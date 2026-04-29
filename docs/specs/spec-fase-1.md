# SPEC — FASE 1: FUNDACIÓN

## Objetivo

Construir la base técnica sólida del backend que permita escalar el sistema sin retrabajos.

Esta fase establece la infraestructura, arquitectura y servicios mínimos necesarios para soportar el resto del producto.

---

## Alcance

Incluye:

* Backend en NestJS (`apps/api`)
* Conexión a PostgreSQL
* ORM con Prisma
* Integración con Redis
* Autenticación (Supabase + validación JWT en backend)
* Módulo de usuarios
* Configuración global tipada
* Health checks

---

## Arquitectura del Monorepo

```
apps/
  api/        → Backend NestJS

packages/
  db/         → Prisma schema + client
  types/      → Tipos compartidos
  config/     → Configuración global (tsconfig, eslint)
```

---

## Estructura del Backend

```
apps/api/src/

├── main.ts
├── app.module.ts

├── modules/
│   ├── auth/
│   ├── users/
│   └── health/

├── common/
│   ├── guards/
│   ├── decorators/
│   ├── filters/
│   └── interceptors/

├── config/
├── database/
├── redis/
```

---

## Configuración Global

### Variables de entorno

```
DATABASE_URL=
REDIS_URL=

SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_JWT_SECRET=
```

### Reglas

* Validación obligatoria con Zod
* Configuración centralizada
* Tipado estricto
* No usar `process.env` directamente en módulos

---

## Base de Datos

### Tecnología

* PostgreSQL
* Prisma

### Ubicación

```
packages/db
```

### Requisitos

* `schema.prisma` centralizado
* Prisma Client exportado como paquete
* Reutilizable en `apps/api`

### Estado esperado

* Conexión funcional
* Migraciones funcionando
* Modelo inicial: `User`

---

## Redis

### Tecnología

* ioredis

### Requisitos

* Conexión singleton
* Disponible globalmente
* Preparado para BullMQ (no implementado aún)

---

## Autenticación

### Estrategia

**Supabase maneja:**

* Login
* Registro
* Sesiones

**Backend maneja:**

* Validación de JWT
* Roles
* Permisos

---

### Flujo

1. Usuario se autentica en frontend (Supabase)
2. Recibe JWT
3. Envía request con `Authorization: Bearer token`
4. Backend valida el token
5. Backend extrae:

   * userId
   * email
6. Backend sincroniza usuario (si no existe → lo crea)
7. Request continúa con usuario inyectado

---

### Componentes requeridos

* `AuthModule`
* `JwtGuard`
* `@CurrentUser()` decorator

---

## Users Module

### Responsabilidades

* Sincronizar usuario desde JWT
* Obtener perfil
* Actualizar datos básicos

---

### Endpoints

```
GET /users/me
PATCH /users/me
```

---

### Reglas

* El backend es la fuente de verdad
* Supabase NO reemplaza la base de datos interna

---

## Health Module

### Endpoint

```
GET /health
```

### Validaciones

* API activa
* Conexión a DB
* Conexión a Redis

---

## Manejo de Errores

* Filtros globales en NestJS
* Errores estructurados
* No exponer errores internos

---

## Logging

* Logs de arranque
* Logs de conexión (DB y Redis)
* Logs de errores

---

## Reglas de Arquitectura

* Arquitectura modular estricta
* Separación de responsabilidades
* Sin lógica de negocio compleja
* Código completamente tipado
* Código comentado
* Preparado para escalar

---

## Fuera de alcance (NO implementar en esta fase)

* Servicios (orders)
* Técnicos
* Agenda
* Chat
* Matching
* IA
* GPS
* Pagos

---

## Decisiones Clave

* Backend separado (NO usar Next.js API Routes)
* Supabase solo para autenticación
* Prisma como ORM único
* Redis desde el inicio

---

## Criterios de Éxito

La fase se considera completa cuando:

* Backend corre sin errores
* Conexión a PostgreSQL funcional
* Conexión a Redis funcional
* Autenticación válida en backend
* Endpoint `/users/me` responde correctamente
* Endpoint `/health` responde correctamente

---

## Resultado Esperado

* Base técnica sólida
* Arquitectura limpia y escalable
* Autenticación funcionando
* Sistema listo para modelar negocio (Fase 2)
