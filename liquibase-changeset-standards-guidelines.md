# Liquibase XML+SQL Changesets Guidelines for AI Agents

This document provides comprehensive guidelines for creating Liquibase changesets using XML format with SQL content. These rules ensure consistency, maintainability, and reliability across database migrations.

## Core Principles

1. **Immutability**: Never modify existing changesets once they've been deployed
2. **Idempotency**: Changesets should be safe to run multiple times
3. **Atomicity**: Each changeset should represent a single logical change
4. **Reversibility**: Always provide rollback instructions where possible
5. **Documentation**: Every changeset must be well-documented
6. **Small and Focused**: Break changes into small units for easier troubleshooting and rollbacks
7. **Environment Awareness**: Use contexts and labels for conditional execution
8. **Security First**: Never store sensitive data in changelogs

## File Structure and Naming

### Directory Structure Options

**Choose the structure that best fits your project:**

### Directory Structure Options

**Option 1: Single Database Project (Recommended for most projects)**
```
db/
├── changelog/
│   ├── db.changelog-master.xml
│   └── releases/
│       ├── v1.0.0/
│       │   ├── ddl/
│       │   │   ├── create-users-table-20250223-v1.0.xml
│       │   │   ├── create-roles-table-20250223-v1.0.xml
│       │   │   ├── add-user-indexes-20250224-v1.0.xml
│       │   │   ├── add-constraints-20250224-v1.0.xml
│       │   │   ├── create-audit-function-20250224-v1.0.xml
│       │   │   └── create-user-summary-view-20250225-v1.0.xml
│       │   └── dml/
│       │       ├── insert-default-roles-20250225-v1.0.xml
│       │       ├── insert-system-config-20250225-v1.0.xml
│       │       └── insert-test-data-20250225-v1.0.xml
│       └── v1.1.0/
│           ├── ddl/
│           │   ├── add-user-phone-column-20250301-v1.1.xml
│           │   ├── create-audit-table-20250301-v1.1.xml
│           │   ├── add-performance-indexes-20250302-v1.1.xml
│           │   └── create-validation-functions-20250302-v1.1.xml
│           └── dml/
│               ├── migrate-user-status-20250303-v1.1.xml
│               └── update-role-permissions-20250303-v1.1.xml
```

**Option 2: Multi-Database Project**
```
db/
├── postgres/
│   ├── changelog/
│   │   ├── db.changelog-master.xml
│   │   └── releases/
│   │       ├── v1.0.0/
│   │       │   ├── ddl/
│   │       │   │   ├── create-users-table-20250223-v1.0.xml
│   │       │   │   └── create-postgres-specific-indexes-20250224-v1.0.xml
│   │       │   └── dml/
│   │       │       └── insert-default-roles-20250225-v1.0.xml
│   │       └── v1.1.0/
│   │           ├── ddl/
│   │           │   └── add-jsonb-column-20250301-v1.1.xml
│   │           └── dml/
│   │               └── migrate-json-data-20250302-v1.1.xml
├── mysql/
│   ├── changelog/
│   │   ├── db.changelog-master.xml
│   │   └── releases/
│   │       ├── v1.0.0/
│   │       │   ├── ddl/
│   │       │   │   ├── create-users-table-20250223-v1.0.xml
│   │       │   │   └── create-mysql-specific-indexes-20250224-v1.0.xml
│   │       │   └── dml/
│   │       │       └── insert-default-roles-20250225-v1.0.xml
│   │       └── v1.1.0/
│   │           ├── ddl/
│   │           │   └── add-json-column-20250301-v1.1.xml
│   │           └── dml/
│   │               └── migrate-json-data-20250302-v1.1.xml
└── oracle/
    ├── changelog/
    │   ├── db.changelog-master.xml
    │   └── releases/
    │       └── v1.0.0/
    │           ├── ddl/
    │           │   ├── create-users-table-20250223-v1.0.xml
    │           │   └── create-oracle-specific-procedures-20250224-v1.0.xml
    │           └── dml/
    │               └── insert-default-roles-20250225-v1.0.xml
```



**When to Use Option 1:**
- Single database type (most common scenario)
- Simpler project structure and maintenance
- All schema objects belong to the same release cycle
- Easier to understand and navigate

**When to Use Option 2:**
- Multiple database types with completely different schemas
- Database-specific features and optimizations needed
- Different deployment schedules for different databases
- Separate teams managing different database types

## Choosing the Right Structure

**Start with Option 1** unless you have specific requirements for multiple databases. You can always refactor to Option 2 later if needed.

**Decision Tree:**
- **Single Database?** → Use Option 1
- **Multiple Different Databases?** → Use Option 2
- **Shared Schema with Database-Specific Features?** → Still use Option 2, but include shared changesets in each database folder

**Migration Path:**
1. **Start Simple**: Use Option 1 for new projects
2. **Scale When Needed**: Move to Option 2 only when you actually need multiple database types

## Handling Multiple Schemas in Single Database

### **The Challenge**
Many applications need multiple schemas within a single database for:
- **Logical Separation**: User management, job processing, reporting schemas
- **Environment Sharing**: Multiple apps sharing one database in dev/test environments
- **Multi-Tenancy**: Tenant-specific schema prefixes

### **❌ Never Hardcode Schema Names**
```xml
<!-- WRONG: Hardcoded schema names -->
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <sql>
        CREATE TABLE user_management.users (  <!-- WRONG: Hardcoded schema -->
            id BIGSERIAL PRIMARY KEY,
            username VARCHAR(50) NOT NULL
        );
    </sql>
</changeSet>
```

### **✅ Recommended Approach: Parameter Substitution**

**Define schema parameters in liquibase.properties:**
```properties
# Development environment (shared database)
appSchema=dev_myapp_users
jobSchema=dev_myapp_jobs
reportingSchema=dev_myapp_reporting

# Production environment (dedicated database)
appSchema=users
jobSchema=jobs
reportingSchema=reporting
```

**Use parameters in changesets:**
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <sql>
        CREATE TABLE ${appSchema}.users (
            id BIGSERIAL PRIMARY KEY,
            username VARCHAR(50) NOT NULL,
            email VARCHAR(255) NOT NULL
        );
    </sql>
    <rollback>
        DROP TABLE ${appSchema}.users;
    </rollback>
</changeSet>

<changeSet id="create-jobs-table-20250223-v1.0" author="jane.doe">
    <sql>
        CREATE TABLE ${jobSchema}.jobs (
            id BIGSERIAL PRIMARY KEY,
            job_name VARCHAR(100) NOT NULL,
            status VARCHAR(20) DEFAULT 'PENDING',
            user_id BIGINT REFERENCES ${appSchema}.users(id)
        );
    </sql>
    <rollback>
        DROP TABLE ${jobSchema}.jobs;
    </rollback>
</changeSet>
```

### **Cross-Schema References**
```xml
<changeSet id="create-user-jobs-view-20250224-v1.0" author="analyst">
    <comment>Create view combining user and job data across schemas</comment>
    <preConditions onFail="HALT">
        <sqlCheck expectedResult="1">
            SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name = '${appSchema}'
        </sqlCheck>
        <sqlCheck expectedResult="1">
            SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name = '${jobSchema}'
        </sqlCheck>
    </preConditions>
    <sql>
        CREATE VIEW ${reportingSchema}.user_jobs AS
        SELECT 
            u.username,
            u.email,
            j.job_name,
            j.status,
            j.created_at
        FROM ${appSchema}.users u
        JOIN ${jobSchema}.jobs j ON u.id = j.user_id;
    </sql>
    <rollback>
        DROP VIEW ${reportingSchema}.user_jobs;
    </rollback>
</changeSet>
```

### **Directory Structure for Multi-Schema**

```
db/
├── liquibase-dev.properties
├── liquibase-sit.properties
├── liquibase-prod.properties
├── changelog/
│   ├── db.changelog-master.xml
│   └── releases/
│       └── v1.0.0/
│           ├── app-schema/
│           │   ├── ddl/
│           │   │   └── create-users-table-20250223-v1.0.xml    # Uses ${appSchema}
│           │   └── dml/
│           │       └── insert-default-users-20250225-v1.0.xml
│           ├── job-schema/
│           │   ├── ddl/
│           │   │   └── create-jobs-table-20250223-v1.0.xml     # Uses ${jobSchema}
│           │   └── dml/
│           │       └── insert-job-types-20250225-v1.0.xml
│           └── reporting-schema/
│               └── ddl/
│                   └── create-user-jobs-view-20250224-v1.0.xml # Uses multiple schemas
```

### **Environment-Specific Properties Files**

**liquibase-dev.properties (Shared Database):**
```properties
# Development - multiple apps share one database with prefixed schemas
appSchema=dev_myapp_users
jobSchema=dev_myapp_jobs
reportingSchema=dev_myapp_reporting
```

**liquibase-prod.properties (Dedicated Database):**
```properties
# Production - dedicated database with clean schema names
appSchema=users
jobSchema=jobs
reportingSchema=reporting
```

### **Deployment Commands**
```bash
# Development environment
liquibase --defaultsFile=liquibase-dev.properties update

# SIT environment  
liquibase --defaultsFile=liquibase-sit.properties update

# Production environment
liquibase --defaultsFile=liquibase-prod.properties update
```

### **Multi-Tenant Example**

**liquibase-tenant-a.properties:**
```properties
appSchema=tenant_a_users
jobSchema=tenant_a_jobs
reportingSchema=tenant_a_reporting
```

**liquibase-tenant-b.properties:**
```properties
appSchema=tenant_b_users
jobSchema=tenant_b_jobs
reportingSchema=tenant_b_reporting
```

### **Why Parameter-Based Approach is Best**

1. **Maximum Flexibility**: Works for shared databases, dedicated databases, and multi-tenant scenarios
2. **Environment Portability**: Same changesets work across all environments with different schema names
3. **Clear Intent**: Parameter names make the purpose and scope obvious
4. **Easy Maintenance**: Change schema names in one place (properties file) without touching changesets
5. **Real-World Practical**: Handles common scenarios like shared lower environments and dedicated production
6. **Cross-Schema Support**: Seamlessly handles references between different schemas
7. **Multi-Tenancy Ready**: Easy to adapt for tenant-specific schema prefixes

### **Best Practices for Schema Parameters**

1. **Use Descriptive Parameter Names**: `appSchema`, `jobSchema`, `reportingSchema` instead of `schema1`, `schema2`
2. **Add Preconditions**: Validate that schemas exist before creating objects
3. **Document Parameters**: Clearly document all required parameters in README
4. **Environment-Specific Files**: Use separate properties files for each environment
5. **Consistent Naming**: Use the same parameter names across all changesets
6. **Validate Early**: Add schema existence checks in early changesets

**The key principle: Use parameter substitution to keep changesets completely environment-agnostic while maintaining full flexibility for different deployment scenarios.**

### File Naming Convention
- **Format**: `{verb}-{object}-{YYYYMMDD}-{version}.xml`
- **Verb**: Action being performed (create, add, insert, migrate, drop, etc.)
- **Object**: Target object (users-table, email-index, default-roles, etc.)
- **Date**: ISO format date (YYYYMMDD) when changeset was created
- **Version**: Release version (v1.0, v1.1, etc.)
- **Examples**:
  - `create-users-table-20250223-v1.0.xml`
  - `add-email-index-20250224-v1.0.xml`
  - `insert-default-roles-20250225-v1.0.xml`
  - `alter-users-add-status-20250301-v1.1.xml`

### DDL vs DML Separation

**Why Separate DDL and DML?**

1. **Deployment Control**: DDL changes (schema) can be deployed separately from DML changes (data)
2. **Risk Management**: Schema changes are typically higher risk and may require different approval processes
3. **Rollback Strategy**: DDL and DML have different rollback complexities and timing requirements
4. **Performance Impact**: DDL operations often require table locks, while DML can be batched
5. **Environment Differences**: Some environments may need schema changes but not data changes

**DDL (Data Definition Language) - Schema Changes:**
- CREATE/DROP/ALTER TABLE
- CREATE/DROP INDEX
- ADD/DROP CONSTRAINTS
- CREATE/DROP FUNCTIONS/PROCEDURES
- CREATE/DROP VIEWS

**DML (Data Manipulation Language) - Data Changes:**
- INSERT/UPDATE/DELETE statements
- Data migrations and transformations
- Reference data population
- Configuration updates

**Execution Order Within Each Release:**
1. **DDL first**: Tables, indexes, constraints, functions, procedures, views
2. **DML second**: Data inserts, updates, migrations

**Why Functions/Views in DDL:**
- Functions and views are schema objects (DDL), not data operations
- They should be versioned with the release that introduces them
- Dependencies are clearer when grouped by release
- Easier to rollback entire releases including all schema objects

## Changeset Format Options

### Choosing the Right Format

**XML + SQL (Recommended)**: Best of both worlds
- Use XML wrapper with SQL content for complex operations
- Provides rich metadata and structured rollback
- Better tooling support and validation

**Pure XML**: For database portability
- Use when you need database-agnostic changesets
- Liquibase handles SQL generation for different databases
- More verbose but highly portable

**Pure SQL**: For readability (when database won't change)
- Use only when you're certain the database type won't change
- More readable for complex queries
- Limited metadata capabilities

### XML + SQL Format (Recommended)

**Example: Table Creation**
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <comment>Create users table with basic user information</comment>
    <sql>
        CREATE TABLE users (
            id BIGSERIAL PRIMARY KEY,
            name VARCHAR(255) NOT NULL
        );
    </sql>
    <rollback>
        DROP TABLE IF EXISTS users;
    </rollback>
</changeSet>
```

**Example: Pure XML (Database Agnostic)**
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <comment>Create users table with basic user information</comment>
    <createTable tableName="users">
        <column name="id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true" nullable="false"/>
        </column>
        <column name="name" type="VARCHAR(255)">
            <constraints nullable="false"/>
        </column>
    </createTable>
    <rollback>
        <dropTable tableName="users"/>
    </rollback>
</changeSet>
```

## Changeset Structure

### Standard Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="changeset-id" author="author" labels="version-1.0.0,feature-name" context="dev,test,prod">
        <comment>Brief description of what this changeset does</comment>
        <sql>
            -- Your SQL statements here
        </sql>
        <rollback>
            -- SQL statements to undo the changes
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Required Attributes
- **id**: Unique identifier matching filename without extension
- **author**: Use consistent identifier (email prefix or username)
- **comment**: Clear, concise description of the change (in comment tag)
- **labels**: Version and feature tags for organization
- **context**: Environments where this should run

## Changeset ID Guidelines

### ID Format Rules
1. **Use filename without extension**: Match the changeset ID to filename
2. **Must be unique across entire project**
3. **Never reuse IDs, even for deleted changesets**
4. **Use descriptive names, not just numbers**
5. **Choose consistent format** across your project

### ID Format Options

**Option 1: Verb-Object-Date-Version (Recommended for enterprise/release-driven projects)**
```xml
<!-- ✅ Excellent for traceability and release mapping -->
<changeSet id="create-jobrunr-jobs-table-20250223-v1.0" author="john.doe">
<changeSet id="add-user-email-index-20250224-v1.0" author="jane.smith">
<changeSet id="insert-default-roles-20250225-v1.0" author="system">
<changeSet id="migrate-user-data-20250301-v1.1" author="jane.smith">
<changeSet id="drop-legacy-column-20250315-v1.2" author="john.doe">
```

**Benefits of Verb-Object-Date-Version Format:**
- **Release Traceability**: Clear connection to specific release cycles
- **Chronological Context**: Date shows when changeset was created
- **Version Alignment**: Direct correlation with release versions
- **Audit Trail**: Excellent for compliance and change tracking
- **Team Coordination**: Helps teams understand timeline of changes

**Option 2: Sequential + Descriptive (Good for smaller teams/continuous deployment)**
```xml
<!-- ✅ Simple and clean -->
<changeSet id="001-create-users-table" author="john.doe">
<changeSet id="002-add-user-indexes" author="jane.smith">
<changeSet id="003-insert-default-roles" author="system">
```

**Option 3: Feature-Based (Good for feature branch workflows)**
```xml
<!-- ✅ Good for feature-driven development -->
<changeSet id="user-mgmt-001-create-users-table" author="john.doe">
<changeSet id="user-mgmt-002-add-user-indexes" author="jane.smith">
<changeSet id="auth-001-create-roles-table" author="system">
```

### Date Format Guidelines (if using dates)
- **Use ISO format**: `YYYYMMDD` (20250223) for consistent sorting
- **Be consistent**: Don't mix date formats within a project
- **Consider timezone**: Use UTC dates for global teams

### ID Best Practices
```xml
<!-- ❌ Avoid these patterns -->
<changeSet id="changeset1" author="john.doe">                    <!-- Too generic -->
<changeSet id="23022025-create-table" author="john.doe">         <!-- Ambiguous date format -->
<changeSet id="001" author="john.doe">                           <!-- Not descriptive -->
<changeSet id="create-users-table-v1.0-23/02/2025" author="john.doe"> <!-- Invalid characters -->
```

### Examples
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
<changeSet id="add-user-indexes-20250224-v1.0" author="jane.smith">
<changeSet id="insert-default-data-20250225-v1.0" author="system">
```

## SQL Best Practices

### Table Creation
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="create-users-table-20250223-v1.0" author="john.doe" labels="version-1.0.0,table-creation" context="dev,test,prod">
        <comment>Create users table with basic user information</comment>
        <sql>
            CREATE TABLE users (
                id BIGSERIAL PRIMARY KEY,
                username VARCHAR(50) NOT NULL UNIQUE,
                email VARCHAR(255) NOT NULL UNIQUE,
                password_hash VARCHAR(255) NOT NULL,
                first_name VARCHAR(100),
                last_name VARCHAR(100),
                is_active BOOLEAN DEFAULT true,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
            );

            -- Add comments to table and columns
            COMMENT ON TABLE users IS 'Application users with authentication information';
            COMMENT ON COLUMN users.username IS 'Unique username for login';
            COMMENT ON COLUMN users.email IS 'User email address for notifications';
            COMMENT ON COLUMN users.password_hash IS 'Hashed password using bcrypt';
        </sql>
        <rollback>
            DROP TABLE users;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Adding Columns
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="add-user-phone-column-20250301-v1.1" author="john.doe" labels="version-1.1.0,column-addition" context="dev,test,prod">
        <comment>Add phone number column to users table</comment>
        <sql>
            ALTER TABLE users 
            ADD COLUMN phone_number VARCHAR(20);

            COMMENT ON COLUMN users.phone_number IS 'User phone number for 2FA';
        </sql>
        <rollback>
            ALTER TABLE users DROP COLUMN phone_number;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Creating Indexes
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="add-user-indexes-20250224-v1.0" author="john.doe" labels="version-1.0.0,indexes" context="dev,test,prod">
        <comment>Add performance indexes on users table</comment>
        <sql>
            CREATE INDEX idx_users_email ON users(email);
            CREATE INDEX idx_users_username ON users(username);
            CREATE INDEX idx_users_active ON users(is_active) WHERE is_active = true;
            CREATE INDEX idx_users_created_at ON users(created_at);
        </sql>
        <rollback>
            DROP INDEX IF EXISTS idx_users_email;
            DROP INDEX IF EXISTS idx_users_username;
            DROP INDEX IF EXISTS idx_users_active;
            DROP INDEX IF EXISTS idx_users_created_at;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Data Insertion
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="insert-default-roles-20250225-v1.0" author="system" labels="version-1.0.0,reference-data" context="dev,test,prod">
        <comment>Insert default system roles</comment>
        <sql>
            INSERT INTO roles (name, description, is_system) VALUES
            ('ADMIN', 'System administrator with full access', true),
            ('USER', 'Regular user with limited access', true),
            ('MODERATOR', 'Content moderator with review permissions', true)
            ON CONFLICT (name) DO NOTHING;
        </sql>
        <rollback>
            DELETE FROM roles WHERE is_system = true;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Rollback Guidelines

### Rollback Requirements
1. **Always provide rollback** unless impossible (e.g., data deletion)
2. **Test rollback statements** before deployment
3. **Use conditional drops** to avoid errors
4. **Document irreversible changes**

### Rollback Examples
```xml
<!-- Simple rollback -->
<rollback>
    DROP TABLE users;
</rollback>

<!-- Conditional rollback -->
<rollback>
    DROP INDEX IF EXISTS idx_users_email;
</rollback>

<!-- Multiple rollback statements -->
<rollback>
    DELETE FROM roles WHERE name = 'ADMIN';
    ALTER TABLE users DROP COLUMN phone_number;
</rollback>

<!-- Complex rollback with data preservation -->
<rollback>
    CREATE TEMPORARY TABLE users_backup AS SELECT * FROM users;
    DROP TABLE users;
    ALTER TABLE users_backup RENAME TO users;
</rollback>
```

## Context and Labels

### Context Usage for Environment-Specific Execution

**Contexts allow you to control which changesets run in which environments.**

```xml
context="dev,test,prod"          <!-- All environments -->
context="!prod"                  <!-- All except production -->
context="dev"                    <!-- Development only -->
context="test,prod"              <!-- Test and production only -->
```

## Valid Use Cases for Contexts

### 1. **Test Data Management**
```xml
<!-- Insert test data only in development -->
<changeSet id="insert-test-users-20250225-v1.0" author="system" context="dev">
    <comment>Insert test users for development environment</comment>
    <sql>
        INSERT INTO users (username, email, password_hash) VALUES 
        ('testuser1', 'test1@example.com', '$2a$10$test.hash.1'),
        ('testuser2', 'test2@example.com', '$2a$10$test.hash.2'),
        ('admin', 'admin@test.com', '$2a$10$admin.test.hash');
    </sql>
    <rollback>
        DELETE FROM users WHERE username IN ('testuser1', 'testuser2', 'admin');
    </rollback>
</changeSet>

<!-- Insert production seed data -->
<changeSet id="insert-prod-admin-20250225-v1.0" author="system" context="prod">
    <comment>Insert production admin user</comment>
    <sql>
        INSERT INTO users (username, email, password_hash, role) 
        VALUES ('admin', '${prod.admin.email}', '${prod.admin.password.hash}', 'ADMIN');
    </sql>
    <rollback>
        DELETE FROM users WHERE username = 'admin';
    </rollback>
</changeSet>
```

### 2. **Environment-Specific Configuration**
```xml
<!-- Development database settings -->
<changeSet id="set-dev-config-20250225-v1.0" author="system" context="dev">
    <comment>Development environment configuration</comment>
    <sql>
        INSERT INTO app_config (key, value) VALUES 
        ('log_level', 'DEBUG'),
        ('cache_ttl', '60'),
        ('email_enabled', 'false'),
        ('debug_mode', 'true');
    </sql>
    <rollback>
        DELETE FROM app_config WHERE key IN ('log_level', 'cache_ttl', 'email_enabled', 'debug_mode');
    </rollback>
</changeSet>

<!-- Production database settings -->
<changeSet id="set-prod-config-20250225-v1.0" author="system" context="prod">
    <comment>Production environment configuration</comment>
    <sql>
        INSERT INTO app_config (key, value) VALUES 
        ('log_level', 'WARN'),
        ('cache_ttl', '3600'),
        ('email_enabled', 'true'),
        ('debug_mode', 'false');
    </sql>
    <rollback>
        DELETE FROM app_config WHERE key IN ('log_level', 'cache_ttl', 'email_enabled', 'debug_mode');
    </rollback>
</changeSet>
```

### 3. **Performance Optimizations**
```xml
<!-- Create indexes concurrently only in production -->
<changeSet id="create-prod-indexes-20250224-v1.0" author="dba" context="prod">
    <comment>Create indexes concurrently to avoid blocking production traffic</comment>
    <sql>
        CREATE INDEX CONCURRENTLY idx_users_last_login ON users(last_login_at);
        CREATE INDEX CONCURRENTLY idx_orders_created_date ON orders(created_at);
    </sql>
    <rollback>
        DROP INDEX IF EXISTS idx_users_last_login;
        DROP INDEX IF EXISTS idx_orders_created_date;
    </rollback>
</changeSet>

<!-- Create indexes normally in dev/test -->
<changeSet id="create-dev-indexes-20250224-v1.0" author="dba" context="dev,test">
    <comment>Create indexes normally in development environments</comment>
    <sql>
        CREATE INDEX idx_users_last_login ON users(last_login_at);
        CREATE INDEX idx_orders_created_date ON orders(created_at);
    </sql>
    <rollback>
        DROP INDEX IF EXISTS idx_users_last_login;
        DROP INDEX IF EXISTS idx_orders_created_date;
    </rollback>
</changeSet>
```

### 4. **Security and Compliance**
```xml
<!-- Enable audit logging only in production -->
<changeSet id="enable-audit-logging-20250226-v1.0" author="security" context="prod">
    <comment>Enable comprehensive audit logging for production compliance</comment>
    <sql>
        CREATE TABLE audit_log (
            id BIGSERIAL PRIMARY KEY,
            table_name VARCHAR(100) NOT NULL,
            operation VARCHAR(10) NOT NULL,
            old_values JSONB,
            new_values JSONB,
            user_id BIGINT,
            timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        
        -- Create audit triggers for sensitive tables
        CREATE OR REPLACE FUNCTION audit_trigger_function()
        RETURNS TRIGGER AS $$
        BEGIN
            INSERT INTO audit_log (table_name, operation, old_values, new_values, user_id)
            VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), row_to_json(NEW), current_setting('app.user_id')::BIGINT);
            RETURN COALESCE(NEW, OLD);
        END;
        $$ LANGUAGE plpgsql;
        
        CREATE TRIGGER users_audit_trigger
        AFTER INSERT OR UPDATE OR DELETE ON users
        FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();
    </sql>
    <rollback>
        DROP TRIGGER IF EXISTS users_audit_trigger ON users;
        DROP FUNCTION IF EXISTS audit_trigger_function();
        DROP TABLE IF EXISTS audit_log;
    </rollback>
</changeSet>
```

### 5. **Development Tools and Debugging**
```xml
<!-- Create development helper views -->
<changeSet id="create-dev-helper-views-20250227-v1.0" author="dev-team" context="dev">
    <comment>Create helper views for development debugging</comment>
    <sql>
        -- View to see user activity with readable timestamps
        CREATE VIEW dev_user_activity AS
        SELECT 
            u.username,
            u.email,
            u.last_login_at,
            CASE 
                WHEN u.last_login_at > NOW() - INTERVAL '1 day' THEN 'Recent'
                WHEN u.last_login_at > NOW() - INTERVAL '1 week' THEN 'This Week'
                ELSE 'Inactive'
            END as activity_status
        FROM users u;
        
        -- View to see system configuration
        CREATE VIEW dev_config_summary AS
        SELECT key, value, 'Current Setting' as status
        FROM app_config
        ORDER BY key;
    </sql>
    <rollback>
        DROP VIEW IF EXISTS dev_user_activity;
        DROP VIEW IF EXISTS dev_config_summary;
    </rollback>
</changeSet>
```

### 6. **Feature Flags and Gradual Rollouts**
```xml
<!-- Enable feature only in test environment first -->
<changeSet id="enable-new-feature-test-20250228-v1.0" author="product" context="test">
    <comment>Enable new payment feature in test environment</comment>
    <sql>
        INSERT INTO feature_flags (feature_name, enabled, environment) 
        VALUES ('new_payment_flow', true, 'test');
    </sql>
    <rollback>
        DELETE FROM feature_flags WHERE feature_name = 'new_payment_flow' AND environment = 'test';
    </rollback>
</changeSet>

<!-- Enable feature in production after testing -->
<changeSet id="enable-new-feature-prod-20250301-v1.0" author="product" context="prod">
    <comment>Enable new payment feature in production after successful testing</comment>
    <sql>
        INSERT INTO feature_flags (feature_name, enabled, environment) 
        VALUES ('new_payment_flow', true, 'prod');
    </sql>
    <rollback>
        DELETE FROM feature_flags WHERE feature_name = 'new_payment_flow' AND environment = 'prod';
    </rollback>
</changeSet>
```

## ❌ Invalid Use Cases (Anti-Patterns)

### Don't Use Contexts For:
```xml
<!-- ❌ WRONG: Using context for business logic -->
<changeSet id="create-premium-features-20250225-v1.0" author="dev" context="premium">
    <!-- This should be handled by application logic, not database deployment -->
</changeSet>

<!-- ❌ WRONG: Using context for temporary fixes -->
<changeSet id="temp-fix-20250225-v1.0" author="dev" context="hotfix">
    <!-- Temporary fixes should be proper changesets, not context-dependent -->
</changeSet>

<!-- ❌ WRONG: Using context for user permissions -->
<changeSet id="admin-only-table-20250225-v1.0" author="dev" context="admin">
    <!-- User permissions should be handled by application security, not deployment contexts -->
</changeSet>
```

## Best Practices for Context Usage

1. **Environment-Specific Only**: Use contexts only for environment differences, not business logic
2. **Clear Naming**: Use standard environment names (dev, test, staging, prod)
3. **Document Dependencies**: Clearly document which contexts are required for full functionality
4. **Test All Contexts**: Ensure your application works regardless of which contexts are applied
5. **Minimize Usage**: Use contexts sparingly - most changesets should run in all environments

### Label Categories
- **Version**: `version-1.0.0`, `version-1.1.0`
- **Feature**: `feature-user-management`, `feature-reporting`
- **Type**: `table-creation`, `data-migration`, `index-optimization`
- **Priority**: `critical`, `optional`, `performance`
- **Destructive**: `approved_destructive` (REQUIRED for destructive operations)

### Label Usage
```xml
labels="version-1.0.0,feature-user-management,table-creation"
labels="version-1.1.0,performance,index-optimization"
labels="version-1.2.0,critical,data-migration"
labels="version-1.2.0,approved_destructive,data-cleanup"
```

## Destructive Operations Policy

### **REQUIRED: approved_destructive Label**

**All changesets performing destructive operations MUST include the `approved_destructive` label.**

**Destructive operations include:**
- `DROP TABLE`, `DROP COLUMN`, `DROP INDEX`, `DROP CONSTRAINT`
- `DELETE` statements (except INSERT conflicts handling)
- `TRUNCATE` statements
- `ALTER TABLE` operations that remove or modify existing data
- Any operation that permanently removes data or schema objects

### **Destructive Changeset Examples**

#### **Table Drop with Safety Checks**
```xml
<changeSet id="drop-legacy-sessions-table-20250223-v1.2" author="john.doe" labels="version-1.2.0,approved_destructive,cleanup">
    <comment>Drop legacy user_sessions table - data migrated to new sessions schema in v1.1</comment>
    <preConditions onFail="HALT">
        <!-- Ensure table is empty or contains only old data -->
        <sqlCheck expectedResult="0">
            SELECT COUNT(*) FROM ${appSchema}.user_sessions 
            WHERE created_at > CURRENT_DATE - INTERVAL '30 days'
        </sqlCheck>
        <!-- Ensure new sessions table exists -->
        <tableExists tableName="sessions" schemaName="${appSchema}"/>
    </preConditions>
    <sql>
        DROP TABLE ${appSchema}.user_sessions;
    </sql>
    <rollback>
        -- Cannot rollback table drop - data would be permanently lost
        -- Manual restoration from backup required
        -- CREATE TABLE ${appSchema}.user_sessions (...); -- Structure only
    </rollback>
</changeSet>
```

#### **Data Cleanup with Validation**
```xml
<changeSet id="delete-expired-audit-logs-20250223-v1.2" author="security-team" labels="version-1.2.0,approved_destructive,compliance">
    <comment>Delete audit logs older than 7 years per compliance policy</comment>
    <preConditions onFail="HALT">
        <!-- Ensure we're not deleting recent data -->
        <sqlCheck expectedResult="0">
            SELECT COUNT(*) FROM ${auditSchema}.audit_logs 
            WHERE created_at > CURRENT_DATE - INTERVAL '7 years' 
            AND created_at < CURRENT_DATE - INTERVAL '6 years 11 months'
        </sqlCheck>
    </preConditions>
    <sql>
        -- Delete in batches to avoid long locks
        DELETE FROM ${auditSchema}.audit_logs 
        WHERE created_at < CURRENT_DATE - INTERVAL '7 years';
        
        -- Log the cleanup action
        INSERT INTO ${auditSchema}.cleanup_log (action, table_name, deleted_count, cleanup_date)
        VALUES ('DELETE_EXPIRED', 'audit_logs', ROW_COUNT(), CURRENT_TIMESTAMP);
    </sql>
    <rollback>
        -- Cannot rollback DELETE - data would be permanently lost
        -- Restore from backup if required for compliance investigation
    </rollback>
</changeSet>
```

#### **Column Drop with Migration Verification**
```xml
<changeSet id="drop-deprecated-user-fields-20250223-v1.2" author="jane.doe" labels="version-1.2.0,approved_destructive,schema-cleanup">
    <comment>Drop deprecated user fields - data migrated to user_profile table in v1.1</comment>
    <preConditions onFail="HALT">
        <!-- Ensure migration completed successfully -->
        <sqlCheck expectedResult="0">
            SELECT COUNT(*) FROM ${appSchema}.users 
            WHERE (old_address IS NOT NULL OR old_phone IS NOT NULL)
            AND id NOT IN (SELECT user_id FROM ${appSchema}.user_profiles)
        </sqlCheck>
    </preConditions>
    <sql>
        ALTER TABLE ${appSchema}.users 
        DROP COLUMN old_address,
        DROP COLUMN old_phone;
    </sql>
    <rollback>
        -- Partial rollback - can recreate columns but data is lost
        ALTER TABLE ${appSchema}.users 
        ADD COLUMN old_address TEXT,
        ADD COLUMN old_phone VARCHAR(20);
        -- Note: Original data cannot be restored
    </rollback>
</changeSet>
```

### **Pipeline Integration for Destructive Changes**

#### **Automated Detection**
```bash
# Check for destructive changes in pipeline
grep -r "approved_destructive" db/changelog/ || {
    echo "Checking for destructive operations without proper labeling..."
    if grep -r -i "DROP\|DELETE\|TRUNCATE" db/changelog/ --include="*.xml"; then
        echo "ERROR: Destructive operations found without approved_destructive label"
        exit 1
    fi
}
```

#### **Manual Approval Required**
```yaml
# GitLab CI/CD example
deploy-destructive:
  stage: deploy
  script:
    - mvn liquibase:update --defaultsFile=liquibase-${CI_ENVIRONMENT_NAME}.properties --labels=approved_destructive
  when: manual
  only:
    variables:
      - $CI_COMMIT_MESSAGE =~ /approved_destructive/
```

### **Best Practices for Destructive Changes**

1. **Always Use Preconditions**: Validate data state before destruction
2. **Document Irreversibility**: Clearly state what cannot be rolled back
3. **Backup Strategy**: Ensure backups exist before destructive operations
4. **Batch Operations**: Use batching for large DELETE operations
5. **Audit Trail**: Log destructive operations for compliance
6. **Manual Approval**: Require manual approval in CI/CD pipelines
7. **Testing**: Test destructive changes thoroughly in non-production environments

## Error Handling and Validation

### Preconditions for Idempotency and Safety

Preconditions ensure changesets are safe to rerun and prevent errors by checking database state before execution.

**Common Precondition Types:**
- `tableExists` / `tableNotExists`
- `columnExists` / `columnNotExists`
- `indexExists` / `indexNotExists`
- `sqlCheck` for custom validations

### Preconditions
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="add-user-constraints-20250224-v1.0" author="john.doe" labels="version-1.0.0,constraints" context="dev,test,prod">
        <comment>Add constraints to users table</comment>
        <preConditions onFail="HALT" onError="HALT">
            <sqlCheck expectedResult="1">
                SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'users'
            </sqlCheck>
        </preConditions>
        <sql>
            ALTER TABLE users 
            ADD CONSTRAINT chk_users_email_format 
            CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
        </sql>
        <rollback>
            ALTER TABLE users DROP CONSTRAINT chk_users_email_format;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Preventing Duplicate Table Creation
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <comment>Create users table if it doesn't exist</comment>
    <preConditions onFail="MARK_RAN">
        <not>
            <tableExists tableName="users"/>
        </not>
    </preConditions>
    <sql>
        CREATE TABLE users (
            id BIGSERIAL PRIMARY KEY,
            username VARCHAR(50) NOT NULL UNIQUE
        );
    </sql>
    <rollback>
        DROP TABLE users;
    </rollback>
</changeSet>
```

### Conditional Logic
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="add-status-column-conditionally-20250301-v1.1" author="john.doe" labels="version-1.1.0,conditional" context="dev,test,prod">
        <comment>Add status column if it doesn't exist</comment>
        <preConditions onFail="MARK_RAN">
            <not>
                <columnExists tableName="users" columnName="status"/>
            </not>
        </preConditions>
        <sql>
            ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';
        </sql>
        <rollback>
            ALTER TABLE users DROP COLUMN IF EXISTS status;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Data Migration Patterns

### Safe Data Updates
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="migrate-user-status-20250315-v1.2" author="john.doe" labels="version-1.2.0,data-migration" context="dev,test,prod">
        <comment>Migrate user active flag to status enum</comment>
        <sql>
            -- Add new column
            ALTER TABLE users ADD COLUMN status VARCHAR(20);

            -- Migrate existing data
            UPDATE users 
            SET status = CASE 
                WHEN is_active = true THEN 'active'
                WHEN is_active = false THEN 'inactive'
                ELSE 'unknown'
            END;

            -- Set default and not null
            ALTER TABLE users ALTER COLUMN status SET DEFAULT 'active';
            ALTER TABLE users ALTER COLUMN status SET NOT NULL;

            -- Note: Removal of old column should be in separate changeset
        </sql>
        <rollback>
            ALTER TABLE users DROP COLUMN status;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Large Data Operations
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="batch-update-users-20250316-v1.2" author="john.doe" labels="version-1.2.0,batch-operation" context="dev,test,prod">
        <comment>Update user data in batches to avoid locks</comment>
        <sql>
            DO $$
            DECLARE
                batch_size INTEGER := 1000;
                processed INTEGER := 0;
                total_rows INTEGER;
            BEGIN
                SELECT COUNT(*) INTO total_rows FROM users WHERE status IS NULL;
                
                WHILE processed &lt; total_rows LOOP
                    UPDATE users 
                    SET status = 'active'
                    WHERE id IN (
                        SELECT id FROM users 
                        WHERE status IS NULL 
                        LIMIT batch_size
                    );
                    
                    processed := processed + batch_size;
                    RAISE NOTICE 'Processed % of % rows', processed, total_rows;
                    
                    -- Small delay to reduce lock contention
                    PERFORM pg_sleep(0.1);
                END LOOP;
            END $$;
        </sql>
        <rollback>
            UPDATE users SET status = NULL WHERE status = 'active';
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Performance Considerations

### Index Creation Guidelines
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="create-indexes-concurrently-20250224-v1.0" author="john.doe" labels="version-1.0.0,performance" context="prod">
        <comment>Create indexes without blocking table access</comment>
        <sql>
            -- Use CONCURRENTLY for production to avoid table locks
            CREATE INDEX CONCURRENTLY idx_users_email_domain 
            ON users(substring(email from '@(.*)$'));

            CREATE INDEX CONCURRENTLY idx_users_last_login 
            ON users(last_login_at) 
            WHERE last_login_at IS NOT NULL;
        </sql>
        <rollback>
            DROP INDEX IF EXISTS idx_users_email_domain;
            DROP INDEX IF EXISTS idx_users_last_login;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Security Best Practices

### Privilege Management
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="create-app-user-20250225-v1.0" author="system" labels="version-1.0.0,security" context="prod" runOnChange="true">
        <comment>Create application database user with limited privileges</comment>
        <sql>
            -- Create application user
            DO $$
            BEGIN
                IF NOT EXISTS (SELECT 1 FROM pg_user WHERE usename = 'app_user') THEN
                    CREATE USER app_user WITH PASSWORD 'secure_password_here';
                END IF;
            END $$;

            -- Grant specific permissions
            GRANT CONNECT ON DATABASE myapp TO app_user;
            GRANT USAGE ON SCHEMA public TO app_user;
            GRANT SELECT, INSERT, UPDATE, DELETE ON users TO app_user;
            GRANT USAGE ON SEQUENCE users_id_seq TO app_user;
        </sql>
        <rollback>
            REVOKE ALL ON users FROM app_user;
            REVOKE USAGE ON SCHEMA public FROM app_user;
            REVOKE CONNECT ON DATABASE myapp FROM app_user;
            DROP USER IF EXISTS app_user;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Testing and Validation

### Changeset Testing Checklist
1. **Syntax validation**: SQL executes without errors
2. **Rollback testing**: Rollback statements work correctly
3. **Data integrity**: No data loss or corruption
4. **Performance impact**: Acceptable execution time
5. **Dependency validation**: Required objects exist

### Validation Queries
```sql
-- Include validation queries as comments
-- Validation: Check table exists
-- SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'users';

-- Validation: Check column exists
-- SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'users' AND column_name = 'email';

-- Validation: Check constraint exists
-- SELECT COUNT(*) FROM information_schema.table_constraints WHERE constraint_name = 'chk_users_email_format';
```

## Deployment and CI/CD Best Practices

### Development Environment
- **Automatic Execution**: Allow Liquibase to run automatically in development
- **Frequent Updates**: Keep schemas up to date with latest changes
- **Test Data**: Include development-specific test data using contexts

### CI/CD Pipeline Integration
- **Automated Testing**: Run Liquibase validate and update in CI pipeline
- **Staging Deployment**: Always test in staging before production
- **Manual Production Review**: Require manual approval for production deployments

### Production Deployment
- **Manual Review Process**: Never execute migrations directly in production without review
- **Rollback Plan**: Always have a tested rollback strategy
- **Monitoring**: Monitor execution time and database performance
- **Backup**: Ensure database backup before major migrations

### Pipeline Commands

#### **Rollforward Operations (Deployment)**

```bash
# 1. Tag the current state before deployment
mvn liquibase:tag \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.tag=v1.0.0

# 2. Validate the changelog syntax and structure
mvn liquibase:validate \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa

# 3. Preview SQL that will be executed (for review)
mvn liquibase:updateSQL \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.outputFile=target/liquibase/update.sql

# 4. Apply the changes to database
mvn liquibase:update \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa

# 5. Generate changelog from existing database (if needed)
mvn liquibase:generateChangeLog \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa
```

#### **Rollback Operations (Emergency Recovery)**

```bash
# 1. Preview rollback SQL (for review before execution)
mvn liquibase:rollbackSQL \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.rollbackTag=v1.0.0 \
  -Dliquibase.outputFile=target/liquibase/rollback.sql

# 2. Perform rollback to specific tag
mvn liquibase:rollback \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.rollbackTag=v1.0.0

# 3. Rollback specific number of changesets
mvn liquibase:rollback \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.rollbackCount=3

# 4. Rollback to specific date
mvn liquibase:rollback \
  -Dliquibase.username=sa \
  -Dliquibase.password=sa \
  -Dliquibase.rollbackDate=2025-01-20
```

### Complete CI/CD Pipeline Example

```yaml
# GitLab CI/CD Pipeline Example
stages:
  - validate
  - preview
  - deploy-dev
  - deploy-staging
  - deploy-prod
  - rollback

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"
  LIQUIBASE_PROPERTIES: "--defaultsFile=liquibase-${CI_ENVIRONMENT_NAME}.properties"

# Validation Stage
validate-changelog:
  stage: validate
  script:
    - mvn liquibase:validate ${LIQUIBASE_PROPERTIES}
    - mvn liquibase:status ${LIQUIBASE_PROPERTIES}
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
    paths:
      - target/liquibase/
    expire_in: 1 hour

# Preview Changes
preview-sql:
  stage: preview
  script:
    - mvn liquibase:updateSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.outputFile=target/liquibase/preview-${CI_ENVIRONMENT_NAME}.sql
  artifacts:
    paths:
      - target/liquibase/preview-${CI_ENVIRONMENT_NAME}.sql
    expire_in: 1 day
  only:
    - merge_requests
    - main

# Development Deployment
deploy-dev:
  stage: deploy-dev
  environment:
    name: dev
  script:
    - mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=${CI_COMMIT_SHORT_SHA}-dev
    - mvn liquibase:update ${LIQUIBASE_PROPERTIES}
  only:
    - develop

# Staging Deployment
deploy-staging:
  stage: deploy-staging
  environment:
    name: staging
  script:
    - mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=${CI_COMMIT_SHORT_SHA}-staging
    - mvn liquibase:updateSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.outputFile=target/liquibase/staging-deployment.sql
    - mvn liquibase:update ${LIQUIBASE_PROPERTIES}
  artifacts:
    paths:
      - target/liquibase/staging-deployment.sql
    expire_in: 7 days
  when: manual
  only:
    - main

# Production Deployment
deploy-prod:
  stage: deploy-prod
  environment:
    name: production
  script:
    - echo "Creating backup tag before production deployment"
    - mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=backup-$(date +%Y%m%d-%H%M%S)
    - echo "Generating deployment SQL for review"
    - mvn liquibase:updateSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.outputFile=target/liquibase/prod-deployment.sql
    - echo "Applying changes to production"
    - mvn liquibase:update ${LIQUIBASE_PROPERTIES}
    - echo "Tagging successful deployment"
    - mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=${CI_COMMIT_TAG:-${CI_COMMIT_SHORT_SHA}}-prod
  artifacts:
    paths:
      - target/liquibase/prod-deployment.sql
    expire_in: 30 days
  when: manual
  only:
    - tags
    - main

# Emergency Rollback
rollback-prod:
  stage: rollback
  environment:
    name: production
  script:
    - echo "EMERGENCY ROLLBACK - Requires manual confirmation"
    - echo "Rolling back to tag: ${ROLLBACK_TAG}"
    - mvn liquibase:rollbackSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.rollbackTag=${ROLLBACK_TAG} -Dliquibase.outputFile=target/liquibase/rollback-${ROLLBACK_TAG}.sql
    - echo "Review the rollback SQL above before proceeding"
    - read -p "Continue with rollback? (yes/no): " confirm
    - if [ "$confirm" = "yes" ]; then mvn liquibase:rollback ${LIQUIBASE_PROPERTIES} -Dliquibase.rollbackTag=${ROLLBACK_TAG}; fi
  artifacts:
    paths:
      - target/liquibase/rollback-${ROLLBACK_TAG}.sql
    expire_in: 30 days
  when: manual
  only:
    - main
```

### Pipeline Best Practices

#### **Pre-Deployment Checks**
```bash
# Always validate before deployment
mvn liquibase:validate ${LIQUIBASE_PROPERTIES}

# Check current database status
mvn liquibase:status ${LIQUIBASE_PROPERTIES}

# Generate and review SQL before applying
mvn liquibase:updateSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.outputFile=deployment-preview.sql
```

#### **Tagging Strategy**
```bash
# Tag before each deployment for rollback capability
mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=pre-deployment-$(date +%Y%m%d-%H%M%S)

# Tag after successful deployment
mvn liquibase:tag ${LIQUIBASE_PROPERTIES} -Dliquibase.tag=release-${VERSION}
```

#### **Environment-Specific Deployment**
```bash
# Development
mvn liquibase:update --defaultsFile=liquibase-dev.properties

# Staging
mvn liquibase:update --defaultsFile=liquibase-staging.properties

# Production
mvn liquibase:update --defaultsFile=liquibase-prod.properties
```

#### **Rollback Procedures**
```bash
# 1. Always preview rollback first
mvn liquibase:rollbackSQL ${LIQUIBASE_PROPERTIES} -Dliquibase.rollbackTag=${TAG} -Dliquibase.outputFile=rollback-preview.sql

# 2. Review the generated SQL carefully

# 3. Execute rollback only after approval
mvn liquibase:rollback ${LIQUIBASE_PROPERTIES} -Dliquibase.rollbackTag=${TAG}

# 4. Verify database state after rollback
mvn liquibase:status ${LIQUIBASE_PROPERTIES}
```

## Code Review and Quality Assurance

### Mandatory Review Process
1. **Peer Review**: All changesets must be reviewed by another developer
2. **DBA Review**: Complex schema changes require database administrator review
3. **Security Review**: Changes affecting permissions or sensitive data need security review
4. **Performance Review**: Index changes and large data operations need performance review

### Review Checklist
- [ ] Changeset ID is unique and descriptive
- [ ] Rollback statement is provided and tested
- [ ] Preconditions are appropriate for the change
- [ ] Context and labels are correctly applied
- [ ] No sensitive data in changeset
- [ ] Performance impact is acceptable
- [ ] Dependencies are properly handled

### Validation Commands
```bash
# Validate changeset syntax
liquibase validate

# Check changeset status
liquibase status

# Generate SQL without executing
liquibase updateSQL

# Test rollback
liquibase rollbackSQL <tag>
```

## Monitoring and Maintenance

### Database Change Tracking
Monitor the `DATABASECHANGELOG` table to track applied changes:

```sql
-- Review recently applied changesets
SELECT * FROM DATABASECHANGELOG 
ORDER BY DATEEXECUTED DESC 
LIMIT 10;

-- Check for failed changesets
SELECT * FROM DATABASECHANGELOG 
WHERE EXECTYPE = 'FAILED';

-- Monitor execution times
SELECT ID, AUTHOR, FILENAME, DATEEXECUTED, 
       CAST(EXECTYPE AS VARCHAR) as EXEC_TYPE
FROM DATABASECHANGELOG 
WHERE DATEEXECUTED > CURRENT_DATE - INTERVAL '7 days'
ORDER BY DATEEXECUTED DESC;
```

## Common Anti-Patterns to Avoid

### ❌ Don't Do This
```xml
<!-- Don't modify existing changesets -->
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <sql>
        ALTER TABLE users ADD COLUMN email VARCHAR(255); <!-- WRONG: Modifying existing changeset -->
    </sql>
</changeSet>

<!-- Don't use generic IDs -->
<changeSet id="changeset1" author="john.doe"> <!-- WRONG: Non-descriptive ID -->

<!-- Don't skip rollback for reversible operations -->
<changeSet id="temp-table" author="john.doe">
    <sql>
        CREATE TABLE temp_table (id INT); <!-- WRONG: No rollback provided -->
    </sql>
</changeSet>

<!-- Don't use hardcoded values without context -->
<changeSet id="config-insert" author="john.doe">
    <sql>
        INSERT INTO config VALUES (1, 'prod_value'); <!-- WRONG: No environment context -->
    </sql>
</changeSet>

<!-- Don't combine DDL and DML in one changeset -->
<changeSet id="user-table-and-data" author="john.doe">
    <sql>
        CREATE TABLE users (id INT, name VARCHAR(50));
        INSERT INTO users VALUES (1, 'Admin'); <!-- WRONG: Mixed DDL and DML -->
    </sql>
</changeSet>

<!-- Don't hardcode schema names -->
<changeSet id="create-table" author="john.doe">
    <sql>
        CREATE TABLE public.users (id INT); <!-- WRONG: Hardcoded schema -->
    </sql>
</changeSet>

<!-- Don't store sensitive data -->
<changeSet id="create-admin" author="john.doe">
    <sql>
        INSERT INTO users (username, password) 
        VALUES ('admin', 'password123'); <!-- WRONG: Sensitive data -->
    </sql>
</changeSet>
```

### ✅ Do This Instead
```xml
<!-- Create new changeset for modifications -->
<changeSet id="add-email-to-users-20250224-v1.0" author="john.doe">
    <sql>
        ALTER TABLE users ADD COLUMN email VARCHAR(255);
    </sql>
    <rollback>
        ALTER TABLE users DROP COLUMN email;
    </rollback>
</changeSet>

<!-- Use descriptive IDs -->
<changeSet id="add-email-to-users-20250224-v1.0" author="john.doe">

<!-- Always provide rollback when possible -->
<changeSet id="create-temp-table-20250224-v1.0" author="john.doe">
    <sql>
        CREATE TABLE temp_table (id INT);
    </sql>
    <rollback>
        DROP TABLE temp_table;
    </rollback>
</changeSet>

<!-- Use context-aware values -->
<changeSet id="insert-config-prod-20250225-v1.0" author="system" context="prod">
    <sql>
        INSERT INTO config VALUES (1, 'prod_value');
    </sql>
</changeSet>

<changeSet id="insert-config-dev-20250225-v1.0" author="system" context="dev,test">
    <sql>
        INSERT INTO config VALUES (1, 'dev_value');
    </sql>
</changeSet>

<!-- Separate DDL and DML changesets -->
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <sql>
        CREATE TABLE users (id INT, name VARCHAR(50));
    </sql>
    <rollback>
        DROP TABLE users;
    </rollback>
</changeSet>

<changeSet id="insert-admin-user-20250224-v1.0" author="john.doe">
    <sql>
        INSERT INTO users VALUES (1, 'Admin');
    </sql>
    <rollback>
        DELETE FROM users WHERE id = 1;
    </rollback>
</changeSet>

<!-- Use portable table references -->
<changeSet id="create-users-table-20250223-v1.0" author="john.doe">
    <sql>
        CREATE TABLE users (id INT);
    </sql>
</changeSet>

<!-- Use environment variables for sensitive data -->
<changeSet id="create-admin-user-20250225-v1.0" author="john.doe" context="prod">
    <sql>
        INSERT INTO users (username, password_hash) 
        VALUES ('admin', '${admin.password.hash}');
    </sql>
</changeSet>
```

## Master Changelog Structure

### db.changelog-master.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <!-- Version 1.0.0 Release -->
    <!-- DDL Changes First (Tables, Indexes, Constraints, Functions, Views) -->
    <include file="releases/v1.0.0/ddl/create-users-table-20250223-v1.0.xml"/>
    <include file="releases/v1.0.0/ddl/create-roles-table-20250223-v1.0.xml"/>
    <include file="releases/v1.0.0/ddl/add-user-indexes-20250224-v1.0.xml"/>
    <include file="releases/v1.0.0/ddl/add-constraints-20250224-v1.0.xml"/>
    <include file="releases/v1.0.0/ddl/create-audit-function-20250224-v1.0.xml"/>
    <include file="releases/v1.0.0/ddl/create-user-summary-view-20250225-v1.0.xml"/>
    
    <!-- DML Changes Second (Data Operations) -->
    <include file="releases/v1.0.0/dml/insert-default-roles-20250225-v1.0.xml"/>
    <include file="releases/v1.0.0/dml/insert-system-config-20250225-v1.0.xml"/>
    <include file="releases/v1.0.0/dml/insert-test-data-20250225-v1.0.xml"/>

    <!-- Version 1.1.0 Release -->
    <!-- DDL Changes First -->
    <include file="releases/v1.1.0/ddl/add-user-phone-column-20250301-v1.1.xml"/>
    <include file="releases/v1.1.0/ddl/create-audit-table-20250301-v1.1.xml"/>
    <include file="releases/v1.1.0/ddl/add-performance-indexes-20250302-v1.1.xml"/>
    <include file="releases/v1.1.0/ddl/create-validation-functions-20250302-v1.1.xml"/>
    
    <!-- DML Changes Second -->
    <include file="releases/v1.1.0/dml/migrate-user-status-20250303-v1.1.xml"/>
    <include file="releases/v1.1.0/dml/update-role-permissions-20250303-v1.1.xml"/>



</databaseChangeLog>
```

## Environment-Specific Considerations

### Development Environment
- Use `--context: dev` for development-only data
- Include test data and debugging helpers
- More verbose logging and validation

### Production Environment
- Use `CONCURRENTLY` for index creation
- Batch large operations
- Minimal logging to reduce overhead
- Careful privilege management

## Monitoring and Maintenance

### Changeset Metadata
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="011-add-changeset-tracking" author="system" labels="version-1.0.0,monitoring" context="dev,test,prod">
        <comment>Create table to track custom changeset metadata</comment>
        <sql>
            CREATE TABLE changeset_metadata (
                changeset_id VARCHAR(255) PRIMARY KEY,
                author VARCHAR(100) NOT NULL,
                deployment_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
                execution_time_ms INTEGER,
                rollback_tested BOOLEAN DEFAULT false,
                notes TEXT
            );
        </sql>
        <rollback>
            DROP TABLE changeset_metadata;
        </rollback>
    </changeSet>

</databaseChangeLog>
```

## Additional Best Practices

### Transaction Management
- Each changeset runs in its own transaction by default
- Use `runInTransaction="false"` for operations that can't run in transactions
- Be cautious with large transactions that may cause locks

### Large Dataset Handling
- **Don't use Liquibase for bulk data migrations**
- Use separate ETL processes for large dataset operations
- If necessary, batch large operations to avoid locks

### Schema Portability
- Avoid database-specific syntax when possible
- Use Liquibase's built-in data types for portability
- Test changesets against target database types

### Documentation Standards
```xml
<changeSet id="create-users-table-20250223-v1.0" author="john.doe" labels="version-1.0.0,user-management">
    <comment>
        Create users table with basic authentication fields.
        
        This table stores user account information including:
        - Unique username and email for login
        - Hashed passwords (never store plain text)
        - Account status and timestamps
        
        Related to: User Management Epic #123
        Dependencies: None
        Impact: New table, no existing data affected
    </comment>
    <sql>
        CREATE TABLE users (
            id BIGSERIAL PRIMARY KEY,
            username VARCHAR(50) NOT NULL UNIQUE,
            email VARCHAR(255) NOT NULL UNIQUE,
            password_hash VARCHAR(255) NOT NULL,
            is_active BOOLEAN DEFAULT true,
            created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
        );
        
        -- Add table and column comments for documentation
        COMMENT ON TABLE users IS 'Application users with authentication information';
        COMMENT ON COLUMN users.username IS 'Unique username for login (3-50 characters)';
        COMMENT ON COLUMN users.email IS 'User email address for notifications and recovery';
        COMMENT ON COLUMN users.password_hash IS 'Bcrypt hashed password (never store plain text)';
    </sql>
    <rollback>
        DROP TABLE users;
    </rollback>
</changeSet>
```

## Summary

These comprehensive guidelines ensure that Liquibase changesets are:

- **Consistent** in format and structure across teams
- **Maintainable** with clear documentation and logical organization
- **Reliable** with proper testing, rollback, and validation
- **Secure** with appropriate privilege management and no sensitive data
- **Performant** with consideration for production impact and scalability
- **Portable** across different database environments when needed
- **Reviewable** with clear processes for code review and approval
- **Monitorable** with proper tracking and validation mechanisms

### Key Reminders

1. **Never modify existing changesets** - create new ones for any changes
2. **Always provide rollback statements** unless impossible
3. **Use preconditions** to ensure idempotency and safety
4. **Separate DDL and DML** operations for better control
5. **Test in staging** before production deployment
6. **Use contexts and labels** for environment-specific execution
7. **Keep changesets small** and focused on single logical changes
8. **Document thoroughly** with comments explaining purpose and impact
9. **Review all changes** through proper code review processes
10. **Monitor execution** and maintain the DATABASECHANGELOG table