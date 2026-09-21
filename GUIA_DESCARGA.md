# Guía de Descarga — Rama 4 Filtros (sencillo, didáctico)

> **Rama 4 = Rama 3 + filtros simples vía query params.** Sin paginación ni complejidad. Ramas: `rama-1-iniciacion`, `rama-2-productos`, `rama-3-ordenes`, `rama-4-filtros` (actual).

**Objetivo didáctico:** aprender a filtrar listados con `@Query()` + DTO + `QueryBuilder` de forma normal y sencilla.

## Requisitos
Node 20+, npm 10+, Git, Postman opcional.

## 1. Clonar rama 4
```bash
git clone -b rama-4-filtros https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
# o
git clone https://github.com/cdtello/backend-nestjs-seed.git && cd backend-nestjs-seed && git checkout rama-4-filtros
```

## 2. Instalar y arrancar
```bash
npm install
cp .env.example .env
npm run start:dev
# Nest application successfully started → http://localhost:3000
```

## 3. Datos previos
Crea 1 usuario y 2-3 productos para probar filtros/órdenes (ver ramas previas). Colección Postman ya incluye datos de ejemplo.

## 4. Probar Filtros Products (nuevo sencillo)

**Cómo funciona (didáctico):**
- Controller: `src/products/controllers/products.controller.ts:24` → `@Get() findAll(@Query() filter: FilterProductDto)`
- DTO: `src/products/dto/filter-product.dto.ts:1` → `@IsOptional()` + `@Type(()=>Number)` para que `?minPrice=50` llegue como `number`
- Service: `src/products/services/products.service.ts:22` → `createQueryBuilder('product').where('isActive=true')` + `andWhere` condicionales + `LOWER(name) LIKE` para case-insensitive

**Endpoints con ejemplos:**
```bash
# Sin filtros (comportamiento Rama 2-3, lista todo activo)
curl http://localhost:3000/products

# Por nombre parcial (whey, proteína, etc.)
curl "http://localhost:3000/products?name=whey"
curl "http://localhost:3000/products?name=PROTE"  # case-insensitive

# Por rango de precio
curl "http://localhost:3000/products?minPrice=50&maxPrice=200"
curl "http://localhost:3000/products?minPrice=100"

# Por stock
curl "http://localhost:3000/products?minStock=10"
curl "http://localhost:3000/products?maxStock=5"

# Combinado (intersección AND)
curl "http://localhost:3000/products?name=prote&minPrice=50&maxPrice=200&minStock=10"

# Validación: si mandas letras en número → 400
curl "http://localhost:3000/products?minPrice=abc"  # Bad Request
```

Postman → carpeta **Products** → 3 requests nuevas: *Filtrar por nombre*, *Filtrar por rango precio*, *Filtrar por stock y nombre*.

## 5. Probar Filtros Orders (nuevo sencillo)

**Cómo funciona:**
- DTO: `src/orders/dto/filter-order.dto.ts:1` → `status` `@IsEnum(OrderStatus)`, `userId` `@Matches(/^\d{5,20}$/)`
- Service: `src/orders/services/orders.service.ts:17` → `createQueryBuilder('order')` + `leftJoinAndSelect` + `andWhere` para `status`/`userId`
- Controller: `src/orders/controllers/orders.controller.ts:24` → `@Get() findAll(@Query() filter: FilterOrderDto)`

```bash
# Sin filtros, lista todo
curl http://localhost:3000/orders

# Por estado
curl "http://localhost:3000/orders?status=PENDING"
curl "http://localhost:3000/orders?status=PAID"
curl "http://localhost:3000/orders?status=CANCELLED"

# Por usuario
curl "http://localhost:3000/orders?userId=1234567890"

# Combinado
curl "http://localhost:3000/orders?status=PENDING&userId=1234567890"

# Sigue funcionando el endpoint dedicado (URL limpia)
curl http://localhost:3000/orders/user/1234567890

# Validación enum → 400
curl "http://localhost:3000/orders?status=INVALID"
```

Postman → carpeta **Orders** → 3 requests nuevas: *Filtrar por status*, *Filtrar por usuario*, *Filtrar por status y usuario*.

## 6. Estructura final Rama 4
```
src/
  products/
    dto/filter-product.dto.ts   # query params productos
    services/products.service.ts # QueryBuilder con filtros
    controllers/products.controller.ts # @Query()
  orders/
    dto/filter-order.dto.ts
    services/orders.service.ts
    controllers/orders.controller.ts
postman/backend-nestjs-seed.postman_collection.json # Users+Products+Orders con filtros
```

## 7. Comandos
```bash
npm run build && npm run lint && npm run start:dev
# Para resetear DB: rm data/app.sqlite y reiniciar
```

Si `GET /products?name=whey` y `GET /orders?status=PENDING` filtran correctamente y `GET /products` sin params sigue listando todo, la rama 4 quedó correcta.
