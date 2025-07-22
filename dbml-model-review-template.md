# DBML Model Review Template for LLM Agents v1.1

This template enables LLM agents to systematically review generated DBML models against requirements and standards, producing actionable feedback and remediation guidance.

---

## 🎯 **Review Objective**

**Purpose**: Validate a generated DBML model against:
1. **Business Requirements** (from database_requirements.md)
2. **DBML Standards** (from dbml-standards-guidelines.md)
3. **Database-Specific Best Practices** (PostgreSQL/Oracle)

**Output**: Structured JSON scorecard with specific remediation actions

---

## 🧩 **LLM Review Instructions**

### **Pre-Review Setup**
```
You are an expert database model reviewer. You will:

1. Parse the provided DBML model completely
2. Cross-reference against the requirements document
3. Validate compliance with DBML standards
4. Generate a comprehensive review with specific fixes

CRITICAL: Return ONLY the JSON object specified in Section 8 (Output Schema).
Do not include explanatory text outside the JSON response.
```

### **Review Context Requirements**
Before starting review, ensure you have:
- [ ] Generated DBML model to review
- [ ] Original database_requirements.md document
- [ ] DBML standards guidelines document
- [ ] Target database type (PostgreSQL or Oracle)
- [ ] Application framework (Spring Boot JPA, etc.)

### **Error Handling Guidelines**
When encountering edge cases:

**Ambiguous Requirements**
- Flag as "REQUIRES_CLARIFICATION" in remediation
- Provide multiple interpretation options
- Set confidence score < 70

**Conflicting Business Rules**
- Document all conflicts in additionalComments
- Prioritize explicit requirements over inferred ones
- Request stakeholder input in recommendedNextSteps

**Incomplete DBML Syntax**
- Attempt to parse what's available
- Flag syntax errors as CRITICAL severity
- Provide specific line-by-line fixes

**Missing Requirement Sections**
- Note missing sections in reviewMetadata
- Adjust scoring to exclude unavailable checks
- Recommend obtaining complete requirements

---

## 📋 **Review Scope Matrix**

| Check ID | Category | Source | Weight | Description |
|----------|----------|---------|---------|-------------|
| **M01** | Target Database | Requirements §1 | 2 | Database type declared (PostgreSQL/Oracle) |
| **M02** | Entity Coverage | Requirements §3 | 2 | All required entities exist as tables |
| **M03** | JSON Field Mapping | Requirements §3 | 2 | All JSON sample fields represented |
| **M04** | ID Strategy | Requirements §7 | 2 | Proper ID implementation (sequence + TSID) |
| **M05** | Relationships | Requirements §4 | 2 | All relationships implemented correctly |
| **M06** | Business Rules | Requirements §4 | 2 | Constraints enforce business rules |
| **M07** | JSON Storage | Requirements §6 | 2 | JSON vs normalized decisions correct |
| **M08** | Enum Implementation | Requirements §3 | 2 | Enum values match requirements exactly |
| **M09** | Naming Conventions | Standards §5 | 2 | Follows snake_case and naming rules |
| **M10** | DBML Syntax | Standards §3 | 2 | Valid DBML syntax and structure |
| **M11** | Documentation | Standards §4 | 2 | Comprehensive table and column notes |
| **M12** | Database Features | Standards §2 | 2 | Uses database-specific features correctly |
| **R01** | Performance Indexes | Requirements §7 | 1 | High-frequency queries have indexes |
| **R02** | Composite Indexes | Requirements §7 | 1 | Multi-column indexes for query patterns |
| **R03** | Audit Fields | Requirements §6 | 1 | Standard audit columns present |
| **R04** | Soft Delete | Requirements §6 | 1 | Soft delete pattern implemented |
| **R05** | Index Priority | Standards §5 | 1 | Index priority follows guidelines |
| **R06** | Table Groups | Standards §6 | 1 | Logical table grouping implemented |
| **R07** | Security Notes | Requirements §5 | 1 | PII and security documented |
| **R08** | JPA Compatibility | Requirements §7 | 1 | JPA-friendly design (if applicable) |

---

## 🔍 **Detailed Review Checklist**

### **Mandatory Checks (Weight = 2)**

#### **M01: Target Database Declaration**
```
✅ PASS Criteria:
- Project block exists with database_type specified
- Database type is either 'PostgreSQL' or 'Oracle'
- Matches target database from requirements

❌ FAIL Examples:
- No Project block
- database_type missing or invalid
- Mismatch with requirements specification

Fix Template:
Project {database_type} {
  database_type: 'PostgreSQL'  // or 'Oracle'
  Note: 'Project description from requirements'
}
```

#### **M02: Entity Coverage**
```
✅ PASS Criteria:
- Every entity from Requirements §3 exists as a table
- Table names follow naming conventions
- No missing core business entities

❌ FAIL Examples:
- Missing tables for entities specified in requirements
- Table names don't match entity names
- Core business entities omitted

Validation Process:
1. Extract entity list from Requirements §3
2. Map each entity to corresponding DBML table
3. Flag missing entities
4. Verify table names follow conventions
```

#### **M03: JSON Field Mapping**
```
✅ PASS Criteria:
- Every field from JSON samples is represented
- Nested JSON objects properly handled (JSONB vs normalized)
- Array fields appropriately modeled
- Field data types match JSON sample types

❌ FAIL Examples:
- Missing columns for JSON sample fields
- Incorrect data type mapping
- Nested objects not properly handled
- Arrays not modeled correctly

Validation Process:
1. Parse all JSON samples from requirements
2. Flatten nested objects and identify arrays
3. Map each field to table column or JSONB path
4. Validate data type compatibility
5. Check JSON vs normalized decisions
```

#### **M04: ID Strategy Implementation**
```
✅ PASS Criteria:
- Business tables have sequence-based internal ID
- Public ID column exists (TSID format only)
- Reference tables can use auto-increment
- Proper indexing on public ID columns
- Application-generated strategy documented

❌ FAIL Examples:
- Using SERIAL/auto-increment for business tables
- Missing public_id columns
- Using UUID instead of TSID
- Incorrect data types for IDs
- Missing indexes on public IDs
- Database-generated public IDs

PostgreSQL Requirements:
- id: BIGINT DEFAULT nextval('table_name_id_seq')
- public_id: VARCHAR(26) NOT NULL UNIQUE (TSID format)

Oracle Requirements:
- id: NUMBER(19) DEFAULT table_name_seq.nextval
- public_id: VARCHAR2(26) NOT NULL UNIQUE (TSID format)

TSID Format Validation:
- 26 characters exactly
- Base32 encoded
- Time-ordered (starts with timestamp)
- Example: 01HN2K8G7QRXM9VWPZ3F4T5Y6B
```

#### **M05: Relationship Implementation**
```
✅ PASS Criteria:
- All relationships from Requirements §4 implemented
- Correct cardinality symbols (>, <, -)
- Foreign key constraints properly defined
- Junction tables for many-to-many relationships

❌ FAIL Examples:
- Missing relationship definitions
- Incorrect cardinality
- Missing foreign key indexes
- Improper junction table structure

Validation Process:
1. Extract relationship definitions from requirements
2. Verify each relationship exists in DBML
3. Check cardinality matches requirements
4. Validate foreign key naming conventions
5. Ensure junction tables follow patterns
```

#### **M06: Business Rules Enforcement**
```
✅ PASS Criteria:
- Unique constraints for business rules
- NOT NULL constraints where required
- Check constraints documented in notes
- Default values match requirements

❌ FAIL Examples:
- Missing unique constraints
- Nullable fields that should be required
- No check constraint documentation
- Incorrect default values

Validation Process:
1. Extract business rules from Requirements §4
2. Map rules to database constraints
3. Verify constraint implementation
4. Check default value assignments
5. Validate enum constraint usage
```

#### **M07: JSON Storage Decisions**
```
✅ PASS Criteria:
- Follows JSON vs Normalized Decision Matrix
- Variable attributes stored as JSON
- Frequently queried fields as columns
- Proper JSON indexing strategy

❌ FAIL Examples:
- Storing structured data as JSON unnecessarily
- Missing JSON indexes for queried paths
- Incorrect JSON data type selection
- Not following hybrid approach

Decision Matrix Validation:
- Variable attributes → JSON ✓
- Fixed schema data → Normalized ✓
- Frequently queried → Columns ✓
- Relationships → Normalized ✓
```

#### **M08: Enum Implementation**
```
✅ PASS Criteria:
- Enum types defined for status fields
- Enum values match requirements exactly
- Proper enum usage in table definitions
- Comprehensive enum documentation

❌ FAIL Examples:
- Missing enum definitions
- Enum values don't match requirements
- Using VARCHAR without enum constraint
- Incomplete enum documentation

Validation Process:
1. Extract enumerated values from Requirements §6
2. Verify Enum blocks exist in DBML
3. Check enum values match exactly
4. Validate enum usage in tables
5. Ensure proper documentation
```

#### **M09: Naming Conventions**
```
✅ PASS Criteria:
- Tables: snake_case, singular
- Columns: snake_case
- Foreign keys: {table}_id format
- Booleans: is_{condition} format
- Timestamps: {action}_at format

❌ FAIL Examples:
- CamelCase or PascalCase names
- Plural table names
- Inconsistent foreign key naming
- Poor boolean naming
- Inconsistent timestamp naming
```

#### **M10: DBML Syntax Validation**
```
✅ PASS Criteria:
- Valid DBML syntax throughout
- Proper block structure
- Correct attribute syntax
- Valid relationship definitions

❌ FAIL Examples:
- Syntax errors preventing parsing
- Malformed table definitions
- Invalid attribute combinations
- Broken relationship syntax

Validation: Run dbml2sql --postgres to check syntax
```

#### **M11: Documentation Quality**
```
✅ PASS Criteria:
- Every table has descriptive Note block
- Business rules documented
- Relationships explained
- Column purposes described

❌ FAIL Examples:
- Missing table notes
- Empty or generic descriptions
- Undocumented business rules
- No relationship explanations
```

#### **M12: Database-Specific Features**
```
PostgreSQL Specific:
✅ PASS: JSONB usage, GIN indexes, proper sequences
❌ FAIL: Using JSON instead of JSONB, missing GIN indexes

Oracle Specific:
✅ PASS: JSON datatype, function-based indexes, sequences
❌ FAIL: Using CLOB without JSON constraints, missing indexes
```

### **Recommended Checks (Weight = 1)**

#### **R01: Performance Indexes**
```
✅ PASS Criteria:
- High-frequency queries have dedicated indexes
- Foreign key columns are indexed
- Unique constraints have indexes
- Query patterns from requirements covered

Validation Process:
1. Extract query patterns from Requirements §7
2. Identify high-frequency access patterns
3. Verify corresponding indexes exist
4. Check index types are appropriate
```

#### **R02: Composite Indexes**
```
✅ PASS Criteria:
- Multi-column queries have composite indexes
- Index column order optimized
- Covers common query patterns
- Proper naming conventions

Example Patterns:
- WHERE category_id = ? AND is_active = true
- ORDER BY created_at DESC, status
- GROUP BY user_id, date_trunc('day', created_at)
```

#### **R03: Audit Fields**
```
✅ PASS Criteria:
- created_at, updated_at timestamps
- created_by, updated_by user references
- version column for optimistic locking
- Proper data types and defaults

Standard Pattern:
created_at timestamptz [not null, default: `CURRENT_TIMESTAMP`]
updated_at timestamptz [not null, default: `CURRENT_TIMESTAMP`]
created_by bigint [not null, ref: > users.id]
updated_by bigint [not null, ref: > users.id]
version integer [not null, default: 1]
```

#### **R04: Soft Delete Pattern**
```
✅ PASS Criteria:
- is_deleted boolean with default false
- deleted_at timestamp (nullable)
- deleted_by user reference (nullable)
- Partial index on active records

Standard Pattern:
is_deleted boolean [not null, default: false]
deleted_at timestamptz
deleted_by bigint [ref: > users.id]

Indexes {
  is_deleted [name: 'idx_table_active', where: 'is_deleted = false']
}
```

#### **R05: Index Priority Compliance**
```
Index Priority Validation:
CRITICAL: Primary keys, foreign keys, unique constraints ✓
HIGH: Frequently filtered columns, authentication fields ✓
MEDIUM: Sorting columns, composite indexes ✓
LOW: JSON paths, reporting queries ✓
```

#### **R06: Table Groups**
```
✅ PASS Criteria:
- Logical domain grouping
- Related tables grouped together
- Clear group naming
- Comprehensive coverage

Example:
TableGroup user_management {
  users
  user_profiles
  user_sessions
}
```

#### **R07: Security Documentation**
```
✅ PASS Criteria:
- PII fields identified and documented
- Encryption requirements noted
- Access control documented
- Compliance requirements mentioned

Example Note:
Note: '''
**Security**: Contains PII data - encrypt at rest
**Access**: Restricted to authorized personnel only
**Compliance**: GDPR Article 32 - data protection
'''
```

#### **R08: JPA Compatibility**
```
✅ PASS Criteria (if Spring Boot JPA specified):
- Single primary keys (no composite)
- Clean entity mapping structure
- Proper foreign key naming for @JoinColumn
- JSON fields compatible with @JdbcTypeCode

JPA-Friendly Patterns:
- Surrogate keys for all business entities
- Foreign keys named for JPA conventions
- Junction tables properly structured
- Sequence allocation considerations
```

---

## 📊 **Scoring Methodology**

### **Score Calculation**
```
Score = (PassedMandatory × 2 + PassedRecommended × 1) / (TotalMandatory × 2 + TotalRecommended × 1) × 100

Grade Thresholds:
A (Excellent): ≥ 90%
B (Good): 80-89%
C (Fair): 70-79%
D (Poor): < 70%
```

### **Severity Levels**
```
ERROR: Failed mandatory check (blocks deployment)
WARNING: Failed recommended check (should fix)
INFO: Suggestion for improvement
```

---

## 🛠️ **Remediation Format**

### **Fix Template Structure**
```json
{
  "checkId": "M03",
  "severity": "ERROR",
  "fixType": "ADD_COLUMN",
  "confidence": 95,
  "requirementId": "BR-03",
  "table": "users",
  "line": 42,
  "targetLocation": "users table, after email column",
  "message": "Missing 'phone_number' column required by JSON sample",
  "fix": "phone_number varchar(20) [note: 'User phone number for 2FA']",
  "context": "JSON sample shows 'phoneNumber' field but no corresponding column found",
  "autoApplicable": true
}
```

### **Fix Type Classification**

| Fix Type | Description | Auto-Applicable | Risk Level |
|----------|-------------|-----------------|------------|
| `ADD_COLUMN` | Add missing column to table | Yes | Low |
| `ADD_INDEX` | Add performance or constraint index | Yes | Low |
| `ADD_RELATIONSHIP` | Add foreign key relationship | No | Medium |
| `ADD_ENUM` | Define new enum type | Yes | Low |
| `ADD_TABLE` | Create missing table | No | High |
| `MODIFY_COLUMN` | Change column definition | No | High |
| `MODIFY_CONSTRAINT` | Update constraint definition | No | Medium |
| `ADD_PROJECT_BLOCK` | Add missing project configuration | Yes | Low |
| `ADD_TABLE_GROUP` | Add logical table grouping | Yes | Low |
| `ADD_DOCUMENTATION` | Add missing notes/comments | Yes | Low |
| `FIX_NAMING` | Correct naming convention violations | No | Medium |
| `FIX_SYNTAX` | Correct DBML syntax errors | No | High |

### **Common Fix Patterns**

#### **Missing Column Fix**
```json
{
  "fixType": "ADD_COLUMN",
  "confidence": 95,
  "autoApplicable": true,
  "fix": "email varchar(255) [not null, unique, note: 'User email address']",
  "targetLocation": "users table, after name column"
}
```

#### **Missing Index Fix**
```json
{
  "fixType": "ADD_INDEX",
  "confidence": 90,
  "autoApplicable": true,
  "fix": "email [name: 'idx_users_email']",
  "targetLocation": "users table Indexes block"
}
```

#### **Missing Relationship Fix**
```json
{
  "fixType": "ADD_RELATIONSHIP",
  "confidence": 85,
  "autoApplicable": false,
  "fix": "user_id bigint [not null, ref: > users.id]",
  "targetLocation": "orders table, after id column"
}
```

#### **Missing Enum Fix**
```json
{
  "fixType": "ADD_ENUM",
  "confidence": 100,
  "autoApplicable": true,
  "fix": "Enum user_status { ACTIVE SUSPENDED INACTIVE }",
  "targetLocation": "Before table definitions"
}
```

### **Enhanced Validation Algorithms**

#### **M01: Target Database Declaration**
```
Validation Algorithm:
1. Search for Project block: `grep -n "Project" dbml_file`
2. Extract database_type: `grep -A5 "Project" | grep "database_type"`
3. Validate against allowed values: ['PostgreSQL', 'Oracle']
4. Cross-reference with requirements specification

✅ PASS Criteria:
- Project block exists with database_type specified
- Database type is either 'PostgreSQL' or 'Oracle'
- Matches target database from requirements

❌ FAIL Examples:
- No Project block
- database_type missing or invalid
- Mismatch with requirements specification

Fix Template:
Project {database_type} {
  database_type: 'PostgreSQL'  // or 'Oracle'
  Note: 'Project description from requirements'
}
```

#### **M02: Entity Coverage**
```
Validation Algorithm:
1. Extract entity list from Requirements §3: `grep -E "Entity:|##\s+\w+" requirements.md`
2. Extract table list from DBML: `grep -E "^Table\s+\w+" model.dbml`
3. Map entity names to table names using snake_case conversion
4. Generate missing_entities list for unmatched entities
5. Validate core business entities are present

✅ PASS Criteria:
- Every entity from Requirements §3 exists as a table
- Table names follow naming conventions
- No missing core business entities

❌ FAIL Examples:
- Missing tables for entities specified in requirements
- Table names don't match entity names
- Core business entities omitted
```

#### **M03: JSON Field Mapping**
```
Enhanced Validation Algorithm:
1. Extract JSON samples: `grep -E '"[^"]+"\s*:' requirements.md`
2. Parse nested structure: handle dot notation (user.profile.name)
3. Map to columns: check table.{snake_case(field)} exists
4. Handle arrays: look for _json suffix or junction tables
5. Validate data types: string->varchar, number->numeric, boolean->boolean
6. Check JSON path indexing for frequently accessed fields

✅ PASS Criteria:
- Every field from JSON samples is represented
- Nested JSON objects properly handled (JSONB vs normalized)
- Array fields appropriately modeled
- Field data types match JSON sample types
- JSON path indexes for high-frequency access

JSON Path Analysis:
- Simple fields: preferences.theme -> preferences JSONB column
- Nested objects: user.address.city -> normalized to address table OR JSON path
- Arrays: user.tags[] -> tags_json JSONB OR junction table user_tags
- Frequently accessed: user.settings.notifications -> separate column
```

#### **Enhanced Performance Analysis**
```
Performance Validation Algorithm:
1. Extract query patterns from Requirements §7
2. Map patterns to required indexes
3. Validate index existence and optimization
4. Check composite index column ordering
5. Assess covering index opportunities
6. Evaluate maintenance overhead

Query Pattern Analysis:
- Single column filters: user_status, created_at
- Multi-column filters: (category_id, is_active)
- Range queries: (created_at, status) with proper ordering
- JOIN performance: all foreign keys indexed
- Sorting: ORDER BY columns have indexes

Index Recommendation Rules:
- WHERE clause columns: Individual or composite indexes
- JOIN columns: Always indexed (foreign keys)
- ORDER BY columns: Include in composite indexes
- GROUP BY columns: Consider covering indexes
- High cardinality first in composite indexes
```

---

## 📊 **Enhanced Scoring Methodology**

### **Score Calculation**
```
Base Score = (PassedMandatory × 2 + PassedRecommended × 1) / (TotalMandatory × 2 + TotalRecommended × 1) × 100

Confidence Adjustment = BaseScore × ConfidenceFactor
where ConfidenceFactor = (HighConfidenceChecks / TotalChecks)

Final Score = ConfidenceAdjustedScore

Grade Thresholds:
A (Excellent): ≥ 90% with high confidence (≥ 85%)
B (Good): 80-89% with medium confidence (≥ 70%)
C (Fair): 70-79% with acceptable confidence (≥ 60%)
D (Poor): < 70% or confidence < 60%
```

### **Enhanced Severity Levels**
```
CRITICAL: Failed mandatory check, blocks all deployment
ERROR: Failed mandatory check, blocks production deployment
WARNING: Failed recommended check, should fix before production
INFO: Suggestion for improvement, nice to have
CLARIFICATION: Requires stakeholder input to resolve
```

### **Confidence Scoring**
```
Confidence Factors:
- Requirement clarity: Clear (100), Ambiguous (70), Missing (30)
- Implementation complexity: Simple (100), Moderate (80), Complex (60)
- Business impact assessment: Clear (100), Uncertain (70), Unknown (40)
- Technical risk evaluation: Low (100), Medium (80), High (60)

Overall Confidence = Average of factor scores
```

---

## 🛠️ **Enhanced Remediation Format**

### **Enhanced Fix Template Structure**
```json
{
  "checkId": "M03",
  "severity": "ERROR",
  "fixType": "ADD_COLUMN",
  "confidence": {
    "score": 95,
    "factors": {
      "requirementClarity": 100,
      "implementationComplexity": 90,
      "businessImpact": 95,
      "technicalRisk": 95
    },
    "reasoning": "Clear requirement with straightforward implementation"
  },
  "requirementId": "BR-03",
  "requirementText": "User contact information must include phone number for 2FA",
  "table": "users",
  "line": 42,
  "targetLocation": "users table, after email column",
  "message": "Missing 'phone_number' column required by JSON sample",
  "fix": "phone_number varchar(20) [note: 'User phone number for 2FA']",
  "context": "JSON sample shows 'phoneNumber' field but no corresponding column found",
  "autoApplicable": true,
  "riskLevel": "LOW",
  "estimatedEffortMinutes": 2,
  "dependsOn": [],
  "impacts": ["entity_mapping", "api_contracts"],
  "testingRequirements": ["Validate phone number format", "Test 2FA integration"],
  "rollbackPlan": "Column can be safely dropped if needed"
}
```

### **Enhanced Fix Type Classification**

| Fix Type | Description | Auto-Applicable | Risk Level | Typical Effort |
|----------|-------------|-----------------|------------|----------------|
| `ADD_COLUMN` | Add missing column to table | Yes | Low | 2-5 min |
| `ADD_INDEX` | Add performance or constraint index | Yes | Low | 3-10 min |
| `ADD_RELATIONSHIP` | Add foreign key relationship | No | Medium | 15-30 min |
| `ADD_ENUM` | Define new enum type | Yes | Low | 5-10 min |
| `ADD_TABLE` | Create missing table | No | High | 30-60 min |
| `MODIFY_COLUMN` | Change column definition | No | High | 15-45 min |
| `MODIFY_CONSTRAINT` | Update constraint definition | No | Medium | 10-20 min |
| `ADD_PROJECT_BLOCK` | Add missing project configuration | Yes | Low | 1-2 min |
| `ADD_TABLE_GROUP` | Add logical table grouping | Yes | Low | 2-5 min |
| `ADD_DOCUMENTATION` | Add missing notes/comments | Yes | Low | 5-15 min |
| `FIX_NAMING` | Correct naming convention violations | No | Medium | 10-30 min |
| `FIX_SYNTAX` | Correct DBML syntax errors | No | High | 5-20 min |
| `OPTIMIZE_PERFORMANCE` | Add performance optimizations | Yes | Low | 5-15 min |
| `ENHANCE_SECURITY` | Add security documentation/features | Yes | Low | 10-20 min |
| `CLARIFY_REQUIREMENT` | Requires stakeholder clarification | No | N/A | Variable |

---

## 📤 **Enhanced Output Schema (Strict JSON)**

```json
{
  "reviewMetadata": {
    "reviewDate": "2025-01-22T10:30:00Z",
    "templateVersion": "1.1.0",
    "targetDatabase": "PostgreSQL",
    "applicationFramework": "Spring Boot JPA",
    "dbmlVersion": "1.0.0",
    "reviewDurationMs": 2847,
    "requirementsVersion": "1.2.0",
    "standardsVersion": "1.1.0",
    "availableSections": {
      "entityDefinitions": true,
      "relationshipSpecs": true,
      "performanceRequirements": true,
      "securityRequirements": false
    },
    "reviewScope": ["M01-M12", "R01-R08"],
    "excludedChecks": [],
    "parsingErrors": []
  },
  "overallScore": {
    "grade": "B",
    "percentage": 85,
    "passedMandatory": 11,
    "totalMandatory": 12,
    "passedRecommended": 6,
    "totalRecommended": 8,
    "confidenceScore": 92,
    "confidenceBreakdown": {
      "requirementClarity": 95,
      "implementationComplexity": 88,
      "businessImpact": 93,
      "technicalRisk": 91
    }
  },
  "categoryScores": {
    "entityCoverage": 100,
    "relationships": 90,
    "businessRules": 80,
    "performance": 75,
    "documentation": 85,
    "standards": 90,
    "security": 70,
    "jpaCompatibility": 95
  },
  "remediations": [
    {
      "checkId": "M03",
      "severity": "ERROR",
      "fixType": "ADD_COLUMN",
      "confidence": {
        "score": 95,
        "factors": {
          "requirementClarity": 100,
          "implementationComplexity": 90,
          "businessImpact": 95,
          "technicalRisk": 95
        },
        "reasoning": "Clear requirement with straightforward implementation"
      },
      "requirementId": "BR-03",
      "requirementText": "User contact information must include phone number for 2FA",
      "table": "users",
      "line": 15,
      "targetLocation": "users table, after email column",
      "message": "Missing 'phone_number' column required by JSON sample",
      "fix": "phone_number varchar(20) [note: 'User phone number for 2FA']",
      "context": "JSON sample includes phoneNumber but no corresponding column found",
      "autoApplicable": true,
      "riskLevel": "LOW",
      "estimatedEffortMinutes": 2,
      "dependsOn": [],
      "impacts": ["entity_mapping", "api_contracts"],
      "testingRequirements": ["Validate phone number format", "Test 2FA integration"],
      "rollbackPlan": "Column can be safely dropped if needed"
    }
  ],
  "remediationsByCategory": {
    "CRITICAL": [
      {
        "checkId": "M01",
        "severity": "ERROR",
        "fixType": "ADD_PROJECT_BLOCK",
        "confidence": {"score": 100},
        "message": "Missing Project block with database type declaration",
        "estimatedEffortMinutes": 1
      }
    ],
    "PERFORMANCE": [
      {
        "checkId": "R01",
        "severity": "WARNING",
        "fixType": "ADD_INDEX",
        "confidence": {"score": 88},
        "message": "Missing index for high-frequency query pattern",
        "estimatedEffortMinutes": 5
      }
    ],
    "DOCUMENTATION": [
      {
        "checkId": "M11",
        "severity": "ERROR",
        "fixType": "ADD_DOCUMENTATION",
        "confidence": {"score": 90},
        "message": "Missing table documentation for business context",
        "estimatedEffortMinutes": 10
      }
    ]
  },
  "remediationsByTable": {
    "users": [
      {
        "checkId": "M03",
        "fixType": "ADD_COLUMN",
        "message": "Missing phone_number column",
        "estimatedEffortMinutes": 2
      }
    ],
    "products": [
      {
        "checkId": "R01",
        "fixType": "ADD_INDEX",
        "message": "Missing performance index",
        "estimatedEffortMinutes": 5
      }
    ]
  },
  "automationSummary": {
    "totalRemediations": 12,
    "autoApplicable": 8,
    "manualReview": 4,
    "highConfidence": 10,
    "lowRisk": 8,
    "estimatedFixTimeMinutes": 45,
    "automationReadiness": "HIGH"
  },
  "missingEntities": [
    {
      "entityName": "UserProfile",
      "reason": "Specified in Requirements §3 but no corresponding table found",
      "suggestedTableName": "user_profiles",
      "requirementId": "BR-02",
      "priority": "HIGH",
      "estimatedEffortMinutes": 30
    }
  ],
  "jsonFieldMapping": [
    {
      "entity": "users",
      "jsonField": "preferences.theme",
      "mapping": "preferences JSONB column",
      "status": "MAPPED",
      "confidence": 95
    },
    {
      "entity": "users", 
      "jsonField": "phoneNumber",
      "mapping": "MISSING",
      "status": "ERROR",
      "requirementId": "BR-03",
      "suggestedFix": "ADD_COLUMN",
      "confidence": 95
    }
  ],
  "performanceAnalysis": {
    "indexCoverage": 75,
    "missingCriticalIndexes": 2,
    "queryPatternsCovered": 6,
    "totalQueryPatterns": 8,
    "estimatedQueryPerformanceImpact": "MEDIUM",
    "recommendedIndexes": [
      {
        "table": "products",
        "columns": ["category_id", "is_active"],
        "type": "COMPOSITE",
        "priority": "HIGH",
        "estimatedImpact": "30% query improvement"
      }
    ]
  },
  "complianceChecks": {
    "namingConventions": "PASS",
    "dbmlSyntax": "PASS",
    "businessRules": "PARTIAL",
    "securityDocumentation": "PASS",
    "requirementTraceability": "GOOD",
    "dataPrivacy": "NEEDS_REVIEW"
  },
  "requirementCoverage": {
    "totalRequirements": 15,
    "implementedRequirements": 12,
    "partiallyImplemented": 2,
    "notImplemented": 1,
    "coveragePercentage": 87,
    "missingRequirements": [
      {
        "requirementId": "BR-08",
        "description": "User activity logging",
        "priority": "MEDIUM"
      }
    ]
  },
  "securityAssessment": {
    "piiFieldsIdentified": 5,
    "encryptionRequirements": 3,
    "accessControlDocumented": true,
    "complianceGaps": ["GDPR data retention policy"],
    "riskLevel": "MEDIUM"
  },
  "additionalComments": [
    "Excellent use of partial indexes for soft delete pattern",
    "JSON storage decisions align well with decision matrix",
    "Consider adding composite index for reporting queries",
    "Table documentation is comprehensive and helpful"
  ],
  "recommendedNextSteps": [
    "Apply 8 auto-applicable fixes (estimated 25 minutes)",
    "Review 4 manual fixes requiring architectural decisions",
    "Add missing UserProfile entity to complete requirements coverage",
    "Implement remaining performance indexes before production deployment",
    "Address security documentation gaps for GDPR compliance"
  ],
  "qualityGates": {
    "readyForDevelopment": false,
    "readyForTesting": false,
    "readyForProduction": false,
    "blockingIssues": 3,
    "criticalIssues": 1,
    "gateDetails": {
      "development": {
        "passed": false,
        "blockers": ["Missing core entities", "Syntax errors"],
        "requirements": ["All mandatory checks pass", "No critical issues"]
      },
      "testing": {
        "passed": false,
        "blockers": ["Performance indexes missing"],
        "requirements": ["Performance optimized", "Documentation complete"]
      },
      "production": {
        "passed": false,
        "blockers": ["Security gaps", "Compliance issues"],
        "requirements": ["Security reviewed", "Compliance validated"]
      }
    }
  },
  "validationResults": {
    "syntaxValid": true,
    "requirementsTraceability": 87,
    "standardsCompliance": 90,
    "performanceOptimized": 75,
    "securityReviewed": 70,
    "documentationComplete": 85
  }
}
```

---

## 🚀 **Quick Reference Commands**

### **Syntax Validation**
```bash
# Validate DBML syntax
dbml2sql model.dbml --postgres > /dev/null

# Check for common issues
grep -n "bigserial" model.dbml  # Should not appear in business tables
grep -n "public_id" model.dbml  # Should appear in all business tables
```

### **Review Execution**
```bash
# Run comprehensive review
llm-review --model model.dbml --requirements requirements.md --standards standards.md --template review-template.md

# Quick syntax check
dbml2sql model.dbml --postgres --validate-only
```

---

## 📋 **Review Checklist Summary**

### **Before Starting Review**
- [ ] DBML model file is syntactically valid
- [ ] Requirements document is available
- [ ] Standards document is available
- [ ] Target database type is identified
- [ ] Application framework is known

### **During Review**
- [ ] Parse DBML completely (tables, relationships, indexes, enums)
- [ ] Map requirements entities to DBML tables
- [ ] Validate JSON field coverage
- [ ] Check relationship implementations
- [ ] Verify business rule enforcement
- [ ] Assess performance optimization
- [ ] Validate naming conventions
- [ ] Review documentation quality

### **After Review**
- [ ] Generate comprehensive JSON output
- [ ] Provide specific remediation actions
- [ ] Include line numbers for fixes
- [ ] Categorize issues by severity
- [ ] Suggest next steps for improvement

---

**Template Version:** 1.0.0  
**Last Updated:** 2025-01-22  
**Compatible with:** database_requirements.md v1.0.0, dbml-standards-guidelines.md v1.0.0cumenta
tion is comprehensive and helpful"
  ],
  "recommendedNextSteps": [
    "Apply 8 auto-applicable fixes (estimated 25 minutes)",
    "Review 4 manual fixes requiring architectural decisions",
    "Add missing UserProfile entity to complete requirements coverage",
    "Implement remaining performance indexes before production deployment",
    "Address security documentation gaps for GDPR compliance"
  ],
  "qualityGates": {
    "readyForDevelopment": false,
    "readyForTesting": false,
    "readyForProduction": false,
    "blockingIssues": 3,
    "criticalIssues": 1,
    "gateDetails": {
      "development": {
        "passed": false,
        "blockers": ["Missing core entities", "Syntax errors"],
        "requirements": ["All mandatory checks pass", "No critical issues"]
      },
      "testing": {
        "passed": false,
        "blockers": ["Performance indexes missing"],
        "requirements": ["Performance optimized", "Documentation complete"]
      },
      "production": {
        "passed": false,
        "blockers": ["Security gaps", "Compliance issues"],
        "requirements": ["Security reviewed", "Compliance validated"]
      }
    }
  },
  "validationResults": {
    "syntaxValid": true,
    "requirementsTraceability": 87,
    "standardsCompliance": 90,
    "performanceOptimized": 75,
    "securityReviewed": 70,
    "documentationComplete": 85
  }
}
```

---

## 🚀 **Quick Reference Commands**

### **Syntax Validation**
```bash
# Validate DBML syntax
dbml2sql model.dbml --postgres > /dev/null

# Check for common issues
grep -n "bigserial" model.dbml  # Should not appear in business tables
grep -n "public_id" model.dbml  # Should appear in all business tables
```

### **Review Execution**
```bash
# Run comprehensive review
llm-review --model model.dbml --requirements requirements.md --standards standards.md --template review-template.md

# Quick syntax check
dbml2sql model.dbml --postgres --validate-only
```

### **Automated Fix Application**
```bash
# Apply auto-applicable fixes
dbml-fixer --input model.dbml --fixes fixes.json --auto-only --output model_fixed.dbml

# Generate migration scripts
dbml2sql model_fixed.dbml --postgres --diff model.dbml
```

---

## 📋 **Review Checklist Summary**

### **Before Starting Review**
- [ ] DBML model file is syntactically valid
- [ ] Requirements document is available and complete
- [ ] Standards document is available
- [ ] Target database type is identified
- [ ] Application framework is known
- [ ] Review scope is defined

### **During Review**
- [ ] Parse DBML completely (tables, relationships, indexes, enums)
- [ ] Map requirements entities to DBML tables
- [ ] Validate JSON field coverage and mapping
- [ ] Check relationship implementations and cardinality
- [ ] Verify business rule enforcement through constraints
- [ ] Assess performance optimization (indexes, query patterns)
- [ ] Validate naming conventions and consistency
- [ ] Review documentation quality and completeness
- [ ] Check database-specific feature usage
- [ ] Evaluate security and compliance considerations

### **After Review**
- [ ] Generate comprehensive JSON output
- [ ] Provide specific remediation actions with confidence scores
- [ ] Include line numbers and target locations for fixes
- [ ] Categorize issues by severity and impact
- [ ] Suggest prioritized next steps for improvement
- [ ] Validate quality gates for deployment readiness

---

## 🔧 **Advanced Features**

### **Batch Review Processing**
```json
{
  "batchReview": {
    "models": [
      {"path": "user_service.dbml", "requirements": "user_requirements.md"},
      {"path": "order_service.dbml", "requirements": "order_requirements.md"}
    ],
    "consolidatedReport": true,
    "crossServiceValidation": true
  }
}
```

### **Custom Check Extensions**
```json
{
  "customChecks": [
    {
      "checkId": "C01",
      "name": "Company Naming Standards",
      "category": "CUSTOM",
      "weight": 1,
      "validator": "custom_validators.company_naming",
      "enabled": true
    }
  ]
}
```

### **Integration Hooks**
```json
{
  "integrations": {
    "jira": {
      "createTickets": true,
      "project": "DB-REVIEW",
      "issueType": "Task"
    },
    "slack": {
      "webhook": "https://hooks.slack.com/...",
      "channel": "#database-reviews"
    },
    "git": {
      "createPR": true,
      "branch": "dbml-review-fixes",
      "autoApplyFixes": true
    }
  }
}
```

---

## 📚 **Reference Materials**

### **DBML Specification**
- [DBML Language Reference](https://www.dbml.org/docs/)
- [Database Types Support](https://www.dbml.org/docs/#database-types)
- [Relationship Syntax](https://www.dbml.org/docs/#relationships-foreign-key-definitions)

### **Database Best Practices**
- [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [Oracle Database Design Guidelines](https://docs.oracle.com/en/database/oracle/oracle-database/)
- [Index Design Patterns](https://use-the-index-luke.com/)

### **Security and Compliance**
- [GDPR Data Protection Guidelines](https://gdpr.eu/data-protection/)
- [Database Security Best Practices](https://owasp.org/www-project-database-security/)
- [PCI DSS Database Requirements](https://www.pcisecuritystandards.org/)

---

## 🏷️ **Version History**

### **v1.1.0** (2025-01-22)
- Enhanced confidence scoring with detailed factor analysis
- Added comprehensive error handling guidelines
- Improved JSON field mapping validation algorithms
- Extended performance analysis with query pattern recognition
- Added security assessment and compliance checking
- Enhanced remediation format with effort estimation
- Added quality gates for deployment readiness
- Improved automation summary with readiness indicators

### **v1.0.0** (2025-01-15)
- Initial template with basic review framework
- Core mandatory and recommended checks
- Basic JSON output schema
- Standard remediation patterns

---

**Template Version:** 1.1.0  
**Last Updated:** 2025-01-22  
**Compatible with:** database_requirements.md v1.2.0, dbml-standards-guidelines.md v1.1.0  
**Minimum DBML Version:** 1.0.0  
**Supported Databases:** PostgreSQL 12+, Oracle 19c+