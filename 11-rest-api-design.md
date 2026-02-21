# Complete REST API Design

_Comprehensive In-Depth Study Guide_

Source video: "11. Complete REST API Design" – Sriniously

---

## Table of Contents

1. History & Context: From World Wide Web to REST
2. Why REST Architecture Matters
3. Understanding REST: Representational State Transfer
4. REST Constraints by Roy Fielding
5. URL Structure & Best Practices
6. Resources: Identifying & Designing Them
7. HTTP Methods & Idempotency
8. CRUD Operations & HTTP Methods
9. Non-CRUD Operations & Custom Actions
10. HTTP Status Codes
11. Request & Response Design
12. API Design Workflow
13. Resource Hierarchies & Relationships
14. Versioning & API Evolution
15. Best Practices & Design Patterns
16. Common API Design Mistakes
17. References & Resources

---

## 1. History & Context: From World Wide Web to REST

### 1990: Tim Berners-Lee Invents the Web

In 1990, Tim Berners-Lee started a project called the **World Wide Web** to share knowledge globally.

**Within one year, he invented:**

```
1. URI (Uniform Resource Identifier)
   - Way to identify resources on the internet
   - Foundation of all URLs

2. HTTP (HyperText Transfer Protocol)
   - Protocol for client-server communication
   - Foundation of web communication

3. HTML (HyperText Markup Language)
   - Markup language for web pages
   - Structure and semantics

4. First Web Server
   - Software to serve content

5. First Web Browser
   - Software to consume content

6. First WYSIWYG HTML Editor
   - Built into the browser

All these technologies are still used today (in evolved forms)
```

### The Scalability Crisis (1993+)

**The Problem:**

As the web grew exponentially, it started to break.

```
Users grew exponentially:
  Thousands → Millions → Billions of users

But infrastructure was designed for:
  Hundreds → Maybe thousands

The web was heading towards breakdown
Technology, architecture, and standards couldn't keep up
```

### Roy Fielding's Solution (1993+)

**Roy Fielding** (co-founder of Apache HTTP Server) became concerned about scalability.

He proposed **architectural constraints** to solve the scalability problem:

```
These constraints became the foundation of REST architecture
Later formalized in his 2000 PhD dissertation
Named: Representational State Transfer (REST)
```

---

## 2. Why REST Architecture Matters

### The Goal

Make the web scalable, reliable, and maintainable.

### The Benefit

By following REST constraints, systems can:

```
✓ Scale to millions of users
✓ Allow independent evolution of components
✓ Improve security and reliability
✓ Work across different protocols and devices
✓ Enable caching and performance optimization
✓ Provide consistent interfaces
```

### Still Relevant Today

Even in 2025, REST architecture is:

```
✓ The industry standard for APIs
✓ Used by virtually all major tech companies
✓ Basis for best practices in API design
✓ Foundation of HTTP-based APIs
```

---

## 3. Understanding REST: Representational State Transfer

### Breaking Down the Name

REST has three components:

#### Component 1: Representational

**Meaning:** Resources have different representations based on context.

```
Same resource, different representations:

User resource:
  As JSON (for APIs):
    {
      "id": 123,
      "name": "Alice",
      "email": "alice@example.com"
    }

  As HTML (for browsers):
    <div class="user">
      <h1>Alice</h1>
      <p>alice@example.com</p>
    </div>

  As XML (for legacy systems):
    <user>
      <id>123</id>
      <name>Alice</name>
      <email>alice@example.com</email>
    </user>

Same data, different representations for different clients
```

**Why it matters:**

```
Server can serve same resource in multiple formats
Client specifies what format it needs
Flexibility for different client types
```

#### Component 2: State

**Meaning:** The current condition/attributes of a resource.

```
Shopping Cart State:
  Items: [{ product: "Book", qty: 2, price: 20 }]
  Total: $40
  Created: 2025-01-06
  Status: "active"

User State:
  Name: "Alice"
  Email: "alice@example.com"
  Status: "active"
  Last Login: 2025-01-06T09:15:00Z

Book State:
  Title: "Clean Code"
  Author: "Robert C. Martin"
  ISBN: "0132350882"
  Available: true

State = current condition of the resource
```

**Transfer of State:**

State moves between client and server through API calls.

```
Client sends state to server:
  POST /users
  Body: { "name": "Alice", "email": "alice@example.com" }

Server returns state to client:
  GET /users/123
  Response: { "id": 123, "name": "Alice", "email": "alice@example.com" }

State flows in both directions
```

#### Component 3: Transfer

**Meaning:** Movement of resource representations between client and server.

```
Transfer mechanisms:

Protocol: HTTP (HyperText Transfer Protocol)
  - Standard protocol for web communication

Methods: GET, POST, PUT, PATCH, DELETE
  - Ways to request state transfer

Format: JSON, XML, HTML
  - How state is serialized

Example transfer:
  Client: GET https://api.example.com/books/123
            (Request representation from server)

  Server: HTTP 200 OK
          Content-Type: application/json
          { "id": 123, "title": "Clean Code", ... }
          (Transfer representation to client)
```

### Complete REST Definition

```
REST = Representational State Transfer

Means:

1. Resources are represented in different formats
   (JSON, XML, HTML, etc.)

2. Resources have state
   (current condition/attributes)

3. State is transferred between client and server
   (through HTTP methods)

4. Within architectural constraints
   (Client-Server, Stateless, Cacheable, etc.)
```

---

## 4. REST Constraints by Roy Fielding

Roy Fielding proposed **6 architectural constraints** for REST:

### Constraint 1: Client-Server

**Principle:** Separation of concerns between client and server.

```
Client Responsibilities:
  ✓ User interface
  ✓ User experience
  ✓ User interaction handling

Server Responsibilities:
  ✓ Data storage
  ✓ Business logic
  ✓ Data processing

Benefits:
  ✓ Each can evolve independently
  ✓ Client updates don't require server changes
  ✓ Server improvements don't affect clients
  ✓ Better scalability
  ✓ Cleaner separation
```

### Constraint 2: Uniform Interface

**Principle:** Standardized way for components to communicate.

**Includes 4 sub-constraints:**

```
1. Resource Identification
   - Each resource has a unique identifier
   - Example: /users/123 identifies user with ID 123

2. Resource Manipulation Through Representation
   - Client manipulates resources through representations
   - Send full or partial representation to update

3. Self-Descriptive Messages
   - Each message contains all info needed to understand it
   - Headers describe format, caching, authentication
   - No separate documentation needed from message itself

4. HATEOAS (Hypermedia As The Engine of Application State)
   - Responses include links to related resources
   - Client navigates through links
   - Server drives state transitions
   - Example: Response includes "next", "prev" links
```

**Benefits:**

```
✓ Consistent interface across all services
✓ Easy for clients to integrate
✓ Easier to maintain and document
✓ Predictable behavior
```

### Constraint 3: Layered System

**Principle:** Architecture composed of hierarchical layers.

```
Layer Structure:

Client Layer
  ↓ (request)
Load Balancer Layer
  ↓ (distributes to one of many)
API Server Layer
  ↓ (processes request)
Cache Layer
  ↓ (may serve from cache or fetch from)
Database Layer
  ↓ (data retrieval)

Each layer:
  ✓ Can only see immediate layer below
  ✓ Doesn't know about layers beyond
  ✓ Can be added or modified independently

Benefits:
  ✓ Better scalability (add more servers)
  ✓ Better security (firewall, proxies)
  ✓ Easier to maintain and update
  ✓ Can add caching, compression, etc.
```

**Real-World Example:**

```
When you access google.com:

1. Client (browser)
2. CDN (caches content globally)
3. Load Balancer (directs to nearest server)
4. API Server (processes request)
5. Cache (Redis, Memcached)
6. Database (persists data)

Each layer has specific role
Multiple layers between client and database
Client doesn't care about internals
```

### Constraint 4: Cacheable

**Principle:** Responses must be explicitly marked as cacheable or non-cacheable.

```
Cacheable Response:
  HTTP/1.1 200 OK
  Cache-Control: public, max-age=3600
  Content-Type: application/json

  { "book": { "id": 1, "title": "Clean Code" } }

  Meaning: This response can be cached for 1 hour
           Any client can cache this

Non-Cacheable Response:
  HTTP/1.1 200 OK
  Cache-Control: private, no-cache
  Content-Type: application/json

  { "user": { "id": 123, "email": "alice@example.com" } }

  Meaning: This response should NOT be cached
           Each request requires fresh data

Benefits:
  ✓ Reduces server load
  ✓ Improves network efficiency
  ✓ Faster response times for clients
  ✓ Reduced bandwidth usage
```

### Constraint 5: Stateless

**Principle:** Server doesn't store client context between requests.

```
Stateless Requirement:

Each request must contain ALL information needed:
  - User identity (token, credentials)
  - Request parameters
  - Request data

Server doesn't remember:
  - Previous requests
  - Client's logged-in status
  - Client's shopping cart
  - Client's preferences

Example:

Stateful (Bad):
  Request 1: Client logs in
  Server: "OK, you're logged in. Remember session #123"

  Request 2: Client: "Get my profile"
  Server: "What profile? I don't know who you are"
  Problem: Lost context!

Stateless (Good):
  Request 1: Client logs in with credentials
  Server: "OK, here's your token: abc123def456"

  Request 2: Client: "Get my profile. Token: abc123def456"
  Server: "Let me verify token... OK, here's your profile"

Each request is independent, complete

Benefits:
  ✓ Any server can handle any request
  ✓ Better scalability (no session affinity needed)
  ✓ Better reliability (lost server doesn't lose state)
  ✓ Easier to cache
  ✓ Better performance
```

### Constraint 6: Code on Demand (Optional)

**Principle:** Server can send executable code to client.

```
Examples:

JavaScript:
  Server sends JavaScript code to browser
  Browser executes it
  Extends client functionality

Applets:
  Server sends compiled code
  Client runs it

Benefits:
  ✓ Reduce initial payload
  ✓ Update client behavior without deploying new version
  ✓ Simplify client implementation

Note: Optional constraint
      Not required for REST
      Used sparingly in modern systems
      Most REST APIs don't use this
```

### Summary: The 6 Constraints

| Constraint        | Purpose                    | Benefit                  |
| ----------------- | -------------------------- | ------------------------ |
| Client-Server     | Separation of concerns     | Independent evolution    |
| Uniform Interface | Standardized communication | Consistency              |
| Layered System    | Hierarchical architecture  | Scalability              |
| Cacheable         | Mark responses explicitly  | Performance              |
| Stateless         | No server-side context     | Scalability, reliability |
| Code on Demand    | Send executable code       | Flexibility (optional)   |

---

## 5. URL Structure & Best Practices

### Typical URL Structure

```
https://api.example.com/v1/books/123?page=1&limit=10#section

├─ Scheme: https
│  (secure HTTP protocol)
│
├─ Authority/Domain: api.example.com
│  └─ Subdomain: api
│     (common convention for APIs)
│
├─ Version: /v1
│  (API version - optional but recommended)
│
├─ Resource Path: /books/123
│  ├─ Resource Collection: books (plural)
│  └─ Resource ID: 123 (specific resource)
│
├─ Query Parameters: ?page=1&limit=10
│  (filters, pagination, sorting)
│
└─ Fragment: #section
   (browser navigation - rarely used in APIs)
```

### URL Best Practices

#### Rule 1: Use Plural Nouns for Collections

```
✓ CORRECT:
  GET /books          (fetch all books)
  GET /books/123      (fetch single book - still plural)
  GET /users          (fetch all users)
  GET /users/456      (fetch single user - still plural)

✗ INCORRECT:
  GET /book           (singular collection is confusing)
  GET /book/123       (singular + ID is inconsistent)
  GET /user           (singular)

Reasoning:
  - Resources are collections of entities
  - Even single resource is part of collection
  - Plural form is more intuitive
  - Industry standard
```

#### Rule 2: Use Hierarchical Paths for Relationships

```
One-to-many relationships:

GET /organizations/789/projects
  (All projects in organization 789)

GET /organizations/789/projects/456
  (Specific project in specific organization)

GET /organizations/789/projects/456/tasks
  (All tasks in project 456 of organization 789)

Structure shows relationship:
  organizations → projects → tasks
  Parent → Child → Grandchild
```

#### Rule 3: Use Lowercase & Hyphens for Multi-Word Slugs

```
✓ CORRECT:
  /api/user-profiles
  /api/payment-methods
  /api/access-control-lists

✗ INCORRECT:
  /api/UserProfiles         (camelCase)
  /api/user_profiles        (underscores)
  /api/User Profiles        (spaces)

Reasoning:
  - URLs travel through various systems
  - Case sensitivity causes issues
  - Underscores are rarely used in URLs
  - Hyphens are standard URL convention
  - Lowercase is standard across browsers and systems
```

#### Rule 4: No File Extensions in REST APIs

```
✓ CORRECT:
  GET /books/123
  Content-Type: application/json

✗ INCORRECT:
  GET /books/123.json
  GET /books/123.xml

Reasoning:
  - Content type specified in header
  - Extensions in URL are outdated (MVC-era)
  - Same resource, multiple representations
  - Clean, consistent URLs
```

#### Rule 5: Use Query Parameters for Filtering & Pagination

```
✓ CORRECT:
  GET /books?category=fiction&sort=title&limit=10&page=2
  (Filters and pagination in query params)

✗ INCORRECT:
  GET /books/fiction/sort-title/limit-10/page-2
  (These belong in path hierarchy)

Correct use:

GET /books
  (All books - default)

GET /books?category=fiction
  (Filter by category)

GET /books?category=fiction&author=tolkien
  (Multiple filters)

GET /books?sort=title&order=asc
  (Sorting)

GET /books?limit=20&page=2
  (Pagination)

GET /books?fields=id,title,author
  (Sparse fieldsets - which fields to return)
```

#### Rule 6: Use Slugs for Human-Readable Identifiers

```
Instead of IDs:
  GET /books/the-hobbit
  (Slug: human-readable, easy to share)

vs

  GET /books/123
  (ID: unique, technical)

Convert slug to database query:
  "the-hobbit" → lowercase
              → replace hyphens with spaces (for search)
              → query database for title

Or use both:
  GET /books/the-hobbit     (user-friendly)
  GET /books/123            (programmatic)
  Both resolve to same book
```

---

## 6. Resources: Identifying & Designing Them

### What is a Resource?

A **resource** is any noun that you can identify in your application domain.

```
Resources are entities that have:
  ✓ Unique identity
  ✓ Properties/attributes
  ✓ State
  ✓ Relationships to other resources
```

### Finding Resources

Start with domain analysis:

#### Example 1: E-Commerce Platform

```
Analyze requirements:
  - Users browse products
  - Users add items to cart
  - Users checkout and order
  - Users can review products
  - Admins manage inventory
  - Admins manage categories

Nouns (potential resources):
  ✓ Users
  ✓ Products
  ✓ Cart (or CartItems)
  ✓ Orders
  ✓ OrderItems
  ✓ Reviews
  ✓ Inventory
  ✓ Categories

All of these become resources in API
```

#### Example 2: Project Management Platform

```
Analyze requirements:
  - Users join organizations
  - Organizations have projects
  - Projects have tasks
  - Tasks can be assigned to users
  - Tasks can have subtasks
  - Teams can be formed
  - Comments on tasks

Nouns (potential resources):
  ✓ Organizations
  ✓ Projects
  ✓ Tasks
  ✓ Subtasks
  ✓ Teams
  ✓ Users
  ✓ Comments
  ✓ Assignments

All of these become resources in API
```

### Resource Characteristics

Good resources have:

```
1. Unique Identity
   - Every resource needs unique identifier
   - ID, slug, UUID, etc.

2. Properties/Attributes
   - Name, description, status, etc.
   - Describe what the resource is

3. Relationships
   - Belongs to another resource
   - Has many other resources
   - Relationships are important

4. Actionable Operations
   - Can be created, read, updated, deleted
   - Can perform actions
   - Has meaningful operations

5. Stateful
   - Resource has state (current condition)
   - State can change
   - State is important to track
```

### Resource vs Data Model

```
Resource (API perspective):
  What the API exposes to clients
  May include computed fields
  May combine multiple database tables
  May exclude sensitive fields
  Example:
    {
      "id": 123,
      "name": "Harry Potter",
      "author": "J.K. Rowling",
      "rating": 4.8,              (computed from reviews)
      "reviewCount": 1500         (computed, not stored)
    }

Data Model (Database perspective):
  How data is stored
  Technical representation
  Normalized for efficiency
  Example:
    books table:
      - id
      - title
      - author_id
      - isbn
      - published_date

    reviews table:
      - id
      - book_id
      - user_id
      - rating
      - comment

They're related but not identical
```

---

## 7. HTTP Methods & Idempotency

### Key Concept: Idempotency

**Idempotency** = Performing the same action multiple times has same effect as performing it once.

```
Mathematical example:
  abs(-5) = 5
  abs(abs(-5)) = abs(5) = 5
  abs(abs(abs(-5))) = 5

  No matter how many times you apply abs(), result is same
  That's idempotency

API example:
  DELETE /users/123

  First call: User deleted, server responds 204 No Content
  Second call: User doesn't exist, server responds 404 Not Found

  Side effect from first call (deletion) doesn't happen again
  State doesn't change with subsequent calls
  That's idempotent
```

### The Five Main HTTP Methods

#### Method 1: GET (Fetch/Retrieve)

**Purpose:** Retrieve data without changing server state.

```
Characteristics:
  ✓ Idempotent (yes)
  ✓ Cacheable (yes)
  ✓ Safe (yes - doesn't modify data)
  ✓ Has request body? (usually no)
  ✓ Has response body? (yes)

Use for:
  - Fetch single resource
  - Fetch collection of resources
  - Fetch with filtering/pagination/sorting

HTTP Status Codes:
  200 OK              (successful fetch)
  404 Not Found       (resource doesn't exist)
  401 Unauthorized    (requires authentication)
  403 Forbidden       (no permission)

Examples:
  GET /books
  (Fetch all books)

  GET /books/123
  (Fetch book with ID 123)

  GET /books?category=fiction&limit=10
  (Fetch books filtered by category)

  GET /books/123/reviews
  (Fetch reviews for book 123)

Why idempotent?
  No matter how many times you call:
  GET /books/123

  You get same book, same data
  Server state doesn't change
  No side effects from multiple calls
```

#### Method 2: POST (Create)

**Purpose:** Create new resource or trigger non-idempotent action.

```
Characteristics:
  ✓ Idempotent? (NO - creates new resource each time)
  ✓ Cacheable (no)
  ✓ Safe (no - modifies data)
  ✓ Has request body? (yes)
  ✓ Has response body? (yes)

Use for:
  - Create new resource
  - Non-CRUD operations
  - Custom actions

HTTP Status Codes:
  201 Created         (resource created successfully)
  400 Bad Request     (invalid data)
  409 Conflict        (resource already exists)
  401 Unauthorized    (requires authentication)
  403 Forbidden       (no permission)

Examples:
  POST /books
  Body: { "title": "New Book", "author": "Author Name" }
  (Create new book)

  POST /books/123/reviews
  Body: { "rating": 5, "comment": "Great book!" }
  (Create review for book)

  POST /users/123/send-email
  Body: { "subject": "Hello", "body": "Message" }
  (Custom action: send email)

Why NOT idempotent?
  First call:
    POST /books
    Body: { "title": "Clean Code" }
    Response: 201 Created, id: 1

  Second call (same data):
    POST /books
    Body: { "title": "Clean Code" }
    Response: 201 Created, id: 2
    (Different result! New book created with different ID)

  Each call creates new resource
  Multiple calls = multiple resources
  Not idempotent
```

#### Method 3: PUT (Replace)

**Purpose:** Replace entire resource representation.

```
Characteristics:
  ✓ Idempotent (yes)
  ✓ Cacheable (no)
  ✓ Safe (no - modifies data)
  ✓ Has request body? (yes)
  ✓ Has response body? (usually yes)

Use for:
  - Replace entire resource
  - Update all fields

HTTP Status Codes:
  200 OK              (successfully updated)
  201 Created         (resource created if didn't exist)
  400 Bad Request     (invalid data)
  404 Not Found       (resource doesn't exist)
  401 Unauthorized    (requires authentication)
  403 Forbidden       (no permission)

Examples:
  PUT /books/123
  Body: {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2008
  }
  (Replace entire book resource)

Important: Must include ALL fields
  If you don't include a field, it might be deleted
  Some servers might reject partial updates

Why idempotent?
  First call:
    PUT /books/123
    Body: { "id": 123, "title": "New Title", "author": "New Author" }
    Response: 200 OK
    (Book updated to: id=123, title="New Title", author="New Author")

  Second call (same data):
    PUT /books/123
    Body: { "id": 123, "title": "New Title", "author": "New Author" }
    Response: 200 OK
    (Book is already: id=123, title="New Title", author="New Author")
    (State doesn't change, same result)

  Multiple calls = same end state
  Idempotent
```

#### Method 4: PATCH (Partial Update)

**Purpose:** Update specific fields only.

```
Characteristics:
  ✓ Idempotent (yes)
  ✓ Cacheable (no)
  ✓ Safe (no - modifies data)
  ✓ Has request body? (yes)
  ✓ Has response body? (usually yes)

Use for:
  - Update specific fields
  - Partial updates

HTTP Status Codes:
  200 OK              (successfully updated)
  400 Bad Request     (invalid data)
  404 Not Found       (resource doesn't exist)
  401 Unauthorized    (requires authentication)
  403 Forbidden       (no permission)

Examples:
  PATCH /books/123
  Body: { "title": "Updated Title" }
  (Update only title, keep other fields)

  PATCH /users/456
  Body: { "email": "newemail@example.com" }
  (Update only email)

Important: Only changed fields needed
  Don't need to send all fields
  Server keeps unchanged fields
  More flexible than PUT

Why idempotent?
  First call:
    PATCH /books/123
    Body: { "title": "New Title" }
    Server updates book: title="New Title"
    Other fields unchanged
    Response: 200 OK

  Second call (same data):
    PATCH /books/123
    Body: { "title": "New Title" }
    Title is already "New Title"
    No change needed
    Response: 200 OK
    (Same end state)

  Multiple calls = same end state
  Idempotent

PUT vs PATCH:

PUT:
  PATCH /users/123
  Body: { "name": "Alice" }
  (If API requires all fields)
  (Might need: {"name": "Alice", "email": "...", "age": ...})

PATCH:
  PATCH /users/123
  Body: { "name": "Alice" }
  (Only field to update)
  (Other fields preserved automatically)
```

#### Method 5: DELETE (Remove)

**Purpose:** Delete/remove a resource.

```
Characteristics:
  ✓ Idempotent (yes)
  ✓ Cacheable (no)
  ✓ Safe (no - modifies data)
  ✓ Has request body? (no)
  ✓ Has response body? (optional)

Use for:
  - Delete resource
  - Remove resource permanently

HTTP Status Codes:
  200 OK              (deleted, response includes deleted object)
  204 No Content      (deleted, no response body)
  404 Not Found       (resource doesn't exist)
  401 Unauthorized    (requires authentication)
  403 Forbidden       (no permission)

Examples:
  DELETE /books/123
  (Delete book with ID 123)
  Response: 204 No Content

  DELETE /users/456/avatar
  (Delete user's avatar)
  Response: 204 No Content

Why idempotent?
  First call:
    DELETE /books/123
    (Book deleted)
    Response: 204 No Content

  Second call:
    DELETE /books/123
    (Book already deleted, doesn't exist)
    Response: 404 Not Found

  Side effect (deletion) only happens once
  Subsequent calls don't cause additional changes
  Idempotent (no additional side effects)
```

### HTTP Methods Summary

| Method     | Purpose        | Idempotent | Cacheable | Safe | Has Body | Status Code |
| ---------- | -------------- | ---------- | --------- | ---- | -------- | ----------- |
| **GET**    | Fetch data     | Yes        | Yes       | Yes  | No       | 200, 404    |
| **POST**   | Create/Action  | No         | No        | No   | Yes      | 201, 400    |
| **PUT**    | Replace        | Yes        | No        | No   | Yes      | 200, 201    |
| **PATCH**  | Partial Update | Yes        | No        | No   | Yes      | 200, 400    |
| **DELETE** | Remove         | Yes        | No        | No   | No       | 204, 404    |

---

## 8. CRUD Operations & HTTP Methods

### CRUD Operations Mapping

**CRUD** = Create, Read, Update, Delete

```
C - Create → POST
R - Read → GET
U - Update → PUT or PATCH
D - Delete → DELETE

Examples:

CREATE (POST):
  POST /books
  Creates new book
  Returns 201 Created

READ (GET):
  GET /books          (fetch all)
  GET /books/123      (fetch one)
  Returns 200 OK

UPDATE (PUT/PATCH):
  PUT /books/123      (replace entire)
  PATCH /books/123    (partial update)
  Returns 200 OK

DELETE (DELETE):
  DELETE /books/123
  Removes resource
  Returns 204 No Content
```

### Complete Example: Books API

```
Create a book:
  POST /books
  Content-Type: application/json
  {
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2008
  }

  Response: 201 Created
  Location: /books/123
  {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2008
  }

Read all books:
  GET /books

  Response: 200 OK
  {
    "data": [
      { "id": 123, "title": "Clean Code", ... },
      { "id": 124, "title": "Code Complete", ... },
      { ... }
    ]
  }

Read single book:
  GET /books/123

  Response: 200 OK
  {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2008
  }

Update (replace) book:
  PUT /books/123
  Content-Type: application/json
  {
    "title": "Clean Code 2nd Edition",
    "author": "Robert C. Martin",
    "year": 2024
  }

  Response: 200 OK
  {
    "id": 123,
    "title": "Clean Code 2nd Edition",
    "author": "Robert C. Martin",
    "year": 2024
  }

Update (partial) book:
  PATCH /books/123
  Content-Type: application/json
  {
    "year": 2009
  }

  Response: 200 OK
  {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2009
  }

Delete book:
  DELETE /books/123

  Response: 204 No Content
  (No response body)
```

---

## 9. Non-CRUD Operations & Custom Actions

### What Are Custom Actions?

**Custom actions** are operations that don't fit into standard CRUD.

```
Examples:

Sending an email
Resending a verification code
Processing a payment
Generating a report
Publishing a draft
Archiving data
Merging resources
Duplicating a resource

These aren't Create, Read, Update, or Delete
They're custom actions specific to your domain
```

### How to Design Custom Actions

#### Option 1: POST to Special Endpoint

**Best approach:** Use POST as a general action trigger.

```
POST /emails/send
Body: { "to": "user@example.com", "subject": "...", "body": "..." }
Response: 201 Created or 200 OK

POST /users/123/send-reset-password-email
Body: {}
Response: 200 OK

POST /orders/456/pay
Body: { "paymentMethod": "credit_card", ... }
Response: 200 OK

POST /reports/generate
Body: { "type": "monthly", "format": "pdf" }
Response: 201 Created (report created)

POST /documents/789/publish
Body: {}
Response: 200 OK

Reasoning:
  - POST is open-ended
  - Handles non-CRUD operations
  - Clear intent in URL
  - Follows REST conventions
```

#### Option 2: Use Action Subresource

```
POST /users/123/actions/send-reset-email
Body: {}
Response: 200 OK

POST /books/456/actions/generate-citation
Body: { "style": "APA" }
Response: 200 OK

POST /orders/789/actions/retry-payment
Body: {}
Response: 200 OK

Reasoning:
  - /actions subresource makes it clear
  - Distinguishes from standard CRUD
  - Explicit about being an action
```

#### Option 3: Use Verbs (Less Preferred)

```
POST /users/123/send-email
POST /orders/456/pay
DELETE /users/123/avatar

Reasoning:
  - Uses verbs in URL (against REST principle)
  - But pragmatic for clarity
  - Still acceptable in real-world APIs

REST principle: URLs should be nouns
Reality: Some verbs make intent clearer
Compromise: Use carefully when needed
```

### Why POST for Custom Actions?

```
GET:
  ✗ Used for fetching
  ✗ Can't send body
  ✗ Idempotent (may cause issues)

PUT/PATCH:
  ✗ Mean "replace/update"
  ✗ Wrong semantic meaning
  ✗ Confusing

DELETE:
  ✗ Means "delete resource"
  ✗ Wrong semantic meaning
  ✗ May trigger 405 errors

POST:
  ✓ Can send request body
  ✓ Not idempotent (OK for actions)
  ✓ Open-ended semantic meaning
  ✓ Industry standard for custom actions
```

---

## 10. HTTP Status Codes

### Status Code Categories

HTTP status codes are 3-digit numbers grouped by first digit:

```
1xx: Informational (1 = information provided)
2xx: Success (2 = request succeeded)
3xx: Redirection (3 = further action needed)
4xx: Client Error (4 = client's fault)
5xx: Server Error (5 = server's fault)
```

### 2xx Success Codes

#### 200 OK

```
Meaning: Request succeeded, here's the result

Use for:
  - GET successful fetch
  - POST successful with response body
  - PUT/PATCH successful update
  - DELETE successful (if returning deleted resource)

Example:
  GET /books/123
  Response: 200 OK
  Body: { "id": 123, "title": "Clean Code", ... }
```

#### 201 Created

```
Meaning: Resource successfully created

Use for:
  - POST that creates new resource

Include:
  - Location header with URL of new resource
  - Response body with created resource

Example:
  POST /books
  Body: { "title": "Clean Code", ... }

  Response: 201 Created
  Location: /books/123
  Body: { "id": 123, "title": "Clean Code", ... }
```

#### 204 No Content

```
Meaning: Request succeeded, no content to return

Use for:
  - DELETE that doesn't return deleted resource
  - UPDATE that doesn't return updated resource
  - POST that doesn't return anything

Example:
  DELETE /books/123
  Response: 204 No Content
  (No response body)
```

#### 202 Accepted

```
Meaning: Request accepted for processing (async)

Use for:
  - Long-running operations
  - Async tasks
  - Operations that don't complete immediately

Example:
  POST /reports/generate
  Body: { "type": "monthly" }

  Response: 202 Accepted
  Body: { "taskId": "task-123", "status": "processing" }
```

### 4xx Client Error Codes

#### 400 Bad Request

```
Meaning: Invalid request (syntax/validation error)

Use for:
  - Invalid JSON syntax
  - Missing required fields
  - Invalid field values
  - Validation failures

Example:
  POST /books
  Body: { "title": "Clean Code" }  (missing author)

  Response: 400 Bad Request
  Body: {
    "error": "VALIDATION_ERROR",
    "message": "author field is required"
  }
```

#### 401 Unauthorized

```
Meaning: Authentication required/failed

Use for:
  - No authentication provided
  - Invalid credentials
  - Expired token
  - Missing authorization header

Example:
  GET /user-profile
  (No Authorization header)

  Response: 401 Unauthorized
  Body: { "error": "AUTHENTICATION_REQUIRED" }
```

#### 403 Forbidden

```
Meaning: Authenticated but not authorized

Use for:
  - User authenticated but lacks permission
  - User trying to access someone else's resource
  - Insufficient permissions

Example:
  GET /users/456/admin-panel
  Authorization: Bearer valid-token
  (But user doesn't have admin role)

  Response: 403 Forbidden
  Body: { "error": "INSUFFICIENT_PERMISSIONS" }
```

#### 404 Not Found

```
Meaning: Resource doesn't exist

Use for:
  - Resource ID doesn't exist
  - Invalid URL path
  - Resource was deleted

Example:
  GET /books/999999
  (Book doesn't exist)

  Response: 404 Not Found
  Body: { "error": "RESOURCE_NOT_FOUND" }
```

#### 409 Conflict

```
Meaning: Request conflicts with existing state

Use for:
  - Duplicate resource (email already registered)
  - Resource state conflict
  - Optimistic locking failure

Example:
  POST /users
  Body: { "email": "alice@example.com" }
  (Email already registered)

  Response: 409 Conflict
  Body: { "error": "EMAIL_ALREADY_EXISTS" }
```

#### 422 Unprocessable Entity

```
Meaning: Syntax correct but semantics invalid

Use for:
  - Validation passed but business logic failed
  - Invalid domain logic
  - Data constraints violated

Example:
  POST /orders
  Body: {
    "productId": 123,
    "quantity": 1000
  }
  (Not enough stock)

  Response: 422 Unprocessable Entity
  Body: {
    "error": "INSUFFICIENT_STOCK",
    "message": "Only 50 units available"
  }
```

#### 429 Too Many Requests

```
Meaning: Rate limit exceeded

Use for:
  - Client exceeded request limit
  - Too many requests in time window

Example:
  GET /books (100th request in 1 minute)

  Response: 429 Too Many Requests
  Headers: Retry-After: 60
  Body: { "error": "RATE_LIMIT_EXCEEDED" }
```

### 5xx Server Error Codes

#### 500 Internal Server Error

```
Meaning: Server encountered unexpected error

Use for:
  - Unhandled exception
  - Database connection failure
  - Unexpected error (bug in code)

Example:
  GET /books/123
  (Database connection failed)

  Response: 500 Internal Server Error
  Body: { "error": "INTERNAL_SERVER_ERROR" }
```

#### 503 Service Unavailable

```
Meaning: Server temporarily unavailable

Use for:
  - Server maintenance
  - Server overload
  - Database down
  - Dependencies down

Example:
  GET /books
  (Database is down)

  Response: 503 Service Unavailable
  Headers: Retry-After: 60
  Body: { "error": "SERVICE_UNAVAILABLE" }
```

### Status Code Decision Tree

```
Client's fault?
  ├─ Yes → 4xx
  │  ├─ No authentication? → 401
  │  ├─ Authenticated but no permission? → 403
  │  ├─ Resource not found? → 404
  │  ├─ Invalid request? → 400
  │  └─ Conflict (duplicate)? → 409
  │
  └─ No → 5xx
     ├─ Server error/bug? → 500
     └─ Service down? → 503

Request succeeded?
  ├─ Yes → 2xx
  │  ├─ Created new? → 201
  │  ├─ No content to return? → 204
  │  └─ Processed successfully? → 200
  │
  └─ No → check above
```

---

## 11. Request & Response Design

### Request Structure

#### Request Headers

```
GET /books?limit=10 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
Accept: application/json
User-Agent: Mozilla/5.0...
```

**Important Headers:**

```
Authorization: Bearer <token>
  - Authentication token (JWT, etc.)
  - Sent with every protected request

Content-Type: application/json
  - Format of request body
  - Tells server what format to expect

Accept: application/json
  - Format client expects in response
  - Tells server what format to send

User-Agent: <client info>
  - Identifies client
  - Browser, mobile app, etc.

X-Request-ID: <unique-id>
  - Unique request identifier
  - Useful for debugging
```

#### Request Body

```
POST /books
Content-Type: application/json

{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "year": 2008,
  "isbn": "0132350882",
  "pages": 464
}
```

**Guidelines:**

```
✓ Use JSON format
✓ Use meaningful field names
✓ Use snake_case or camelCase (consistent)
✓ Include all required fields
✓ Use appropriate data types
✓ Validate in server
✗ Don't include IDs (auto-generated)
✗ Don't include timestamps (server generates)
```

### Response Structure

#### Successful Response (2xx)

```
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 06 Jan 2025 09:15:00 GMT
Cache-Control: public, max-age=3600

{
  "success": true,
  "data": {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "year": 2008
  }
}
```

#### Error Response (4xx/5xx)

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Date: Tue, 06 Jan 2025 09:15:00 GMT

{
  "success": false,
  "error": {
    "type": "VALIDATION_ERROR",
    "message": "Invalid request",
    "details": [
      {
        "field": "title",
        "message": "title is required"
      },
      {
        "field": "author",
        "message": "author must be at least 3 characters"
      }
    ]
  }
}
```

### Response Best Practices

```
1. Consistent Structure
   Always use same structure for all responses

   Good:
   {
     "success": true/false,
     "data": { ... },
     "error": { ... }
   }

2. Include Timestamps
   When resource was created/updated

   {
     "id": 123,
     "title": "...",
     "createdAt": "2025-01-06T09:15:00Z",
     "updatedAt": "2025-01-06T09:15:00Z"
   }

3. Include IDs
   Primary ID and any foreign IDs

   {
     "id": 123,
     "title": "...",
     "authorId": 456
   }

4. Use Meaningful Error Messages
   Help client understand what went wrong

   ✗ Bad: "Error"
   ✓ Good: "Email address is already registered"
   ✓ Better: Include error code for programmatic handling

5. Consistent Naming
   Use same naming convention throughout

   ✓ snake_case: user_id, created_at
   ✓ camelCase: userId, createdAt
   (Pick one, use consistently)

6. Include Metadata
   Total count, pagination info, etc.

   {
     "data": [ ... ],
     "pagination": {
       "total": 100,
       "page": 1,
       "limit": 10,
       "pages": 10
     }
   }
```

---

## 12. API Design Workflow

### Step-by-Step API Design Process

#### Step 1: Analyze Requirements

```
Start with:
  - Figma/design mockups
  - User stories
  - Feature requirements
  - Business requirements

Understand:
  - How users will interact with app
  - What data users need
  - What operations users perform
  - What constraints exist
```

#### Step 2: Identify Resources

```
From requirements, find nouns:

Project Management Platform:
  - Organizations
  - Projects
  - Tasks
  - Teams
  - Users
  - Comments

E-Commerce Platform:
  - Products
  - Categories
  - Cart
  - Orders
  - Users
  - Reviews

List all resources
```

#### Step 3: Design Database Schema

```
For each resource:
  - Define fields/columns
  - Define data types
  - Define relationships
  - Define constraints

(This is covered in next video - database design)
```

#### Step 4: Identify Operations

```
For each resource:

Standard CRUD:
  - List all (GET)
  - Get single (GET)
  - Create (POST)
  - Update (PUT/PATCH)
  - Delete (DELETE)

Custom actions:
  - Publish (POST)
  - Archive (POST)
  - Export (POST)
  - Send notification (POST)

List all operations needed
```

#### Step 5: Design Routes

```
For each operation:
  - Choose HTTP method
  - Design URL path
  - Define request format
  - Define response format
  - Define error scenarios
  - Choose status codes

Example:

List books:
  GET /books
  Response: 200 OK
  Body: { "data": [...], "pagination": {...} }

Get single book:
  GET /books/:id
  Response: 200 OK
  Body: { "id": 1, "title": "...", ... }

Create book:
  POST /books
  Body: { "title": "...", "author": "..." }
  Response: 201 Created
  Body: { "id": 1, "title": "...", ... }

Update book:
  PUT /books/:id
  Body: { "title": "...", "author": "..." }
  Response: 200 OK
  Body: { "id": 1, "title": "...", ... }

Delete book:
  DELETE /books/:id
  Response: 204 No Content
```

#### Step 6: Document API

```
Create API documentation:
  - Endpoint descriptions
  - Request/response examples
  - Error scenarios
  - Authentication requirements
  - Rate limits

Use tools:
  - Swagger/OpenAPI
  - Postman
  - Insomnia
  - API Blueprint
```

#### Step 7: Implement API

```
Write code:
  - Implement routes
  - Implement handlers
  - Implement business logic
  - Add validation
  - Add error handling
```

#### Step 8: Test API

```
Test thoroughly:
  - Happy path tests
  - Error scenario tests
  - Edge case tests
  - Integration tests
```

#### Step 9: Deploy & Monitor

```
After implementation:
  - Deploy to server
  - Monitor usage
  - Track errors
  - Collect feedback
  - Iterate
```

---

## 13. Resource Hierarchies & Relationships

### Hierarchical Resources

Resources can have parent-child relationships:

```
Organization
  └─ Project
      └─ Task
          └─ Comment

API paths reflect hierarchy:

GET /organizations
GET /organizations/123
POST /organizations/123/projects
GET /organizations/123/projects
GET /organizations/123/projects/456
POST /organizations/123/projects/456/tasks
GET /organizations/123/projects/456/tasks
GET /organizations/123/projects/456/tasks/789
```

### Nested Routes Rules

```
When to nest:
  ✓ Resource strongly belongs to parent
  ✓ Resource identity includes parent
  ✓ Can't exist without parent
  ✓ Operations only make sense in parent context

When NOT to nest:
  ✗ Resource can exist independently
  ✗ Resource has global identity
  ✗ Paths get too deep (>3 levels)
  ✗ Parent relationship is optional

Too deep (avoid):
  GET /organizations/1/projects/2/tasks/3/subtasks/4/comments/5
  (Hard to read, hard to use)

Better:
  GET /organizations/1/projects/2/tasks/3
  GET /comments/5  (if comment has independent identity)
```

### Many-to-Many Relationships

When resources relate through junction table:

```
Books <-> Authors (many-to-many)
Users <-> Organizations (many-to-many)

Option 1: Query string
  GET /books?author=tolkien
  (Filter books by author)

Option 2: Subresource
  GET /authors/123/books
  (Get all books by author 123)

Option 3: Separate endpoint
  GET /book-authors
  (Get all author-book relationships)

Choose based on:
  - How commonly used
  - How client needs data
  - Natural query patterns
```

---

## 14. Versioning & API Evolution

### API Versioning Strategies

#### Strategy 1: URL Path Versioning

```
v1 endpoint:
  GET /v1/books

v2 endpoint:
  GET /v2/books

Breaking change → new version number
Clients explicitly choose version

Advantages:
  ✓ Clear which version
  ✓ Easy to run multiple versions
  ✓ Easy for clients to migrate

Disadvantages:
  ✗ Duplicate code/maintenance
  ✗ More paths to maintain
  ✗ Takes more effort
```

#### Strategy 2: Accept Header Versioning

```
Request header specifies version:
  GET /books
  Accept: application/vnd.api+json; version=1

  GET /books
  Accept: application/vnd.api+json; version=2

Clean URLs
Versions in header

Advantages:
  ✓ Clean URLs
  ✓ Semantic versioning info
  ✗ Less visible to clients
  ✗ Harder to test/debug
```

#### Strategy 3: Query Parameter Versioning

```
GET /books?version=1
GET /books?version=2

Simple approach
Easy to implement

Advantages:
  ✓ Simple
  ✓ Easy to test
  ✗ Less RESTful
  ✗ Can't differentiate versions well
```

#### Strategy 4: No Versioning (Evolve Gracefully)

```
Approach:
  - Add new fields without removing old ones
  - Never change field meaning
  - Deprecate gracefully with warnings
  - Support multiple response formats

Best practice:
  - Only version when absolutely necessary
  - Try backward compatibility first
  - Document deprecations clearly
```

### API Deprecation

```
When deprecating an API version:

1. Announce deprecation (6-12 months notice)
2. Document migration path
3. Provide deprecation warnings in headers
   Deprecation: true
   Sunset: Wed, 06 Jan 2026 09:00:00 GMT

4. Keep old version running
5. Provide support for migration
6. Finally sunset the old version
```

---

## 15. Best Practices & Design Patterns

### Practice 1: Use Content Negotiation

```
Client requests format via Accept header:
  GET /books
  Accept: application/json

Server responds with appropriate format:
  Content-Type: application/json

Multiple formats:
  Accept: application/json     (JSON)
  Accept: application/xml      (XML)
  Accept: text/csv             (CSV)
  Accept: application/pdf      (PDF)

Benefits:
  ✓ Same endpoint, multiple formats
  ✓ Flexible for different clients
```

### Practice 2: Pagination for Large Collections

```
Always paginate large result sets:
  GET /books?limit=20&page=2

Response includes:
  {
    "data": [...20 items...],
    "pagination": {
      "total": 500,
      "page": 2,
      "limit": 20,
      "pages": 25
    }
  }

Benefits:
  ✓ Faster responses
  ✓ Lower bandwidth
  ✓ Better user experience
```

### Practice 3: Filtering & Searching

```
GET /books?category=fiction&year=2023
GET /books?search=clean code
GET /books?author=martin&sort=title

Parameters:
  ?field=value        (filter by exact value)
  ?search=term        (full-text search)
  ?sort=field         (sort by field)
  ?order=asc|desc     (sort order)

Benefits:
  ✓ Flexible querying
  ✓ Powerful searching
  ✓ Better UX
```

### Practice 4: Sparse Fieldsets

```
Client requests only needed fields:
  GET /books/123?fields=id,title,author

Response:
  {
    "id": 123,
    "title": "Clean Code",
    "author": "Robert C. Martin"
  }

Benefits:
  ✓ Smaller response size
  ✓ Lower bandwidth
  ✓ Faster over slow connections
```

### Practice 5: Rate Limiting

```
Prevent abuse:

Response headers indicate limits:
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 950
  X-RateLimit-Reset: 1641400200

When exceeded:
  HTTP 429 Too Many Requests
  Retry-After: 60

Benefits:
  ✓ Prevent abuse
  ✓ Fair resource allocation
  ✓ Protect infrastructure
```

### Practice 6: Webhooks for Async Events

```
Register webhook:
  POST /webhooks
  Body: {
    "url": "https://client.com/webhooks/orders",
    "events": ["order.created", "order.paid"]
  }

Server sends POST to webhook URL:
  POST https://client.com/webhooks/orders
  Body: {
    "event": "order.created",
    "data": { "orderId": 123, ... }
  }

Benefits:
  ✓ Real-time notifications
  ✓ Asynchronous updates
  ✓ Event-driven architecture
```

---

## 16. Common API Design Mistakes

### Mistake 1: Mixing Singular & Plural

```
✗ WRONG:
  GET /book          (singular)
  GET /books/123     (plural for single resource)
  GET /user          (inconsistent)

✓ CORRECT:
  GET /books         (always plural)
  GET /books/123     (plural for collection)
  GET /users         (consistent)
```

### Mistake 2: Using Verbs in URLs

```
✗ WRONG:
  GET /getAllBooks
  POST /createBook
  DELETE /deleteBook/123

✓ CORRECT:
  GET /books
  POST /books
  DELETE /books/123

Reasoning:
  HTTP method implies action
  URL should identify resource, not action
```

### Mistake 3: Wrong HTTP Methods

```
✗ WRONG:
  GET /users/123/delete
  POST /books/123        (for reading)
  PUT /books             (for multiple resources)

✓ CORRECT:
  DELETE /users/123
  GET /books/123
  POST /books
```

### Mistake 4: Inconsistent Response Format

```
✗ WRONG:
  API 1: { "success": true, "data": {...} }
  API 2: { "result": {...} }
  API 3: {...}
  (Different formats)

✓ CORRECT:
  All endpoints: { "success": true, "data": {...}, "error": null }
  (Consistent format)
```

### Mistake 5: Poor Error Messages

```
✗ WRONG:
  { "error": "Invalid" }
  { "error": "Error occurred" }

✓ CORRECT:
  {
    "error": {
      "type": "VALIDATION_ERROR",
      "message": "Email is required",
      "field": "email"
    }
  }
```

### Mistake 6: No Pagination

```
✗ WRONG:
  GET /users
  Response: [1000 user objects]
  (Slow, high bandwidth)

✓ CORRECT:
  GET /users?limit=20&page=1
  Response: [20 user objects + pagination info]
```

### Mistake 7: No Rate Limiting

```
✗ WRONG:
  No rate limiting
  Bots spam API
  Legitimate users get 503

✓ CORRECT:
  Rate limit per IP/user
  Return 429 when exceeded
  Include Retry-After header
```

---

## Summary Table: Complete API Design Checklist

| Aspect           | ✓ Do This                              | ✗ Avoid This                             |
| ---------------- | -------------------------------------- | ---------------------------------------- |
| **Resources**    | Use nouns                              | Use verbs                                |
| **Plural**       | Always plural (books, users)           | Mixed (book, books)                      |
| **Hierarchy**    | /orgs/1/projects/2                     | Overly nested (>3 levels)                |
| **HTTP Methods** | GET, POST, PUT/PATCH, DELETE           | Custom verbs                             |
| **Idempotency**  | GET, PUT, PATCH, DELETE are idempotent | POST multiple times (creates duplicates) |
| **Status Codes** | 200/201/204, 400/401/403/404, 500/503  | Using only 200 or 500                    |
| **Errors**       | Consistent format with error type      | Generic error messages                   |
| **Versions**     | URL path (/v1) or accept header        | Breaking changes without version         |
| **Pagination**   | Limit, page, total count               | Return all results                       |
| **Caching**      | Cache-Control headers                  | No caching strategy                      |
| **Auth**         | Authorization header (Bearer token)    | Custom headers, URL params               |

---

## Key Principles

### REST Principles

```
1. Client-Server: Separation of concerns
2. Statelessness: Each request self-contained
3. Uniform Interface: Consistent, standard interface
4. Resource-Based: URLs identify resources
5. Representations: Resources have multiple formats
6. Hypermedia: Links to related resources
```

### API Design Principles

```
1. Consistency: Same patterns everywhere
2. Simplicity: Easy to understand and use
3. Intuitiveness: Follows conventions
4. Discoverability: Obvious how to use
5. Extensibility: Can evolve over time
6. Security: Secure by default
7. Documentation: Clear, complete docs
```

---

## References & Links

**Video Source:**

- Complete REST API Design (Sriniously): https://www.youtube.com/watch?v=RG6q57DwV8Y

**Sriniously Channel & Playlist:**

- Sriniously Channel: https://www.youtube.com/channel/UCYkDx5W-v5qjkVVm1MrA1-w
- Backend from First Principles Playlist: https://www.youtube.com/playlist?list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1

**Related Videos in Series:**

- Controllers, Services, Repositories: https://www.youtube.com/watch?v=hyc-7w3pee8&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1
- Validations & Transformations: https://www.youtube.com/watch?v=qedj_JjjL-U&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1
- Authentication & Authorization: https://www.youtube.com/watch?v=A95rliroC8Q&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1

**REST Architecture & Standards:**

- Roy Fielding's REST Dissertation (2000): https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- REST Constraints Explained: https://en.wikipedia.org/wiki/Representational_state_transfer
- HATEOAS Explained: https://restfulapi.net/hateoas/

**HTTP & Web Standards:**

- HTTP Status Codes (RFC 7231): https://tools.ietf.org/html/rfc7231#section-6
- HTTP Methods (RFC 7231): https://tools.ietf.org/html/rfc7231#section-4.3
- HTTP Semantics: https://httpwg.org/

**API Design Best Practices:**

- RESTful API Design Best Practices: https://restfulapi.net/
- OpenAPI Specification: https://www.openapis.org/
- JSON API Standard: https://jsonapi.org/

**API Documentation Tools:**

- Swagger/OpenAPI: https://swagger.io/
- Postman: https://www.postman.com/
- Insomnia: https://insomnia.rest/
- Stoplight: https://stoplight.io/

**Related Topics:**

- Graph QL (Alternative to REST): https://graphql.org/
- gRPC (Binary Protocol): https://grpc.io/
- Webhook Design: https://webhooks.fyi/

**Idempotency & Safety:**

- Idempotent Operations: https://en.wikipedia.org/wiki/Idempotence
- HTTP Method Properties: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- Safe Methods vs Idempotent: https://stackoverflow.com/questions/45016234/what-is-idempotency-in-http-methods

short summary

# REST Architecture — Clear, Practical, Expanded Notes

These notes expand the lecture into a concise, friendly guide you can use for learning, interviews, and building real APIs. I explain key principles, show examples, offer best practices, and call out common pitfalls.

---

## What is REST (short)

- **REST = Representational State Transfer.**
- It’s an **architectural style** (not a protocol) for designing web APIs so they’re logical, predictable, and easy to consume.
- REST focuses on **resources** (nouns), HTTP methods, statelessness, and standard representations (usually JSON).

---

## Core REST Principles (what to follow)

1. **Resources (noun-first)**
   - Everything you expose is a resource: `users`, `movies`, `orders`.
   - Resource identifiers are URLs (URIs). Use plural nouns: `/movies`, `/users`.
   - Avoid verbs in paths (`/getUser`, `/deleteMovie`) — use HTTP methods instead.

2. **HTTP Methods as Intent**
   - `GET /movies` → read (safe, idempotent)

   - `GET /movies/21` → read single resource

   - `POST /movies` → create (not idempotent; server assigns ID)

   - `PUT /movies/21` → replace/update entire resource (idempotent)

   - `PATCH /movies/21` → partial update (idempotent if same payload repeated)

   - `DELETE /movies/21` → delete (idempotent)

   > **Idempotent** means repeated requests have the same effect as one request (except for side effects like logs).

3. **Endpoints / URL Design**
   - Use resource hierarchy for relationships:
     `/users/23/movies` (movies rented by user 23)
   - Use path parameters for identity, query parameters for filtering/pagination:
     `/movies?page=4&genre=action&sort=rating_desc`

4. **Statelessness**
   - Each request must carry all information needed to process it.
   - The server does **not** store client session state between requests.
   - State lives on the client (or is passed in each request, e.g., JWT token).

5. **Representations**
   - Typically JSON (`application/json`) for request/response bodies.
   - Responses often follow an envelope shape for consistency (optional but common — see example below).

6. **Use of HTTP Status Codes**
   - `200 OK` — successful GET/PUT/PATCH
   - `201 Created` — after successful POST creating a resource; include `Location` header with new URI
   - `204 No Content` — success with no body (common on DELETE)
   - `400 Bad Request` — client sent invalid data
   - `401 Unauthorized` — authentication missing/invalid
   - `403 Forbidden` — authenticated but not allowed
   - `404 Not Found` — resource missing
   - `409 Conflict` — e.g., duplicate unique field
   - `422 Unprocessable Entity` — validation semantics
   - `500` / `502` / etc — server errors

---

## Simple JSON response formatting (envelope)

A small, consistent wrapper reduces ambiguity.

**Raw data:**

```json
[{ "id": 21, "title": "Inception" }]
```

**Enveloped response:**

```json
{
  "status": "success",
  "data": [{ "id": 21, "title": "Inception" }],
  "meta": { "count": 1 }
}
```

- `status`: success / error helps clients.
- `meta`: pagination, totals, etc.

(There are formal specs like JSON:API and OData if you need stricter rules.)

---

## Examples & patterns

### CRUD examples

- List movies: `GET /movies`
- Get one movie: `GET /movies/21`
- Create movie: `POST /movies` (body contains movie fields)
- Update movie fully: `PUT /movies/21` (body contains full resource)
- Update partially: `PATCH /movies/21` (body contains changed fields)
- Delete: `DELETE /movies/21`

### Pagination

- Offset/limit (simple): `GET /movies?page=4&limit=25` or `?offset=75&limit=25`
- Cursor-based (more scalable): `GET /movies?cursor=t0kEn&limit=25`

### Filtering & sorting

- `GET /movies?genre=comedy&release_year=2022&sort=rating_desc`

### Non-CRUD actions

- Login / search / bulk operations — create explicit endpoints:
  - `POST /auth/login` (returns token)
  - `GET /search?q=matrix`
  - `POST /movies/bulk-delete` (when bulk actions are necessary)

---

## Authentication & statelessness

- Prefer **stateless auth** (token-based): JWT in `Authorization: Bearer <token>`.
- Server validates token on each request — no server-side session required.
- If sessions are used, accept they break strict REST statelessness (but are common).

---

## Error handling strategy

- Return meaningful HTTP status plus structured error body:

```json
{
  "status": "error",
  "error": {
    "code": "INVALID_INPUT",
    "message": "title is required",
    "fields": { "title": "required" }
  }
}
```

- Avoid leaking internals (stack traces) in production.

---

## HATEOAS (optional)

- HATEOAS = hypermedia links in responses (`_links`) to let client discover actions.
- Useful in hypermedia-driven APIs but heavier; many APIs skip it.

---

## Versioning

- Version your API so you can evolve safely:
  - URI versioning: `/v1/movies` (common and simple)
  - Header versioning: `Accept: application/vnd.myapi.v1+json` (cleaner but more complex)

- Avoid breaking changes without a new version.

---

## Caching & performance

- Use HTTP caching headers: `Cache-Control`, `ETag`, `Last-Modified`.
- Make read endpoints cacheable when possible.
- Use streaming for large payloads (avoid reading entire large datasets into memory).

---

## When to deviate from “pure REST”

- Some operations are RPC-style or complex workflows (e.g., execute job, batch processing). In those cases:
  - Use descriptive endpoints: `POST /reports/generate`
  - Document behavior clearly
  - Ensure idempotency where needed

---

## Best practices & design tips

- Keep endpoints **consistent and predictable**.
- Always use **proper HTTP verbs**.
- Use **plural nouns** for collections.
- Validate inputs and return clear errors.
- Keep responses small and paginated if large.
- Use HTTPS for all APIs.
- Rate-limit and protect sensitive endpoints.
- Provide good API docs (OpenAPI / Swagger).
- Provide examples in docs and include sample request/response.

---

## Common beginner mistakes

- Using verbs in URLs (`/getUsers`) instead of HTTP verbs.
- Putting state on the server (endpoints like `/nextPage`).
- Returning inconsistent response formats.
- Ignoring proper HTTP status codes (always return appropriate code).
- Not handling pagination and returning huge payloads.
- Storing authentication state purely server-side (breaks statelessness).

---

## Beginner mental model (how to think about REST)

- **Server = storage & behavior around resources.**
- **Client = actor that manipulates resources** by making stateless HTTP requests.
- Each request is a **self-contained instruction**: endpoint (resource), HTTP method (action), headers (metadata), body (payload).

---

## Interview-ready key points (short bullets)

- REST is architectural style based on resources and HTTP methods.
- Use nouns (resources) in URLs; use HTTP verbs for actions.
- Stateless: each request contains everything to process it.
- Common HTTP methods: GET, POST, PUT, PATCH, DELETE; know idempotency differences.
- Use standard HTTP status codes; return structured JSON (envelope).
- Version your API; document it (OpenAPI).
- For large data use pagination and streaming; use caching headers.
- Auth: prefer stateless tokens (JWT) for REST APIs.

---

### Quick curl examples

```bash
# List movies
curl -i https://api.example.com/v1/movies

# Get one movie
curl -i https://api.example.com/v1/movies/21

# Create a movie
curl -X POST -H "Content-Type: application/json" -d '{"title":"X","year":2024}' https://api.example.com/v1/movies

# Update partially
curl -X PATCH -H "Content-Type: application/json" -d '{"rating":9.0}' https://api.example.com/v1/movies/21

# Delete
curl -X DELETE https://api.example.com/v1/movies/21
```
