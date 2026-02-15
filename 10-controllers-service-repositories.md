# Controllers, Services, Repositories, Middlewares & Request Context

_Complete In-Depth Study Guide_

Source video: "10. What are controllers, services, repositories, middlewares and request context?" – Sriniously

---

## Table of Contents

1. The Request Lifecycle: Overview
2. Request Object & Response Object
3. Controllers/Handlers: The Entry Point
4. Services: The Business Logic Layer
5. Repositories: The Data Access Layer
6. Why Separate These Three Layers?
7. Middlewares: The Middleware Pattern
8. Middleware Execution Order & Chain
9. Common Middleware Examples
10. Request Context: Sharing State Across Boundaries
11. Complete Request Lifecycle Flow
12. Best Practices & Design Patterns
13. References & Resources

---

## 1. The Request Lifecycle: Overview

### What is a Request Lifecycle?

A **request lifecycle** is the complete journey a request takes from the moment it arrives at your server until the moment a response is sent back to the client.

```
Client sends request
        ↓
Operating System forwards to server port
        ↓
Server receives request at entry point
        ↓
[Multiple processing steps]
        ↓
Server sends response
        ↓
Client receives response
```

### Why Does This Matter?

Understanding the request lifecycle helps you:

- Design code that follows best practices
- Understand where each piece of logic belongs
- Debug issues more effectively
- Build scalable, maintainable systems

### The Simplified Lifecycle

```
REQUEST ARRIVES
    ↓
ROUTING (which endpoint?)
    ↓
MIDDLEWARE CHAIN (pre-processing)
    ↓
CONTROLLER/HANDLER (extract request data)
    ↓
SERVICE LAYER (execute business logic)
    ↓
REPOSITORY LAYER (database operations)
    ↓
SERVICE RETURNS RESULT (back up the chain)
    ↓
CONTROLLER SENDS RESPONSE
    ↓
RESPONSE SENT TO CLIENT
```

---

## 2. Request Object & Response Object

### What is the Request Object?

The **request object** contains all information about the HTTP request sent by the client.

```
Request object includes:
  - HTTP method (GET, POST, PUT, DELETE, etc.)
  - URL path (/api/users/123)
  - Query parameters (page=1, limit=10)
  - Request headers (Content-Type, Authorization, etc.)
  - Request body (JSON payload)
  - Cookies (if present)
  - IP address of client
  - User agent (browser info)
  - And more...
```

### What is the Response Object?

The **response object** is used to send an HTTP response back to the client.

```
Response object allows you to:
  - Set HTTP status code (200, 404, 500, etc.)
  - Set response headers (Content-Type, Set-Cookie, etc.)
  - Send response body (JSON data)
  - End the response
  - Redirect the client
  - And more...
```

### How They're Provided

In most web frameworks, these objects are automatically provided to your handler/controller by the runtime:

```javascript
// Node.js Express example
app.get('/users/:id', (req, res) => {
  // req = request object (automatically provided)
  // res = response object (automatically provided)

  const userId = req.params.id;        // Extract from request
  res.json({ id: userId });            // Send response
});

// Go example
func GetUser(w http.ResponseWriter, r *http.Request) {
  // w = ResponseWriter (like response object)
  // r = Request (like request object)

  id := r.URL.Query().Get("id")         // Extract from request
  w.Header().Set("Content-Type", "application/json")
  w.WriteHeader(http.StatusOK)
  // Write response body
}

// Python example
@app.route('/users/<id>', methods=['GET'])
def get_user(id):
  # request = global object with request data
  # return = automatically creates response

  return jsonify({'id': id})
```

### Key Point

**You don't create these objects yourself.** Your web framework automatically provides them. Your job is to use them properly.

---

## 3. Controllers/Handlers: The Entry Point

### What is a Controller/Handler?

A **controller** (or **handler**) is the first function that processes an incoming HTTP request.

```
Client Request
    ↓
Routing matches endpoint
    ↓
Handler/Controller function is called with (req, res)
    ↓
Handler processes and sends response
```

### The Four Responsibilities of a Controller

#### Responsibility 1: Extract Request Data

**Purpose:** Take data from the request object and convert it into a format your application understands.

```
Data sources:
  1. Request body (POST/PUT/PATCH)
     req.body → { "name": "Alice", "email": "alice@example.com" }

  2. URL path parameters (GET /users/:id)
     req.params.id → "123"

  3. Query parameters (GET /users?page=1&limit=10)
     req.query → { "page": "1", "limit": "10" }

  4. Headers
     req.headers["authorization"] → "Bearer token123..."

Example:
  POST /users
  Body: { "name": "John", "email": "john@example.com" }

  In controller:
    const { name, email } = req.body;
    // Extract name: "John"
    // Extract email: "john@example.com"
```

#### Responsibility 2: Deserialize (Binding)

**Purpose:** Convert serialized data (JSON) into native objects.

```
Why needed?
  - Client sends JSON (text format)
  - Server (e.g., Go) uses structs (native format)
  - Need to convert between them

Example in Go:
  Raw request body: {"name":"John","age":30}

  Struct definition:
    type User struct {
      Name string `json:"name"`
      Age  int    `json:"age"`
    }

  Deserialization:
    var user User
    json.Unmarshal([]byte(requestBody), &user)
    // user.Name = "John"
    // user.Age = 30

Example in Node.js:
  Most frameworks auto-deserialize JSON
  Request body automatically parsed by middleware
  req.body already contains parsed object
  No explicit deserialization needed in controller

Note:
  In Node.js (with auto-parsing), this step is transparent
  In Go, Python, Rust, you do this explicitly
```

#### Responsibility 3: Validate & Transform

**Purpose:** Ensure data is valid before processing.

```
Validation:
  - Check required fields present
  - Check data types correct
  - Check values within expected range
  - Check format correct (email, phone, date, etc.)

Transformation:
  - Convert formats (string to number)
  - Normalize data (lowercase email)
  - Set defaults (if not provided)
  - Create computed fields

Example:
  Raw request: { email: "Alice@GMAIL.COM", page: "2", limit: "20" }

  After validation & transformation:
    - email validated as valid format
    - email transformed to lowercase: "alice@gmail.com"
    - page converted string to number: 2
    - limit converted string to number: 20
    - status set to default: "active"

  Result: { email: "alice@gmail.com", page: 2, limit: 20, status: "active" }
```

#### Responsibility 4: Call Service & Send Response

**Purpose:** Orchestrate the business logic and return appropriate response.

```
Controller flow:
  1. Receive (req, res)
  2. Extract data from request
  3. Deserialize to native format
  4. Validate & transform data
  5. Call service layer method
  6. Receive result from service
  7. Determine appropriate response code
  8. Send response with data & status code

Example:
  // POST /books
  // Body: { "title": "Clean Code", "author": "Robert C. Martin" }

  async createBook(req, res) {
    try {
      // Step 1-4: Extract, deserialize, validate
      const { title, author } = req.body;

      // Validation
      if (!title || !author) {
        return res.status(400).json({ error: "Title and author required" });
      }

      // Step 5: Call service
      const newBook = await bookService.createBook(title, author);

      // Step 6-8: Send response
      res.status(201).json({
        message: "Book created successfully",
        data: newBook
      });
    } catch (error) {
      res.status(500).json({ error: "Internal server error" });
    }
  }
```

### Key Principle: Controllers Deal with HTTP Concerns

Controllers are responsible for:

- ✓ Extracting request data
- ✓ HTTP status codes (200, 201, 400, 500)
- ✓ HTTP headers
- ✓ Request/Response serialization
- ✓ Error handling & error codes

Controllers should NOT:

- ✗ Contain business logic
- ✗ Contain database queries
- ✗ Know about implementation details
- ✗ Mix HTTP and business concerns

### Controller Template Pattern

```javascript
async handleRequest(req, res) {
  try {
    // 1. Extract request data
    const userId = req.params.id;
    const { name, email } = req.body;
    const { page, limit } = req.query;

    // 2. Deserialize (often automatic)
    // Already in native format

    // 3. Validate & Transform
    if (!userId) {
      return res.status(400).json({ error: "User ID required" });
    }

    // 4. Call service
    const result = await this.service.doSomething(userId, name, email);

    // 5. Determine response code
    const statusCode = result ? 200 : 400;

    // 6. Send response
    res.status(statusCode).json({
      success: true,
      data: result
    });

  } catch (error) {
    // Global error handling
    res.status(500).json({
      success: false,
      error: "Internal server error"
    });
  }
}
```

---

## 4. Services: The Business Logic Layer

### What is a Service?

A **service** is a function that contains all the business logic for your application.

```
Business logic = What your app actually does

Examples:
  - Creating a user (validation, generating ID, etc.)
  - Processing payment (calling payment gateway, updating database)
  - Sending email notification
  - Calculating complex metrics
  - Orchestrating multiple operations
```

### Key Principle: Services Are HTTP-Agnostic

A good service doesn't know or care that it's being used in an API.

```
Good service:
  // This could be called from API, CLI, scheduled job, etc.
  async createUser(name, email, password) {
    // Validate inputs
    // Hash password
    // Save to database
    // Return created user
  }

  // Can use from:
  // 1. API: res.json(await userService.createUser(...))
  // 2. CLI: console.log(await userService.createUser(...))
  // 3. Scheduled job: await userService.createUser(...)
  // 4. Event listener: await userService.createUser(...)

Bad service:
  // This is tightly coupled to HTTP
  // Can ONLY be used from API
  async createUser(req, res) {
    // Do business logic
    res.json(...); // Can't reuse without response object
  }
```

### What Services Do

#### 1. Orchestrate Operations

Services coordinate between multiple components:

```
Single responsibility:
  Repository: Get user by ID from database
  Repository: Update user in database

Orchestration:
  Service: Get user from repository
          → Validate the user
          → Update user using repository
          → Notify user by email
          → Log the action
          → Return result
```

#### 2. Implement Business Rules

Services enforce business logic:

```
Example: Create a bank transfer

Business rules:
  1. Both accounts must exist
  2. Sender must have sufficient balance
  3. Cannot transfer to same account
  4. Amount must be positive
  5. Both accounts must be active
  6. Daily transfer limit must not be exceeded

Service encapsulates all these rules:
  async transferMoney(fromAccountId, toAccountId, amount) {
    // Rule 1: Check both accounts exist
    const fromAccount = await accountRepo.getById(fromAccountId);
    const toAccount = await accountRepo.getById(toAccountId);

    if (!fromAccount || !toAccount) {
      throw new Error("Account not found");
    }

    // Rule 2: Check sufficient balance
    if (fromAccount.balance < amount) {
      throw new Error("Insufficient balance");
    }

    // Rule 3: Cannot transfer to same account
    if (fromAccountId === toAccountId) {
      throw new Error("Cannot transfer to same account");
    }

    // Rule 4: Amount must be positive
    if (amount <= 0) {
      throw new Error("Amount must be positive");
    }

    // Rule 5: Both accounts must be active
    if (!fromAccount.isActive || !toAccount.isActive) {
      throw new Error("Account is not active");
    }

    // Rule 6: Daily limit check
    const todaysTransfers = await transferRepo.getDailyTotal(fromAccountId);
    const dailyLimit = 100000;
    if (todaysTransfers + amount > dailyLimit) {
      throw new Error("Daily transfer limit exceeded");
    }

    // All rules passed, perform transfer
    fromAccount.balance -= amount;
    toAccount.balance += amount;

    await accountRepo.update(fromAccount);
    await accountRepo.update(toAccount);

    await transferRepo.create({
      from: fromAccountId,
      to: toAccountId,
      amount,
      timestamp: new Date()
    });

    // Notify both users
    await emailService.send(fromAccount.email, "Transfer sent");
    await emailService.send(toAccount.email, "Transfer received");

    return { success: true, newBalance: fromAccount.balance };
  }
```

#### 3. Call Repository Methods

Services use repositories for database operations:

```
Service method structure:
  1. Extract parameters
  2. Validate business rules
  3. Call repository for database operations
  4. Transform/combine results
  5. Perform additional processing (emails, notifications, etc.)
  6. Return result

Example:
  async getUserProfile(userId) {
    // 1. Extract parameters (passed in)

    // 2. Validate
    if (!userId) throw new Error("User ID required");

    // 3. Call repositories
    const user = await userRepository.getById(userId);
    const posts = await postRepository.getByUserId(userId);
    const followers = await followerRepository.countByUserId(userId);

    // 4. Transform/combine
    const profile = {
      user,
      posts,
      followerCount: followers
    };

    // 5. Additional processing
    await loggingService.log(`Profile accessed: ${userId}`);

    // 6. Return
    return profile;
  }
```

### Service Methods Should Have Single Responsibility

```
Good:
  // Each method does ONE thing
  async getUserById(userId) { ... }
  async createUser(name, email, password) { ... }
  async updateUser(userId, updates) { ... }
  async deleteUser(userId) { ... }
  async getUserPosts(userId) { ... }

Bad:
  // One method tries to do everything
  async manageUser(operation, userId, data) {
    if (operation === "get") {
      // Get user
    } else if (operation === "create") {
      // Create user
    } else if (operation === "update") {
      // Update user
    } else if (operation === "delete") {
      // Delete user
    }
    // Becomes unmaintainable
  }
```

### Service Template Pattern

```javascript
class UserService {
  constructor(userRepository, emailService, loggingService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
    this.loggingService = loggingService;
  }

  // Pure business logic - no HTTP knowledge
  async createUser(name, email, password) {
    // 1. Validate business rules
    if (!name || !email || !password) {
      throw new Error("Name, email, and password required");
    }

    const existingUser = await this.userRepository.getByEmail(email);
    if (existingUser) {
      throw new Error("Email already registered");
    }

    // 2. Process
    const hashedPassword = await hashPassword(password);

    // 3. Call repository
    const user = await this.userRepository.create({
      name,
      email,
      password: hashedPassword,
      createdAt: new Date(),
    });

    // 4. Additional operations
    await this.emailService.sendWelcomeEmail(email, name);
    await this.loggingService.log(`User created: ${user.id}`);

    // 5. Return (no HTTP concerns)
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      // Don't return password!
    };
  }

  async getUserById(userId) {
    if (!userId) throw new Error("User ID required");
    const user = await this.userRepository.getById(userId);
    if (!user) throw new Error("User not found");
    return user;
  }

  async updateUser(userId, updates) {
    if (!userId) throw new Error("User ID required");

    const user = await this.userRepository.getById(userId);
    if (!user) throw new Error("User not found");

    const updatedUser = await this.userRepository.update(userId, updates);
    await this.loggingService.log(`User updated: ${userId}`);

    return updatedUser;
  }
}

// Usage from controller
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  async create(req, res) {
    try {
      const { name, email, password } = req.body;
      const user = await this.userService.createUser(name, email, password);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// Usage from CLI (same service, no controller)
async function createUserFromCLI() {
  const user = await userService.createUser(
    "Alice",
    "alice@example.com",
    "password123",
  );
  console.log("User created:", user);
}
```

---

## 5. Repositories: The Data Access Layer

### What is a Repository?

A **repository** is a function that handles all database operations for a specific entity.

```
Repository = Data Access Object

Responsibilities:
  - Database queries
  - Data persistence
  - Fetching data
  - Updating data
  - Deleting data
```

### Repository Pattern Principle

The repository pattern separates data access logic from business logic.

```
Without repository pattern:
  Service has database queries mixed with business logic
  ↓
  Hard to test
  Hard to change database
  Tight coupling

With repository pattern:
  Repository handles all database operations
  Service handles business logic
  ↓
  Easy to test
  Easy to change database
  Loose coupling
```

### Repository Methods

Each repository method has **single responsibility**.

```
Good:
  // Each method does ONE database operation

  async getById(id) {
    return await db.query(
      "SELECT * FROM users WHERE id = ?",
      [id]
    );
  }

  async getByEmail(email) {
    return await db.query(
      "SELECT * FROM users WHERE email = ?",
      [email]
    );
  }

  async create(userData) {
    return await db.query(
      "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
      [userData.name, userData.email, userData.password]
    );
  }

  async update(id, updates) {
    return await db.query(
      "UPDATE users SET ? WHERE id = ?",
      [updates, id]
    );
  }

  async delete(id) {
    return await db.query(
      "DELETE FROM users WHERE id = ?",
      [id]
    );
  }

  async getAll() {
    return await db.query("SELECT * FROM users");
  }

Bad:
  // One method tries to do everything
  async handleUser(operation, id, data) {
    if (operation === "get") {
      // Get user
    } else if (operation === "create") {
      // Create user
    } else if (operation === "update") {
      // Update user
    }
    // Becomes unmaintainable
  }
```

### Why Single Responsibility?

```
Problem with multi-operation methods:
  async handleData(operation, id, data) {
    if (operation === "get") { ... }
    else if (operation === "fetch-all") { ... }
    else if (operation === "create") { ... }
    else if (operation === "update") { ... }
    else if (operation === "delete") { ... }
  }

  When you want to:
  ✗ Fetch user by ID → Can't do with "fetch-all"
  ✗ Fetch all with sorting → Can't do with "get"
  ✗ Update multiple fields → Complex logic

Solution: Specific methods:
  getById(id) → fetch one
  getAll() → fetch all
  getBySortedBy(sortField) → fetch with sort
  create(data) → insert
  update(id, data) → update
  delete(id) → remove

  Service can combine:
  const users = await repo.getAll();
  users = users.sort(...);
  return users;
```

### Repository Template Pattern

```javascript
class UserRepository {
  constructor(database) {
    this.db = database;
  }

  // Single responsibility: Get by ID
  async getById(userId) {
    const result = await this.db.query("SELECT * FROM users WHERE id = $1", [
      userId,
    ]);
    return result.rows[0] || null;
  }

  // Single responsibility: Get by email
  async getByEmail(email) {
    const result = await this.db.query("SELECT * FROM users WHERE email = $1", [
      email,
    ]);
    return result.rows[0] || null;
  }

  // Single responsibility: Get all
  async getAll() {
    const result = await this.db.query("SELECT id, name, email FROM users");
    return result.rows;
  }

  // Single responsibility: Get with pagination
  async getPaginated(page, limit) {
    const offset = (page - 1) * limit;
    const result = await this.db.query(
      "SELECT * FROM users LIMIT $1 OFFSET $2",
      [limit, offset],
    );
    return result.rows;
  }

  // Single responsibility: Create
  async create(userData) {
    const result = await this.db.query(
      "INSERT INTO users (name, email, password) VALUES ($1, $2, $3) RETURNING *",
      [userData.name, userData.email, userData.password],
    );
    return result.rows[0];
  }

  // Single responsibility: Update
  async update(userId, updates) {
    const fields = [];
    const values = [];
    let paramIndex = 1;

    for (const [key, value] of Object.entries(updates)) {
      fields.push(`${key} = $${paramIndex++}`);
      values.push(value);
    }

    values.push(userId);

    const query = `
      UPDATE users SET ${fields.join(", ")} WHERE id = $${paramIndex}
      RETURNING *
    `;

    const result = await this.db.query(query, values);
    return result.rows[0];
  }

  // Single responsibility: Delete
  async delete(userId) {
    const result = await this.db.query(
      "DELETE FROM users WHERE id = $1 RETURNING *",
      [userId],
    );
    return result.rows[0] || null;
  }
}

// Usage from Service
class UserService {
  constructor(userRepository) {
    this.repo = userRepository;
  }

  async getUserProfile(userId) {
    // Service calls specific repository method
    const user = await this.repo.getById(userId);
    if (!user) throw new Error("User not found");
    return user;
  }

  async listUsers(page = 1, limit = 10) {
    // Service calls paginated method
    return await this.repo.getPaginated(page, limit);
  }

  async createUser(name, email, password) {
    // Service handles validation
    const existing = await this.repo.getByEmail(email);
    if (existing) throw new Error("Email exists");

    // Service calls create method
    return await this.repo.create({
      name,
      email,
      password, // Already hashed by service
    });
  }
}
```

### Repository vs Service

| Aspect             | Repository      | Service                   |
| ------------------ | --------------- | ------------------------- |
| **Purpose**        | Data access     | Business logic            |
| **Knows about**    | Database, SQL   | Business rules            |
| **What it does**   | CRUD operations | Orchestration, validation |
| **Example method** | `getById(id)`   | `getUserProfile(userId)`  |
| **Called by**      | Service         | Controller                |
| **HTTP aware?**    | No              | No                        |

---

## 6. Why Separate These Three Layers?

### The Separation Principle

Separating controllers, services, and repositories creates **separation of concerns**.

```
Each layer has one responsibility:
  Controller → HTTP concerns
  Service → Business logic
  Repository → Data access

Benefits:
  ✓ Testability (test each layer independently)
  ✓ Reusability (service can be called from multiple places)
  ✓ Maintainability (changes in one layer don't affect others)
  ✓ Scalability (easy to add new features)
  ✓ Debugging (know exactly where to look for issues)
```

### Scenario: Why Not Combine?

#### Option 1: Mixed Everything (Bad)

```javascript
// Everything in one function
app.post('/users', async (req, res) => {
  try {
    // HTTP stuff
    const { name, email, password } = req.body;

    // Validation
    if (!name || !email || !password) {
      return res.status(400).json({ error: "Missing fields" });
    }

    // Hash password (business logic)
    const bcrypt = require('bcrypt');
    const hashedPassword = await bcrypt.hash(password, 10);

    // Database query (data access)
    const result = await db.query(
      "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
      [name, email, hashedPassword]
    );

    // Email logic (business logic)
    const nodemailer = require('nodemailer');
    await nodemailer.send({
      to: email,
      subject: "Welcome"
    });

    // HTTP response
    res.status(201).json({
      id: result.insertId,
      name, email
    });
  } catch (error) {
    res.status(500).json({ error: "Server error" });
  }
});

Problems:
  ✗ Can't reuse logic for CLI commands
  ✗ Can't test business logic separately
  ✗ Can't test database logic separately
  ✗ Hard to modify (changing one thing affects everything)
  ✗ Can't call from scheduled jobs
  ✗ Can't call from event listeners
  ✗ Over 100 lines in one function
  ✗ Multiple responsibilities
```

#### Option 2: Separated Layers (Good)

```javascript
// REPOSITORY LAYER
class UserRepository {
  async create(userData) {
    return await db.query(
      "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
      [userData.name, userData.email, userData.password]
    );
  }
}

// SERVICE LAYER
class UserService {
  constructor(userRepository, emailService) {
    this.repo = userRepository;
    this.emailService = emailService;
  }

  async createUser(name, email, password) {
    // Validation
    if (!name || !email || !password) {
      throw new Error("Missing fields");
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create user via repository
    const user = await this.repo.create({
      name,
      email,
      password: hashedPassword
    });

    // Send email
    await this.emailService.sendWelcome(email, name);

    return user;
  }
}

// CONTROLLER LAYER
app.post('/users', async (req, res) => {
  try {
    const { name, email, password } = req.body;
    const user = await userService.createUser(name, email, password);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// CLI COMMAND (same service, no controller)
async function createUserFromCLI(name, email, password) {
  try {
    const user = await userService.createUser(name, email, password);
    console.log("User created:", user);
  } catch (error) {
    console.error("Error:", error.message);
  }
}

// SCHEDULED JOB (same service)
schedule.every().day.at("10:00").do(async () => {
  const user = await userService.createUser(
    "DailyUser",
    `user${Date.now()}@example.com`,
    "randomPassword"
  );
  console.log("Daily user created");
});

Benefits:
  ✓ Service logic reused in API, CLI, scheduled jobs, events
  ✓ Can test each layer independently
  ✓ Easy to modify (change only what's needed)
  ✓ Each function has one responsibility
  ✓ Clear separation of concerns
  ✓ Scalable and maintainable
```

### Testability Example

```javascript
// Testing separated layers is easy

// Test repository in isolation
describe('UserRepository', () => {
  it('should create a user', async () => {
    const mockDb = { ... };
    const repo = new UserRepository(mockDb);
    const user = await repo.create({
      name: 'John',
      email: 'john@example.com',
      password: 'hash'
    });
    expect(user.name).toBe('John');
  });
});

// Test service in isolation (mock repository)
describe('UserService', () => {
  it('should hash password before creating user', async () => {
    const mockRepo = { create: jest.fn() };
    const mockEmail = { send: jest.fn() };
    const service = new UserService(mockRepo, mockEmail);

    await service.createUser('John', 'john@example.com', 'password123');

    // Verify repository was called with hashed password
    expect(mockRepo.create).toHaveBeenCalledWith(
      expect.objectContaining({
        password: expect.not.stringContaining('password123')
      })
    );
  });
});

// Test controller in isolation (mock service)
describe('UserController', () => {
  it('should return 201 on successful creation', async () => {
    const mockService = {
      createUser: jest.fn().mockResolvedValue({ id: 1, name: 'John' })
    };
    const controller = new UserController(mockService);

    const mockRes = { status: jest.fn().returnThis(), json: jest.fn() };
    await controller.create({ body: {...} }, mockRes);

    expect(mockRes.status).toHaveBeenCalledWith(201);
  });
});

Testing mixed handler (hard):
  - Can't isolate business logic
  - Can't mock database
  - Can't test without full HTTP setup
  - Very slow and fragile tests
```

---

## 7. Middlewares: The Middleware Pattern

### What is a Middleware?

A **middleware** is a function that sits between the request entry point and the final handler, with access to the request, response, and a `next()` function.

```
Middleware = Processing function in request lifecycle

Structure:
  function middleware(req, res, next) {
    // Do something with request
    // Do something with response
    // Call next() to pass to next middleware
    // OR send response to end chain
  }
```

### Three Objects Provided to Middleware

```javascript
// Middleware receives three parameters:
app.use((req, res, next) => {
  // req = Request object (input)
  // res = Response object (output)
  // next = Function to proceed to next middleware

  next(); // Pass to next middleware
  // OR
  res.json({ error: "Unauthorized" }); // End here
});
```

### The `next()` Function

The `next()` function is critical for middleware execution flow.

```
Middleware with next():

Middleware 1
    ↓ (next called)
Middleware 2
    ↓ (next called)
Middleware 3
    ↓ (next called)
Handler/Controller
    ↓
Response sent

If Middleware 2 sends response:

Middleware 1
    ↓ (next called)
Middleware 2
    ↓ (res.send() - chain breaks)
[Middleware 3 not called]
[Handler not called]
Response sent

Example code:

// Middleware 1
app.use((req, res, next) => {
  console.log("Middleware 1");
  next(); // Pass to Middleware 2
});

// Middleware 2
app.use((req, res, next) => {
  console.log("Middleware 2");

  // If condition true, stop chain
  if (someCondition) {
    return res.status(401).json({ error: "Unauthorized" });
  }

  next(); // Pass to Middleware 3
});

// Middleware 3
app.use((req, res, next) => {
  console.log("Middleware 3");
  next(); // Pass to Handler
});

// Handler
app.get('/api/users', (req, res) => {
  console.log("Handler");
  res.json({ users: [] });
});

Execution order:
  Middleware 1 ➜ Middleware 2 ➜ Middleware 3 ➜ Handler ➜ Response
```

### Key Characteristics of Middleware

| Characteristic              | Meaning                              |
| --------------------------- | ------------------------------------ |
| **Optional**                | May or may not be executed           |
| **Order matters**           | Executed in order they're registered |
| **Can end chain**           | Can send response and stop chain     |
| **Can modify request**      | Can add data to `req` object         |
| **Can modify response**     | Can set headers, cookies, etc.       |
| **Has access to req & res** | Full HTTP request/response           |
| **Executed before handler** | By default (unless placed after)     |

---

## 8. Middleware Execution Order & Chain

### Understanding Middleware Order

Middleware executes in the exact order you register it.

```
Registration order = Execution order

Code:
  app.use(corsMiddleware);         // 1st to execute
  app.use(loggingMiddleware);      // 2nd to execute
  app.use(authMiddleware);         // 3rd to execute
  app.use(rateLimitMiddleware);    // 4th to execute

Request comes in:
  corsMiddleware ➜ loggingMiddleware ➜ authMiddleware ➜ rateLimitMiddleware ➜ Handler

If authentication fails:
  corsMiddleware ➜ loggingMiddleware ➜ authMiddleware ⛔ (sends 401, chain stops)
  Handler never executes, Response sent
```

### Why Order Matters

```
Example: CORS before Authentication

WRONG ORDER:
  Authentication ➜ CORS

  Request from different origin fails auth (401)
  But CORS headers not set
  Browser blocks response (CORS error)
  User sees confusing error

RIGHT ORDER:
  CORS ➜ Authentication

  Request from different origin
  CORS middleware sets headers
  Authentication middleware checks credentials
  If fails, CORS headers already set
  Browser allows response, shows proper error
```

### Example: Middleware Chain

```javascript
// Middleware chain for API request

// 1. CORS Middleware (very first)
app.use((req, res, next) => {
  const allowedOrigins = ['https://example.com'];
  if (allowedOrigins.includes(req.origin)) {
    res.setHeader('Access-Control-Allow-Origin', req.origin);
  }
  next(); // Must proceed
});

// 2. Logging Middleware
app.use((req, res, next) => {
  const startTime = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
  });
  next(); // Must proceed
});

// 3. Request Parsing Middleware
app.use(express.json()); // Parse JSON body automatically

// 4. Authentication Middleware
app.use((req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: "No token" }); // Chain ends
  }

  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded; // Add user to request
    next(); // Proceed with user attached
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" }); // Chain ends
  }
});

// 5. Rate Limit Middleware
app.use((req, res, next) => {
  const clientIp = req.ip;
  const requestCount = store.get(clientIp) || 0;

  if (requestCount > 100) { // 100 requests per minute
    return res.status(429).json({ error: "Too many requests" }); // Chain ends
  }

  store.set(clientIp, requestCount + 1);
  next(); // Proceed
});

// 6. Route Handler
app.get('/api/users', (req, res) => {
  // req.user is available (from auth middleware)
  res.json({ users: [], userId: req.user.id });
});

Execution for valid request:
  CORS ➜ Logging (started) ➜ Parse ➜ Auth ➜ Rate Limit ➜ Handler ➜ Logging (finished) ➜ Response

Execution for no token:
  CORS ➜ Logging (started) ➜ Parse ➜ Auth ⛔ 401 response sent ➜ Logging (finished)
  Handler never called
```

---

## 9. Common Middleware Examples

### Middleware 1: CORS (Cross-Origin Resource Sharing)

**Purpose:** Control which external domains can access your API

```javascript
// CORS Middleware
app.use((req, res, next) => {
  const allowedOrigins = [
    'https://frontend.example.com',
    'https://app.example.com'
  ];

  if (allowedOrigins.includes(req.headers.origin)) {
    res.setHeader('Access-Control-Allow-Origin', req.headers.origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }

  next();
});

Why middleware?
  ✓ Runs for every request
  ✓ Needs to check origin and set headers
  ✓ Must run before handler
  ✓ Same logic for all endpoints
```

### Middleware 2: Authentication

**Purpose:** Verify user identity from token

```javascript
app.use((req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const token = authHeader.split(' ')[1]; // "Bearer TOKEN"
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Add user info to request for downstream
    req.user = {
      id: decoded.sub,
      role: decoded.role,
      email: decoded.email
    };

    next(); // Proceed with user attached
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
});

Why middleware?
  ✓ Runs for every request
  ✓ Extracts and verifies token
  ✓ Stops unauthorized requests early
  ✓ Adds user info to request
```

### Middleware 3: Rate Limiting

**Purpose:** Prevent too many requests from single client

```javascript
const requestStore = new Map();

app.use((req, res, next) => {
  const clientIp = req.ip;
  const now = Date.now();

  if (!requestStore.has(clientIp)) {
    requestStore.set(clientIp, []);
  }

  const requests = requestStore.get(clientIp);

  // Remove requests older than 1 minute
  const oneMinuteAgo = now - 60000;
  const recentRequests = requests.filter(time => time > oneMinuteAgo);

  if (recentRequests.length >= 100) { // Max 100 per minute
    return res.status(429).json({ error: "Rate limit exceeded" });
  }

  recentRequests.push(now);
  requestStore.set(clientIp, recentRequests);

  next();
});

Why middleware?
  ✓ Runs for every request
  ✓ Checks rate limit and blocks if exceeded
  ✓ Must block before expensive operations
```

### Middleware 4: Security Headers

**Purpose:** Add security headers to responses

```javascript
app.use((req, res, next) => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');

  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');

  // Enable XSS protection
  res.setHeader('X-XSS-Protection', '1; mode=block');

  // Content Security Policy
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' trusted.com"
  );

  next();
});

Why middleware?
  ✓ Runs for every response
  ✓ Adds security headers
  ✓ Must happen for all endpoints
```

### Middleware 5: Logging

**Purpose:** Log request/response information

```javascript
app.use((req, res, next) => {
  const startTime = Date.now();
  const originalSend = res.send;

  // Intercept response to log it
  res.send = function(data) {
    const duration = Date.now() - startTime;

    console.log({
      timestamp: new Date().toISOString(),
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      durationMs: duration,
      userAgent: req.get('user-agent'),
      ip: req.ip
    });

    // Call original send
    return originalSend.call(this, data);
  };

  next();
});

Why middleware?
  ✓ Runs for every request
  ✓ Logs all important info
  ✓ Captures response status
```

### Middleware 6: Global Error Handler

**Purpose:** Catch errors from anywhere and format response

```javascript
app.use((error, req, res, next) => {
  // Note: 4 parameters means this is error handler middleware

  console.error(error); // Log error

  // Determine status code
  const statusCode = error.statusCode || 500;

  // Determine error type
  const errorType = error.type || 'INTERNAL_ERROR';

  // Send formatted error response
  res.status(statusCode).json({
    success: false,
    error: {
      type: errorType,
      message: error.message,
      details: process.env.NODE_ENV === 'development' ? error : undefined
    }
  });
});

Why middleware?
  ✓ Catches all errors from app
  ✓ Centralizes error handling
  ✓ Ensures consistent error format
  ✓ Prevents unhandled rejections
  ✓ Must be registered LAST
```

### Middleware 7: Compression

**Purpose:** Compress response body for faster transfer

```javascript
app.use((req, res, next) => {
  const acceptEncoding = req.headers['accept-encoding'] || '';

  if (acceptEncoding.includes('gzip')) {
    // Add gzip compression
    res.setHeader('Content-Encoding', 'gzip');
    // Compress response body
  }

  next();
});

Why middleware?
  ✓ Runs for every response
  ✓ Reduces response size
  ✓ Faster transfer to client
```

---

## 10. Request Context: Sharing State Across Boundaries

### What is Request Context?

A **request context** is storage scoped to a single HTTP request.

```
Concept:
  Each request gets its own context
  Context is accessible throughout request lifecycle
  Context dies after response sent
  Next request gets new context

Purpose:
  Share data between middleware and handler
  Without passing parameters explicitly
  Without global state
```

### Why Context Matters

```
Problem without context:

Middleware 1: Extract user ID from token
↓
How to pass user ID to handler?

Option 1: Attach to request object (works but feels hacky)
  req.userId = decoded.userId;

Option 2: Pass as function parameter
  handler(req, res, req.userId, ...) // Lots of parameters

Option 3: Return from middleware (doesn't work, no return)

Option 4: Use context (clean and proper)
  ctx.userId = decoded.userId;
  // Handler can access ctx.userId anytime
```

### How Context Works

```
Request arrives
  ↓
Create new context { }
  ↓
Middleware 1: ctx.ip = req.ip
  ↓
Middleware 2: ctx.userId = user.id
  ↓
Middleware 3: ctx.role = user.role
  ↓
Handler: Can read ctx.ip, ctx.userId, ctx.role
  ↓
Service: Can access ctx.userId for logging
  ↓
Repository: Can access ctx.ip for audit trail
  ↓
Response sent
  ↓
Context destroyed
```

### Request Context in Different Languages

#### Node.js/Express

```javascript
// Not built-in, need library like cls-hooked or AsyncLocalStorage

const { AsyncLocalStorage } = require("async_hooks");
const requestContext = new AsyncLocalStorage();

// Middleware to create context
app.use((req, res, next) => {
  const context = {
    requestId: uuid(),
    startTime: Date.now(),
    ip: req.ip,
  };

  requestContext.run(context, () => {
    next();
  });
});

// Auth middleware adds to context
app.use((req, res, next) => {
  const context = requestContext.getStore();
  if (!context) return next();

  const token = req.headers.authorization?.split(" ")[1];
  context.userId = decoded.sub;
  context.role = decoded.role;

  next();
});

// Handler reads context
app.get("/api/data", (req, res) => {
  const context = requestContext.getStore();

  console.log(`User ${context.userId} from IP ${context.ip}`);
  res.json({ data: [] });
});

// Service reads context
async function getData() {
  const context = requestContext.getStore();
  const user = await userRepo.getById(context.userId);
  return user;
}
```

#### Go (Built-in with Context)

```go
// Go has context.Context built into standard library

// Middleware creates context with values
func authMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    token := r.Header.Get("Authorization")
    user := verifyToken(token)

    // Create new context with user info
    ctx := context.WithValue(r.Context(), "userId", user.ID)
    ctx = context.WithValue(ctx, "userRole", user.Role)

    // Continue with new context
    next.ServeHTTP(w, r.WithContext(ctx))
  })
}

// Handler reads from context
func getDataHandler(w http.ResponseWriter, r *http.Request) {
  userId := r.Context().Value("userId").(string)
  userRole := r.Context().Value("userRole").(string)

  // Use userId and userRole
  data := getData(r.Context())
  w.Header().Set("Content-Type", "application/json")
  json.NewEncoder(w).Encode(data)
}

// Service receives context
func getData(ctx context.Context) []Data {
  userId := ctx.Value("userId").(string)
  // Use userId for database query
  return db.GetData(ctx, userId)
}
```

#### Python (Request/Application Context)

```python
from flask import Flask, request, g

app = Flask(__name__)

# Before request, set up context
@app.before_request
def before_request():
  token = request.headers.get('Authorization', '').split(' ')[-1]
  user = verify_token(token)

  # g is Flask's application context
  g.user_id = user['id']
  g.user_role = user['role']
  g.request_start = time.time()

# Handler can access g
@app.route('/api/data')
def get_data():
  user_id = g.user_id  # From context
  role = g.user_role   # From context

  data = get_data_service(user_id)
  return jsonify(data)

# Service receives user_id as parameter (or could access g)
def get_data_service(user_id):
  data = db.get_data(user_id)

  # Optional: can also access g directly
  # duration = time.time() - g.request_start

  return data
```

### Context Usage Example

```javascript
// Complete flow with context

// 1. Middleware creates context
app.use((req, res, next) => {
  const context = {
    requestId: uuid(),
    ip: req.ip,
    startTime: Date.now(),
    userId: null,
    userRole: null,
  };

  requestContext.run(context, () => next());
});

// 2. Auth middleware populates context
app.use((req, res, next) => {
  try {
    const token = req.headers.authorization?.split(" ")[1];
    const decoded = jwt.verify(token, SECRET);

    const context = requestContext.getStore();
    context.userId = decoded.sub;
    context.userRole = decoded.role;

    next();
  } catch (error) {
    res.status(401).json({ error: "Unauthorized" });
  }
});

// 3. Controller calls service
app.get("/api/users/:id", (req, res) => {
  const context = requestContext.getStore();
  const userId = req.params.id;

  userService
    .getUserProfile(userId)
    .then((user) => res.json(user))
    .catch((error) => res.status(400).json({ error: error.message }));
});

// 4. Service accesses context
async function getUserProfile(userId) {
  const context = requestContext.getStore();

  // Log with context info
  console.log(
    `User ${context.userId} requesting profile for ${userId} from ${context.ip}`,
  );

  const user = await userRepository.getById(userId);

  // Add audit log with context
  await auditLog.create({
    requestId: context.requestId,
    userId: context.userId,
    action: "VIEW_PROFILE",
    targetId: userId,
    timestamp: new Date(),
  });

  return user;
}

// 5. Repository can access context
async function getById(userId) {
  const context = requestContext.getStore();

  const result = await db.query("SELECT * FROM users WHERE id = ?", [userId]);

  // Add trace info
  console.log(
    `Query executed for request ${context.requestId} in ${Date.now() - context.startTime}ms`,
  );

  return result[0];
}
```

---

## 11. Complete Request Lifecycle Flow

### The Full Journey

```
┌─────────────────────────────────────────────────────────────┐
│                   HTTP REQUEST ARRIVES                       │
│                  POST /api/users (port 3000)                │
│        Body: { "name": "Alice", "email": "alice@..." }      │
│        Headers: { "Authorization": "Bearer token...", ... }  │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│           OPERATING SYSTEM FORWARDS TO SERVER                │
│              Server listening on port 3000                   │
│                  Request received at entry                   │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│              MIDDLEWARE 1: CORS MIDDLEWARE                   │
│  Check if origin is allowed                                 │
│  ✓ Origin is allowed                                        │
│  Add Access-Control-Allow-Origin header                     │
│  Call next()                                                │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│              MIDDLEWARE 2: LOGGING MIDDLEWARE                │
│  Record request method, path, start time                    │
│  Call next()                                                │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│          MIDDLEWARE 3: JSON PARSING MIDDLEWARE               │
│  Parse request body JSON string                             │
│  req.body = { name: "Alice", email: "alice@..." }          │
│  Call next()                                                │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│            MIDDLEWARE 4: AUTH MIDDLEWARE                     │
│  Extract token from Authorization header                    │
│  Verify token with JWT secret                              │
│  ✓ Token valid                                              │
│  Add user info to request context:                          │
│    context.userId = "user_123"                             │
│    context.userRole = "admin"                              │
│  Call next()                                                │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│             MIDDLEWARE 5: RATE LIMIT                         │
│  Check IP address request count                             │
│  ✓ Under limit (50 requests this minute)                    │
│  Increment counter                                          │
│  Call next()                                                │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                  ROUTE MATCHING                              │
│  URL path: /api/users                                       │
│  Method: POST                                               │
│  ✓ Matches route: app.post('/api/users', handler)          │
│  Call handler with (req, res)                              │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│            CONTROLLER/HANDLER: CREATE USER                   │
│  1. Extract data from req.body:                             │
│     name: "Alice"                                           │
│     email: "alice@example.com"                              │
│                                                              │
│  2. Deserialize JSON (already done by middleware)           │
│     req.body is native JavaScript object                    │
│                                                              │
│  3. Validate & Transform:                                   │
│     ✓ Email format valid                                    │
│     ✓ Name length OK                                        │
│     Transform: email to lowercase                           │
│     Transform: set createdAt = now                          │
│                                                              │
│  4. Call service layer:                                     │
│     await userService.createUser("Alice", "alice@...")     │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│           SERVICE LAYER: CREATE USER SERVICE                 │
│  1. Receive cleaned data from controller                    │
│                                                              │
│  2. Business Logic:                                         │
│     - Hash password (not shown in this example)            │
│     - Check if email already exists via repository         │
│     - Validate business rules                              │
│     ✓ All checks passed                                     │
│                                                              │
│  3. Call repository to persist data:                        │
│     const user = await userRepository.create({             │
│       name: "Alice",                                        │
│       email: "alice@example.com"                            │
│     })                                                       │
│                                                              │
│  4. Additional processing:                                  │
│     - Send welcome email                                    │
│     - Log action with context                               │
│     - Return user object                                    │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│       REPOSITORY LAYER: USER REPOSITORY                      │
│  1. Receive data to persist:                                │
│     { name: "Alice", email: "alice@example.com" }          │
│                                                              │
│  2. Database Operation:                                     │
│     SQL: INSERT INTO users (name, email) VALUES (?, ?)     │
│     Execute query on database                              │
│     ✓ Insert successful                                     │
│     Database returns generated ID: 42                       │
│                                                              │
│  3. Return result:                                          │
│     { id: 42, name: "Alice", email: "alice@..." }         │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│        BACK TO SERVICE LAYER (with result)                  │
│  Repository returned: { id: 42, name: "Alice", ... }      │
│                                                              │
│  Continue processing:                                       │
│  - Send email to alice@example.com                         │
│  - Return user (without sensitive data)                     │
│  Return to controller: { id: 42, name: "Alice", ... }     │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│       BACK TO CONTROLLER (with result)                      │
│  Service returned user: { id: 42, name: "Alice", ... }    │
│                                                              │
│  Determine HTTP response:                                   │
│  - Success ✓                                                │
│  - Status code: 201 Created                                 │
│  - Response body: { success: true, data: user }            │
│                                                              │
│  res.status(201).json({ success: true, data: user })      │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│         MIDDLEWARE 6: LOGGING MIDDLEWARE (RESPONSE)         │
│  Record response status: 201                                │
│  Record duration: 145ms                                     │
│  Log: POST /api/users - 201 - 145ms                        │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│       MIDDLEWARE 7: ERROR HANDLER (if error occurred)       │
│  No error occurred, skip                                    │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│              RESPONSE SENT TO CLIENT                         │
│  HTTP/1.1 201 Created                                       │
│  Content-Type: application/json                             │
│  Access-Control-Allow-Origin: https://frontend.com         │
│                                                              │
│  {                                                           │
│    "success": true,                                         │
│    "data": {                                                │
│      "id": 42,                                              │
│      "name": "Alice",                                       │
│      "email": "alice@example.com"                           │
│    }                                                         │
│  }                                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 12. Best Practices & Design Patterns

### Practice 1: Single Responsibility

Each layer has one job:

```
Controller:
  - Extract request data
  - Validate/transform
  - Call service
  - Return HTTP response

Service:
  - Implement business logic
  - Orchestrate operations
  - Call repositories
  - Return result (not HTTP response)

Repository:
  - Database operations only
  - No business logic
  - No HTTP knowledge
```

### Practice 2: Dependency Injection

Pass dependencies instead of creating them:

```javascript
// BAD: Dependencies created inside
class UserService {
  async createUser(data) {
    const userRepo = new UserRepository(); // Wrong!
    const emailService = new EmailService(); // Wrong!
    const user = await userRepo.create(data);
    await emailService.send(...);
    return user;
  }
}

// GOOD: Dependencies injected
class UserService {
  constructor(userRepository, emailService) {
    this.userRepo = userRepository;
    this.emailService = emailService;
  }

  async createUser(data) {
    const user = await this.userRepo.create(data);
    await this.emailService.send(...);
    return user;
  }
}

// Usage
const userRepo = new UserRepository();
const emailService = new EmailService();
const userService = new UserService(userRepo, emailService);

// Easy to test - inject mocks
const mockRepo = { create: jest.fn() };
const mockEmail = { send: jest.fn() };
const testService = new UserService(mockRepo, mockEmail);
```

### Practice 3: Error Handling

Errors should bubble up properly:

```javascript
// Repository throws database errors
async function getById(id) {
  try {
    return await db.query("SELECT * FROM users WHERE id = ?", [id]);
  } catch (error) {
    throw new Error(`Database error: ${error.message}`);
  }
}

// Service catches and decides
async function getUserProfile(userId) {
  try {
    const user = await userRepository.getById(userId);
    if (!user) {
      throw new Error("User not found"); // Business logic error
    }
    return user;
  } catch (error) {
    // Could log, transform, etc.
    throw error; // Re-throw for controller
  }
}

// Controller catches and formats HTTP response
app.get("/users/:id", async (req, res) => {
  try {
    const user = await userService.getUserProfile(req.params.id);
    res.json(user);
  } catch (error) {
    // Transform to HTTP response
    if (error.message === "User not found") {
      res.status(404).json({ error: error.message });
    } else {
      res.status(500).json({ error: "Internal server error" });
    }
  }
});
```

### Practice 4: Middleware Order

Standard recommended order:

```javascript
// 1. Security/Setup (early)
app.use(cors());                    // CORS headers
app.use(helmet());                  // Security headers

// 2. Parsing/Data (before auth)
app.use(express.json());            // Parse JSON
app.use(express.urlencoded(...));   // Parse form

// 3. Identification/Auth
app.use(authMiddleware);            // Authenticate user

// 4. Rate limiting (after auth)
app.use(rateLimit);                 // Rate limit

// 5. Logging (optional, early or late)
app.use(loggingMiddleware);         // Log requests

// 6. Routes
app.get('/api/users', getUsersHandler);
app.post('/api/users', createUserHandler);

// 7. Error handling (last!)
app.use(errorHandler);              // Catch all errors
```

### Practice 5: Request Context for Tracing

Use context for debugging:

```javascript
// Middleware sets up context
app.use((req, res, next) => {
  const context = {
    requestId: uuid(),
    startTime: Date.now(),
    userId: null,
    path: req.path,
    method: req.method,
  };

  requestContext.run(context, () => next());
});

// Every layer can access and log with context
function log(message) {
  const context = requestContext.getStore();
  console.log(
    JSON.stringify({
      requestId: context.requestId,
      userId: context.userId,
      duration: Date.now() - context.startTime,
      message,
    }),
  );
}

// Makes debugging easier
// Can track single request through entire system
// Correlate logs across services
```

---

## 13. Key Takeaways & Summary

### The Three Layers

| Layer          | Responsibility | Input            | Output          |
| -------------- | -------------- | ---------------- | --------------- |
| **Controller** | HTTP concerns  | Request object   | HTTP response   |
| **Service**    | Business logic | Clean data       | Result data     |
| **Repository** | Data access    | Query parameters | Database result |

### Middleware Characteristics

- Runs for every request (or filtered requests)
- Has access to request & response objects
- Can modify request/response
- Can end the chain with response
- Executed in order registered
- Order matters (especially CORS, Auth, Error)

### Request Context

- Storage scoped to single request
- Accessible across all middleware/handlers
- Dies after response sent
- Useful for passing data (userId, requestId)
- Avoid global state with context

### Separation of Concerns Benefits

- Testability (test each layer independently)
- Reusability (service used from API, CLI, jobs)
- Maintainability (one responsibility per layer)
- Scalability (easy to add features)
- Debugging (know where to look)

### Complete Request Flow

```
Request → CORS → Auth → Logging → Route → Controller → Service → Repository →
Database → Service → Controller → Logging → Error Handler → Response
```

---

## References & Links

**Video Source:**

- Controllers, Services, Repositories, Middlewares & Request Context (Sriniously): https://www.youtube.com/watch?v=hyc-7w3pee8

**Sriniously Channel & Playlist:**

- Sriniously Channel: https://www.youtube.com/channel/UCYkDx5W-v5qjkVVm1MrA1-w
- Backend from First Principles Playlist: https://www.youtube.com/playlist?list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1

**Related Videos in Series:**

- Serialization & Deserialization: https://www.youtube.com/watch?v=VNzXHJiRKxI&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1
- Validations and Transformations: https://www.youtube.com/watch?v=qedj_JjjL-U&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1
- Authentication and Authorization: https://www.youtube.com/watch?v=A95rliroC8Q&list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1

**Design Patterns & Architecture:**

- Separation of Concerns: https://en.wikipedia.org/wiki/Separation_of_concerns
- Repository Pattern: https://martinfowler.com/eaaCatalog/repository.html
- Dependency Injection: https://en.wikipedia.org/wiki/Dependency_injection
- Middleware Pattern: https://en.wikipedia.org/wiki/Middleware

**Framework Documentation:**

- Express.js Middleware: https://expressjs.com/en/guide/using-middleware.html
- Go HTTP Context: https://golang.org/pkg/context/
- Flask Request Context: https://flask.palletsprojects.com/en/2.0.x/appcontext/
- Django Middleware: https://docs.djangoproject.com/en/4.0/topics/http/middleware/

**Best Practices:**

- Clean Architecture: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- SOLID Principles: https://en.wikipedia.org/wiki/SOLID
- Error Handling Best Practices: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html

**Testing:**

- Unit Testing Patterns: https://martinfowler.com/articles/injection.html
- Jest Testing Library: https://jestjs.io/
- Dependency Injection for Testing: https://en.wikipedia.org/wiki/Dependency_injection#Benefits
