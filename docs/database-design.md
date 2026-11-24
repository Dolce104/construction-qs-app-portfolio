# Database Design

## 1. Overview

The application uses a relational database hosted on Neon PostgreSQL, with Prisma ORM handling database access and schema management.

The database is designed to support a multi-tenant construction management platform. Each company maintains its own operational data, including projects, tasks, tenders, documents and financial records.

The schema is organised into related models, with relationships, constraints and indexes used to maintain data integrity and support efficient queries.

## 2. Multi-Tenant Data Structure

The application uses a shared database architecture in which multiple companies store their data within the same database.

Company-specific records are associated with a `companyId`, allowing the application to identify which organisation owns each record.

This approach avoids maintaining a separate database for every customer while providing a structure for separating company data.

### User and Company Membership

The relationship between `User` and `Company` is managed through the `CompanyMembership` model.

Rather than assigning a single `companyId` directly to a user, the application uses a many-to-many relationship. This allows one user account to belong to multiple companies.

`CompanyMembership` also stores information specific to that user's relationship with a company, including:

- Company ID
- User ID
- Role ID
- Membership status
- Job title

A composite unique constraint on `[companyId, userId]` ensures that a user cannot have duplicate membership records for the same company.

This structure allows users to have different roles and permissions across different organisations.

## 3. Database Domain Structure

The schema is organised around the main business functions of the application.

| Domain | Representative models |
|---|---|
| Organisation and access | Company, User, CompanyMembership, CompanyRole |
| CRM and customers | Client, CrmLead, CrmOpportunity, TenderInvite |
| Preconstruction | PreconstructionJob, PreconstructionEstimate, EstimateRevision |
| Project delivery | Project, ProjectPhase, ProjectTask, ProjectMilestone |
| Scheduling | ProjectScheduleItem, ProjectScheduleDependency, CalendarEvent |
| Documents | ProjectDocumentFile, ProjectDocumentVersion, CompanyDocumentFile |
| Procurement | Supplier, ProjectPurchaseOrder, ProjectSupplierInvoice |
| Financial management | ProjectBillingContract, ProjectBillingClaim, ProjectBillingInvoice, ProjectBillingReceipt |
| Administration | AuditLog, Notification, ApprovalQueueItem |

This modular organisation separates different business responsibilities while allowing related records to be connected through database relationships.

## 4. Relationships and Referential Integrity

The database uses foreign-key relationships to connect related records.

For example:

- A company can have multiple projects.
- A project can have multiple phases.
- A project can have multiple tasks.
- A task can have multiple comments and document links.

This avoids storing unrelated information in a single large table and makes it easier to retrieve and maintain specific records.

### Deletion Behaviour

Relationships use different deletion rules depending on the purpose of the relationship.

Examples from the schema include:

- `Cascade`: Deleting a project also deletes dependent tasks and phases.
- `SetNull`: If an optional relationship is removed, the dependent record can remain without that reference.
- `Restrict`: Prevents deletion where dependent records must remain valid.

These rules help maintain referential integrity and avoid orphaned records.

## 5. Constraints and Data Integrity

The schema uses unique constraints to prevent duplicate records where a particular combination of values must be unique.

Examples include:

- `CompanyMembership`: Unique combination of company ID and user ID.
- `CompanyRole`: Unique combination of company ID and role name.
- `ProjectCostCode`: Unique combination of project ID and cost code.
- `ProjectDocumentVersion`: Unique combination of document ID and version number.

These constraints enforce data integrity at the database level rather than relying exclusively on application logic.

## 6. Indexing and Query Performance

Indexes are used to improve query performance, particularly when retrieving records by company, project, status or other commonly queried fields.

The schema includes both individual and composite indexes.

For example, `ProjectTask` includes indexes for:

- Company and project.
- Company, project and task status.
- Company, project and priority.
- Company, project and assigned user.
- Company, project and due date.

These indexes support common application queries, such as retrieving tasks for a particular project, filtering by status or finding tasks assigned to a particular user.

Composite index field order is selected to support the expected query patterns.

## 7. Application-Level Access Control

Database structure is one part of the application's multi-tenant security model. Access is also controlled by the backend.

The general request flow is:

1. Firebase Authentication provides the user's authentication token.
2. Firebase Admin verifies the token on the backend.
3. The backend identifies the corresponding application user.
4. Company membership is checked to confirm the user belongs to the requested organisation.
5. Role-based permissions are checked to determine whether the requested action is allowed.
6. Database queries are scoped to the authorised company ID.

This combines authentication, company membership, application-level authorisation and company-scoped data retrieval.

The database schema supports this structure, while the backend is responsible for enforcing access checks.

## 8. Design Considerations

The database design focuses on several key requirements:

- **Multi-tenancy:** Support multiple organisations within one application.
- **Data separation:** Associate company-specific records with their owning organisation.
- **Flexible membership:** Allow users to belong to multiple companies with different roles.
- **Maintainability:** Separate business domains into related models.
- **Data integrity:** Use relationships, constraints and deletion rules to maintain consistency.
- **Performance:** Use indexes to support common filtering and retrieval patterns.
- **Extensibility:** Allow additional construction management functionality to be added without requiring a completely different database structure.

The schema is designed to support the application's broader goal of providing a central operating platform for small and medium-sized construction businesses.
