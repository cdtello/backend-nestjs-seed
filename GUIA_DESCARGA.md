# Guía de Descarga — Rama 3 Órdenes (Tienda didáctica)

> **Rama 3 = Rama 2 + módulo `orders` con relaciones.** Si ya tienes Rama 2, es el mismo flujo de instalación. Ramas: `rama-1-iniciacion` (users), `rama-2-productos` (users+products), `rama-3-ordenes` (tienda completa).

**Objetivo didáctico de esta rama:** entender relaciones TypeORM 1-N, tabla pivote con datos (`OrderItem`), y transacción atómica para crear una orden.

## Requisitos
Node 20+, npm 10+, Git, Postman opcional.

## 1. Clonar rama 3
```bash
git clone -b rama-3-ordenes https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
# o
git clone https://github.com/cdtello/backend-nestjs-seed.git && cd backend-nestjs-seed && git checkout rama-3-ordenes
```

## 2. Instalar y configurar
```bash
npm install
cp .env.example .env
cat .env  # DB_TYPE=sqlite por defecto crea data/app.sqlite solo
```

## 3. Arrancar
```bash
npm run start:dev
# Nest application successfully started → http://localhost:3000
```

## 4. Probar Users y Products (resumen)
Ya vistos en ramas 1-2. Recuerda crear al menos 1 usuario y 1 producto antes de crear órdenes, porque la orden los referencia.

```bash
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"id":"1234567890","name":"Ana","email":"ana@seed.local","age":25,"phone":"+573001234567"}'
curl -X POST http://localhost:3000/products -H "Content-Type: application/json" -d '{"name":"Proteína Whey","price":129.9,"stock":50}'
# guarda el uuid del producto → lo usarás como {{productId}}
```

Colección Postman: `postman/backend-nestjs-seed.postman_collection.json` carpetas Users y Products (5 req c/u).

## 5. Probar Orders — Tienda (nuevo didáctico)

### Diagrama de relaciones que vas a ver en código
```
User 1 —— N Order 1 —— N OrderItem N —— 1 Product
```
- `src/orders/entities/order.entity.ts` — `Order` tiene `@ManyToOne(User)` con `@JoinColumn(userId)` y `@OneToMany(OrderItem, cascade:true)`
- `src/orders/entities/order-item.entity.ts` — `OrderItem` tiene `@ManyToOne(Order, onDelete:CASCADE)` y `@ManyToOne(Product, eager:true)` + `quantity`, `unitPrice`
- Por qué no `ManyToMany` directo: necesitamos guardar `quantity` y `unitPrice` histórico por item.

### Endpoints Orders
| Método | Ruta | Qué hace |
|---|---|---|
| `POST` | `/orders` | Crea orden: valida user activo, valida cada producto activo y `stock >= quantity`, calcula `total`, descuenta stock, crea `Order` + `OrderItem`s en **transacción** |
| `GET` | `/orders` | Lista con `user` e `items.product` (eager) |
| `GET` | `/orders/:id` | Detalle |
| `GET` | `/orders/user/:userId` | Órdenes de un usuario |
| `PUT` | `/orders/:id` | Cambia status (`PENDING`→`PAID`/`CANCELLED`) |
| `PUT` | `/orders/:id/cancel` | Cancela `PENDING` y **restaura stock** en transacción |
| `DELETE` | `/orders/:id` | Borra (si PENDING primero cancela/restaura) |

### Crear orden — ejemplo completo
```bash
# 1 producto
curl -X POST http://localhost:3000/orders -H "Content-Type: application/json" \
  -d '{"userId":"1234567890","items":[{"productId":"<uuid>","quantity":2}]}'
# → 201 { "id":"<orderId>", "total":"259.80", "status":"PENDING", "items":[...] }

# 2 productos (usa dos uuid distintos)
curl -X POST http://localhost:3000/orders -H "Content-Type: application/json" \
  -d '{"userId":"1234567890","items":[{"productId":"<uuid1>","quantity":1},{"productId":"<uuid2>","quantity":3}]}'
```

Qué pasa dentro (`src/orders/services/orders.service.ts:create`):
1. `queryRunner.startTransaction()`
2. `findOneBy(User)` → 404 si no existe/inactivo
3. Loop items → `findOneBy(Product)` → 404, check `stock < quantity` → 400, `total += price*quantity`, `product.stock -= quantity` + `save(product)`
4. `create(Order)` con `items` + `save(Order)` (cascade guarda OrderItems)
5. `commitTransaction()` o `rollbackTransaction()` si falla

### Consultas y flujo de prueba sugerido en Postman
Importa `postman/backend-nestjs-seed.postman_collection.json` → carpeta **Orders** (8 requests):
1. Crear orden → copia `id` retornado a variable `orderId`
2. Listar órdenes `GET /orders` → debe aparecer con `total` y `items[].product`
3. Consultar `GET /orders/{{orderId}}`
4. Órdenes por usuario `GET /orders/user/{{userId}}`
5. Verifica stock descontado `GET /products/{{productId}}` → `stock` bajó
6. Cancelar `PUT /orders/{{orderId}}/cancel` → `status:CANCELLED` y `GET /products/{{productId}}` vuelve a subir stock
7. Crear otra orden y cambiar a `PAID` `PUT /orders/{{orderId}}` `{"status":"PAID"}`
8. Intentar cancelar una `PAID` → `400`
9. Eliminar `DELETE /orders/{{orderId}}`

### Errores didácticos que debes provocar
- Crear orden con `userId` inexistente → `404`
- Con `productId` uuid inexistente → `404`
- Con `quantity` 0 o `stock` insuficiente → `400`
- Cancelar una orden ya `CANCELLED` o `PAID` → `400`
- Cambiar `status` desde `CANCELLED` → `400`

### DTOs didácticos
- `src/orders/dto/create-order.dto.ts` — `userId` con `Matches(/^\d{5,20}$/)`, `items` con `@IsArray() @ArrayMinSize(1) @ValidateNested({each:true}) @Type(()=>CreateOrderItemDto)`, cada item `productId` `@IsUUID()` y `quantity` `@IsInt() @Min(1) @Max(100)`
- `src/orders/dto/update-order.dto.ts` — solo `status` `@IsEnum(OrderStatus)`

## 6. Estructura final Rama 3
```
src/
  main.ts
  app.module.ts            # UsersModule + ProductsModule + OrdersModule
  config/env.validation.ts
  users/                   # Rama 1
  products/                # Rama 2
  orders/                  # Rama 3
    orders.module.ts
    controllers/orders.controller.ts
    services/orders.service.ts
    entities/order.entity.ts
    entities/order-item.entity.ts
    dto/create-order.dto.ts
    dto/update-order.dto.ts
postman/backend-nestjs-seed.postman_collection.json # Users+Products+Orders
```

## 7. Comandos útiles
```bash
npm run build && npm run lint && npm run start:dev
# DB es SQLite data/app.sqlite (ignorado por git). Para resetear: rm data/app.sqlite y reiniciar.
```

Si todo lo anterior devuelve `201`/`200` y ves `total` calculado y `stock` moviéndose, la rama 3 quedó correcta.
