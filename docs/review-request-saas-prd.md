# Product Request: Multi-Tenant Review Outreach SaaS (Microsoft 365 + Google Business)

## 1) Product Objective
Build a secure, multi-tenant web application that helps businesses request Google reviews from their customers by sending branded outreach emails through each business's own Microsoft 365 tenant. The platform must support:
- Central administration across all companies.
- Self-service onboarding for business users.
- Subscription-based paid access with optional admin-managed free plans.
- Delivery/open tracking and campaign analytics.
- Google Business integration to read existing reviews and profile metadata.

## 2) Business Goals
1. Increase review volume and review recency for enrolled businesses.
2. Enable agencies or platform operators to manage many businesses from one admin console.
3. Monetize via recurring monthly subscriptions.
4. Reduce operational friction with self-service onboarding and automated workflows.

## 3) Core Personas
- **Platform Admin**: Oversees all companies, users, billing status, and support operations.
- **Company Owner/Manager (Tenant Admin)**: Configures Google Business + Microsoft 365, manages team users, sends campaigns, and monitors outcomes.
- **Company User (Tenant Member)**: Adds customer lists and launches campaigns subject to role permissions.

## 4) Functional Requirements

### 4.1 Authentication & Authorization
- Implement secure login for:
  - **Platform Admin** (global scope).
  - **Tenant Users** (company-scoped).
- Support role-based access control (RBAC):
  - `platform_admin`
  - `tenant_admin`
  - `tenant_user`
- Enforce strict tenant data isolation for non-platform users.
- Support password reset + optional MFA for sensitive roles.

### 4.2 Multi-Tenant Company Management
- Platform admins can:
  - Create, edit, deactivate, and reactivate companies.
  - View all tenant usage, connection status, and billing status.
  - Impersonate/read-only view for support troubleshooting (audited).
- Tenant admins can:
  - Manage only users and settings within their own company.

### 4.3 Microsoft 365 Integration (Per Tenant)
- Each tenant can connect their own Microsoft 365 tenant via OAuth.
- Required capabilities:
  - Send emails from tenant-owned mailbox/account.
  - Maintain token lifecycle (refresh, revoke, reconnect UX).
  - Validate sender identity before campaign launch.
- Configuration UI should include:
  - Sender mailbox selection.
  - From name customization.
  - Signature/footer management.

### 4.4 Google Business Integration (Per Tenant)
- Tenant provides Google Business access through OAuth.
- System can:
  - Link one or more business locations.
  - Retrieve profile/location metadata used in templates.
  - Pull existing reviews and store synchronized snapshots.
- Review sync requirements:
  - Initial historical sync on connect.
  - Scheduled incremental sync (e.g., daily/hourly).
  - Visibility into review date, rating, author alias, text, and response status.

### 4.5 Customer & Campaign Management
- Tenant users can upload/import customer contacts (CSV + manual entry).
- Data model for contacts should support:
  - Name, email, transaction/service date, optional tags/segment.
- Campaign workflow:
  1. Select audience.
  2. Select template.
  3. Insert review link pointing to Google Business listing.
  4. Schedule or send immediately through tenant’s Microsoft 365 connection.
- Guardrails:
  - Email validation, duplicate suppression, send throttling, and opt-out honoring.

### 4.6 Email Open Tracking & Delivery Telemetry
- Track at minimum:
  - Sent, delivered (if available), bounced (if available), opened, clicked.
- Open tracking approach:
  - Tracking pixel with campaign/message identifiers.
- Click tracking approach:
  - Redirect links for click analytics.
- Dashboard metrics:
  - Open rate, click-through rate, send volume, campaign performance by date range.

### 4.7 Subscription Billing & Access Control
- Implement monthly subscription model with at least:
  - Active paid plan
  - Grace/past-due state
  - Suspended/canceled state
- Platform admin can manually override access and grant free usage:
  - Per company perpetual free flag or time-bound free period.
- Billing status must gate premium actions (e.g., sending campaigns) according to plan rules.

### 4.8 Admin Overrides, Auditing, and Compliance
- Every sensitive action should generate audit events:
  - Connection changes, user role changes, billing overrides, campaign sends.
- Provide immutable activity logs for platform admin review.
- Include data retention controls and deletion workflows (account closure, contact deletion).

## 5) Non-Functional Requirements
- **Security**: Encrypt data at rest/in transit; secure token storage; least-privilege OAuth scopes.
- **Scalability**: Support growth to many tenants and high email volume.
- **Reliability**: Queue-based sending with retry/backoff and idempotency keys.
- **Observability**: Structured logs, metrics, error monitoring, alerting.
- **Performance**: Responsive dashboards and near-real-time campaign status updates.

## 6) Suggested Information Architecture
- **Public**: Landing page, pricing, login, signup.
- **Tenant App**:
  - Overview dashboard
  - Customers
  - Campaigns
  - Templates
  - Reviews (synced from Google Business)
  - Integrations (Microsoft 365 + Google)
  - Billing
  - Settings / Team
- **Platform Admin App**:
  - Companies
  - Users
  - Integrations health
  - Billing & plan overrides
  - Audit logs

## 7) Data Model (High-Level)
- `users` (id, email, role, tenant_id, status)
- `tenants` (id, company_name, plan_status, free_access_override, created_at)
- `m365_connections` (tenant_id, oauth_metadata, sender_config, status)
- `google_business_connections` (tenant_id, oauth_metadata, selected_locations, status)
- `google_reviews` (tenant_id, location_id, review_id_external, rating, text, reviewed_at)
- `contacts` (tenant_id, name, email, tags, consent_flags)
- `campaigns` (tenant_id, template_id, status, scheduled_at, created_by)
- `messages` (campaign_id, contact_id, delivery_status, open_count, click_count)
- `billing_subscriptions` (tenant_id, provider_subscription_id, state, renewal_at)
- `audit_events` (actor_id, tenant_id, action, entity_type, entity_id, timestamp)

## 8) Key User Flows
1. **Tenant onboarding**
   - Signup → create company → connect Microsoft 365 → connect Google Business → import contacts → send first campaign.
2. **Admin company oversight**
   - View all companies → inspect connection health → grant free access override if needed.
3. **Review intelligence**
   - Tenant opens Reviews page → sees synced historical and new Google reviews with filters and trend summaries.

## 9) Reporting & Analytics Requirements
- Campaign analytics by tenant and date range.
- Review growth analytics:
  - Total reviews over time
  - Average rating trend
  - New reviews after campaign sends
- Admin-level aggregate stats across all tenants.

## 10) Compliance & Legal Considerations
- Provide consent/disclaimer language for email outreach.
- Include unsubscribe handling for campaign emails where required.
- Provide data export and deletion pathways to support privacy obligations.
- Store only necessary scopes/data from external providers.

## 11) Delivery Plan (Phased)

### Phase 1 (MVP)
- Auth + tenant architecture + RBAC.
- Microsoft 365 send integration.
- Google Business connect + review sync.
- Contact import + basic campaign sending.
- Open/click tracking + core dashboards.
- Admin console with company management and free-access override.

### Phase 2
- Advanced templates and segmentation.
- Enhanced analytics and cohort views.
- Usage limits by plan.
- Expanded audit and compliance tooling.

### Phase 3
- Automation rules (e.g., send after service completion).
- Multi-location optimization and benchmark insights.
- API/webhooks for CRM integrations.

## 12) Acceptance Criteria (Must-Have)
1. A tenant can connect their own Microsoft 365 and successfully send review request emails from their own tenant account.
2. A tenant can connect Google Business and view synced existing reviews in-app.
3. Platform admin can manage all companies and toggle free access independent of subscription state.
4. System tracks opened emails and displays open metrics per campaign.
5. Billing status is enforced for paid features while respecting explicit admin free-access overrides.
6. All tenant data remains isolated from other tenants except for platform admin scope.
