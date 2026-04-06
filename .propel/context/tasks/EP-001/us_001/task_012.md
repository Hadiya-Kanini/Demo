## Task ID: task_012
## Task Title: Implement Login UI component and client integration (wireframe pending)
## Category: Frontend

## Requirement Reference
- Parent Story: us_001
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria mapped from parent story:
  - Given valid credentials, POST /auth/login returns HTTP 200 with access_token (JWT), token_type "Bearer", expires_in and no sensitive debug details.
  - Given invalid credentials, server returns HTTP 401 with generic "Invalid credentials" and failed-login counters increment.
  - Given malformed or missing fields, server returns HTTP 400 with validation errors referencing invalid fields.
  - Given valid login, token expiry equals configured TTL, server records audit log and response latency median < 1s in integration tests.

Edge Cases (from parent story):
- Database unavailable -> HTTP 503; client surfaces a retryable error without leaking internals.
- Excessive failed attempts -> server may return 429 / 423; client must surface generic message and not reveal policy details.
- Legacy password-hash verification may require different handling server-side (client should not attempt migration).

## Task Overview
Implement an accessible, reusable Login form component in the frontend codebase (React + TypeScript). The component must provide:
- Field-level validation (email/username and password) and form-level error summary.
- Loading state and disabled submit while request is in flight.
- Clear display of server-side validation and authentication errors.
- Client integration that POSTs to /auth/login with correct JSON payload and handles the documented success and error responses.
- WCAG 2.2 AA accessibility checks (labels, ARIA, focus management, live regions).
- Design/wireframe is pending — component must be implemented to match design tokens and be easily adaptable to final wireframe.

## Design References (Frontend Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **UI Impact** | Yes |
| **Figma URL** | N/A |
| **Wireframe Status** | PENDING |
| **Wireframe Type** | N/A |
| **Wireframe Path/URL** | TODO: Provide wireframe at .propel/context/wireframes/Hi-Fi/wireframe-SCR-XXX-login.[html|png|jpg] |
| **Screen Spec** | figma_spec.md#SCR-XXX (placeholder) |
| **UXR Requirements** | UXR-XXX (referenced in parent story if any) |
| **Design Tokens** | designsystem.md#forms (reference; confirm tokens once wireframe available) |

> Wireframe Status: PENDING — implement based on design tokens and accessible, responsive layout. Update styling to exact wireframe when provided.

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React | 18.x |
| Frontend | TypeScript | 5.x |
| Forms | React Hook Form | 7.x |
| Data Fetching | React Query | 4.x (mutations) |
| HTTP Client | Axios | 1.x |
| Testing (unit) | Jest + React Testing Library | latest in repo |
| E2E | Playwright or Cypress (project default) |