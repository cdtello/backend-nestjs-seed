# Guía de Descarga - Rama 1 Iniciación

Esta rama es la **base limpia** del seed. No tiene lógica adicional: solo NestJS + TypeORM + módulo `users` de ejemplo.

## Requisitos previos

- Node.js 20+ y npm 10+ (`node -v` / `npm -v`)
- Git
- Postman o similar para probar la API (opcional)

## 1. Clonar el repositorio (rama 1)

```bash
# Clonar solo la rama de iniciación
git clone -b rama-1-iniciacion https://github.com/cdtello/backend-nestjs-seed.git

# o clonar todo y cambiarse a la rama
git clone https://github.com/cdtello/backend-nestjs-seed.git
cd backend-nestjs-seed
git checkout rama-1-iniciacion
```

## 2. Instalar dependencias

```bash
cd backend-nestjs-seed
npm install
```

## 3. Configurar variables de entorno

```bash
cp .env.example .env
# Edita .env si quieres cambiar PORT o usar Postgres
cat .env
```

Por defecto usa SQLite y crea `data/app.sqlite` automáticamente.

## 4. Ejecutar en desarrollo

```bash
npm run start:dev
```

Verás: `Nest application successfully started` en `http://localhost:3000`

Para salir: `Ctrl + C`

Otros comandos:
```bash
npm run build      # compila a dist/
npm run start:prod # ejecuta dist/main.js
npm run lint       # verifica eslint
npm run format     # formatea con prettier
```

## 5. Probar con Postman

Colección incluida: `postman/backend-nestjs-seed.postman_collection.json`

1. Abre Postman → Import → selecciona el archivo `postman/backend-nestjs-seed.postman_collection.json`
2. Verifica variable `baseUrl` = `http://localhost:3000`
3. Ejecuta en orden:
   - **Crear usuario** (`POST /users`)
   - **Listar usuarios** (`GET /users`)
   - **Consultar usuario** (`GET /users/{{userId}}`) - usa el id creado
   - **Actualizar usuario** (`PUT /users/{{userId}}`)
   - **Desactivar usuario** (`DELETE /users/{{userId}}`)

Ejemplo `POST /users`:
```json
{
  "id": "1234567890",
  "name": "Usuario de prueba",
  "email": "prueba@seed.local",
  "age": 25,
  "phone": "+573101234567"
}
```

También puedes probar con curl:
```bash
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"id":"1234567890","name":"Usuario de prueba","email":"prueba@seed.local","age":25,"phone":"+573101234567"}'
curl http://localhost:3000/users
```

## 6. Estructura que debes ver

```
src/
  main.ts
  app.module.ts
  config/env.validation.ts
  users/
    users.module.ts
    controllers/users.controller.ts
    services/users.service.ts
    entities/user.entity.ts
    dto/create-user.dto.ts
    dto/update-user.dto.ts
```

Si el paso 4 arrancó sin errores y el 5 devuelve `201`/`200`, la instalación fue exitosa.
