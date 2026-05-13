## SPEC — FASE 0: MODELO DE DATOS

## Objetivo

Diseñar el modelo de datos completo del sistema antes de escribir
cualquier línea de código de negocio.

Este schema es la fuente de verdad de toda la plataforma.

---

## Ubicación
packages/db/prisma/schema.prisma

---

## Reglas generales

- Todos los IDs son UUID
- El ID de User NO se genera internamente, proviene de Supabase Auth
- Todos los modelos tienen `createdAt` y `updatedAt`
- Soft delete donde aplique (campo `deletedAt`)
- Enums en UPPER_SNAKE_CASE
- Nombres de modelos en PascalCase
- Nombres de campos en camelCase

---

## Enums

```prisma
enum Role {
  ADMIN
  CLIENT
  TECHNICIAN
}

enum TechnicianStatus {
  ONLINE
  OFFLINE
  BUSY
}

enum SlotStatus {
  AVAILABLE
  RESERVED
  BLOCKED
}

enum ServiceType {
  EXPRESS
  STANDARD
}

enum ServiceStatus {
  TECHNICIAN_ASSIGNED
  ON_THE_WAY
  ON_SITE
  DIAGNOSIS_DONE
  IN_PROGRESS
  COMPLETED
  CANCELLED
}

enum MessageType {
  TEXT
  AUDIO
  IMAGE
  VIDEO
  FILE
  SYSTEM
}

enum ServiceCategory {
  PLUMBING
  ELECTRICITY
  LOCKSMITH
  FURNITURE_ASSEMBLY
  NETWORKING
  ELECTRONIC_SECURITY
  COMPUTER_MAINTENANCE
}

enum PaymentMethod {
  BANK_TRANSFER
  BREB_TRANSFER
  PAYMENT_LINK
}

enum PaymentStatus {
  PENDING
  CONFIRMED
}

enum CancellationRequestStatus {
  PENDING
  APPROVED
  REJECTED
}

enum SupportCaseStatus {
  OPEN
  IN_REVIEW
  RESOLVED
}

enum CancellationRequestType {
  CLIENT_CANCELLATION
  TECHNICIAN_REASSIGNMENT
}
```

---

## Entidades principales

### Core
- User
- TechnicianProfile
- Service

### Agenda
- TechnicianAvailability
- TechnicianSlot

### Communication
- Chat
- Message

### Diagnosis
- DiagnosisUpdate

### Facturación
- Invoice
- Survey

### Operación
- TechnicianLocation
- TechnicianInventoryItem

### CancellationRequest

Solicitud de cancelación del cliente o abandono de servicio por parte del técnico.

Puede originarse desde:

- CLIENT → puede cancelar el servicio (puede implicar cobro de visita)
- TECHNICIAN → NO puede atender el servicio, solicita reasignación

La lógica depende del rol y del estado del servicio.

### Post-servicio
- SupportCase

### Auditoría
- ServiceStatusHistory


## Modelo: User

Representa cualquier persona en el sistema.
Un usuario puede tener rol CLIENT, TECHNICIAN o ADMIN.
Un mismo usuario puede tener perfil de cliente y de técnico.

```prisma
model User {
  id        String   @id @db.Uuid // proveniente de Supabase Auth
  email     String   @unique
  name      String
  phone     String?
  avatarUrl String?
  role      Role     @default(CLIENT)
  isActive  Boolean  @default(true)
  deletedAt DateTime?
  

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  technicianProfile TechnicianProfile?
  services          Service[]           @relation("ClientServices")
  messages          Message[] @relation("UserMessages")
  supportCases      SupportCase[]
  cancellationRequestsRequested CancellationRequest[] @relation("UserCancellationRequests")
  cancellationRequestsResolved  CancellationRequest[] @relation("ResolvedCancellation")
}
```
---

### TechnicianProfile

Perfil extendido de un usuario con rol TECHNICIAN.
Relación 1 a 1 con User.

```prisma
model TechnicianProfile {
  id          String           @id @default(uuid()) @db.Uuid
  userId      String           @unique @db.Uuid

  bio         String?
  photoUrl    String?

  rating      Float            @default(0)
  totalRatings Int             @default(0)

  status      TechnicianStatus @default(OFFLINE)

  isActive    Boolean          @default(true)

  deletedAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  user              User                      @relation(fields: [userId], references: [id])
  serviceCategories TechnicianServiceCategory[]
  availability      TechnicianAvailability[]
  slots             TechnicianSlot[]
  services          Service[]                  @relation("TechnicianServices")
  inventory         TechnicianInventoryItem[]
  locationUpdates   TechnicianLocation[]
  penalties         TechnicianPenalty[]
  cancellationRequests CancellationRequest[]
}
```

---

### TechnicianServiceCategory

Tipos de servicio que cubre un técnico.
Relación muchos a muchos entre TechnicianProfile y ServiceCategory.

```prisma
model TechnicianServiceCategory {
  id                  String          @id @default(uuid()) @db.Uuid
  technicianProfileId String @db.Uuid
  category            ServiceCategory

  createdAt DateTime @default(now())

  // Relations
  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])

  @@unique([technicianProfileId, category])
}
```

---

### TechnicianAvailability

Disponibilidad recurrente del técnico por día de semana y franja horaria.

```prisma
model TechnicianAvailability {
  id                  String @id @default(uuid()) @db.Uuid
  technicianProfileId String @db.Uuid
  dayOfWeek           Int    // 0 = domingo, 6 = sábado
  startTime           String // "08:00"
  endTime             String // "18:00"

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])

  @@unique([technicianProfileId, dayOfWeek])
}
```

---

### TechnicianSlot

Slots individuales de agenda del técnico por fecha y hora.
Se crean automáticamente basados en TechnicianAvailability.

```prisma
model TechnicianSlot {
  id                  String     @id @default(uuid())  @db.Uuid
  technicianProfileId String  @db.Uuid

  date                DateTime
  startTime           String     // "08:00"
  endTime             String     // "10:00"

  status              SlotStatus @default(AVAILABLE)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])
  serviceSlots      ServiceSlot[]
  
  @@unique([technicianProfileId, date, startTime])
}
```

---
### TechnicianLocation

Historial de ubicaciones GPS del técnico durante el servicio.
Solo se registra cuando el estado del servicio es ON_THE_WAY.

```prisma
model TechnicianLocation {
  id                  String   @id @default(uuid())  @db.Uuid
  technicianProfileId String  @db.Uuid
  serviceId           String?  @db.Uuid
  latitude            Float
  longitude           Float
  recordedAt          DateTime @default(now())

  // Relations
  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])
  service   Service? @relation(fields: [serviceId], references: [id])
}
```

---

### TechnicianInventoryItem

Inventario de herramientas e insumos por técnico.
Se actualiza diariamente via job de BullMQ.

```prisma
model TechnicianInventoryItem {
  id                  String  @id @default(uuid())  @db.Uuid
  technicianProfileId String  @db.Uuid
  name                String
  quantity            Int     @default(0)
  unit                String? // "unidades", "metros", etc.
  lastUpdatedAt       DateTime @default(now())

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])
}
```

###  model TechnicianPenalty

Cuando un técnico abandona o solicita reasignar un servicio faltando menos de 24 horas y habiendo aceptado la orden hace mas de 48 horas

```prisma
model TechnicianPenalty {
  id                  String   @id @default(uuid()) @db.Uuid
  technicianProfileId String   @db.Uuid
  serviceId           String   @db.Uuid

  amount   Float
  reason   String

  createdAt DateTime @default(now())

  @@index([technicianProfileId])

  technicianProfile TechnicianProfile @relation(fields: [technicianProfileId], references: [id])
  service           Service           @relation(fields: [serviceId], references: [id])
}

```

---

## Modelo: Service

Orden de servicio. Núcleo del sistema.
Conecta cliente, técnico, chat y facturación.

```prisma
model Service {
  id                  String        @id @default(uuid())  @db.Uuid
  clientId String @db.Uuid //supabase 
  technicianProfileId String?  @db.Uuid

  type                ServiceType
  category            ServiceCategory
  status              ServiceStatus @default(TECHNICIAN_ASSIGNED)

  description         String
  address             String
  city                String        @default("Bogotá")

  serviceSlots ServiceSlot[]

  // Pricing
  visitCost           Float         @default(70000)
  estimatedCost       Float?
  finalCost           Float?
  visitCostDiscounted Boolean       @default(false)

  // Diagnosis
  diagnosisNotes      String?
  diagnosisProgress   Int?          // porcentaje avance si se detiene

  // Timer
  confirmationExpiresAt DateTime?
  confirmedAt           DateTime?

  // Session
  sessionAbandonedAt    DateTime?
  inactivityAlertSentAt DateTime?

  deletedAt DateTime?

  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([clientId])
  @@index([technicianProfileId])

  //Relations
  client            User                  @relation("ClientServices", fields: [clientId], references: [id])
  technicianProfile TechnicianProfile?    @relation("TechnicianServices", fields: [technicianProfileId], references: [id])
  chat              Chat?
  invoice           Invoice?
  statusHistory     ServiceStatusHistory[]
  cancellationRequests CancellationRequest[]

  diagnosisUpdates     DiagnosisUpdate[]
  locationUpdates      TechnicianLocation[]
  supportCases         SupportCase[]
  penalties            TechnicianPenalty[]   
}
```

---

### Modelo ServiceSlot

```prisma
model ServiceSlot {
  id        String   @id @default(uuid()) @db.Uuid
  serviceId String   @db.Uuid
  slotId    String   @db.Uuid

  createdAt DateTime @default(now())

  // Relations
  service Service @relation(fields: [serviceId], references: [id])
  slot    TechnicianSlot @relation(fields: [slotId], references: [id])

  @@unique([slotId]) // un slot solo pertenece a un servicio
  @@index([serviceId])
}
```

### ServiceStatusHistory

Historial de cambios de estado del servicio.
Necesario para la línea de tiempo visible en el panel fijo del chat.

```prisma
model ServiceStatusHistory {
  id        String        @id @default(uuid())  @db.Uuid
  serviceId String  @db.Uuid
  status    ServiceStatus
  note      String?
  createdAt DateTime      @default(now())

  // Relations
  service Service @relation(fields: [serviceId], references: [id])
}
```

---
### CancellationRequest


Solicitud de cancelación de un técnico a admin.
Admin decide si aprueba y reasigna.

```prisma
model CancellationRequest {
  id          String                    @id @default(uuid()) @db.Uuid

  serviceId   String                    @db.Uuid
  requestedBy String                    @db.Uuid

  type        CancellationRequestType

  reason      String

  status      CancellationRequestStatus @default(PENDING)

  serviceStatusAtRequest ServiceStatus

  appliesVisitCharge Boolean @default(false)

  technicianProfileId String? @db.Uuid

  resolvedBy String? @db.Uuid
  resolvedAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  service             Service             @relation(fields: [serviceId], references: [id])

  requester           User                @relation("UserCancellationRequests", fields: [requestedBy], references: [id])

  resolver            User?               @relation("ResolvedCancellation", fields: [resolvedBy], references: [id])

  technicianProfile   TechnicianProfile?  @relation(fields: [technicianProfileId], references: [id])

  @@index([serviceId])
  @@index([requestedBy])
  @@index([technicianProfileId])
}

```
---

### SupportCase

Caso de soporte o garantía creado por la IA cuando un cliente
reporta un problema post-servicio.

```prisma
model SupportCase {
  id          String           @id @default(uuid())  @db.Uuid
  clientId    String  @db.Uuid
  serviceId   String?  @db.Uuid

  description String

  status      SupportCaseStatus @default(OPEN)

  assignedTo  String?            @db.Uuid          // userId del admin
  resolvedAt  DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  client  User    @relation(fields: [clientId], references: [id])
  service Service? @relation(fields: [serviceId], references: [id])
}
```

---

## Modelo Chat

Canal de comunicación de un servicio.
Relación 1 a 1 con Service.

```prisma
model Chat {
  id        String   @id @default(uuid())  @db.Uuid
  serviceId String   @unique  @db.Uuid
  isActive  Boolean  @default(true)
  deletedAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  service  Service   @relation(fields: [serviceId], references: [id])
  messages Message[]
}
```

---

## Modelo Message

Mensaje individual dentro de un chat.
Soporta todos los tipos: texto, audio, imagen, video, archivo, sistema.

```prisma
model Message {
  id              String      @id @default(uuid())  @db.Uuid
  chatId          String  @db.Uuid
  senderId        String? @db.Uuid    // null si es mensaje del sistema o IA
  type            MessageType
  content         String?     // texto o transcripción
  audioUrl        String?     // URL del audio original en R2
  fileUrl         String?     // URL de imagen, video o archivo en R2
  fileName        String?
  isTranscription Boolean     @default(false)
  isSystemMessage Boolean     @default(false)

  // Approval (para diagnósticos actualizados)
  requiresApproval Boolean  @default(false)
  isApproved       Boolean?
  approvedAt       DateTime?

  deletedAt DateTime?

  createdAt DateTime @default(now())

  // Relations
  chat   Chat  @relation(fields: [chatId], references: [id])
  sender User? @relation("UserMessages", fields: [senderId], references: [id])
  diagnosisUpdate DiagnosisUpdate?
}
```
---

## Modelo DiagnosisUpdate

Actualización de diagnóstico propuesta por el técnico durante el servicio.
El cliente debe aprobar o rechazar.

```prisma
model DiagnosisUpdate {
  id          String   @id @default(uuid())  @db.Uuid
  serviceId   String  @db.Uuid
  messageId   String   @unique  @db.Uuid
  newCost     Float
  description String
  isApproved  Boolean?
  resolvedAt  DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  service Service @relation(fields: [serviceId], references: [id])
  message Message @relation(fields: [messageId], references: [id])
}
```

---

## Modelo Invoice

Factura generada al completar el servicio.

```prisma
model Invoice {
  id            String        @id @default(uuid())  @db.Uuid
  serviceId     String        @unique  @db.Uuid

  totalAmount   Float
  visitCost     Float         @default(70000)
  serviceAmount Float

  fileUrl       String?       // PDF en R2

  paymentMethod PaymentMethod?
  paymentStatus PaymentStatus @default(PENDING)

  paidAt        DateTime?

  confirmedBy   String?       @db.Uuid      // userId del admin que confirmó

  deletedAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  service Service @relation(fields: [serviceId], references: [id])
  survey  Survey?
}
```

---

## Modelo Survey

Encuesta de satisfacción post-pago.

```prisma
model Survey {
  id        String  @id @default(uuid())  @db.Uuid
  invoiceId String  @unique @db.Uuid
  rating    Int     // 1 a 5
  comment   String?

  createdAt DateTime @default(now())

  // Relations
  invoice Invoice @relation(fields: [invoiceId], references: [id])
}
```

---

## Fuera de alcance en esta fase

- No implementar lógica de negocio
- No implementar endpoints
- No implementar eventos
- Solo definir el schema y verificar que las migraciones corren

---

## Criterios de éxito

La fase se considera completa cuando:

- Schema válido sin errores de Prisma
- Migración inicial corre sin errores
- Prisma Client generado correctamente
- Todos los modelos y enums están definidos
- Relaciones correctamente tipadas
