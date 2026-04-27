# Proyecta Multiservicios

Plataforma de servicios técnicos domiciliarios con arquitectura modular, tiempo real y soporte de IA.

## Características

* Chat con agente IA para solicitud de servicios
* Asignación inteligente de técnicos
* Seguimiento en tiempo real (GPS)
* Chat cliente ↔ técnico
* Gestión completa de órdenes de servicio
* Sistema de diagnóstico dinámico con aprobación del cliente

---

## Arquitectura

Monorepo basado en pnpm workspaces:

### Apps

* `apps/web` → Aplicación cliente (Next.js)
* `apps/admin` → Panel administrativo
* `apps/mobile` → App de técnicos (React Native)
* `apps/api` → Backend principal (NestJS)

### Packages

* `packages/types` → Tipos compartidos
* `packages/validation` → Esquemas Zod
* `packages/db` → Prisma + configuración de base de datos
* `packages/events` → Definición de eventos del sistema
* `packages/utils` → Utilidades compartidas
* `packages/ai` → Lógica de IA (RAG, agentes)
* `packages/auth` → Helpers de autenticación
* `packages/ui` → Componentes reutilizables

---

## Stack Tecnológico

* TypeScript
* Node.js

### Frontend

* React
* Next.js
* Tailwind CSS

### Mobile

* React Native (Expo)

### Backend

* NestJS
* PostgreSQL + Prisma
* Redis + BullMQ
* Socket.io

### IA

* OpenAI / modelos LLM
* RAG (Retrieval-Augmented Generation)

### Infraestructura

* Docker
* Cloudflare R2


---

## Comandos

Instalar dependencias:

```bash
pnpm install
```

Desarrollo:

```bash
pnpm dev
```

Build:

```bash
pnpm build
```

---

## Estado del proyecto

En desarrollo — arquitectura base en construcción.
