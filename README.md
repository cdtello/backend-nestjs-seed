# Backend NestJS Seed — Rama 2 Productos

> **Rama actual:** `rama-2-productos` — añade módulo `products` standalone (sin relaciones aún, base para tienda). Rama 1 queda intacta en `rama-1-iniciacion`. Guía en [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md) · Postman en [`postman/backend-nestjs-seed.postman_collection.json`](./postman/backend-nestjs-seed.postman_collection.json).

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
  app.module.ts           # ConfigModule + TypeOrmModule.forRootAsync + UsersModule + ProductsModule
  config/
    env.validation.ts
  users/
    users.module.ts
    controllers/users.controller.ts
    services/users.service.ts
    entities/user.entity.ts
    dto/create-user.dto.ts
    dto/update-user.dto.ts
  products/               # ← NUEVO Rama 2 (standalone)
    products.module.ts
    controllers/products.controller.ts
    services/products.service.ts
    entities/product.entity.ts
    dto/create-product.dto.ts
    dto/update-product.dto.ts
data/
  app.sqlite
```

Clonado 1:1 de `nutrifit-backend/src`.

## Guía rápida (Rama 2)

Ver guía completa en [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md).

```bash
git clone -b rama-2-productos https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
npm install
cp .env.example .env
npm run start:dev
# API en http://localhost:3000
# Postman: importar postman/backend-nestjs-seed.postman_collection.json (carpetas Users + Products)
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

Ejemplos CRUD Users (ver también `GUIA_DESCARGA.md` y `postman/backend-nestjs-seed.postman_collection.json`):

```bash
# 1. POST /users - crear
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" \
  -d '{"id":"1234567890","name":"Carlos Tello","email":"carlos@ejemplo.com","age":30,"phone":"+573101234567"}'
# → 201 Created

# 2. GET /users - listar
curl http://localhost:3000/users

# 3. GET /users/:id - consultar
curl http://localhost:3000/users/1234567890

# 4. PUT /users/:id - actualizar (campos opcionales)
curl -X PUT http://localhost:3000/users/1234567890 -H "Content-Type: application/json" \
  -d '{"name":"Carlos Tello Actualizado","age":31}'

# 5. DELETE /users/:id - desactivar (soft delete)
curl -X DELETE http://localhost:3000/users/1234567890
```

Detalle JSON `POST`:
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

## Módulo nuevo Rama 2: Products (standalone)

Sin relaciones aún — base para la tienda (Rama 3 unirá User + Product → Order).

| Método | Ruta | Descripción |
| --- | --- | --- |
| `POST` | `/products` | Crea producto activo |
| `GET` | `/products` | Lista productos activos |
| `GET` | `/products/:id` | Obtiene producto por UUID |
| `PUT` | `/products/:id` | Actualiza campos |
| `DELETE` | `/products/:id` | Soft delete (`isActive=false`) |

Entidad `Product` (`src/products/entities/product.entity.ts`):
- `id` UUID PK auto
- `name` 2-120
- `description` opcional 0-500
- `price` 0.01 - 999999 (decimal 10,2)
- `stock` 0 - 100000 (int)
- `isActive` + `createdAt`

Ejemplos:
```bash
# Crear
curl -X POST http://localhost:3000/products -H "Content-Type: application/json" \
  -d '{"name":"Proteína Whey","description":"900g vainilla","price":129.9,"stock":50}'
# → 201 { "id":"uuid", "name":"Proteína Whey", ... }

# Listar
curl http://localhost:3000/products

# Consultar (usa el uuid retornado)
curl http://localhost:3000/products/<uuid>

# Actualizar
curl -X PUT http://localhost:3000/products/<uuid> -H "Content-Type: application/json" \
  -d '{"price":119.9,"stock":45}'

# Eliminar (soft)
curl -X DELETE http://localhost:3000/products/<uuid>
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
