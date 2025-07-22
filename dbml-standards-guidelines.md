# DBML Standards and Guidelines for AI Agents

This document provides comprehensive guidelines for creating Database Markup Language (DBML) specifications. These standards ensure consistency, maintainability, and clarity in database design and documentation.

## What is DBML?

**Database Markup Language (DBML)** is a simple, readable DSL language designed to define database structures. It's designed to be:
- **Human-readable**: Easy to write and understand
- **Version-controllable**: Works well with Git and other VCS
- **Tool-agnostic**: Can generate SQL for multiple database systems
- **Visualization-ready**: Integrates with dbdiagram.io for visual ERDs

## Core Principles

1. **Clarity**: Use descriptive names and comprehensive documentation
2. **Consistency**: Follow naming conventions throughout the schema
3. **Completeness**: Document all relationships, constraints, and business rules
4. **Maintainability**: Structure for easy updates and modifications
5. **Standards Compliance**: Follow database design best practices

## File Structure and Organization

### Directory Structure
```
db-design/
├── schemas/
│   ├── core/
│   │   ├── users.dbml
│   │   ├── authentication.dbml
│   │   └── audit.dbml
│   ├── business/
│   │   ├── orders.dbml
│   │   ├── products.dbml
│   │   └── inventory.dbml
│   └── reporting/
│       ├── analytics.dbml
│       └── dashboards.dbml
├── master-schema.dbml
├── README.md
└── docs/
    ├── business-rules.md
    └── data-dictionary.md
```

### File Naming Convention
- **Format**: `{domain}-{entity}.dbml` or `{entity}.dbml`
- **Examples**:
  - `users.dbml`
  - `order-management.dbml`
  - `product-catalog.dbml`
  - `user-authentication.dbml`

## DBML Syntax Standards

### Table Definition Standards

#### Basic Table Structure
```dbml
Table users {
  id bigserial [pk, increment, note: 'Primary key - auto-incrementing user ID']
  username varchar(50) [not null, unique, note: 'Unique username for login (3-50 characters)']
  email varchar(255) [not null, unique, note: 'User email address for notifications and recovery']
  password_hash varchar(255) [not null, note: 'Bcrypt hashed password - never store plain text']
  first_name varchar(100) [note: 'User first name']
  last_name varchar(100) [note: 'User last name']
  is_active boolean [not null, default: true, note: 'Account status - false for suspended accounts']
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`, note: 'Account creation timestamp']
  updated_at timestamp [not null, default: `CURRENT_TIMESTAMP`, note: 'Last profile update timestamp']
  
  Note: '''
  Core user account information table.
  
  Business Rules:
  - Username must be unique across the system
  - Email must be unique and valid format
  - Passwords are hashed using bcrypt with salt
  - Inactive accounts cannot login but data is preserved
  - Created/updated timestamps are in UTC
  
  Related Tables: user_profiles, user_sessions, audit_logs
  '''
}
```

#### Table with Indexes
```dbml
Table products {
  id bigserial [pk, increment]
  sku varchar(50) [not null, unique, note: 'Stock Keeping Unit - unique product identifier']
  name varchar(200) [not null, note: 'Product display name']
  description text [note: 'Detailed product description']
  category_id bigint [not null, ref: > categories.id, note: 'Product category reference']
  price decimal(10,2) [not null, note: 'Product price in base currency']
  cost decimal(10,2) [note: 'Product cost for margin calculation']
  stock_quantity integer [not null, default: 0, note: 'Current stock level']
  min_stock_level integer [default: 0, note: 'Minimum stock level for reorder alerts']
  is_active boolean [not null, default: true, note: 'Product availability status']
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  updated_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  
  Indexes {
    sku [unique, name: 'idx_products_sku']
    category_id [name: 'idx_products_category']
    (name, category_id) [name: 'idx_products_name_category']
    is_active [name: 'idx_products_active', where: 'is_active = true']
    created_at [name: 'idx_products_created']
  }
  
  Note: '''
  Product catalog with inventory tracking.
  
  Business Rules:
  - SKU must be unique across all products
  - Price must be positive
  - Stock quantity cannot be negative
  - Inactive products are hidden from catalog but orders preserved
  - Cost is optional for margin analysis
  '''
}
```

### Relationship Standards

#### One-to-Many Relationships

**Relationship Cardinality Symbols:**
- `>` = **One-to-Many** (one parent, many children)
- `<` = **Many-to-One** (many children, one parent)  
- `-` = **One-to-One** (one-to-one relationship)

```dbml
Table categories {
  id bigserial [pk, increment]
  name varchar(100) [not null, unique]
  description text
  parent_id bigint [ref: > categories.id, note: 'Self-reference for category hierarchy']
  is_active boolean [not null, default: true]
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
}

Table products {
  id bigserial [pk, increment]
  category_id bigint [not null, ref: > categories.id]  // Many products > One category
  name varchar(200) [not null]
  // ... other fields
}

// Alternative relationship syntax with explicit cardinality
Ref: products.category_id > categories.id [delete: restrict, update: cascade]
// Read as: "Many products belong to one category"
```

#### Many-to-Many Relationships
```dbml
Table users {
  id bigserial [pk, increment]
  username varchar(50) [not null, unique]
  email varchar(255) [not null, unique]
}

Table roles {
  id bigserial [pk, increment]
  name varchar(50) [not null, unique]
  description text
  permissions jsonb [note: 'JSON array of permission strings']
}

Table user_roles {
  id bigserial [pk, increment]
  user_id bigint [not null, ref: > users.id]    // Many user_roles > One user
  role_id bigint [not null, ref: > roles.id]    // Many user_roles > One role
  assigned_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  assigned_by bigint [ref: > users.id, note: 'User who assigned this role']
  expires_at timestamp [note: 'Optional role expiration date']
  
  Indexes {
    (user_id, role_id) [unique, name: 'idx_user_roles_unique']
    user_id [name: 'idx_user_roles_user']
    role_id [name: 'idx_user_roles_role']
    expires_at [name: 'idx_user_roles_expiry', where: 'expires_at IS NOT NULL']
  }
  
  Note: '''
  Junction table for user-role assignments.
  Supports temporary role assignments with expiration.
  
  Relationship Pattern:
  - Users >-< Roles (many-to-many through user_roles junction table)
  - One user can have many roles
  - One role can be assigned to many users
  '''
}

// Alternative explicit relationship definitions
Ref: user_roles.user_id > users.id [delete: cascade]
Ref: user_roles.role_id > roles.id [delete: restrict]
```

#### Self-Referencing Relationships
```dbml
Table employees {
  id bigserial [pk, increment]
  employee_code varchar(20) [not null, unique]
  first_name varchar(100) [not null]
  last_name varchar(100) [not null]
  manager_id bigint [ref: > employees.id, note: 'Reports to relationship - self reference']
  department_id bigint [not null, ref: > departments.id]
  hire_date date [not null]
  is_active boolean [not null, default: true]
  
  Note: '''
  Employee hierarchy with manager relationships.
  
  Self-Referencing Pattern:
  - manager_id > employees.id (Many employees report to one manager)
  - CEO/Top-level managers have NULL manager_id
  - Creates hierarchical tree structure
  '''
}

// One-to-One Relationship Example
Table user_profiles {
  user_id bigint [pk, ref: - users.id, note: 'One-to-one with users table']
  first_name varchar(100)
  last_name varchar(100)
  bio text
  avatar_url varchar(500)
  
  Note: '''
  One-to-One Relationship Pattern:
  - user_id - users.id (One user has exactly one profile)
  - Uses '-' symbol for one-to-one cardinality
  - Primary key is also the foreign key
  '''
}
```

## Data Types and Constraints

### Standard Data Types
```dbml
Table data_type_examples {
  // Numeric Types
  id bigserial [pk, increment]
  small_number smallint [note: '2-byte integer (-32,768 to 32,767)']
  regular_number integer [note: '4-byte integer']
  big_number bigint [note: '8-byte integer']
  price decimal(10,2) [note: 'Fixed precision decimal']
  percentage float [note: 'Floating point number']
  
  // String Types
  code varchar(10) [note: 'Variable length string with limit']
  name varchar(255) [note: 'Standard name field']
  description text [note: 'Unlimited length text']
  fixed_code char(5) [note: 'Fixed length string']
  
  // Date/Time Types
  birth_date date [note: 'Date only (YYYY-MM-DD)']
  created_at timestamp [note: 'Date and time without timezone']
  updated_at timestamptz [note: 'Date and time with timezone']
  duration interval [note: 'Time interval']
  
  // Boolean
  is_active boolean [not null, default: true]
  
  // JSON
  metadata jsonb [note: 'Binary JSON for better performance']
  settings json [note: 'Text JSON']
  
  // Arrays (PostgreSQL)
  tags varchar[] [note: 'Array of strings']
  scores integer[] [note: 'Array of integers']
  
  // UUID
  external_id uuid [unique, note: 'External system identifier']
}
```

### Constraint Standards
```dbml
Table constraint_examples {
  id bigserial [pk, increment]
  
  // Unique Constraints
  email varchar(255) [unique, not null]
  username varchar(50) [unique, not null]
  
  // Not Null
  required_field varchar(100) [not null]
  
  // Default Values
  status varchar(20) [not null, default: 'PENDING']
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  is_deleted boolean [not null, default: false]
  
  // Check Constraints (database-specific)
  age integer [note: 'CHECK (age >= 0 AND age <= 150)']
  email_validated varchar(255) [note: 'CHECK (email_validated ~* email_regex)']
  
  Note: '''
  Examples of various constraint types.
  Check constraints are documented in notes as DBML doesn't have native syntax.
  '''
}

// Composite Primary Key Examples
Table composite_pk_example {
  organization_id bigint [not null]
  user_id bigint [not null]
  role varchar(50) [not null]
  assigned_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  
  // Method 1: Using pk tags (inline)
  // organization_id bigint [pk, not null]
  // user_id bigint [pk, not null]
  
  // Method 2: Using Indexes block (recommended for clarity)
  Indexes {
    (organization_id, user_id) [pk, name: 'pk_org_user']
  }
  
  Note: '''
  Composite primary key using Indexes block syntax.
  This approach is cleaner and more explicit than inline [pk] tags.
  '''
}
```

## Schema Organization Patterns

### Domain-Driven Design Approach
```dbml
// Core Domain - User Management
Table users {
  id bigserial [pk, increment]
  username varchar(50) [not null, unique]
  email varchar(255) [not null, unique]
  // ... user fields
}

Table user_profiles {
  user_id bigint [pk, ref: - users.id, note: 'One-to-one relationship']
  first_name varchar(100)
  last_name varchar(100)
  bio text
  avatar_url varchar(500)
  // ... profile fields
}

// Business Domain - E-commerce
Table orders {
  id bigserial [pk, increment]
  user_id bigint [not null, ref: > users.id]
  order_number varchar(50) [not null, unique]
  status order_status [not null, default: 'PENDING']
  total_amount decimal(10,2) [not null]
  // ... order fields
}

Table order_items {
  id bigserial [pk, increment]
  order_id bigint [not null, ref: > orders.id]
  product_id bigint [not null, ref: > products.id]
  quantity integer [not null]
  unit_price decimal(10,2) [not null]
  // ... item fields
}

// Supporting Domain - Audit
Table audit_logs {
  id bigserial [pk, increment]
  table_name varchar(100) [not null]
  record_id bigint [not null]
  action varchar(20) [not null, note: 'INSERT, UPDATE, DELETE']
  old_values jsonb
  new_values jsonb
  user_id bigint [ref: > users.id]
  timestamp timestamptz [not null, default: `CURRENT_TIMESTAMP`]
}
```

### Enum Definitions
```dbml
Enum order_status {
  PENDING [note: 'Order created but not paid']
  PAID [note: 'Payment confirmed']
  PROCESSING [note: 'Order being prepared']
  SHIPPED [note: 'Order dispatched']
  DELIVERED [note: 'Order received by customer']
  CANCELLED [note: 'Order cancelled']
  REFUNDED [note: 'Order refunded']
}

Enum user_role {
  ADMIN [note: 'System administrator']
  MANAGER [note: 'Department manager']
  USER [note: 'Regular user']
  GUEST [note: 'Limited access user']
}

Table orders {
  id bigserial [pk, increment]
  status order_status [not null, default: 'PENDING']
  // ... other fields
}
```

## Documentation Standards

### Table Documentation
```dbml
Table comprehensive_example {
  id bigserial [pk, increment, note: 'Primary key - auto-incrementing']
  name varchar(255) [not null, note: 'Display name - required field']
  
  Note: '''
  **Purpose**: Brief description of table purpose
  
  **Business Rules**:
  - Rule 1: Specific business constraint
  - Rule 2: Another important rule
  - Rule 3: Data validation requirement
  
  **Relationships**:
  - Parent tables: List of parent relationships
  - Child tables: List of child relationships
  - Related tables: Other related entities
  
  **Indexes**:
  - Performance considerations
  - Query patterns supported
  
  **Data Volume**: Expected size and growth
  **Retention**: Data retention policy
  **Security**: Access control requirements
  '''
}
```

### Project Documentation
```dbml
Project ecommerce_platform {
  database_type: 'PostgreSQL'
  Note: '''
  **E-commerce Platform Database Schema**
  
  **Version**: 1.0.0
  **Last Updated**: 2025-01-21
  **Author**: Development Team
  
  **Overview**:
  This schema supports a multi-tenant e-commerce platform with:
  - User management and authentication
  - Product catalog and inventory
  - Order processing and fulfillment
  - Payment processing integration
  - Audit logging and compliance
  
  **Key Features**:
  - Multi-tenant architecture
  - Soft delete patterns
  - Audit trail for all changes
  - Performance optimized indexes
  - Data retention policies
  
  **Dependencies**:
  - PostgreSQL 13+
  - UUID extension
  - JSONB support
  
  **Deployment**:
  - Use Liquibase for migrations
  - Follow semantic versioning
  - Test in staging before production
  '''
}
```

## Advanced DBML Features

### Table Groups
```dbml
TableGroup user_management {
  users
  user_profiles
  user_sessions
  user_roles
}

TableGroup ecommerce {
  products
  categories
  orders
  order_items
  shopping_carts
}

TableGroup system {
  audit_logs
  system_config
  error_logs
}
```

### Complex Relationships
```dbml
Table orders {
  id bigserial [pk]
  user_id bigint [ref: > users.id]
  shipping_address_id bigint [ref: > addresses.id]
  billing_address_id bigint [ref: > addresses.id]
}

// Multiple references to same table
Ref: orders.shipping_address_id > addresses.id [delete: restrict]
Ref: orders.billing_address_id > addresses.id [delete: restrict]

// Conditional relationships
Table payments {
  id bigserial [pk]
  order_id bigint [ref: > orders.id]
  payment_method_id bigint [ref: > payment_methods.id]
  
  Note: '''
  Payment can be associated with either an order or a subscription,
  but not both (enforced by application logic).
  '''
}
```

## Best Practices

### Naming Conventions

#### Table Names
- Use **singular nouns**: `user`, `order`, `product`
- Use **snake_case**: `user_profile`, `order_item`
- Be **descriptive**: `audit_log` not `log`
- Avoid **abbreviations**: `category` not `cat`

#### Column Names
- Use **snake_case**: `first_name`, `created_at`
- Be **descriptive**: `email_address` not `email`
- Use **consistent suffixes**:
  - `_id` for foreign keys: `user_id`, `category_id`
  - `_at` for timestamps: `created_at`, `updated_at`
  - `_date` for dates: `birth_date`, `due_date`
  - `is_` for booleans: `is_active`, `is_deleted`

#### Relationship Names
- Use **descriptive references**: `user_id` references `users.id`
- Be **consistent**: Always use `id` for primary keys
- Use **meaningful junction table names**: `user_roles` not `user_role_mapping`

### Performance Considerations
```dbml
Table performance_optimized {
  id bigserial [pk, increment]
  user_id bigint [not null, ref: > users.id]
  status varchar(20) [not null, default: 'ACTIVE']
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  
  Indexes {
    user_id [name: 'idx_perf_user_id']
    status [name: 'idx_perf_status', where: 'status != \'DELETED\'']
    created_at [name: 'idx_perf_created_at']
    (user_id, status) [name: 'idx_perf_user_status']
    (created_at, status) [name: 'idx_perf_created_status', where: 'status = \'ACTIVE\'']
  }
  
  Note: '''
  **Performance Optimizations**:
  - Partial indexes on status exclude soft-deleted records
  - Composite indexes support common query patterns
  - Foreign key indexes prevent lock escalation
  - Date indexes support time-based queries
  '''
}
```

### Security Patterns
```dbml
Table secure_users {
  id bigserial [pk, increment]
  username varchar(50) [not null, unique]
  email varchar(255) [not null, unique]
  password_hash varchar(255) [not null, note: 'Bcrypt hash - never store plain text']
  salt varchar(32) [not null, note: 'Random salt for password hashing']
  failed_login_attempts integer [not null, default: 0]
  locked_until timestamp [note: 'Account lock expiration']
  last_password_change timestamp [not null, default: `CURRENT_TIMESTAMP`]
  requires_password_change boolean [not null, default: false]
  
  Note: '''
  **Security Features**:
  - Passwords are hashed with bcrypt and salt
  - Account locking after failed attempts
  - Password change tracking
  - Force password change capability
  '''
}

Table sensitive_data {
  id bigserial [pk, increment]
  user_id bigint [not null, ref: > secure_users.id]
  encrypted_ssn varchar(255) [note: 'AES encrypted - PII data']
  encrypted_phone varchar(255) [note: 'AES encrypted - PII data']
  data_classification varchar(20) [not null, default: 'CONFIDENTIAL']
  
  Note: '''
  **Data Classification**: CONFIDENTIAL
  **Encryption**: AES-256 encryption for PII fields
  **Access**: Restricted to authorized personnel only
  **Retention**: 7 years per compliance requirements
  '''
}
```

## Integration with Development Workflow

### Version Control
```
# .gitignore additions
*.dbml.bak
generated-sql/
temp-diagrams/

# Include in version control
schemas/
master-schema.dbml
README.md
```

### CI/CD Integration
```yaml
# Example GitHub Actions workflow
name: DBML Validation
on: [push, pull_request]

jobs:
  validate-dbml:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install DBML CLI
        run: npm install -g @dbml/cli
      - name: Validate DBML syntax
        run: dbml2sql master-schema.dbml --postgres
      - name: Generate documentation
        run: dbml2sql master-schema.dbml --postgres > generated-schema.sql
      - name: Upload artifacts
        uses: actions/upload-artifact@v2
        with:
          name: generated-sql
          path: generated-schema.sql
```

### Documentation Generation
```bash
# Generate SQL from DBML
dbml2sql master-schema.dbml --postgres > schema.sql
dbml2sql master-schema.dbml --mysql > schema-mysql.sql

# Generate visual diagrams
# Upload to dbdiagram.io or use local tools
```

### Code Generation Tool Limitations

**Important Note about dbml2sql and Similar Tools:**

While `dbml2sql` is excellent for generating initial DDL scripts, keep in mind:

**DBML is the Source of Truth:**
- The DBML specification is your design document
- Generated SQL is a starting point, not the final implementation
- Always review and customize generated SQL for your specific needs

**Manual Adjustments Often Required:**
- **Check Constraints**: DBML notes need to be converted to actual CHECK constraints
- **Triggers**: Database triggers must be added manually to generated SQL
- **Stored Procedures**: Functions and procedures are not generated by dbml2sql
- **Database-Specific Features**: Advanced features may need manual implementation
- **Performance Optimizations**: Fine-tuning indexes and constraints for your workload

**Best Practice Workflow:**
1. Design schema in DBML (source of truth)
2. Generate initial SQL with dbml2sql
3. Review and enhance generated SQL
4. Add database-specific optimizations
5. Test thoroughly before deployment
6. Keep DBML updated with any manual changes

**Tool Expectations:**
- **Great for**: Initial schema generation, documentation, visualization
- **Limited for**: Complex constraints, triggers, stored procedures, advanced features
- **Remember**: DBML is a design language first, code generation is a bonus feature

## Common Patterns and Templates

### Audit Pattern
```dbml
Table auditable_entity {
  id bigserial [pk, increment]
  name varchar(255) [not null]
  
  // Audit fields - include in all business tables
  created_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  created_by bigint [not null, ref: > users.id]
  updated_at timestamp [not null, default: `CURRENT_TIMESTAMP`]
  updated_by bigint [not null, ref: > users.id]
  version integer [not null, default: 1, note: 'Optimistic locking version']
  
  Note: '''
  Standard audit pattern for all business entities.
  Tracks creation, modification, and version for optimistic locking.
  '''
}
```

### Soft Delete Pattern
```dbml
Table soft_deletable {
  id bigserial [pk, increment]
  name varchar(255) [not null]
  
  // Soft delete fields
  is_deleted boolean [not null, default: false]
  deleted_at timestamp [note: 'Timestamp when record was soft deleted']
  deleted_by bigint [ref: > users.id, note: 'User who performed soft delete']
  
  Indexes {
    is_deleted [name: 'idx_soft_del_active', where: 'is_deleted = false']
  }
  
  Note: '''
  Soft delete pattern preserves data while marking as deleted.
  Use partial index to optimize queries for active records.
  '''
}
```

### Multi-Tenant Pattern
```dbml
Table tenant_aware {
  id bigserial [pk, increment]
  tenant_id bigint [not null, ref: > tenants.id, note: 'Multi-tenant isolation']
  name varchar(255) [not null]
  
  Indexes {
    tenant_id [name: 'idx_tenant_aware_tenant']
    (tenant_id, id) [name: 'idx_tenant_aware_composite']
  }
  
  Note: '''
  Multi-tenant pattern with tenant isolation.
  All queries must include tenant_id for data isolation.
  '''
}
```

## Summary

These DBML standards ensure:
- **Consistency** across all database designs
- **Maintainability** through clear documentation and naming
- **Performance** through proper indexing strategies
- **Security** through data classification and encryption guidance
- **Compliance** through audit trails and retention policies
- **Collaboration** through version control and CI/CD integration

Always remember: **Good database design starts with clear documentation and consistent standards.**