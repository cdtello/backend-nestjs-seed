# Backend NestJS Seed — Rama 3 Órdenes (Tienda con relaciones)

> **Rama actual:** `rama-3-ordenes` — añade **módulo `orders` con relaciones TypeORM + transacción** (didáctica Tienda). Ramas previas: `main` (seed), `rama-1-iniciacion` (users), `rama-2-productos` (users+products). Guía en [`GUIA_DESCARGA.md`](./GUIA_DESCARGA.md) · Postman en [`postman/backend-nestjs-seed.postman_collection.json`](./postman/backend-nestjs-seed.postman_collection.json).

Seed NestJS 11 + TypeORM 0.3 + SQLite/Postgres. En esta rama se demuestra **cómo modelar una tienda mínima donde un Usuario compra Productos y se genera una Orden**.

## Qué aprende en esta rama (didáctico)

- **Relaciones TypeORM:** `@ManyToOne`, `@OneToMany`, `@JoinColumn`, `cascade:true`, `eager:true`, `onDelete`
- **Tabla pivote con datos:** `OrderItem` guarda `quantity` y `unitPrice` (precio histórico) — por qué no usar `ManyToMany` directo
- **Transacción con `QueryRunner`:** validar stock, descontar stock, calcular total y crear orden de forma atómica (todo o nada)
- **Enum + decimales:** `OrderStatus` y `decimal(10,2)` para dinero

## Stack
- NestJS 11 + TypeScript 5.7 + ValidationPipe (`whitelist`, `forbidNonWhitelisted`, `transform`)
- TypeORM + `@nestjs/typeorm` + `autoLoadEntities:true` + `synchronize:true` (dev)
- `class-validator` / `class-transformer` para DTOs

## Estructura

```
src/
  main.ts
  app.module.ts           # + UsersModule + ProductsModule + OrdersModule
  config/env.validation.ts
  users/                  # Rama 1 - standalone
    users.module.ts / controllers / services / entities / dto
  products/               # Rama 2 - standalone
    products.module.ts / controllers / services / entities / dto
  orders/                 # ← NUEVO Rama 3 (con relaciones)
    orders.module.ts
    controllers/orders.controller.ts
    services/orders.service.ts
    entities/order.entity.ts        # @ManyToOne(User) + @OneToMany(OrderItem) cascade
    entities/order-item.entity.ts   # @ManyToOne(Order) + @ManyToOne(Product eager)
    dto/create-order.dto.ts         # @ValidateNested + @ArrayMinSize(1)
    dto/update-order.dto.ts
data/app.sqlite
```

## Guía rápida (Rama 3)

```bash
git clone -b rama-3-ordenes https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
npm install
cp .env.example .env
npm run start:dev
# API http://localhost:3000
# Postman: importar postman/backend-nestjs-seed.postman_collection.json (Users + Products + Orders)
```

## Módulos previos

### Users (Rama 1)
| `POST` | `/users` | Crea | `GET` | `/users` | lista | `GET` | `/users/:id` | `PUT` | `/users/:id` | `DELETE` | `/users/:id` soft |
Entidad `User`: `id` 5-20 dígitos PK, `email` único, `name` 2-100, `age` 0-130, `phone` +? 7-15.

### Products (Rama 2)
| `POST` | `/products` | `GET` | `/products` | `GET` | `/products/:id` | `PUT` | `/products/:id` | `DELETE` | `/products/:id` soft |
Entidad `Product`: `id` uuid, `name` 2-120, `price` decimal 0.01-999999, `stock` 0-100000.

## Módulo nuevo Rama 3: Orders (con relaciones)

**Diagrama:**
```
User 1──N Order 1──N OrderItem N──1 Product
```
- `Order.user` → `@ManyToOne(User)` + `@JoinColumn(userId)` — FK explícita `userId`
- `Order.items` → `@OneToMany(OrderItem, cascade:true)` — al guardar Order con items se guardan juntos
- `OrderItem.order` → `@ManyToOne(Order, onDelete:CASCADE)`
- `OrderItem.product` → `@ManyToOne(Product, eager:true)` — al traer items ya trae producto

**Endpoints:**

| Método | Ruta | Descripción |
|---|---|---|
| `POST` | `/orders` | Crea orden con transacción (valida user, productos, stock, descuenta, calcula total) |
| `GET` | `/orders` | Lista todas con `user` e `items.product` |
| `GET` | `/orders/:id` | Detalle |
| `GET` | `/orders/user/:userId` | Órdenes de un usuario |
| `PUT` | `/orders/:id` | Cambia `status` (`PENDING`→`PAID`/`CANCELLED`) |
| `PUT` | `/orders/:id/cancel` | Cancela y **restaura stock** (transacción) |
| `DELETE` | `/orders/:id` | Borra (si PENDING primero restaura stock) |

**Ejemplo crear orden:**
```bash
# 1. Crear usuario y producto primero, guarda sus ids en variables
curl -X POST http://localhost:3000/orders -H "Content-Type: application/json" \
  -d '{"userId":"1234567890","items":[{"productId":"<uuid>","quantity":2}]}'
# → 201 { "id":"uuid", "userId":"1234567890", "total":"259.80", "status":"PENDING",
#         "items":[{"quantity":2,"unitPrice":"129.90","product":{...}}] }

# Con 2 productos
curl -X POST http://localhost:3000/orders -H "Content-Type: application/json" \
  -d '{"userId":"1234567890","items":[{"productId":"<uuid1>","quantity":1},{"productId":"<uuid2>","quantity":3}]}'

# Listar / consultar / por usuario
curl http://localhost:3000/orders
curl http://localhost:3000/orders/<orderId>
curl http://localhost:3000/orders/user/1234567890

# Cambiar estado / cancelar
curl -X PUT http://localhost:3000/orders/<orderId> -H "Content-Type: application/json" -d '{"status":"PAID"}'
curl -X PUT http://localhost:3000/orders/<orderId>/cancel
```

**Validaciones y errores:**
- `userId` debe existir y `isActive=true` → `404`
- `productId` uuid debe existir y `isActive=true` → `404`
- `quantity` 1-100, `stock` insuficiente → `400 BadRequest`
- Solo `PENDING` puede pasar a `PAID`/`CANCELLED` → `400`
- Transacción `QueryRunner`: si falla algo hace `rollback`, no queda stock a medias

**Código clave a estudiar:**
- `src/orders/entities/order.entity.ts:1` y `order-item.entity.ts:1` — decoradores de relación
- `src/orders/services/orders.service.ts:17` — `create()` con `createQueryRunner().startTransaction()`, `commit`/`rollback`
- `src/orders/dto/create-order.dto.ts:1` — `@ValidateNested({each:true})` + `@Type(()=>CreateOrderItemDto)`

## Scripts
```bash
npm run format && npm run lint && npm run build && npm run start:dev
```

## Generadores usados
```bash
# Rama 2: nest g resource products --type rest --no-spec
# Rama 3: nest g module orders && nest g controller/service para orders
```
