# Backend NestJS Seed — Rama 1 Iniciación

> **Rama actual:** `rama-1-iniciacion` — base limpia tal cual iniciación. Para la guía paso a paso ver [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md) y colección Postman en [`postman/`](./postman/backend-nestjs-seed.postman_collection.json).

Seed base genérico NestJS + TypeORM + TypeScript, clonado de `nutrifit-backend`. Misma estructura y mismo módulo `users` como ejemplo.

## Stack
- NestJS 11 + TypeScript 5.7
- TypeORM 0.3 + SQLite (por defecto) / Postgres
- Validación con `class-validator` + `class-transformer` y `ValidationPipe` global (`whitelist`, `forbidNonWhitelisted`, `transform`)
- Configuración con `@nestjs/config` + validación de entorno en `src/config/env.validation.ts`

## Estructura

```
src/
  main.ts                 # bootstrap + ValidationPipe global
  app.module.ts           # ConfigModule + TypeOrmModule.forRootAsync + UsersModule
  config/
    env.validation.ts     # valida PORT, DB_TYPE, DB_SYNCHRONIZE, DB_LOGGING (+ Postgres)
  users/
    users.module.ts
    controllers/users.controller.ts
    services/users.service.ts
    entities/user.entity.ts
    dto/create-user.dto.ts
    dto/update-user.dto.ts
data/
  app.sqlite              # creado automáticamente con DB_TYPE=sqlite (ignorado por git)
```

Clonado 1:1 de `nutrifit-backend/src`.

## Guía rápida (Rama 1)

Ver guía completa en [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md).

```bash
git clone -b rama-1-iniciacion https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
npm install
cp .env.example .env
npm run start:dev
# API en http://localhost:3000
# Probar con Postman: importar postman/backend-nestjs-seed.postman_collection.json
```

## Inicio local

```bash
npm install
cp .env.example .env   # ya viene creado, revísalo
npm run start:dev
```

API en http://localhost:3000

## Configuración DB (.env)

`.env` no se versiona (ver `.gitignore`), usa `.env.example` como referencia.

SQLite (default):
```dotenv
DB_TYPE=sqlite
DB_DATABASE=data/app.sqlite
DB_SYNCHRONIZE=true
DB_LOGGING=false
```

PostgreSQL:
```dotenv
DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=una_clave_segura
DB_DATABASE=app_db
DB_SYNCHRONIZE=true
DB_LOGGING=false
```

`DB_SYNCHRONIZE=true` solo para desarrollo. En producción poner `false` y usar migraciones.

## Módulo de ejemplo: Users

Mismo comportamiento que en NutriFit:

| Método | Ruta | Descripción |
| --- | --- | --- |
| `POST` | `/users` | Crea usuario activo |
| `GET` | `/users` | Lista usuarios activos |
| `GET` | `/users/:id` | Obtiene usuario activo |
| `PUT` | `/users/:id` | Actualiza campos |
| `DELETE` | `/users/:id` | Soft delete (`isActive=false`) |

Entidad `User` (`src/users/entities/user.entity.ts`):
- `id` PK string 5-20 dígitos, único
- `name` 2-100
- `email` único, normalizado a lowercase/trim
- `age` 0-130
- `phone` +? 7-15 dígitos
- `isActive` boolean default true

DTOs con validación y lógica de unicidad en `UsersService` (`ConflictException` / `NotFoundException`).

Ejemplo:
```http
POST /users
Content-Type: application/json

{
  "id": "1234567890",
  "name": "Carlos Tello",
  "email": "carlos@ejemplo.com",
  "age": 30,
  "phone": "+573101234567"
}
```

## Scripts

```bash
npm run format
npm run lint
npm run lint:fix
npm run build
npm run start:dev
npm run start:prod
```

## Generadores Nest CLI

Desde la raíz del proyecto:

```bash
nest g module products
nest g controller products
nest g service products
nest g resource products --type rest --crud --no-spec
```

## Crear un nuevo proyecto desde este seed

1. Copia la carpeta `backend-nestjs-seed` y renómbrala.
2. Cambia `name` en `package.json`.
3. Ajusta `DB_DATABASE` en `.env` si quieres otro nombre de archivo.
4. Borra/adapta `users` o genera tu nuevo recurso.

## Verificación

```bash
npm run format
npm run lint
npm run build
```
