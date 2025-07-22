# Database Requirements - Bank Account Opening System

**For Designers:** Fill out the business requirements below. Technical standards and implementation details are handled separately.

---

## 1. Project Information

| Field | Your Answer |
|-------|-------------|
| **Project Name** | `Digital Bank Account Opening System` |
| **Database Type** | `PostgreSQL` |
| **Application Framework** | `Spring Boot JPA` |
| **Business Purpose** | `A digital onboarding system that allows customers to open various types of bank accounts online. The system handles customer verification, document collection, compliance checks, and account provisioning. Used by retail customers, business customers, and bank staff for account management.` |

---

## 2. What are your main business entities?

List the main "things" your system needs to track:

```
Your entities:
- Customers
- Account Applications
- Bank Accounts
- Documents
- Identity Verification
- Address Verification
- Compliance Checks
- Account Types
- Branches
- Staff Members
```

---

## 3. How do these entities relate to each other?

Describe the relationships in simple terms:

```
Your relationships:
- Customers can have multiple Account Applications
- Account Applications can result in one Bank Account
- Customers can have multiple Bank Accounts
- Account Applications require multiple Documents
- Account Applications undergo multiple Compliance Checks
- Identity Verification belongs to one Customer
- Address Verification belongs to one Customer
- Bank Accounts belong to one Account Type
- Bank Accounts are managed by one Branch
- Staff Members can process multiple Account Applications
- Customers can have multiple Address Verifications (historical)
```

---

## 4. Entity Details

### Entity 1: Customers

**What is this?** `Represents individuals or businesses who want to open bank accounts. Contains personal/business information and contact details.`

**Sample data:**
```
- first_name: "Sarah"
- last_name: "Johnson"
- date_of_birth: "1985-03-15"
- email: "sarah.johnson@email.com"
- phone: "+1-555-234-5678"
- customer_type: "INDIVIDUAL"
- nationality: "US"
- tax_id: "123-45-6789"
- employment_status: "EMPLOYED"
- annual_income: 75000.00
- registration_date: "2024-01-20"
- kyc_status: "VERIFIED"
```

**Business rules:**
```
- Email must be unique across all customers
- Tax ID must be unique for individual customers
- Phone number must be valid format
- Date of birth must indicate customer is at least 18 years old
- KYC status can be: PENDING, IN_PROGRESS, VERIFIED, REJECTED
- Customer type can be: INDIVIDUAL, BUSINESS, JOINT
- Annual income must be positive number
- Nationality must be valid country code
```

---

### Entity 2: Account Applications

**What is this?** `Represents a customer's request to open a new bank account. Tracks the application process from submission to approval/rejection.`

**Sample data:**
```
- application_number: "APP-2024-001234"
- customer_id: "CUST-789012"
- account_type: "CHECKING"
- initial_deposit: 500.00
- application_status: "UNDER_REVIEW"
- submitted_at: "2024-01-20T10:30:00Z"
- branch_code: "NYC001"
- assigned_staff_id: "STAFF-456"
- purpose: "Primary checking account"
- estimated_monthly_transactions: 75
- wants_direct_deposit: true
- wants_online_banking: true
- wants_debit_card: true
- risk_score: 25
- priority: "STANDARD"
```

**Business rules:**
```
- Application number must be unique and auto-generated
- Customer can have only one PENDING application per account type
- Initial deposit must meet minimum requirements for account type
- Application status can be: DRAFT, SUBMITTED, UNDER_REVIEW, APPROVED, REJECTED, CANCELLED
- Risk score must be between 0-100
- Priority can be: LOW, STANDARD, HIGH, URGENT
- Applications expire after 30 days if not completed
```

---

### Entity 3: Bank Accounts

**What is this?** `Actual bank accounts created after successful application approval. Contains account details and current status.`

**Sample data:**
```
- account_number: "1234567890"
- account_type: "CHECKING"
- customer_id: "CUST-789012"
- branch_code: "NYC001"
- current_balance: 500.00
- available_balance: 500.00
- account_status: "ACTIVE"
- opened_date: "2024-01-22"
- interest_rate: 0.01
- minimum_balance: 25.00
- monthly_fee: 0.00
- overdraft_limit: 0.00
- last_statement_date: "2024-01-31"
```

**Business rules:**
```
- Account number must be unique and system-generated
- Current balance cannot be negative unless overdraft is approved
- Available balance = current balance - holds - minimum balance
- Account status can be: ACTIVE, SUSPENDED, CLOSED, DORMANT
- Interest rate must be non-negative
- Minimum balance requirements vary by account type
- Accounts automatically become DORMANT after 12 months of inactivity
```

---

### Entity 4: Documents

**What is this?** `Digital copies of documents submitted by customers for identity verification and compliance purposes.`

**Sample data:**
```
- document_type: "DRIVERS_LICENSE"
- application_id: "APP-2024-001234"
- file_name: "drivers_license_front.pdf"
- file_path: "/documents/2024/01/20/drivers_license_front.pdf"
- file_size: 2048576
- upload_date: "2024-01-20T11:15:00Z"
- verification_status: "VERIFIED"
- expiry_date: "2028-03-15"
- issuing_state: "NY"
- document_number: "D123456789"
- issue_date: "2020-03-15"
- verification_notes: "Document verified by automated system"
```

**Business rules:**
```
- Document type can be: DRIVERS_LICENSE, PASSPORT, UTILITY_BILL, BANK_STATEMENT, TAX_RETURN, BUSINESS_LICENSE
- File size cannot exceed 10MB
- Verification status can be: PENDING, VERIFIED, REJECTED, EXPIRED
- Identity documents must not be expired
- Utility bills must be dated within last 3 months
- Each application must have at least one identity document and one address proof
```

---

### Entity 5: Identity Verification

**What is this?** `Records of identity verification checks performed on customers using various verification methods.`

**Sample data:**
```
- customer_id: "CUST-789012"
- verification_method: "DOCUMENT_SCAN"
- verification_status: "PASSED"
- verification_date: "2024-01-20T12:00:00Z"
- verification_score: 95
- provider_name: "Jumio"
- provider_reference_id: "JUM-789012345"
- face_match_score: 98
- document_authenticity: "AUTHENTIC"
- extracted_name: "Sarah Johnson"
- extracted_date_of_birth: "1985-03-15"
- extracted_address: "123 Main St, New York, NY 10001"
- verification_notes: "Automated document scan completed successfully"
```

**Business rules:**
```
- Verification method can be: DOCUMENT_SCAN, BIOMETRIC, KNOWLEDGE_BASED, MANUAL_REVIEW
- Verification status can be: PENDING, PASSED, FAILED, REQUIRES_MANUAL_REVIEW
- Verification score must be between 0-100
- Score above 80 is considered PASSED for automated verification
- Score below 60 is considered FAILED
- Scores 60-80 require manual review
- Identity verification is required before account opening
```

---

### Entity 6: Compliance Checks

**What is this?** `Records of various compliance and regulatory checks performed on customers and applications.`

**Sample data:**
```
- application_id: "APP-2024-001234"
- check_type: "AML_SCREENING"
- check_status: "CLEARED"
- check_date: "2024-01-20T13:30:00Z"
- check_provider: "WorldCheck"
- risk_level: "LOW"
- matches_found: 0
- sanctions_check_result: "CLEAR"
- pep_check_result: "CLEAR"
- adverse_media_result: "CLEAR"
- manual_review_required: false
- reviewer_notes: "Automated screening completed successfully"
```

**Business rules:**
```
- Check type can be: AML_SCREENING, SANCTIONS_CHECK, PEP_CHECK, CREDIT_CHECK, FRAUD_CHECK
- Check status can be: PENDING, CLEARED, FLAGGED, FAILED, MANUAL_REVIEW_REQUIRED
- All applications must pass AML screening before approval
- High-risk customers require additional manual review
- Credit checks are optional for basic checking accounts
- Compliance checks must be completed within 24 hours of application
```

---

## 5. Important Business Rules

List any important constraints or rules:

```
Your rules:
- Customers under 18 cannot open accounts independently
- Joint accounts require all parties to complete verification
- Business accounts require additional documentation (EIN, Articles of Incorporation)
- Maximum of 5 accounts per customer across all account types
- Minimum initial deposit varies by account type: Checking $25, Savings $100, Premium $1000
- Applications automatically expire after 30 days if incomplete
- Failed identity verification blocks account opening for 24 hours
- Three failed verification attempts require manual review
- Dormant accounts (no activity for 12 months) incur monthly fees
- Account closure requires zero balance and no pending transactions
- Staff can only process applications assigned to their branch
- High-risk applications (score >70) require manager approval
```

---

## 6. How will you access/search this data?

**This is important for database performance!** Describe how users will search and access data:

### Common Access Patterns

**For Customers:**
```
How will you find/search customers?
- By email address (for login and duplicate check) - [High frequency - every login attempt]
- By customer public ID (for API calls) - [High frequency]
- By tax ID/SSN (for compliance checks) - [Medium frequency]
- By phone number (for customer service) - [Medium frequency]
- By name (for customer service search) - [Medium frequency]
- By KYC status (for compliance reporting) - [Low frequency]
```

**For Account Applications:**
```
How will you find/search applications?
- By application number (customer and staff lookup) - [High frequency]
- By customer ID (customer's application history) - [High frequency]
- By application status (staff workflow) - [High frequency]
- By assigned staff (staff workload) - [High frequency]
- By branch code (branch management) - [Medium frequency]
- By submission date (reporting and SLA tracking) - [Medium frequency]
- By risk score range (risk management) - [Low frequency]
```

**For Bank Accounts:**
```
How will you find/search accounts?
- By account number (transactions and customer service) - [High frequency]
- By customer ID (customer account portfolio) - [High frequency]
- By account status (active accounts management) - [High frequency]
- By branch code (branch account management) - [Medium frequency]
- By account type (product reporting) - [Medium frequency]
- By balance ranges (risk management) - [Low frequency]
```

**For Documents:**
```
How will you find/search documents?
- By application ID (application processing) - [High frequency]
- By document type (compliance requirements) - [Medium frequency]
- By verification status (pending document review) - [High frequency]
- By upload date (document aging reports) - [Low frequency]
```

### Cross-Entity Searches (Joins)
```
Common combined searches:
- Customer with their active applications - [High frequency]
- Application with all required documents - [High frequency]
- Customer with all their bank accounts - [High frequency]
- Application with compliance check results - [High frequency]
- Customer with identity verification status - [Medium frequency]
- Staff member with assigned applications - [High frequency]
- Branch with all customer accounts - [Medium frequency]
```

### Reporting Needs
```
What reports do you need?
- Daily application volume by branch - [Medium frequency]
- Application approval rates by risk score - [Low frequency]
- Average processing time by application type - [Low frequency]
- Compliance check failure rates - [Low frequency]
- Customer acquisition by channel - [Low frequency]
- Account opening trends by product type - [Low frequency]
- Staff productivity metrics - [Low frequency]
- Regulatory compliance reports - [Low frequency, but critical]
```

**Frequency Guide:**
- **High**: Multiple times per minute (needs fast indexes)
- **Medium**: Few times per hour (needs good indexes) 
- **Low**: Few times per day (basic indexes okay)

---

## 7. Performance Expectations

Expected data volume:
```
Expected data volume:
- How many Customers records? [50,000 customers]
- How many Account Applications records? [100,000 applications annually]
- How many Bank Accounts records? [75,000 active accounts]
- How many Documents records? [500,000 documents]
- How fast should searches be? [Under 500ms for customer-facing, under 2 seconds for staff]

Growth expectations:
- How many new customers per month? [2,000 new customers/month]
- How many new applications per month? [8,000 applications/month]
- Document storage growth? [40,000 documents/month]
```

---

## 8. Security/Compliance

```
- Do you need to comply with GDPR? [Yes - for international customers]
- Do you handle payment card data? [No - accounts only, no card processing]
- Do you need audit trails (who changed what when)? [Yes - regulatory requirement]
- Do you need soft delete (hide instead of permanently delete)? [Yes - for compliance and audit]
- Additional compliance: BSA/AML, KYC, SOX, PCI-DSS (for future card integration)
- Data encryption required for: SSN, account numbers, document content
- Data retention: Customer data 7 years after account closure, transaction logs 5 years
- Access controls: Role-based access, branch-level data isolation
```

---

**That's it!** 

The technical implementation details (indexes, data types, naming conventions, JSON storage strategies, etc.) will be handled automatically using the standards defined in `database_standards-guidelines.md`.

**Usage Instructions:**
1. Designer fills out this requirements template
2. LLM references both this requirements file AND the standards/guidelines file
3. Result: Production-ready DBML that follows best practices