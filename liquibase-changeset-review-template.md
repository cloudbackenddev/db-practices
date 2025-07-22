# Liquibase Changeset Review Template for LLM Agents

This template enables LLM agents to systematically review generated Liquibase changesets against database requirements, DBML models, and Liquibase best practices, producing actionable feedback and remediation guidance.

---

## 🎯 **Review Objective**

**Purpose**: Validate generated Liquibase changesets against:
1. **Database Requirements** (from database_requirements.md)
2. **DBML Model** (from generated DBML)
3. **Liquibase Best Practices** (from liquibase-changeset-standards-guidelines.md)

**Output**: Structured JSON scorecard with specific remediation actions

---

## 🧩 **LLM Review Instructions**

### **Pre-Review Setup**
```
You are an expert Liquibase changeset reviewer. You will:

1. Parse the provided Liquibase XML changeset(s) completely
2. Cross-reference against the DBML model
3. Validate against database requirements
4. Check compliance with Liquibase best practices
5. Generate a comprehensive review with specific fixes

CRITICAL: Return ONLY the JSON object specified in Section 8 (Output Schema).
Do not include explanatory text outside the JSON response.
```

### **Review Context Requirements**
Before starting review, ensure you have:
- [ ] Generated Liquibase changeset XML to review
- [ ] DBML model for cross-reference
- [ ] Original database requirements document
- [ ] Target database type (PostgreSQL or Oracle)

---

## 📋 **Review Scope Matrix**

| Check ID | Category | Source | Weight | Description |
|----------|----------|---------|---------|-------------|
| **M01** | Target Database | Requirements §1 | 2 | Database type declared and matches requirements |
| **M02** | Entity Coverage | DBML | 2 | All DBML tables present in changesets |
| **M03** | Field Coverage | Requirements §3 | 2 | All JSON sample fields mapped to columns |
| **M04** | ID Strategy | Requirements §7 | 2 | Proper ID implementation (sequence + TSID) |
| **M05** | Relationships | DBML | 2 | All DBML relationships implemented as constraints |
| **M06** | Business Rules | Requirements §4 | 2 | Business rules enforced via constraints |
| **M07** | JSON Storage | Requirements §6 | 2 | JSON fields stored correctly (JSONB/JSON) |
| **M08** | Rollback | Liquibase §4 | 2 | Rollback provided for all reversible changes |
| **M09** | Changeset ID | Liquibase §3 | 2 | IDs follow verb-object-date-version format |
| **M10** | Schema Parameters | Liquibase §3 | 2 | Schema names parameterized (${schemaName}) |
| **M11** | Security Constraints | Security §1 | 2 | SQL injection prevention, privilege validation, PII handling |
| **M12** | Data Migration Safety | Security §2 | 2 | Backup verification, data validation, rollback testing |
| **R01** | DDL/DML Separation | Liquibase §2 | 1 | DDL and DML in separate changesets |
| **R02** | Performance Indexes | Requirements §6 | 1 | Indexes for high-frequency queries |
| **R03** | Preconditions | Liquibase §5 | 1 | Appropriate preconditions for safety |
| **R04** | Context/Labels | Liquibase §5 | 1 | Proper context and labels usage |
| **R05** | Destructive Labels | Liquibase §6 | 1 | Destructive operations labeled appropriately |
| **R06** | Naming Conventions | DBML §5 | 1 | Follows naming conventions |
| **R07** | Transaction Control | Liquibase §7 | 1 | Appropriate transaction management |
| **R08** | Batch Operations | Liquibase §7 | 1 | Large operations properly batched |
| **R09** | Execution Order | Liquibase §8 | 1 | DDL changesets before DML in master changelog |
| **R10** | JPA Compatibility | Requirements §7 | 1 | JPA-friendly design (if applicable) |

---

## 🔍 **Detailed Review Checklist**

### **Mandatory Checks (Weight = 2)**

#### **M01: Target Database Declaration**
```
✅ PASS Criteria:
- Database type declared in XML or properties
- Matches target database from requirements (PostgreSQL/Oracle)
- Database-specific features used correctly

❌ FAIL Examples:
- No database type declaration
- Mismatch with requirements
- Using features not available in target database

Validation:
- Check for database type in XML or properties
- Verify SQL syntax matches target database
- Ensure data types are appropriate for target database
```

#### **M02: Entity Coverage**
```
✅ PASS Criteria:
- Every table in DBML has corresponding CREATE TABLE changeset
- Table names match between DBML and changesets
- All required tables are created

❌ FAIL Examples:
- Missing tables from DBML
- Table names don't match
- Incomplete table creation

Validation:
- Extract table list from DBML
- Map to createTable or CREATE TABLE statements
- Flag missing tables
```

#### **M03: Field Coverage**
```
✅ PASS Criteria:
- Every field from JSON samples is represented
- Data types match between JSON and columns
- Required fields marked as NOT NULL

❌ FAIL Examples:
- Missing columns for JSON fields
- Incorrect data types
- Nullable constraints on required fields

Validation:
- Parse JSON samples from requirements
- Map each field to column definition
- Check constraints match requirements
```

#### **M04: ID Strategy Implementation**
```
✅ PASS Criteria:
- Business tables use sequence-based internal ID
- Public ID column (TSID format) present and unique
- Reference tables can use auto-increment
- Application-generated strategy documented
- TSID format validation (26 chars, Base32, time-ordered)

❌ FAIL Examples:
- Using SERIAL/auto-increment for business tables
- Missing public_id columns
- Using UUID instead of TSID
- Database-generated public IDs
- Incorrect TSID format or length

PostgreSQL Requirements:
- id BIGINT DEFAULT nextval('table_name_id_seq')
- public_id VARCHAR(26) NOT NULL UNIQUE (TSID format)

Oracle Requirements:
- id NUMBER(19) DEFAULT table_name_seq.nextval
- public_id VARCHAR2(26) NOT NULL UNIQUE (TSID format)

TSID Format Validation:
- Exactly 26 characters
- Base32 encoded (0-9, A-Z excluding I, L, O, U)
- Time-ordered (starts with timestamp)
- Example: 01HN2K8G7QRXM9VWPZ3F4T5Y6B
- Application-generated (not database triggers/functions)
```

#### **M05: Relationship Implementation**
```
✅ PASS Criteria:
- All relationships from DBML implemented as constraints
- Foreign key constraints match DBML relationships
- Proper ON DELETE/UPDATE actions

❌ FAIL Examples:
- Missing foreign key constraints
- Incorrect reference columns
- Missing cascade/restrict actions

Validation:
- Extract relationships from DBML
- Map to addForeignKeyConstraint or REFERENCES
- Check ON DELETE/UPDATE actions
```

#### **M06: Business Rules Enforcement**
```
✅ PASS Criteria:
- Unique constraints for business rules
- Check constraints for validation rules
- Default values match requirements

❌ FAIL Examples:
- Missing unique constraints
- No check constraints for validation
- Incorrect default values

Validation:
- Extract business rules from requirements
- Map to constraints in changesets
- Verify constraint implementation
```

#### **M07: JSON Storage Implementation**
```
✅ PASS Criteria:
- JSON fields stored as JSONB (PostgreSQL) or JSON (Oracle)
- GIN indexes for PostgreSQL JSONB
- Function-based indexes for Oracle JSON

❌ FAIL Examples:
- Using TEXT/VARCHAR for JSON
- Missing JSON indexes
- Incorrect JSON column definition

PostgreSQL:
- Column type JSONB
- GIN index: CREATE INDEX idx_json ON table USING GIN (json_column)

Oracle:
- Column type JSON or CLOB with IS JSON check
- Function index: CREATE INDEX idx_json ON table (JSON_VALUE(json_column, '$.path'))
```

#### **M08: Rollback Implementation**
```
✅ PASS Criteria:
- <rollback> element for all reversible changes
- Appropriate rollback statements
- Documentation for irreversible changes

❌ FAIL Examples:
- Missing rollback elements
- Incorrect rollback statements
- No documentation for irreversible changes

Validation:
- Check for rollback element in each changeset
- Verify rollback statements are correct
- Ensure irreversible changes are documented
```

#### **M09: Changeset ID Format**
```
✅ PASS Criteria:
- IDs follow verb-object-date-version format
- Globally unique across project
- Matches filename without extension

❌ FAIL Examples:
- Generic or numeric-only IDs
- Non-descriptive IDs
- Duplicate IDs

Correct Format:
- create-users-table-20250223-v1.0
- add-email-index-20250224-v1.0
- insert-default-roles-20250225-v1.0
```

#### **M10: Schema Parameter Usage**
```
✅ PASS Criteria:
- Schema names use ${schemaName} parameters
- No hardcoded schema names
- Cross-schema references use parameters

❌ FAIL Examples:
- Hardcoded schema names (e.g., public.users)
- Missing schema parameters
- Inconsistent parameter usage

Correct Usage:
- CREATE TABLE ${appSchema}.users
- REFERENCES ${appSchema}.users(id)
- CREATE VIEW ${reportingSchema}.user_summary
```

#### **M11: Security Constraints**
```
✅ PASS Criteria:
- No dynamic SQL construction vulnerable to injection
- PII columns have appropriate constraints/encryption
- No GRANT ALL or excessive privileges
- Sensitive data properly masked in rollback statements
- Row-level security implemented where required

❌ FAIL Examples:
- Dynamic SQL with string concatenation
- PII stored in plain text without constraints
- Overly permissive privilege grants
- Sensitive data exposed in rollback SQL
- Missing row-level security for multi-tenant data

Security Validations:
PostgreSQL:
- PII encryption: CREATE TABLE users (ssn VARCHAR(255) ENCRYPTED)
- Row-level security: ALTER TABLE users ENABLE ROW LEVEL SECURITY
- Privilege validation: No GRANT ALL TO PUBLIC statements

Oracle:
- Data masking: ALTER TABLE users MODIFY (ssn VARCHAR2(255) ENCRYPT)
- VPD policies: CREATE POLICY user_access_policy ON users
- Privilege validation: No GRANT ALL ON schema TO PUBLIC

Critical Checks:
- Scan for CONCAT(), ||, + operators in SQL
- Validate PII column constraints (SSN, email, phone)
- Check for appropriate encryption/masking
- Verify privilege grants are minimal and specific
```

#### **M12: Data Migration Safety**
```
✅ PASS Criteria:
- Backup verification preconditions before destructive operations
- Data validation queries in DML changesets
- Rollback testing for large dataset operations
- Zero-downtime deployment patterns for production

❌ FAIL Examples:
- Destructive operations without backup verification
- No data validation after migration
- Untested rollback for large data changes
- Blocking operations during business hours

Safety Validations:
- Backup preconditions for DROP/TRUNCATE operations
- Row count validation before/after data migration
- Rollback verification for data-heavy changesets
- Lock duration estimation for large operations

Required Preconditions:
```xml
<preConditions onFail="HALT">
  <sqlCheck expectedResult="1">
    SELECT CASE WHEN backup_completed_at > NOW() - INTERVAL '1 hour' 
    THEN 1 ELSE 0 END FROM backup_status
  </sqlCheck>
</preConditions>
```

Data Validation Pattern:
```xml
<changeSet id="validate-migration-data" author="system">
  <sql>
    INSERT INTO migration_validation (table_name, expected_count, actual_count, status)
    SELECT 'users', 
           (SELECT count(*) FROM users_backup),
           (SELECT count(*) FROM users),
           CASE WHEN (SELECT count(*) FROM users_backup) = (SELECT count(*) FROM users) 
           THEN 'PASS' ELSE 'FAIL' END;
  </sql>
</changeSet>
```
```

### **Recommended Checks (Weight = 1)**

#### **R01: DDL/DML Separation**
```
✅ PASS Criteria:
- DDL (schema changes) in separate changesets from DML (data changes)
- DDL executed before DML in same release
- Clear separation in directory structure

❌ FAIL Examples:
- Mixed DDL and DML in same changeset
- DML before dependent DDL
- Unclear separation

Validation:
- Check each changeset for mixed operations
- Verify execution order in master changelog
- Check directory structure follows guidelines
```

#### **R02: Performance Indexes**
```
✅ PASS Criteria:
- Indexes for high-frequency query columns
- Composite indexes for multi-column queries
- Appropriate index types (B-tree, GIN, etc.)

❌ FAIL Examples:
- Missing indexes for query patterns
- Inefficient index types
- Redundant indexes

Validation:
- Extract query patterns from requirements
- Map to index definitions
- Check index types and columns
```

#### **R03: Preconditions Usage**
```
✅ PASS Criteria:
- Appropriate preconditions for safety
- Idempotent operations with MARK_RAN
- Validation checks before destructive operations

❌ FAIL Examples:
- Missing preconditions for risky operations
- Incorrect onFail/onError actions
- Insufficient validation

Validation:
- Check for preconditions in changesets
- Verify onFail/onError actions
- Ensure destructive operations have validation
```

#### **R04: Context and Labels Usage**
```
✅ PASS Criteria:
- Appropriate contexts (dev, test, prod)
- Version labels (version-1.0.0)
- Feature labels (feature-user-management)

❌ FAIL Examples:
- Missing contexts for environment-specific changes
- No version labels
- Inconsistent label usage

Validation:
- Check context attributes
- Verify label attributes
- Ensure consistency across changesets
```

#### **R05: Destructive Operation Labeling**
```
✅ PASS Criteria:
- approved_destructive label for DROP, DELETE, TRUNCATE
- Preconditions for destructive operations
- Documentation for irreversible changes

❌ FAIL Examples:
- Destructive operations without approved_destructive label
- Missing preconditions for destructive operations
- No documentation for data loss

Validation:
- Scan for DROP, DELETE, TRUNCATE operations
- Check for approved_destructive label
- Verify preconditions and documentation
```

#### **R06: Naming Conventions**
```
✅ PASS Criteria:
- Tables: snake_case, singular
- Columns: snake_case
- Foreign keys: {table}_id format
- Indexes: idx_{table}_{column}

❌ FAIL Examples:
- CamelCase or PascalCase names
- Plural table names
- Inconsistent naming

Validation:
- Check table and column names
- Verify foreign key naming
- Check index naming
```

#### **R07: Transaction Management**
```
✅ PASS Criteria:
- Appropriate runInTransaction attributes
- Large operations outside transactions
- DDL that can't be in transactions marked accordingly

❌ FAIL Examples:
- Missing runInTransaction for operations that need it
- Large operations in transactions
- Incorrect transaction management

Validation:
- Check for runInTransaction attributes
- Identify large operations
- Verify transaction management for DDL
```

#### **R08: Batch Operation Patterns**
```
✅ PASS Criteria:
- Large data operations use batching
- Appropriate batch size
- Delay between batches to reduce lock contention

❌ FAIL Examples:
- Large operations without batching
- Excessive batch size
- No delay between batches

Validation:
- Identify large data operations
- Check for batching patterns
- Verify batch size and delay
```

#### **R09: Master Changelog Execution Order**
```
✅ PASS Criteria:
- DDL changesets included before their corresponding DML changesets
- Logical execution order maintained (tables before data)
- Dependencies properly sequenced (parent tables before child tables)

❌ FAIL Examples:
- DML changesets before dependent DDL changesets
- INSERT statements before CREATE TABLE
- Child table creation before parent table creation

Validation:
- Parse db.changelog-master.xml include order
- Verify DDL changesets precede DML changesets within same release
- Check table creation order respects foreign key dependencies
- Validate index creation after table creation
```

#### **R10: JPA Compatibility**
```
✅ PASS Criteria (if Spring Boot JPA specified in requirements):
- Single-column primary keys (no composite keys)
- Foreign key naming supports @JoinColumn mapping
- Sequence allocation size appropriate for batch operations
- JSON fields compatible with @JdbcTypeCode annotations
- Junction tables properly structured for @ManyToMany

❌ FAIL Examples:
- Composite primary keys that complicate JPA mapping
- Foreign key names that don't follow JPA conventions
- Sequence increment of 1 (poor for batch operations)
- JSON column types incompatible with JPA JSON handling
- Junction tables missing proper primary keys

JPA-Friendly Patterns:
PostgreSQL:
- Sequences: CREATE SEQUENCE users_id_seq INCREMENT BY 50 (for batch operations)
- JSON: JSONB columns for @JdbcTypeCode(SqlTypes.JSON) compatibility
- Foreign keys: user_id, order_id (supports @JoinColumn naming)

Oracle:
- Sequences: CREATE SEQUENCE users_id_seq INCREMENT BY 50 CACHE 50
- JSON: JSON datatype for JPA custom converters
- Foreign keys: USER_ID, ORDER_ID (Oracle naming conventions)

Validation:
- Check for composite primary key definitions
- Verify foreign key naming conventions
- Validate sequence increment sizes (should be > 1 for batch operations)
- Ensure JSON column types support JPA mapping
```

---

## 📊 **Risk-Based Scoring Methodology**

### **Enhanced Score Calculation**
```
Base Score = (PassedMandatory × 2 + PassedRecommended × 1) / (TotalMandatory × 2 + TotalRecommended × 1) × 100

Risk-Adjusted Score = Base Score × Risk Multipliers

Risk Multipliers:
- Production deployment: ×1.5 weight on mandatory checks
- Large dataset operations (>100K rows): ×1.3 weight on performance checks  
- Multi-tenant systems: ×1.2 weight on security checks
- Financial/Healthcare data: ×1.4 weight on security and compliance checks
- High-availability systems: ×1.2 weight on rollback and safety checks
```

### **Weighted Category Scoring**
```
Security & Compliance: 25% (M11, M12, R05)
Data Integrity & Relationships: 20% (M02, M03, M05, M06)
Performance & Scalability: 20% (M07, R02, R07, R08)
Maintainability & Standards: 15% (M09, M10, R01, R06)
Deployment Safety: 10% (M08, R03, R09)
Documentation & Rollback: 10% (M01, M04, R04)
```

### **Enhanced Grade Thresholds**
```
A+ (Exceptional): ≥ 95% - Production-ready with exemplary practices
A (Excellent): 90-94% - Production-ready with minor improvements
B+ (Very Good): 85-89% - Near production-ready, address warnings
B (Good): 80-84% - Requires attention to critical issues
C (Fair): 70-79% - Significant improvements needed
D (Poor): 60-69% - Major rework required
F (Failing): < 60% - Not suitable for deployment
```

### **Severity Levels with Risk Context**
```
CRITICAL: Failed mandatory security/safety check (blocks all deployments)
ERROR: Failed mandatory check (blocks deployment to target environment)
WARNING: Failed recommended check (should fix before production)
INFO: Suggestion for improvement (consider for future releases)
```

---

## 🛠️ **Remediation Format**

### **Enhanced Fix Template Structure**
```json
{
  "checkId": "M03",
  "severity": "ERROR",
  "file": "create-users-table-20250223-v1.0.xml",
  "line": 42,
  "message": "Missing 'email' column required by JSON sample",
  "fixType": "ADD_XML_ELEMENT",
  "fix": "Add: <column name=\"email\" type=\"VARCHAR(255)\"><constraints nullable=\"false\" unique=\"true\"/></column>",
  "context": {
    "description": "JSON sample shows 'email' field but no corresponding column found",
    "affectedTables": ["users", "user_profiles"],
    "dependentChangesets": ["create-user-profiles-20250224-v1.0"],
    "estimatedImpact": "HIGH",
    "suggestedTestQueries": [
      "SELECT COUNT(*) FROM users WHERE email IS NULL",
      "EXPLAIN SELECT * FROM users WHERE email = 'test@example.com'"
    ],
    "sourceReferences": {
      "dbmlLine": 15,
      "requirementSection": "§3.2 User Authentication Fields",
      "jsonSchemaPath": "$.user.email"
    }
  },
  "crossReference": {
    "reviewType": "DBML",
    "checkId": "M03",
    "suggestion": "Verify if email field was properly mapped in DBML generation",
    "relatedIssues": ["M05", "R02"]
  },
  "preDeploymentChecks": [
    {
      "checkType": "DISK_SPACE",
      "query": "SELECT pg_size_pretty(pg_total_relation_size('${schemaName}.users'))",
      "threshold": "100MB",
      "action": "MONITOR_SPACE_USAGE"
    }
  ]
}
```

### **Fix Types for Automation**
```
ADD_XML_ELEMENT: Add new XML element (column, index, constraint)
REPLACE_ATTRIBUTE_VALUE: Change existing attribute value
ADD_ATTRIBUTE: Add new attribute to existing element
REMOVE_ELEMENT: Remove XML element (for cleanup)
REPLACE_ELEMENT: Replace entire XML element
ADD_ROLLBACK: Add rollback element to changeset
REORDER_ELEMENTS: Change order of elements in master changelog
PARAMETERIZE_SCHEMA: Replace hardcoded schema with parameter
```

### **Common Fix Patterns**

#### **Missing Column Fix**
```
"fix": "Add: <column name=\"email\" type=\"VARCHAR(255)\"><constraints nullable=\"false\" unique=\"true\"/></column>"
```

#### **Missing Index Fix**
```
"fix": "Add: <createIndex tableName=\"users\" indexName=\"idx_users_email\"><column name=\"email\"/></createIndex>"
```

#### **Missing Foreign Key Fix**
```
"fix": "Add: <addForeignKeyConstraint baseTableName=\"orders\" baseColumnNames=\"user_id\" constraintName=\"fk_orders_user\" referencedTableName=\"users\" referencedColumnNames=\"id\" onDelete=\"CASCADE\"/>"
```

#### **Missing Rollback Fix**
```
"fix": "Add: <rollback><dropTable tableName=\"users\"/></rollback>"
```

#### **Schema Parameter Fix**
```
"fix": "Replace 'public.users' with '${appSchema}.users'"
```

---

## � **Pure-Deployment Safety Validation**

### **Automated Safety Checks**
```json
{
  "preDeploymentValidation": {
    "diskSpaceChecks": [
      {
        "checkType": "DISK_SPACE",
        "query": "SELECT pg_size_pretty(pg_total_relation_size('${schemaName}.users'))",
        "threshold": "1GB",
        "action": "REQUIRE_MAINTENANCE_WINDOW"
      }
    ],
    "concurrentUserChecks": [
      {
        "checkType": "CONCURRENT_USERS",
        "query": "SELECT count(*) FROM pg_stat_activity WHERE datname = current_database()",
        "threshold": 100,
        "action": "SCHEDULE_LOW_USAGE_WINDOW"
      }
    ],
    "lockDurationEstimation": [
      {
        "operation": "ALTER TABLE users ADD COLUMN email VARCHAR(255)",
        "estimatedLockTime": "< 1 second",
        "riskLevel": "LOW"
      },
      {
        "operation": "CREATE INDEX CONCURRENTLY idx_users_email ON users(email)",
        "estimatedLockTime": "5-10 minutes",
        "riskLevel": "MEDIUM"
      }
    ],
    "backupValidation": [
      {
        "checkType": "BACKUP_RECENCY",
        "query": "SELECT backup_completed_at FROM backup_status WHERE table_name = 'users'",
        "threshold": "1 hour",
        "action": "HALT_IF_NO_RECENT_BACKUP"
      }
    ]
  }
}
```

### **Risk Assessment Matrix**
```
HIGH RISK Operations:
- DROP TABLE/COLUMN (data loss)
- Large data migrations (>100K rows)
- Schema changes during business hours
- Operations without rollback capability

MEDIUM RISK Operations:
- ADD COLUMN with NOT NULL constraint
- CREATE INDEX on large tables
- Foreign key constraint additions
- Batch data updates

LOW RISK Operations:
- ADD COLUMN (nullable)
- CREATE INDEX CONCURRENTLY
- INSERT reference data
- View/function creation
```

---

## 📤 **Enhanced Output Schema (Strict JSON)**

```json
{
  "reviewMetadata": {
    "reviewDate": "2025-01-22T10:30:00Z",
    "targetDatabase": "PostgreSQL",
    "dbmlVersion": "1.0.0",
    "liquibaseVersion": "4.20.0",
    "riskProfile": {
      "deploymentType": "PRODUCTION",
      "datasetSize": "LARGE",
      "systemType": "MULTI_TENANT",
      "complianceRequirements": ["FINANCIAL", "PCI_DSS"]
    }
  },
  "overallScore": {
    "grade": "B+",
    "percentage": 87,
    "baseScore": 85,
    "riskAdjustedScore": 87,
    "passedMandatory": 11,
    "totalMandatory": 12,
    "passedRecommended": 7,
    "totalRecommended": 9,
    "appliedRiskMultipliers": {
      "productionDeployment": 1.5,
      "multiTenantSystem": 1.2,
      "financialCompliance": 1.4
    }
  },
  "categoryScores": {
    "securityCompliance": 85,
    "dataIntegrityRelationships": 95,
    "performanceScalability": 80,
    "maintainabilityStandards": 90,
    "deploymentSafety": 75,
    "documentationRollback": 88
  },
  "securityAssessment": {
    "sqlInjectionRisk": "LOW",
    "piiHandling": "COMPLIANT",
    "privilegeEscalation": "NONE_DETECTED",
    "dataExposureRisk": "LOW"
  },
  "preDeploymentValidation": {
    "diskSpaceRequired": "250MB",
    "estimatedDowntime": "< 30 seconds",
    "concurrentUserImpact": "MINIMAL",
    "backupRequired": true,
    "maintenanceWindowRecommended": false
  },
  "remediations": [
    {
      "checkId": "M03",
      "severity": "ERROR",
      "file": "create-users-table-20250223-v1.0.xml",
      "line": 15,
      "message": "Missing 'phoneNumber' column required by JSON sample",
      "fixType": "ADD_XML_ELEMENT",
      "fix": "Add: <column name=\"phone_number\" type=\"VARCHAR(20)\"/>",
      "context": "JSON sample includes phoneNumber but no corresponding column found. This issue may originate from the DBML model. Cross-reference DBML review check 'M03'.",
      "crossReference": {
        "reviewType": "DBML",
        "checkId": "M03",
        "suggestion": "Verify if phoneNumber field was properly mapped in DBML generation"
      }
    },
    {
      "checkId": "M08",
      "severity": "ERROR",
      "file": "create-users-table-20250223-v1.0.xml",
      "line": 5,
      "message": "Missing rollback element",
      "fixType": "ADD_ROLLBACK",
      "fix": "Add: <rollback><dropTable tableName=\"users\"/></rollback>",
      "context": "All changesets must have rollback instructions"
    },
    {
      "checkId": "R02",
      "severity": "WARNING",
      "file": "create-users-table-20250223-v1.0.xml",
      "line": 25,
      "message": "Missing index for high-frequency query on email column",
      "fixType": "ADD_XML_ELEMENT",
      "fix": "Add: <createIndex tableName=\"users\" indexName=\"idx_users_email\"><column name=\"email\"/></createIndex>",
      "context": "Requirements specify frequent authentication queries by email"
    },
    {
      "checkId": "R09",
      "severity": "WARNING",
      "file": "db.changelog-master.xml",
      "line": 12,
      "message": "DML changeset included before dependent DDL changeset",
      "fixType": "REORDER_ELEMENTS",
      "fix": "Move 'insert-default-roles-20250225-v1.0.xml' after 'create-roles-table-20250223-v1.0.xml'",
      "context": "DDL operations must precede DML operations within the same release"
    }
  ],
  "missingEntities": [
    {
      "entityName": "user_profiles",
      "dbmlTable": "user_profiles",
      "reason": "Table exists in DBML but no corresponding changeset found"
    }
  ],
  "missingFields": [
    {
      "entity": "users",
      "jsonField": "phoneNumber",
      "suggestedColumn": "phone_number VARCHAR(20)",
      "status": "MISSING"
    }
  ],
  "schemaParameterIssues": [
    {
      "file": "create-orders-table-20250223-v1.0.xml",
      "line": 42,
      "hardcodedSchema": "public.orders",
      "suggestedFix": "${appSchema}.orders"
    }
  ],
  "additionalComments": [
    "Good use of preconditions for idempotent operations",
    "Appropriate transaction management for large operations",
    "Consider adding more descriptive comments to changesets",
    "Batch size for data migration looks appropriate"
  ],
  "recommendedNextSteps": [
    "Fix all ERROR severity issues before deployment",
    "Add missing indexes for performance optimization",
    "Create changesets for missing DBML tables",
    "Add schema parameters for all hardcoded schema references"
  ]
}
```

---

## 🚀 **Quick Reference Commands**

### **Syntax Validation**
```bash
# Validate XML syntax
xmllint --noout changeset.xml

# Validate with Liquibase
liquibase validate --changelog-file=changeset.xml

# Preview SQL
liquibase updateSQL --changelog-file=changeset.xml
```

### **Review Execution**
```bash
# Run comprehensive review
llm-review --changeset changeset.xml --dbml model.dbml --requirements requirements.md --template review-template.md

# Quick syntax check
liquibase validate --changelog-file=changeset.xml
```

---

## 📋 **Review Checklist Summary**

### **Before Starting Review**
- [ ] Liquibase changeset XML is syntactically valid
- [ ] DBML model is available for cross-reference
- [ ] Database requirements document is available
- [ ] Target database type is identified

### **During Review**
- [ ] Parse XML completely (changesets, columns, constraints, indexes)
- [ ] Map DBML tables to changesets
- [ ] Validate JSON field coverage
- [ ] Check relationship implementations
- [ ] Verify business rule enforcement
- [ ] Assess rollback completeness
- [ ] Validate schema parameter usage
- [ ] Check performance optimization

### **After Review**
- [ ] Generate comprehensive JSON output
- [ ] Provide specific remediation actions
- [ ] Include line numbers for fixes
- [ ] Categorize issues by severity
- [ ] Suggest next steps for improvement

---

**Template Version:** 1.0.0  
**Last Updated:** 2025-01-22  
**Compatible with:** database_requirements-simple.md v1.0.0, database_standards-guidelines.md v1.0.0, liquibase-changeset-standards-guidelines.md v1.0.0