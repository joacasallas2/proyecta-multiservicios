# Contexto del Proyecto — Proyecta Multiservicios

## El problema que resolvemos
Proyecto: Plataforma de servicios técnicos domiciliarios tipo Uber + Airbnb + IA

Inspiración real:

Problema vivido: dificultad para conseguir técnicos confiables en Bogotá
Mala experiencia con proveedores → oportunidad de negocio

## Propuesta de valor:
- NO marketplace abierto
- Técnicos propios y verificados
- IA como interfaz principal
- Control total de la operación
- Trazabilidad completa vía chat

---

## Modelo operativo

### Tipos de servicio
- **Express** → asignación automática inmediata
- **Estándar** → cliente elige entre top 3 técnicos seleccionados por IA

### Flujo principal

- Cliente entra a web
- IA inicia conversación
- Cliente describe problema
- IA pide:
* nombre
* dirección
- Validación cobertura (Bogotá)
- Prediagnóstico + costo estimado
- Cliente decide continuar o no
- Matching de técnicos
- Cliente selecciona técnico

Se crea orden + chat
- Chat (núcleo del sistema)
- Estilo Airbnb

Todo queda registrado
- Técnico: solo audio, fotos, videos, archivos
- IA: transcribe audio
- Cliente:
    ve texto + opción de audio

Botones de aprobación:
    aceptar cambios
    rechazar

---

### Estados del servicio
TECNICO_ASIGNADO → EN_CAMINO → EN_SITIO → 
DIAGNOSTICO_REALIZADO → EN_EJECUCION → SERVICIO_TERMINADO

### Reglas críticas
- Técnico NO puede cancelar — puede solicitar cancelación a admin
- Solo admin cancela y reasigna (la orden no se cierra)
- Todo acuerdo debe quedar en el chat
- Visita técnica: $70.000 (descontable del servicio)
- Agenda se sincroniza automáticamente con servicios confirmados

---

## Dominios del sistema
| Dominio | Responsabilidad |
|---------|----------------|
| Users | Auth, perfiles, roles |
| Technicians | Perfil, disponibilidad, agenda |
| Services | Órdenes, estados, asignación |
| Chat | Mensajes, permisos, adjuntos |
| AI | Agentes, transcripción, RAG |
| Tracking | GPS en tiempo real |
| Billing | Facturas, pagos, encuestas |
| Inventory | Herramientas e insumos |

---

## Fases del proyecto
| Fase | Contenido | Estado |
|------|-----------|--------|
| Fase 0 | Modelo de datos (schema Prisma) | 
| Fase 1 | Fundación (infra, auth, users) | 
| Fase 2 | Core del negocio | 
| Fase 3 | Chat | 
| Fase 4 | Matching | 
| Fase 5 | IA básica | 
| Fase 6 | Tracking GPS | 
| Fase 7 | Colas BullMQ | 
| Fase 8 | IA avanzada + multiagentes | 
| Fase 9 | Post-servicio | 

---

## Modelo de técnicos
[perfil, agenda, slots, reglas]

Perfil incluye:

bio
foto
tipos de servicio (skills)
disponibilidad
agenda

estado:
ONLINE
OFFLINE
OCUPADO

slots:
DISPONIBLE
RESERVADO
BLOQUEADO



## Matching
[filtros, flujo express vs estándar]


Filtros:
- tipo de servicio
- disponibilidad
- agenda
- estado técnico
- rating (futuro)

Estándar:
- técnicos levantan la mano
- IA selecciona top 3
- cliente elige

Express:
- asignación automática

## IA y multiagentes
[agentes, RAG, pgvector]

- prediagnóstico
- estimación de costo
- RAG con pgvector

## Reglas de arquitectura
- Backend separado — NestJS en `apps/api`, NO Next.js API Routes
- Supabase SOLO para login, registro y sesiones
- El rol del usuario SIEMPRE viene de la base de datos, nunca del JWT
- Event-driven: cada dominio emite y escucha eventos, no llama directamente a otro
- Stateless backend
- Config centralizada — nunca `process.env` directamente en módulos

## PRINCIPIOS CLAVE DEL SISTEMA
Arquitectura modular
Event-driven (preparado para escalar)
IA controlada (no acceso a fuentes abiertas)
Todo queda registrado en chat (evita acuerdos verbales)
Sistema cerrado (no marketplace abierto)
Control total de técnicos

---

### Multiagentes:
- recepción
- diagnóstico
- pricing
- soporte
- transcripción

---

## COLAS (BullMQ)

Jobs:

- matching
- notificaciones
- transcripción
- timers (30 min)
- inactividad (21h + 3h)
- inventario técnico

--- 

## POST-SERVICIO
- factura automática
- pago por transferencia / llave Breb / link de pago
- confirmación manual admin
- encuesta
- soporte y garantías

---

## FASES DEFINIDAS

### Fase 0 — Modelo de datos (CRÍTICA)
entidades
relaciones
enums
Prisma schema

---

### Fase 1 — Fundación
NestJS
Prisma
PostgreSQL
Redis
Supabase Auth
Users

---

### Fase 2 — Core negocio
servicios
técnicos
agenda
estados

---

### Fase 3 — Chat
Socket.io
rooms
mensajes tipados
transcripción

---

### Fase 4 — Matching
algoritmo selección
top 3 técnicos
express vs estándar

---

### Fase 5 — IA básica
agente conversacional
flujo completo cliente

---

### Fase 6 — Tracking
GPS tiempo real

---

### Fase 7 — Colas
BullMQ
jobs

---

### Fase 8 — IA avanzada
RAG
multiagentes

---

### Fase 9 — Post-servicio
facturación
pagos
soporte

---

## TRACKING
GPS técnico (app móvil)
envío cada 5–10s
visible cuando:
estado = "en camino"

---
## Convenciones de código

### Nombrado
- Archivos: `kebab-case` (ej: `jwt.guard.ts`)
- Clases: `PascalCase` (ej: `JwtGuard`)
- Variables y funciones: `camelCase`
- Constantes: `UPPER_SNAKE_CASE`
- Enums: `PascalCase` con valores `UPPER_SNAKE_CASE`

### Commits
Formato convencional:
feat(auth): add JWT validation guard
fix(users): correct role sync on first login
chore(config): update tsconfig base paths

## AUTENTICACIÓN
- Supabase gestiona login
- Backend valida JWT
- El rol SIEMPRE viene de la DB
- Nunca confiar en datos del cliente

---

## USUARIO
- Roles: ADMIN, CLIENT, TECHNICIAN
- Sincronización automática desde JWT
- No existe creación manual

---

## Fase activa
**Fase 1 — Fundación**
Spec: `docs/specs/spec-fase-1.md`
Contexto completo: `docs/context.md`

Lee el spec antes de implementar cualquier cosa.

---

## ARQUITECTURA DEFINIDA
Monorepo (pnpm)
/apps
  /web        → Next.js (cliente)
  /admin      → Next.js (panel admin)
  /mobile     → React Native (Expo)
  /api        → NestJS (backend)

/packages
  /types
  /ui
  /config
  /db (Prisma)
  /auth
  /events
  /utils
  /ai
  /validation
  /constants

/infra
  /docker
  /scripts

---

## STACK TECNOLÓGICO
- Backend: NestJS (apps/api)
- Frontend web:  Next.js + tailwind (apps/web  apps/admin)
- Mobile app: React Native + Expo (apps/mobile)
- ORM: Prisma (packages/db)
- BASE DE DATOS: PostgreSQL+ pgvector
- Cache/Eventos: Redis (ioredis)
- Colas: BullMQ (sobre Redis)
- Auth: Supabase (frontend) + validación JWT (backend)
- Realtime: Socket.io
- IA: Claude API + LangChain.js + RAG(pgvector)
- Storage: Cloudflare R2
- Mapas: Leaflet + OpenStreetMap
- Arquitectura modular estricta
- Event-driven (Redis Pub/Sub)
- Validación: Zod
- Tests: Vitest + Playwright
- Infra: Docker + Cloudflare

---
