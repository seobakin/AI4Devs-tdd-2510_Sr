# Prompts utilizados para generar tests unitarios - PG

## Prompt Principal

```
Tu misión será crear una suite de tests unitarios en Jest para la funcionalidad de insertar candidatos en base de datos. Apóyate en la IA y utiliza el contexto del proyecto para identificar aquellos tests que puedan ser relevantes en este caso.

Pista 1: hay 2 familias principales de tests, recepción de los datos del formulario, y guardado en la base de datos. Queremos ver tests que cubran ambos procesos con al menos un test.

(BONUS) Aunque para esta sesión no es necesario, si alguno de los tests requiere modificar algo en base de datos, recuerda que lo ideal cuando las pruebas unitarias requieren interacción con base de datos, es mockearla para no alterar los datos.
```

## Contexto del Proyecto

El proyecto LTI es un sistema de seguimiento de talento (Talent Tracking System) con:
- Backend en Express/TypeScript con Prisma ORM
- Frontend en React
- Base de datos PostgreSQL

## Estructura Analizada

Para crear los tests, se analizaron los siguientes archivos:

1. **`backend/src/application/validator.ts`**: Contiene las funciones de validación de datos del candidato (nombre, email, teléfono, dirección, educación, experiencia laboral, CV).

2. **`backend/src/application/services/candidateService.ts`**: Contiene la función `addCandidate()` que orquesta la inserción de candidatos, incluyendo validación, creación de instancia y guardado en base de datos.

3. **`backend/src/domain/models/Candidate.ts`**: Modelo de dominio del candidato con método `save()` que interactúa con Prisma.

4. **`backend/src/application/validator.test.ts`**: Tests existentes de validación que sirvieron como referencia de estilo.

## Decisiones de Diseño de Tests

### Familia 1: Recepción de datos del formulario

Se crearon tests para validar:
- Campos obligatorios (firstName, lastName, email)
- Formato de email
- Formato de teléfono español (empieza por 6, 7 o 9, seguido de 8 dígitos)
- Datos de educación (institución, título, fechas)
- Datos de experiencia laboral (empresa, posición, descripción, fechas)
- Datos de CV (filePath, fileType)

### Familia 2: Guardado en base de datos

Se implementó mocking de Prisma siguiendo las recomendaciones de la documentación oficial de Prisma para tests unitarios:

```typescript
jest.mock('@prisma/client', () => {
    const mockPrismaClient = {
        candidate: {
            create: jest.fn(),
            update: jest.fn(),
            findUnique: jest.fn(),
        },
        // ... otros modelos
    };
    return {
        PrismaClient: jest.fn(() => mockPrismaClient),
        Prisma: {
            PrismaClientInitializationError: class PrismaClientInitializationError extends Error {},
        },
    };
});
```

Se crearon tests para:
- Creación de candidato con datos mínimos
- Creación de candidato con todos los campos
- Manejo de error de email duplicado (código P2002)
- Propagación de errores desconocidos
- Validación antes de intentar guardar
- Creación de instancia del modelo Candidate

## Buenas Prácticas Aplicadas

1. **Aislamiento de tests**: Cada test es independiente gracias a `beforeEach(() => jest.clearAllMocks())`
2. **Mocking de dependencias externas**: Prisma está mockeado para evitar conexiones reales a la base de datos
3. **Estructura AAA (Arrange-Act-Assert)**: Cada test sigue este patrón
4. **Nombres descriptivos en español**: Los tests describen claramente qué se está probando
5. **Agrupación lógica con `describe`**: Tests organizados en familias y subcategorías
6. **Tests de casos positivos y negativos**: Se prueban tanto casos válidos como inválidos

## Comandos para Ejecutar Tests

```bash
cd backend
npm test
# o para ejecutar solo este archivo de tests:
npm test -- tests-pg.test.ts
```

## Referencias

- [Prisma Testing Guide](https://www.prisma.io/blog/testing-series-1-8eRB5p0Y8o#mock-prisma-client)
- [Jest Mock Functions](https://jestjs.io/docs/mock-functions)
