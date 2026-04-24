Build a complete production-ready MVP for a Tenant Management System with Capmo ticket integration.

PROJECT NAME:
mietermanagement-capmo

LANGUAGE:
English code, English UI labels, clean modern SaaS UI.

GOAL:
Create a web application where tenants can report defects. The system creates tickets in Capmo and synchronizes status changes from Capmo back into our system.

TECH STACK:
- Next.js 14 App Router
- TypeScript
- Tailwind CSS
- Prisma ORM
- PostgreSQL
- NextAuth or JWT-based auth
- REST API routes
- File upload support
- Mock Capmo API service if real credentials are missing

CORE ROLES:
1. Tenant
2. Property Manager
3. Warranty Manager
4. Admin

CORE REQUIREMENTS:

1. Tenant Login
Tenants log in using:
- tenant number
- last name

Add captcha placeholder logic, but allow local development without captcha.

2. Tenant Dashboard
After login, tenant can:
- create a new defect report
- view existing reports
- see current ticket status
- upload images
- view messages/comments
- select appointments if proposed

3. Defect Report Form
Fields:
- title
- description
- category
- type
- property
- unit / apartment
- floor
- room
- tenant contact data
- phone
- email
- images / attachments

4. Ticket Creation Logic
When a tenant submits a defect:
- create local ticket first
- upload images to storage
- create Capmo ticket via integration service
- save Capmo ticket ID locally
- initial status must ALWAYS be "OPEN"

VERY IMPORTANT:
Every newly created ticket sent to Capmo must always have initial status:

OPEN

This is mandatory and must not depend on tenant input.

5. Capmo Ticket Field Mapping

Tenant Management System → Capmo Ticket

title → short text
description → description
category → category
type → type
location / room / unit → location
due date → due date
company → company
responsible persons → responsible persons
images → attachments
status → status

Example Capmo ticket title:
"KUS9-159 - 3rd floor left | WE 33 - Loose skirting boards - 530204"

Example description:
"In 2 rooms the skirting boards are loose.

Contact details tenant:
Bernd Fritz Georg Japp or Hannah Lena Japp
Phone landline: 0175/7286292
Phone mobile: 0151/67214190
Email: b.japp@japp.de"

6. Status Synchronization

Two-way logic:

A) Our system → Capmo
Only when creating a new ticket:
status = OPEN

B) Capmo → Our system
All future status changes in Capmo must be synchronized back into our system.

Implement both:
- webhook endpoint
- polling fallback

Webhook endpoint:
POST /api/webhooks/capmo/status-update

Polling service:
GET /api/capmo/sync-status

Status mapping:
Capmo "Open" → System "Open"
Capmo "In Progress" → System "In Progress"
Capmo "On Hold" → System "On Hold"
Capmo "Done" → System "Done"
Capmo "Closed" → System "Closed"

7. Capmo Integration Service

Create:
src/lib/capmo.ts

Functions:
- createCapmoTicket()
- uploadCapmoAttachment()
- getCapmoTicketStatus()
- mapSystemTicketToCapmoPayload()
- mapCapmoStatusToSystemStatus()

Use environment variables:
CAPMO_API_BASE_URL
CAPMO_API_KEY
CAPMO_PROJECT_ID

If environment variables are missing, use mock mode.

8. Attachments / Images
Supported:
- jpg
- jpeg
- png
- pdf

Max file size:
10 MB

Attachments should be linked to local ticket and Capmo ticket.

9. Admin Dashboard
Admin can:
- view all organizations
- view properties
- view tenants
- view tickets
- see Capmo sync status
- manually trigger status sync
- see failed Capmo API calls

10. Property Manager Dashboard
Property managers can:
- view tickets for assigned properties
- filter by status
- filter by category
- filter by property
- open ticket detail
- see tenant contact data
- see Capmo ticket ID
- see attachments

11. Ticket Detail Page
Ticket detail must show:
- title
- description
- tenant contact details
- status
- Capmo ticket ID
- category
- type
- due date
- company
- responsible persons
- attachments
- comments
- status history

12. Database Models

Use Prisma models for:

User
- id
- email
- name
- role
- organizationId
- createdAt
- updatedAt

Organization
- id
- name
- createdAt
- updatedAt

Property
- id
- organizationId
- name
- address
- createdAt
- updatedAt

Tenant
- id
- propertyId
- tenantNumber
- firstName
- lastName
- email
- phone
- mobile
- unit
- floor
- createdAt
- updatedAt

Ticket
- id
- organizationId
- propertyId
- tenantId
- capmoTicketId
- title
- description
- category
- type
- location
- status
- dueDate
- company
- responsiblePersons
- capmoSyncStatus
- capmoLastSyncedAt
- createdAt
- updatedAt

Attachment
- id
- ticketId
- fileName
- fileUrl
- fileType
- fileSize
- capmoAttachmentId
- createdAt

Comment
- id
- ticketId
- authorId
- body
- source
- createdAt

StatusHistory
- id
- ticketId
- oldStatus
- newStatus
- source
- createdAt

CapmoSyncLog
- id
- ticketId
- direction
- status
- requestPayload
- responsePayload
- errorMessage
- createdAt

13. API Routes

Implement:

POST /api/auth/tenant-login
GET /api/tickets
POST /api/tickets
GET /api/tickets/:id
PATCH /api/tickets/:id
POST /api/tickets/:id/attachments
POST /api/tickets/:id/comments
POST /api/capmo/create-ticket
GET /api/capmo/sync-status
POST /api/webhooks/capmo/status-update
GET /api/admin/sync-logs

14. UI Design
Create a clean modern SaaS interface.

Design style:
- white background
- green as primary color
- blue for Capmo integration
- card-based layout
- rounded corners
- subtle shadows
- mobile responsive
- dashboard sidebar
- status badges
- upload dropzone
- ticket timeline

Pages:
- /login
- /tenant/dashboard
- /tenant/tickets/new
- /tenant/tickets/[id]
- /manager/dashboard
- /manager/tickets/[id]
- /admin/dashboard
- /admin/sync-logs

15. Seed Data
Create seed data:

Organization:
Demo Real Estate GmbH

Property:
KUS9-159

Tenant:
tenantNumber: 530204
lastName: Japp
firstName: Bernd
unit: WE 33
floor: 3rd floor left
email: b.japp@japp.de
phone: 0175/7286292
mobile: 0151/67214190

Example Ticket:
Title:
KUS9-159 - 3rd floor left | WE 33 - Loose skirting boards - 530204

Description:
In 2 rooms the skirting boards are loose.

Category:
Floor

Type:
Warranty defect

Status:
Open

Due date:
2026-04-17

Company:
P&P Group GmbH

Responsible persons:
Kenneth Wegner
Katharina Walter
Florian Zengerle
Dariia Biliavtseva

16. README
Create a complete README with:
- project description
- installation steps
- env variables
- Prisma setup
- seed command
- development start command
- Capmo integration explanation
- status sync explanation

17. Important Acceptance Criteria

The app is accepted only if:

- tenant can log in with tenant number + last name
- tenant can create ticket
- ticket is saved locally
- Capmo payload is generated
- new ticket status is always OPEN
- attachments can be uploaded
- Capmo ticket ID is stored
- Capmo status changes can be received by webhook
- Capmo status changes update local ticket
- admin can see sync logs
- UI is usable and modern

18. Mock Mode
If no Capmo API key exists:
- create fake Capmo ticket ID like CAPMO-MOCK-12345
- simulate attachment upload
- simulate status sync
- show visible “Mock Mode” badge in admin dashboard

19. Deliverables
Generate the full project with:
- package.json
- next.config.js
- tailwind.config.ts
- prisma/schema.prisma
- prisma/seed.ts
- src/app pages
- src/components
- src/lib/capmo.ts
- src/lib/auth.ts
- src/lib/db.ts
- src/lib/status.ts
- API routes
- README.md

20. Final instruction
Do not only describe the system.
Actually generate the complete working codebase.
