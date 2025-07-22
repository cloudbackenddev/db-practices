# Database Requirements - Designer Template

**For Designers:** Fill out the business requirements below. Technical standards and implementation details are handled separately.

---

## 1. Project Information

| Field | Your Answer |
|-------|-------------|
| **Project Name** | `[Enter your project name]` |
| **Database Type** | `[PostgreSQL or Oracle]` |
| **Application Framework** | `[Spring Boot JPA / Other ORM / Direct SQL]` |
| **Business Purpose** | `[2-3 sentences: What does this system do? Who uses it?]` |

---

## 2. What are your main business entities?

List the main "things" your system needs to track:

```
Examples:
- Users/Customers
- Products
- Orders
- Payments
- Categories
- Reviews

Your entities:
- [Entity 1]
- [Entity 2] 
- [Entity 3]
- [Add more as needed]
```

---

## 3. How do these entities relate to each other?

Describe the relationships in simple terms:

```
Examples:
- Users can place multiple Orders
- Orders contain multiple Products
- Products belong to Categories

Your relationships:
- [Entity A] can have [one/many] [Entity B]
- [Entity C] belongs to [Entity D]
- [Add more relationships]
```

---

## 4. Entity Details

For each main entity, provide:

### Entity 1: [Name]

**What is this?** `[Brief description of what this entity represents]`

**Sample data:** (Choose the format that matches your data)

**Option A - Simple structured data:**
```
- name: "John Doe"
- email: "john@example.com"
- phone: "+1-555-123-4567"
- status: "ACTIVE"
- registration_date: "2024-01-15"
```

**Option B - Complex/flexible data (use JSON format):**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "preferences": {
    "newsletter": true,
    "theme": "dark"
  },
  "address": {
    "street": "123 Main St",
    "city": "New York"
  }
}
```

**Business rules:**
```
- Email must be unique
- Name is required
- Status can be: ACTIVE, SUSPENDED, INACTIVE
- [Add other rules]
```

---

### Entity 2: [Name]

**What is this?** `[Brief description]`

**Sample data:**
```
[Provide sample data in the format that makes sense for your entity]
```

**Business rules:**
```
[List the important rules for this entity]
```

---

### Entity 3: [Name]

**What is this?** `[Brief description]`

**Sample data:**
```
[Provide sample data]
```

**Business rules:**
```
[List the important rules]
```

---

## 5. Important Business Rules

List any important constraints or rules:

```
Examples:
- Users cannot place orders if their account is suspended
- Product prices must be positive numbers
- Orders cannot be deleted, only cancelled
- Stock quantity cannot go below zero

Your rules:
- [Rule 1]
- [Rule 2]
- [Rule 3]
```

---

## 6. How will you access/search this data?

**This is important for database performance!** Describe how users will search and access data:

### Common Access Patterns

**For [Entity 1 - e.g., Users]:**
```
How will you find/search users?
- By email address (for login) - [High frequency - happens every login]
- By user ID (for profile pages) - [High frequency]
- By name (for admin search) - [Medium frequency]
- By registration date (for reports) - [Low frequency]
- By status (active/inactive users) - [Medium frequency]
```

**For [Entity 2 - e.g., Products]:**
```
How will you find/search products?
- By product name (customer search) - [High frequency]
- By category (browse by category) - [High frequency] 
- By price range (filter by price) - [Medium frequency]
- By availability/stock status - [High frequency]
- By creation date (newest products) - [Medium frequency]
```

**For [Entity 3 - e.g., Orders]:**
```
How will you find/search orders?
- By user (user's order history) - [High frequency]
- By order date (recent orders) - [High frequency]
- By status (pending, completed orders) - [High frequency]
- By order number (customer service lookup) - [Medium frequency]
```

### Cross-Entity Searches (Joins)
```
Common combined searches:
- User's orders with order details - [High frequency]
- Products in a category with stock info - [High frequency]
- Order items with product details - [High frequency]
- User's reviews for products - [Medium frequency]
```

### Reporting Needs
```
What reports do you need?
- Daily/monthly sales totals - [Low frequency, but important]
- Top selling products - [Medium frequency]
- User activity reports - [Low frequency]
- Inventory reports - [Medium frequency]
```

**Frequency Guide:**
- **High**: Multiple times per minute (needs fast indexes)
- **Medium**: Few times per hour (needs good indexes) 
- **Low**: Few times per day (basic indexes okay)

---

## 7. Performance Expectations (Optional)

Only fill this out if you have specific requirements:

```
Expected data volume:
- How many [Entity 1] records? [e.g., 10,000 users]
- How many [Entity 2] records? [e.g., 100,000 products]
- How fast should searches be? [e.g., under 1 second]

Growth expectations:
- How many new records per month? [e.g., 1,000 new users/month]
```

---

## 8. Security/Compliance (Optional)

Only fill this out if you have specific requirements:

```
- Do you need to comply with GDPR? [Yes/No]
- Do you handle payment card data? [Yes/No]
- Do you need audit trails (who changed what when)? [Yes/No]
- Do you need soft delete (hide instead of permanently delete)? [Yes/No]
```

---

**That's it!** 

The technical implementation details (indexes, data types, naming conventions, JSON storage strategies, etc.) will be handled automatically using the standards defined in `database_standards-guidelines.md`.

**Usage Instructions:**
1. Designer fills out this requirements template
2. LLM references both this requirements file AND the standards/guidelines file
3. Result: Production-ready DBML that follows best practices