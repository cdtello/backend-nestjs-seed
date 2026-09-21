# Backend NestJS Seed — Rama 4 Filtros (sencillo)

> **Rama actual:** `rama-4-filtros` — añade **filtros simples vía query params + QueryBuilder** sobre la tienda. Ramas: `main`, `rama-1-iniciacion` (users), `rama-2-productos`, `rama-3-ordenes`, `rama-4-filtros` (actual). Guía en [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md) · Postman en [`postman/backend-nestjs-seed.postman_collection.json`](./postman/backend-nestjs-seed.postman_collection.json).

Seed NestJS 11 + TypeORM. En esta rama se demuestra **cómo filtrar listados de forma normal y sencilla sin meter complejidad**: `@Query()` + DTO + `QueryBuilder`.

## Qué aprende en esta rama (didáctico, sencillo)

- **`@Query()` + DTO validado:** cómo Nest + `ValidationPipe(transform:true)` convierte `?minPrice=50` (string) a `number` con `@Type(()=>Number)` y valida
- **`QueryBuilder` básico:** `where`, `andWhere`, `LIKE` case-insensitive con `LOWER()`, rangos `>=`/`<=`, `orderBy`
- **Filtros sin paginación ni sorting complejo:** solo `where` opcional, ideal para clase inicial
- **Por qué no `findBy` simple:** con filtros opcionales es más claro construir la query paso a paso que armar un `FindOptionsWhere` dinámico

## Stack
NestJS 11 + TypeORM + SQLite/Postgres + `class-validator`/`class-transformer`. `synchronize:true` dev.

## Estructura

```
src/
  main.ts
  app.module.ts           # UsersModule + ProductsModule + OrdersModule
  users/                  # Rama 1
  products/               # Rama 2 + Rama 4 filtros
    products.module.ts / controllers / services / entities / dto
    dto/filter-product.dto.ts   # ← NUEVO: name, minPrice, maxPrice, minStock, maxStock
  orders/                 # Rama 3 + Rama 4 filtros
    orders.module.ts / controllers / services / entities / dto
    dto/filter-order.dto.ts     # ← NUEVO: status, userId
data/app.sqlite
```

## Guía rápida (Rama 4)

```bash
git clone -b rama-4-filtros https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed && npm install && cp .env.example .env && npm run start:dev
# http://localhost:3000
# Postman: importar postman/backend-nestjs-seed.postman_collection.json (Users+Products+Orders con filtros)
```

## Módulos previos (resumen)

**Users:** `POST/GET/GET:id/PUT/DELETE` soft. **Products:** `POST/GET/GET:id/PUT/DELETE` soft (uuid). **Orders:** `POST /orders` con transacción + `GET /orders`, `GET /orders/:id`, `GET /orders/user/:userId`, `PUT /orders/:id`, `PUT /orders/:id/cancel`, `DELETE /orders/:id`.

## Novedad Rama 4: Filtros simples

**Products — `GET /products` con query opcional:**
| Query | Tipo | Ejemplo | Efecto |
|---|---|---|---|
| `name` | string | `?name=whey` | `LOWER(name) LIKE %whey%` case-insensitive |
| `minPrice` | number | `?minPrice=50` | `price >= 50` |
| `maxPrice` | number | `?maxPrice=200` | `price <= 200` |
| `minStock` | int | `?minStock=10` | `stock >= 10` |
| `maxStock` | int | `?maxStock=100` | `stock <= 100` |

Combinables: `GET /products?name=prote&minPrice=50&maxPrice=200&minStock=5`

Código: `src/products/dto/filter-product.dto.ts:1` + `src/products/services/products.service.ts:22` (QueryBuilder) + `src/products/controllers/products.controller.ts:24` (`@Query() filter: FilterProductDto`).

```bash
curl "http://localhost:3000/products?name=whey"
curl "http://localhost:3000/products?minPrice=50&maxPrice=200"
curl "http://localhost:3000/products?name=prote&minStock=10&maxStock=100"
```

**Orders — `GET /orders` con query opcional:**
| Query | Tipo | Ejemplo |
|---|---|---|
| `status` | enum `PENDING/PAID/CANCELLED` | `?status=PENDING` |
| `userId` | 5-20 dígitos | `?userId=1234567890` |

Combinables: `GET /orders?status=PENDING&userId=1234567890`

Código: `src/orders/dto/filter-order.dto.ts:1` + `src/orders/services/orders.service.ts:17` + `src/orders/controllers/orders.controller.ts:24`.

```bash
curl "http://localhost:3000/orders?status=PENDING"
curl "http://localhost:3000/orders?userId=1234567890"
curl "http://localhost:3000/orders?status=PENDING&userId=1234567890"
# Sigue funcionando GET /orders/user/:userId para URL limpia
curl http://localhost:3000/orders/user/1234567890
```

**Validación:** si mandas `?minPrice=abc` o `?status=INVALID` → `400` por `ValidationPipe`. Si no mandas nada, lista todo (comportamiento previo).

## Scripts
```bash
npm run format && npm run lint && npm run build && npm run start:dev
```
