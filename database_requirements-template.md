# Database Requirements Template for DBML Design

This template helps designers provide comprehensive business requirements and data structures that LLM models can use to generate professional DBML database designs.

---

## 🤖 **LLM PROCESSING INSTRUCTIONS**

### **INPUT VALIDATION CHECKLIST**
Before generating DBML, verify these requirements are met:

**✅ MANDATORY Requirements (Must be present):**
- [ ] Target database specified (PostgreSQL or Oracle)
- [ ] Project name and basic info provided
- [ ] At least 2-3 core business entities identified
- [ ] Sample data structure provided for each main entity (normalized or JSON as appropriate)
- [ ] Primary relationships between entities described
- [ ] Basic business rules specified

**� RECOMMENDED Requirements (Highly beneficial):**
- [ ] Performance requirements specified
- [ ] Security requirements mentioned
- [ ] Critical query patterns provided
- [ ] Naming conventions specified
- [ ] Standard field patterns defined

**🔸 OPTIONAL Requirements (Process if present):**
- [ ] Migration constraints specified
- [ ] Validation criteria provided
- [ ] Special project constraints mentioned

### **DBML GENERATION REQUIREMENTS**

**Target Database Support:**
- ✅ **PostgreSQL**: Use PostgreSQL-specific features (JSONB, UUID, sequences)
- ✅ **Oracle**: Use Oracle-specific features (JSON, RAW, NUMBER, sequences)
- ❌ **Other databases**: Not supported - request clarification

**Application Framework Support:**
- ✅ **Spring Boot JPA**: Optimize for JPA entity mapping and query patterns
- ✅ **Other ORM**: General ORM-friendly design
- ✅ **Direct SQL**: Database-optimized design without ORM constraints

**ID Strategy (MANDATORY):**
- **Business Entities**: Use sequence + TSID/UUID pattern (no auto-increment)
  - Internal ID: Sequence-generated BIGINT for joins/foreign keys
  - Public ID: TSID (preferred) or UUID for APIs/external references
  - **Public ID Generation**: Application-generated (not database-generated)
- **Reference Tables**: Auto-increment acceptable for lookup tables only
- **PostgreSQL**: Use sequences with `nextval()`, not `bigserial`
- **Oracle**: Use `SEQUENCE` objects with `nextval`

**JSON Handling (MANDATORY when JSON data present):**
- PostgreSQL: Use `JSONB` with GIN indexes
- Oracle: Use `JSON` datatype or `CLOB` with constraints
- Create appropriate indexes for queried JSON paths
- JPA Integration: Design JSON fields for `@JdbcTypeCode` or custom converters

### **ACCEPTANCE CRITERIA FOR GENERATED DBML**

**Schema Completeness:**
- [ ] All entities from requirements are represented as tables
- [ ] All JSON sample fields are mapped to appropriate columns
- [ ] JSON vs normalized decisions follow the decision matrix
- [ ] All relationships are defined with proper cardinality
- [ ] All business rules are implemented as constraints

**Database-Specific Implementation:**
- [ ] Correct data types for target database
- [ ] Proper ID generation strategy implemented
- [ ] JSON fields handled with database-specific approach
- [ ] Indexes follow the priority guide (CRITICAL → HIGH → MEDIUM → LOW)

**Performance Optimization:**
- [ ] High-frequency access patterns have dedicated indexes
- [ ] Foreign key indexes created for all relationships
- [ ] Composite indexes for common query patterns
- [ ] JSON indexes for frequently queried paths

**Production Readiness:**
- [ ] Audit fields included (created_at, updated_at, etc.)
- [ ] Soft delete pattern implemented where applicable
- [ ] Multi-tenant support if specified
- [ ] Security considerations addressed
- [ ] JPA-friendly design (if Spring Boot JPA specified)

**Documentation Quality:**
- [ ] All tables have descriptive comments
- [ ] Business rules documented in table notes
- [ ] Relationships explained with comments
- [ ] Index purposes and priorities documented
- [ ] JSON storage decisions explained

### **ERROR HANDLING**
If requirements are incomplete:
1. **Missing Target Database**: Request clarification - PostgreSQL or Oracle?
2. **Insufficient Entity Data**: Request at least 2-3 entities with JSON samples
3. **Missing Relationships**: Request clarification on how entities relate
4. **Ambiguous Business Rules**: Request specific constraint details

---

## 📋 **DESIGNER INSTRUCTIONS**

### **MANDATORY Sections** (Required for all projects):
- ✅ **Project Overview** - Basic project information and target database
- ✅ **Business Domain and Entities** - Core business entities
- ✅ **Detailed Entity Requirements** - At least 2-3 main entities with JSON samples
- ✅ **Relationships and Constraints** - How entities relate to each other

### **RECOMMENDED Sections** (Highly beneficial):
- 🔶 **Technical Requirements** - Performance and security needs
- 🔶 **Sample Queries and Use Cases** - Critical query patterns
- 🔶 **Data Patterns and Standards** - Naming conventions and field standards

### **OPTIONAL Sections** (Use only when applicable):
- 🔸 **Entity Relationships Diagram** - Visual representation using Mermaid syntax
- 🔸 **Migration and Deployment** - Only for existing system migrations
- 🔸 **Success Metrics and Validation** - For complex enterprise projects
- 🔸 **Additional Context** - For projects with special constraints

**Quick Start**: Fill out only the MANDATORY sections for a basic but functional DBML design. Add RECOMMENDED sections for production-quality results.

**Note on Sample Data**: Please ensure all sample JSON data is anonymized and does not contain any real user information, sensitive credentials, or personally identifiable information. Use fictional names, emails, and data as shown in the template examples.

---

## 1. Project Overview

### Project Information
| Field | Value |
|-------|-------|
| **Project Name** | `[Enter project name]` |
| **Version** | `[e.g., 1.0.0]` |
| **Database Type** | `[PostgreSQL/Oracle]` |
| **Application Framework** | `[Spring Boot JPA/Other ORM/Direct SQL]` |
| **Author** | `[Your name/team]` |
| **Date** | `[YYYY-MM-DD]` |

### Business Context
```
[Provide 2-3 paragraphs describing:]
- What business problem this database solves
- Who will use this system
- Key business processes it supports
- Expected scale and growth
```

### Success Criteria
```
[List 3-5 key success criteria, e.g.:]
- Support 10,000 concurrent users
- Handle 1M transactions per day
- Maintain 99.9% uptime
- Comply with GDPR/SOX requirements
```

---

## 2. Business Domain and Entities

### Core Business Entities
```
[List the main business entities, e.g.:]
- Users/Customers
- Products/Services
- Orders/Transactions
- Payments
- Inventory
- Categories
- Reviews
```

### Entity Relationships Overview
```
[Describe high-level relationships, e.g.:]
- Users can place multiple Orders
- Orders contain multiple Order Items
- Products belong to Categories
- Users can write Reviews for Products
- Orders have one Payment
```

---

## 3. Detailed Entity Requirements

### Entity 1: [Entity Name]

**Business Purpose:**
```
[Describe what this entity represents in business terms]
```

**Sample Data Structure:**
```
Choose ONE approach based on your data structure:

OPTION A: Normalized Structure (Most Common)
Provide sample data as key-value pairs for structured fields:
- id: 12345
- public_id: "01HN2K8G7QRXM9VWPZ3F4T5Y6B"
- username: "john_doe"
- email: "john.doe@example.com"
- first_name: "John"
- last_name: "Doe"
- date_of_birth: "1990-05-15"
- phone_number: "+1-555-123-4567"
- account_status: "ACTIVE"
- registration_date: "2024-01-15T10:30:00Z"
- last_login_date: "2025-01-21T14:22:00Z"

OPTION B: Hybrid Structure (When some fields are variable/flexible)
Provide JSON only for truly variable data:
```json
{
  "id": 12345,
  "publicId": "01HN2K8G7QRXM9VWPZ3F4T5Y6B",
  "username": "john_doe",
  "email": "john.doe@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "accountStatus": "ACTIVE",
  "preferences": {
    "newsletter": true,
    "theme": "dark",
    "language": "en"
  },
  "metadata": {
    "source": "web_registration",
    "campaign": "summer_2024"
  }
}
```

**Data Storage Strategy:**
```
Use JSON Storage ONLY when:
- Data structure varies significantly between records
- You have truly flexible/dynamic attributes
- Configuration or preference data with unknown fields
- Metadata that doesn't need to be queried frequently

Use Normalized Tables when:
- Data structure is consistent across records
- Fields are frequently queried or filtered
- You need referential integrity
- Data is used in JOINs or complex queries

Example: User preferences and metadata are good JSON candidates,
but core user fields (name, email, status) should be normalized columns.
```

**Business Rules:**
```
- Username must be unique across the system
- Email must be unique and validated
- Phone number is optional but must be valid format if provided
- Account status can be: ACTIVE, SUSPENDED, DELETED
- Registration date cannot be modified
- Address is optional during registration
```

**Enumerated Values:**
```
- accountStatus: [ACTIVE, SUSPENDED, PENDING_DELETION, INACTIVE]
- registrationSource: [WEB, MOBILE, API, ADMIN]
- preferredTheme: [LIGHT, DARK, AUTO]
- notificationPreference: [ALL, IMPORTANT_ONLY, NONE]
```

**Data Volume & Growth:**
```
- Initial: 10,000 records
- Growth: 1,000 new records per month
- Retention: Keep all data (soft delete)
```

---

### Entity 2: [Entity Name]

**Business Purpose:**
```
[Describe what this entity represents in business terms]
```

**Sample JSON Data:**
```json
{
  "id": 67890,
  "publicId": "01HN2K9H8SRYN0WXQZ4G5U6V7C",
  "sku": "LAPTOP-DELL-XPS13-001",
  "name": "Dell XPS 13 Laptop",
  "description": "High-performance ultrabook with 13-inch display",
  "category": {
    "id": 5,
    "name": "Electronics",
    "subcategory": "Laptops"
  },
  "pricing": {
    "basePrice": 1299.99,
    "salePrice": 1199.99,
    "currency": "USD",
    "taxRate": 0.08,
    "priceHistory": [
      {"price": 1299.99, "effectiveDate": "2024-06-01T00:00:00Z"},
      {"price": 1199.99, "effectiveDate": "2024-12-01T00:00:00Z"}
    ]
  },
  "inventory": {
    "stockQuantity": 25,
    "reservedQuantity": 3,
    "minStockLevel": 5,
    "maxStockLevel": 100,
    "locations": [
      {"warehouse": "NYC", "quantity": 15},
      {"warehouse": "LA", "quantity": 10}
    ]
  },
  "specifications": {
    "processor": "Intel i7-1165G7",
    "memory": "16GB RAM",
    "storage": "512GB SSD",
    "display": "13.3-inch FHD",
    "weight": "2.8 lbs",
    "dimensions": {
      "length": 11.6,
      "width": 7.8,
      "height": 0.58,
      "unit": "inches"
    }
  },
  "attributes": {
    "color": "Silver",
    "warranty": "1 year",
    "energyRating": "A+",
    "certifications": ["FCC", "CE", "RoHS"]
  },
  "status": "ACTIVE",
  "createdDate": "2024-06-01T09:00:00Z",
  "lastUpdated": "2025-01-20T16:45:00Z",
  "images": [
    "https://cdn.example.com/products/67890/main.jpg",
    "https://cdn.example.com/products/67890/side.jpg"
  ]
}
```

**JSON Storage Strategy:**
```
Complex JSON Fields to Store:
- specifications: Product technical details (varies by category)
- attributes: Dynamic product attributes (color, size, etc.)
- pricing.priceHistory: Historical pricing data
- inventory.locations: Multi-warehouse inventory tracking

PostgreSQL Implementation:
- Use JSONB for specifications and attributes
- Create GIN indexes: CREATE INDEX idx_product_specs ON products USING GIN (specifications);
- Query example: WHERE specifications @> '{"processor": "Intel i7-1165G7"}'

Oracle Implementation:
- Use JSON datatype or CLOB with JSON constraints
- Create function-based indexes on commonly queried JSON paths
- Query example: WHERE JSON_VALUE(specifications, '$.processor') = 'Intel i7-1165G7'
```

**JSON Sample Guidelines:**
```
Provide 2-3 representative samples per entity:
1. Minimal data sample (required fields only)
2. Complete data sample (all fields populated)
3. Edge case sample (maximum complexity, special scenarios)

Example for User entity:
- Minimal: Basic user with required fields only
- Complete: Full profile with preferences, address, metadata
- Edge case: Admin user with special permissions, multiple addresses
```

**Business Rules:**
```
- SKU must be unique across all products
- Base price must be positive
- Sale price cannot exceed base price
- Stock quantity cannot be negative
- Reserved quantity cannot exceed stock quantity
- Product status can be: ACTIVE, DISCONTINUED, OUT_OF_STOCK
```

**Data Volume & Growth:**
```
- Initial: 50,000 products
- Growth: 500 new products per month
- Updates: 1,000 price/inventory updates per day
```

---

### Entity 3: [Entity Name]

**Business Purpose:**
```
[Continue with additional entities following the same pattern]
```

**Sample JSON Data:**
```json
{
  // Add sample JSON for this entity
}
```

**Business Rules:**
```
[List specific business rules for this entity]
```

---

## 4. Relationships and Constraints

### Primary Relationships
```
[Describe key relationships with cardinality, e.g.:]

User (1) ←→ (Many) Orders
- One user can have multiple orders
- Each order belongs to exactly one user
- Cascade delete: If user is deleted, archive their orders

Product (1) ←→ (Many) OrderItems  
- One product can appear in multiple order items
- Each order item references exactly one product
- Restrict delete: Cannot delete product if it has order items

Order (1) ←→ (Many) OrderItems
- One order contains multiple order items
- Each order item belongs to exactly one order
- Cascade delete: If order is deleted, delete all order items
```

### Entity Relationships Diagram *(OPTIONAL - Visual representation)*
```
[Provide a simple visual diagram using Mermaid syntax for quick comprehension:]

erDiagram
    USERS ||--o{ ORDERS : places
    ORDERS ||--|{ ORDER_ITEMS : contains
    PRODUCTS ||--|{ ORDER_ITEMS : includes
    CATEGORIES ||--o{ PRODUCTS : categorizes
    USERS ||--o{ USER_PROFILES : has
    USERS }|--|| USER_ROLES : assigned

Alternative text format:
USERS (1) → (Many) ORDERS → (Many) ORDER_ITEMS ← (Many) PRODUCTS
CATEGORIES (1) → (Many) PRODUCTS
USERS (1) → (1) USER_PROFILES
```

### Business Constraints
```
[List important business constraints, e.g.:]

Data Integrity:
- Order total must equal sum of order item totals
- Stock quantity must be updated when orders are placed
- Payment amount must match order total

Business Logic:
- Users cannot place orders if account is suspended
- Products cannot be ordered if out of stock
- Orders cannot be modified after payment is processed

Temporal Constraints:
- Order date cannot be in the future
- Delivery date must be after order date
- Return period is 30 days from delivery
```

---

## 5. Technical Requirements

### Performance Requirements
```
[Specify performance needs, e.g.:]

Query Performance:
- User login: < 200ms response time
- Product search: < 500ms for 10,000 results
- Order placement: < 1 second end-to-end
- Report generation: < 30 seconds for monthly reports

Concurrency:
- Support 1,000 concurrent users
- Handle 100 orders per minute during peak
- Support 50 concurrent admin users

Data Volume:
- 1M users, 100K products, 10M orders expected in year 1
- 100GB database size expected
- 1TB of file storage for images/documents
```

### Security Requirements
```
[Specify database-level security needs, e.g.:]

Data Protection:
- PII data must be encrypted at rest (affects column design)
- Sensitive fields need special handling (separate tables, encryption)
- Audit trail required for all data changes (audit columns needed)
- Soft delete for user accounts (is_deleted, deleted_at columns)

Compliance Requirements:
- GDPR compliance: Right to be forgotten (soft delete strategy)
- Data retention policy: 7 years for financial records (affects delete strategy)
- PCI DSS: No payment card data stored directly (tokenization approach)

Database-Level Security:
- Row-level security requirements (tenant isolation)
- Column-level encryption needs
- Audit logging requirements (who changed what when)
```



---

## 6. Data Patterns and Standards

### Naming Conventions
```
Tables: snake_case, singular (user, order, product)
Columns: snake_case (first_name, created_at, is_active)
Primary Keys: id (bigserial)
Foreign Keys: {table}_id (user_id, product_id)
Booleans: is_{condition} (is_active, is_deleted)
Timestamps: {action}_at (created_at, updated_at)

JPA Considerations (if Spring Boot JPA specified):
- Table/column names should map cleanly to Java entities
- Avoid reserved keywords that conflict with JPA
- Foreign key naming should support JPA relationship mapping
- Junction tables: {entity1}_{entity2} (user_roles, order_items)
```

### Public ID Generation Strategy
```
Public ID Generation: Application-generated (recommended)

Rationale:
- Application controls ID generation for batch operations
- Better performance for bulk inserts
- Supports distributed systems without coordination
- Enables offline-first applications
- TSID/UUID generation is more efficient in application code

Implementation:
- Generate TSID/UUID in application before database insert
- Database stores the generated ID (no triggers or functions needed)
- Application handles ID uniqueness and collision detection
```

### Enumerated Types *(RECOMMENDED - For status and category fields)*
```
Define possible values for enum-like fields from your JSON samples:

Example Enums:
- account_status: [ACTIVE, SUSPENDED, PENDING_DELETION, INACTIVE]
- order_status: [PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED, REFUNDED]
- user_role: [ADMIN, MANAGER, USER, GUEST]
- product_status: [ACTIVE, DISCONTINUED, OUT_OF_STOCK, COMING_SOON]
- payment_method: [CREDIT_CARD, DEBIT_CARD, PAYPAL, BANK_TRANSFER]

Benefits:
- Creates DBML Enum types for better schema documentation
- Enables database-level validation
- Improves API documentation and client code generation
- Prevents invalid status values
```

### Standard Fields

#### **PostgreSQL Standard Fields:**
```
Business Entity Tables:
- id: BIGINT DEFAULT nextval('table_name_id_seq') PRIMARY KEY
- public_id: VARCHAR(26) NOT NULL UNIQUE (for TSID)
- created_at: TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
- updated_at: TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
- created_by: BIGINT REFERENCES users(id)
- updated_by: BIGINT REFERENCES users(id)
- version: INTEGER DEFAULT 1 (optimistic locking)

Reference/Lookup Tables:
- id: SERIAL PRIMARY KEY
- name: VARCHAR(100) NOT NULL
- is_active: BOOLEAN DEFAULT true

Soft Delete Pattern:
- is_deleted: BOOLEAN DEFAULT false
- deleted_at: TIMESTAMPTZ (nullable)
- deleted_by: BIGINT REFERENCES users(id) (nullable)

JSON Fields:
- metadata: JSONB (flexible attributes)
- preferences: JSONB (user preferences)
```

#### **Oracle Standard Fields:**
```
Business Entity Tables:
- id: NUMBER(19) DEFAULT table_name_seq.nextval PRIMARY KEY
- public_id: VARCHAR2(26) NOT NULL UNIQUE (for TSID)
- created_at: TIMESTAMP WITH TIME ZONE DEFAULT SYSTIMESTAMP
- updated_at: TIMESTAMP WITH TIME ZONE DEFAULT SYSTIMESTAMP
- created_by: NUMBER(19) REFERENCES users(id)
- updated_by: NUMBER(19) REFERENCES users(id)
- version: NUMBER(10) DEFAULT 1 (optimistic locking)

Reference/Lookup Tables:
- id: NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY
- name: VARCHAR2(100) NOT NULL
- is_active: NUMBER(1) DEFAULT 1 CHECK (is_active IN (0,1))

Soft Delete Pattern:
- is_deleted: NUMBER(1) DEFAULT 0 CHECK (is_deleted IN (0,1))
- deleted_at: TIMESTAMP WITH TIME ZONE (nullable)
- deleted_by: NUMBER(19) REFERENCES users(id) (nullable)

JSON Fields:
- metadata: JSON (flexible attributes)
- preferences: JSON (user preferences)
```

#### **ID Generation Strategy:**
```
Business Entity Tables (users, orders, products):
- Internal ID: BIGINT with sequence (for joins/foreign keys)
- Public ID: TSID or UUID (for APIs/external use)
- PostgreSQL: id BIGINT DEFAULT nextval('table_name_id_seq')
- Oracle: id NUMBER(19) DEFAULT table_name_seq.nextval

Reference/Lookup Tables (countries, roles, statuses):
- Simple auto-increment acceptable
- PostgreSQL: id SERIAL PRIMARY KEY
- Oracle: id NUMBER GENERATED BY DEFAULT AS IDENTITY

TSID vs UUID:
- TSID: 01HN2K8G7QRXM9VWPZ3F4T5Y6B (26 chars, time-ordered, preferred)
- UUID: 550e8400-e29b-41d4-a716-446655440000 (36 chars, random)

Required Indexes:
- Always index public_id columns
- Use B-tree indexes for TSID/UUID lookups
```

### JSON vs Normalized Decision Matrix

**Use JSON Storage For:**
- Variable attributes that differ by category/type
- User preferences and settings
- Metadata and flexible properties
- Product specifications that vary by category
- Configuration data with unknown structure
- Audit trail details and change logs

**Use Normalized Tables For:**
- Relationships between entities
- Frequently queried and filtered fields
- Structured data with fixed schema
- Data requiring referential integrity
- Fields used in JOINs and complex queries
- Data requiring strong consistency

**Hybrid Approach (Recommended):**
- Store frequently queried fields as columns
- Store flexible/variable data as JSON
- Example: `name VARCHAR(255), category_id BIGINT, specifications JSONB`

### Data Types Preferences

#### **PostgreSQL Specific:**
```
Strings: varchar with appropriate limits (avoid text unless needed)
Numbers: decimal for money, integer for counts, bigint for IDs
Dates: timestamp with timezone (timestamptz) for all datetime fields
Booleans: boolean with meaningful names (is_active, not active)
JSON: jsonb (better performance and indexing than json)
UUIDs: uuid type with proper indexing (consider TSID for better performance)
Arrays: Use PostgreSQL arrays sparingly, prefer junction tables for relationships
Sequences: Use bigserial or custom sequences for auto-incrementing IDs
```

#### **Oracle Specific:**
```
Strings: VARCHAR2 with appropriate limits (avoid CLOB unless needed)
Numbers: NUMBER for money/decimals, INTEGER for counts, NUMBER(19) for IDs
Dates: TIMESTAMP WITH TIME ZONE for all datetime fields
Booleans: NUMBER(1) CHECK (value IN (0,1)) or VARCHAR2(1) CHECK (value IN ('Y','N'))
JSON: JSON datatype (Oracle 21c+) or CLOB with JSON constraints (older versions)
UUIDs: RAW(16) for storage efficiency, or VARCHAR2(36) for readability
Sequences: Use Oracle SEQUENCE objects for auto-incrementing IDs
LOBs: Use BLOB for binary data, CLOB for large text
```

#### **Cross-Database ID Strategy:**
```
Primary Keys:
- PostgreSQL: Use bigserial or TSID (Time Series ID) for better performance
- Oracle: Use NUMBER(19) with SEQUENCE for auto-increment

External/Public IDs:
- Both: Use TSID or UUID with proper indexing
- TSID advantages: Time-ordered, shorter, better performance
- UUID advantages: Globally unique, standard format

Index-Friendly UUID/TSID:
- Store as binary format when possible (RAW(16) in Oracle, uuid in PostgreSQL)
- Create dedicated indexes on UUID/TSID columns
- Consider TSID for better clustering and performance
```

---

## 7. Data Access Patterns and Index Tuning

### **Primary Data Access Paths** *(RECOMMENDED - Critical for performance)*

**Define how data will be accessed to optimize indexing strategy:**

#### **Entity 1: [Entity Name] Access Patterns**
```
Primary Access Patterns:
1. Lookup by public_id (API calls): WHERE public_id = ?
   - Frequency: High (1000+ queries/minute)
   - Index: B-tree on public_id (unique)

2. User authentication: WHERE email = ? AND is_deleted = false
   - Frequency: High (500+ queries/minute)
   - Index: B-tree on email, partial index on is_deleted = false

3. Search by name with filters: WHERE name ILIKE ? AND category_id = ? AND is_active = true
   - Frequency: Medium (100+ queries/minute)
   - Index: Composite index on (category_id, is_active, name)

4. Time-based queries: WHERE created_at BETWEEN ? AND ? ORDER BY created_at DESC
   - Frequency: Medium (reporting queries)
   - Index: B-tree on created_at

5. JSON attribute filtering: WHERE preferences->>'theme' = 'dark'
   - Frequency: Low (admin queries)
   - Index: GIN on preferences (PostgreSQL) or function-based (Oracle)

Range Queries:
- Date ranges: created_at, updated_at
- Numeric ranges: price, quantity, age
- Status filtering: is_active, status, is_deleted

Sorting Requirements:
- Default sort: created_at DESC
- Secondary sort: name ASC, updated_at DESC
- Pagination: LIMIT/OFFSET or cursor-based
```

#### **Entity 2: [Entity Name] Access Patterns**
```
Primary Access Patterns:
1. [Define access pattern]
   - Frequency: [High/Medium/Low]
   - Index: [Specific index recommendation]

2. [Define access pattern]
   - Frequency: [High/Medium/Low]
   - Index: [Specific index recommendation]

Join Patterns:
- Most common joins: [entity1] JOIN [entity2] ON [condition]
- Join frequency: [High/Medium/Low]
- Foreign key indexes: Required on all FK columns

Aggregation Queries:
- COUNT queries: GROUP BY [columns]
- SUM/AVG queries: GROUP BY [columns]
- Reporting queries: Complex aggregations with date ranges
```

### **Cross-Entity Access Patterns**
```
Common Join Patterns:
1. User → Orders → Order Items (user order history)
   - Frequency: High
   - Indexes: users(public_id), orders(user_id, created_at), order_items(order_id)

2. Product → Category → Inventory (product catalog with stock)
   - Frequency: High
   - Indexes: products(category_id, is_active), inventory(product_id)

3. Order → Payment → User (payment processing)
   - Frequency: Medium
   - Indexes: orders(id), payments(order_id), users(id)

Reporting Access Patterns:
1. Daily sales: Orders grouped by date with totals
   - Query: SELECT DATE(created_at), COUNT(*), SUM(total) FROM orders GROUP BY DATE(created_at)
   - Index: orders(created_at, status) where status = 'COMPLETED'

2. Top products: Products with highest sales volume
   - Query: Complex join between products, order_items, orders
   - Index: Composite indexes on join columns and aggregation fields

3. User activity: User login patterns and behavior
   - Query: User sessions with date ranges and activity metrics
   - Index: user_sessions(user_id, created_at, is_active)
```

### **Index Priority Guide**
```
CRITICAL (Must have - create first):
- Primary keys (automatic)
- Foreign keys (all FK columns)
- Unique constraints (business rules)
- Public ID lookups (API access)

HIGH (Performance essential):
- Frequently filtered columns (WHERE clauses)
- Authentication fields (email, username)
- Status/active flags with partial indexes
- Time-based queries (created_at, updated_at)

MEDIUM (Performance beneficial):
- Sorting columns (ORDER BY clauses)
- Join columns beyond foreign keys
- Composite indexes for common query patterns
- Range query columns (price, quantity)

LOW (Nice to have):
- JSON paths (specific use cases only)
- Text search columns (full-text search)
- Reporting-only queries
- Admin/infrequent operations
```

### **Performance Requirements by Access Pattern**
```
High-Frequency Queries (>100 queries/minute):
- Response time: <50ms
- Must have dedicated indexes (CRITICAL + HIGH priority)
- Consider read replicas for scaling

Medium-Frequency Queries (10-100 queries/minute):
- Response time: <200ms
- Shared indexes acceptable (MEDIUM priority)
- Monitor for optimization opportunities

Low-Frequency Queries (<10 queries/minute):
- Response time: <1000ms
- Can use existing indexes (LOW priority)
- Acceptable for table scans on small tables

Batch Operations:
- Bulk inserts: Optimize for write performance
- Data exports: Optimize for large result sets
- ETL processes: Consider maintenance windows
```

---

## 8. Sample Queries and Use Cases

### Critical Query Patterns

#### **PostgreSQL Specific Queries:**
```sql
-- User authentication with JSON preferences
SELECT id, username, password_hash, is_active, preferences
FROM users 
WHERE email = ? AND is_deleted = false;

-- Product search with JSONB specifications
SELECT p.*, c.name as category_name,
       p.specifications->>'processor' as processor,
       p.specifications->>'memory' as memory
FROM products p 
JOIN categories c ON p.category_id = c.id 
WHERE p.name ILIKE ? 
  AND p.specifications @> '{"processor": "Intel i7"}' 
  AND p.is_active = true 
ORDER BY p.created_at DESC 
LIMIT 20;

-- JSON aggregation for order summary
SELECT o.*, 
       jsonb_agg(
         jsonb_build_object(
           'product_name', p.name,
           'quantity', oi.quantity,
           'price', oi.unit_price
         )
       ) as items
FROM orders o 
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.user_id = ? 
GROUP BY o.id 
ORDER BY o.created_at DESC;

-- TSID/UUID lookup (index-friendly)
SELECT * FROM users WHERE public_id = ?;
```

#### **Oracle Specific Queries:**
```sql
-- User authentication with JSON preferences
SELECT id, username, password_hash, is_active, preferences
FROM users 
WHERE email = ? AND is_deleted = 0;

-- Product search with JSON specifications
SELECT p.*, c.name as category_name,
       JSON_VALUE(p.specifications, '$.processor') as processor,
       JSON_VALUE(p.specifications, '$.memory') as memory
FROM products p 
JOIN categories c ON p.category_id = c.id 
WHERE UPPER(p.name) LIKE UPPER(?) 
  AND JSON_VALUE(p.specifications, '$.processor') LIKE '%Intel i7%'
  AND p.is_active = 1
ORDER BY p.created_at DESC 
FETCH FIRST 20 ROWS ONLY;

-- JSON aggregation using JSON_ARRAYAGG
SELECT o.*, 
       JSON_ARRAYAGG(
         JSON_OBJECT(
           'product_name' VALUE p.name,
           'quantity' VALUE oi.quantity,
           'price' VALUE oi.unit_price
         )
       ) as items
FROM orders o 
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.user_id = ? 
GROUP BY o.id, o.order_number, o.created_at
ORDER BY o.created_at DESC;

-- RAW UUID lookup (index-friendly)
SELECT * FROM users WHERE public_id = HEXTORAW(?);
```

#### **Cross-Database JSON Query Patterns:**
```sql
-- Specify which JSON fields need to be queryable:

User Preferences Queries:
- Filter by theme preference
- Filter by notification settings
- Filter by language preference

Product Specification Queries:
- Search by processor type
- Filter by memory size
- Search by specific attributes

Inventory Location Queries:
- Find products in specific warehouse
- Calculate total stock across locations
- Find low-stock items by location

Required JSON Indexes:
PostgreSQL: GIN indexes on JSONB columns
Oracle: Function-based indexes on JSON_VALUE expressions
```

### Reporting Requirements
```sql
-- Example reports that need to be generated

-- Daily sales report
SELECT DATE(o.created_at) as sale_date,
       COUNT(o.id) as order_count,
       SUM(o.total_amount) as total_revenue
FROM orders o 
WHERE o.status = 'COMPLETED'
  AND o.created_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE(o.created_at)
ORDER BY sale_date DESC;

-- Top selling products
SELECT p.name, p.sku, 
       COUNT(oi.id) as times_ordered,
       SUM(oi.quantity) as total_quantity_sold
FROM products p
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.created_at >= CURRENT_DATE - INTERVAL '90 days'
  AND o.status = 'COMPLETED'
GROUP BY p.id, p.name, p.sku
ORDER BY total_quantity_sold DESC
LIMIT 10;
```

---

## 8. Migration and Deployment *(OPTIONAL - Only for existing system migrations)*

### Data Migration Requirements
```
[Complete this section ONLY if migrating from an existing system:]

Source System: [Describe current system]
Data Volume: [Amount of data to migrate]
Migration Strategy: [Big bang vs phased approach]
Data Mapping: [How old data maps to new schema]
Validation Rules: [How to verify migration success]
Rollback Plan: [How to rollback if migration fails]
```



---

## 9. Success Metrics and Validation *(OPTIONAL - For complex enterprise projects)*

### Acceptance Criteria
```
[Complete this section for enterprise projects with strict validation requirements:]

Schema Validation:
- All entities from requirements are represented
- All relationships are properly defined
- All business rules are enforced through constraints
- Naming conventions are followed consistently

Performance Validation:
- All critical queries have appropriate indexes
- Query execution plans meet performance requirements
- Database can handle expected concurrent load

Security Validation:
- Sensitive data is properly classified
- Access controls are defined
- Audit trails are implemented
- Compliance requirements are met
```

---

## 10. Schema Validation and Testing *(RECOMMENDED - Quality assurance)*

### **Schema Validation Queries**
```
Provide queries to validate the generated schema works correctly:

Constraint Validation:
- Check all business rules are enforced
- Verify foreign key relationships work
- Test unique constraints prevent duplicates
- Validate check constraints reject invalid data

Data Integrity Tests:
- INSERT valid data samples
- INSERT invalid data (should fail)
- UPDATE operations with business rule validation
- DELETE operations with cascade/restrict behavior

Performance Testing:
- Execute high-frequency queries with EXPLAIN PLAN
- Test query response times with sample data
- Verify indexes are being used correctly
- Check for missing indexes on slow queries

JSON Schema Validation (if applicable):
- Validate JSON structure matches expected format
- Test JSON queries and indexing
- Verify JSON constraints work correctly
```

### **Sample Validation Scenarios**
```
Business Rule Testing:
1. Try to create user with duplicate email (should fail)
2. Try to create order with invalid status (should fail)
3. Try to delete user with existing orders (should fail/cascade based on design)
4. Try to set negative inventory quantity (should fail)

Performance Testing:
1. Query user by public_id (should use index, <50ms)
2. Search products by category (should use index, <200ms)
3. Generate daily sales report (should complete <30s)
4. Bulk insert 1000 records (should complete <10s)

JSON Testing (if applicable):
1. Query products by JSON specification (should use GIN/function-based index)
2. Update JSON preferences (should maintain structure)
3. Search within JSON arrays (should perform adequately)
```

---

## 11. JPA/ORM Considerations *(RECOMMENDED - If using Spring Boot JPA)*

### **JPA Entity Mapping Requirements**
```
If application framework is Spring Boot JPA, ensure:

Entity Design:
- Single primary key per table (avoid composite keys)
- Surrogate keys (id) for all business entities
- Public IDs as separate columns for API exposure
- Clean mapping to Java entity classes

Relationship Design:
- Foreign keys named for JPA @JoinColumn mapping
- Junction tables for @ManyToMany relationships
- Proper cascade delete/update for JPA cascade types
- Bidirectional relationship support

JSON Field Mapping:
- PostgreSQL JSONB → @JdbcTypeCode(SqlTypes.JSON) in JPA 3.1+
- Oracle JSON → Custom @Converter implementation
- JSON structure should map to Java POJOs

Query Optimization:
- Indexes optimized for JPQL and Criteria API queries
- Foreign key indexes to prevent N+1 query problems
- Composite indexes for common JPA query patterns
```

### **JPA Performance Patterns**
```
Lazy Loading Support:
- All foreign keys must be indexed
- Avoid deep object graphs without proper indexing
- Consider fetch strategies in relationship design

Batch Operations:
- Design for JPA batch insert/update operations
- Sequence allocation size considerations
- Bulk operation query patterns

Common JPA Query Patterns:
- findBy{Property} methods need single-column indexes
- findBy{Property1}And{Property2} need composite indexes
- @Query with JOIN operations need proper foreign key indexes
```

---

## 12. Additional Context *(OPTIONAL - For projects with special constraints)*

### Known Limitations
```
[Complete this section only if you have specific constraints that affect database design:]
- Budget constraints affecting infrastructure choices
- Timeline constraints affecting feature scope
- Technical constraints (existing systems, compliance requirements)
- Resource constraints (team size, expertise limitations)
```

### Future Considerations
```
[Complete this section if future requirements might affect current design:]
- Multi-language support
- Mobile app requirements
- International expansion
- Additional payment methods
- Advanced analytics and reporting
- Machine learning integration
```

### Questions and Assumptions
```
[List any questions or assumptions that affect database design:]

Questions:
- Should we support multiple currencies?
- Do we need real-time inventory updates?
- What level of audit detail is required?

Assumptions:
- Single timezone operation initially
- English language only for phase 1
- Credit card payments only initially
```

---

## 🎯 **LLM PROCESSING WORKFLOW**

### **Step 1: Validate Input Requirements**
```
1. Check target database is specified (PostgreSQL or Oracle)
2. Verify at least 2-3 entities with JSON samples are provided
3. Confirm basic relationships are described
4. Validate business rules are specified
5. If validation fails, request missing information before proceeding
```

### **Step 2: Process Requirements in Priority Order**

**MANDATORY Processing (Must complete):**
1. **Extract Entities**: Convert JSON samples to table structures
2. **Define Relationships**: Implement foreign keys and constraints
3. **Apply Business Rules**: Create check constraints and validations
4. **Implement ID Strategy**: Add TSID/UUID fields with proper indexing
5. **Handle JSON Data**: Use database-specific JSON storage and indexing

**RECOMMENDED Processing (If data available):**
1. **Analyze Data Access Patterns**: Create indexes based on access frequency and patterns
2. **Add Performance Indexes**: Optimize for high-frequency queries first
3. **Apply Naming Standards**: Use consistent naming conventions
4. **Add Security Features**: Implement audit trails and soft delete
5. **Optimize for Scale**: Add appropriate indexes and constraints

**OPTIONAL Processing (If specified):**
1. **Migration Considerations**: Account for existing system constraints
2. **Validation Rules**: Implement additional validation criteria
3. **Special Constraints**: Handle project-specific requirements

### **Step 3: Generate Production-Ready DBML**
```
1. Create complete table definitions with all columns
2. Define all relationships with proper cardinality
3. Add comprehensive indexes for performance
4. Include detailed documentation and comments
5. Implement database-specific optimizations
6. Validate against acceptance criteria
```

### **Step 4: Self-Validation Checklist**
Before delivering DBML, verify:
- [ ] All JSON sample data is represented in schema
- [ ] All relationships from requirements are implemented
- [ ] Database-specific features are used correctly
- [ ] All business rules are enforced through constraints
- [ ] Proper indexing strategy is implemented
- [ ] Documentation is comprehensive and clear

### **Output Format Requirements**
The generated DBML must:
- Follow DBML syntax standards exactly
- Include comprehensive table and column documentation
- Use database-specific data types correctly
- Implement proper indexing strategies
- Include all business rules as constraints
- Be immediately usable for schema generation

### **Error Response Format**
If requirements are insufficient, respond with:
```
❌ INSUFFICIENT REQUIREMENTS

Missing Required Information:
- [ ] Target database not specified (PostgreSQL or Oracle?)
- [ ] Entity JSON samples missing for: [list entities]
- [ ] Relationships not defined between: [list entity pairs]
- [ ] Business rules not specified for: [list areas]

Please provide the missing information to generate a complete DBML schema.
```

---

**Template Version:** 1.0.0  
**Last Updated:** 2025-01-22  
**Compatible with:** DBML Standards Guidelines v1.0.0