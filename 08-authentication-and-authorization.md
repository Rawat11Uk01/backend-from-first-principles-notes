# Authentication and Authorization for Backend Engineers

_Comprehensive In-Depth Study Guide_

Source video: "8. Authentication and authorization for backend engineers" – Sriniously

---

## Table of Contents

1. Foundational Concepts: Authentication vs Authorization
2. Historical Evolution of Authentication
3. Core Components (Sessions, JWTs, Cookies)
4. Authentication Methods in Detail
5. Authorization Concepts & Models
6. OAuth 2.0 & OpenID Connect
7. Password Storage & Security
8. API Key Authentication
9. Modern Authentication Challenges
10. Best Practices & Recommendations
11. Security Resources & References

---

## 1. Foundational Concepts: Authentication vs Authorization

### Two Sentence Summary

**Authentication:** Answers the question "Who are you?" - A mechanism to assign an identity to a subject in a given context.

**Authorization:** Answers the question "What can you do?" - The process of determining what capabilities and permissions a user has in a particular context.

### The Difference

#### Authentication

- **Purpose:** Verify the identity of a user
- **When:** At login/initial request
- **Question:** "Are you really who you claim to be?"
- **Mechanism:** Username/password, biometrics, tokens, API keys
- **Result:** User identity is confirmed or denied

```
User logs in with email and password
→ System verifies credentials
→ Authentication succeeds: User is who they say they are
```

#### Authorization

- **Purpose:** Verify what a user can do
- **When:** On every request after authentication
- **Question:** "Do you have permission to access this resource?"
- **Mechanism:** Role-based access control (RBAC), permission checks, ACLs
- **Result:** User can or cannot perform the requested action

```
Authenticated user requests: "Delete all users"
→ System checks: Does user have admin role?
→ Authorization check: User is admin → Delete allowed
  OR User is regular member → Delete denied (403 Forbidden)
```

### Context Matters

Both authentication and authorization depend on **context**:

```
Same person, different contexts:

Context 1: Company A platform
  Authentication: User is John (CEO of Company A)
  Authorization: Can access all company data, can fire employees

Context 2: Social media platform
  Authentication: User is John (person)
  Authorization: Can only view own profile, can edit own posts

Context 3: Operating system
  Authentication: User is John (computer user)
  Authorization: Can install software, cannot access system files
```

---

## 2. Historical Evolution of Authentication

Understanding how authentication evolved helps us understand why we have the systems we do today.

### Phase 1: Pre-Industrial Societies (Ancient Times)

**Mechanism:** Implicit Trust & Human Recognition

How it worked:

- Identities were implicit (assumed, not proven)
- A trusted community member (village elder) would vouch for a person
- Deals were sealed with a **handshake** symbolizing mutual recognition
- Trust was based on personal acquaintance and reputation

**Limitations:**

- Didn't scale beyond familiar communities
- When populations grew and strangers interacted, implicit trust failed
- No mechanism for proving identity to unknown people

### Phase 2: Medieval Period - The Seal Era (1000s-1500s)

**Mechanism:** Physical Seals (Cryptographic Tokens)

How it worked:

- Wax seals attached to documents and letters
- Each seal had a **unique pattern** (like a signature)
- Seal ownership implied authentication authority
- Seal on a document proved authenticity

**Advantages:**

- Scaled beyond personal relationships
- Physical token-based authentication
- Could authenticate identity without knowing the person

**Vulnerabilities:**

- Seals could be **forged** (first documented authentication bypass attacks)
- Forgery led to evolution of security: watermarks, encrypted codes
- Set foundation for cryptographic thinking

**Impact:** Marked transition from implicit to explicit authentication

### Phase 3: Industrial Revolution - Passwords (1800s)

**Mechanism:** Shared Secrets & Passphrases

How it worked:

- Telegraph operators used pre-agreed **passphrases** (early passwords)
- Same static password used repeatedly
- Principle: "Something you know" (mental possession)

**Evolution:**

- Marked shift from "something you have" (seal) to "something you know" (password)
- Based on shared secrets between two parties
- More secure than physical tokens

**Limitation:** Static passwords are susceptible to discovery

### Phase 4: Early Digital Era (1960s) - Mainframes

**Key Event:** CTSS Password System (1961)

At MIT's Project MAC, researchers developed CTSS (Compatible Time Sharing System) - the first multi-user computing system.

**The Problem:**

- Multiple users needed to use one computer without sharing data
- Needed a way to isolate user data

**The Solution:**

- Introduced passwords for multi-user systems
- Stored passwords in **plain text** (critical vulnerability!)

**The Incident:**

- Someone printed the entire password file
- Every user's password was exposed
- This incident revealed the danger of plain text password storage

**The Impact:**

- Marked genesis of secure password storage mechanisms
- Led to philosophy: "Passwords should never be stored in plain text"
- Motivated development of hashing algorithms

### Phase 5: Hash-Based Password Storage (1970s)

**Mechanism:** Irreversible Hashing

How hashing works:

```
Plain text password: "myPassword123"
    ↓ (through hash function)
Hash: "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"

Properties:
- Same input always produces same output
- Fixed length (doesn't matter if input is 3 chars or 1000 chars)
- Irreversible (cannot reverse hash to get original password)
- Deterministic (same password = same hash every time)
```

**Why it's better:**

- Server only stores hash, not password
- Even if database is compromised, attacker gets useless hashes
- When user logs in: server hashes their input and compares with stored hash

**Implementation in database:**

```
User table:
  id | username | password_hash
  1  | alice    | a1b2c3d4e5f6g7h8...
  2  | bob      | f7g8h9i0j1k2l3m4...
```

### Phase 6: Asymmetric Cryptography Era (1970s)

**Key Contributors:** Whitfield Diffie & Martin Hellman

**Discovery:** Diffie-Hellman Key Exchange

How it changed authentication:

- For the first time, two parties could establish a **shared secret over an untrusted medium**
- No need to meet in person or send secret through secure channel
- Could agree on secret while eavesdroppers listened

**Impact:**

- Foundation of all modern authentication
- Led to Public Key Infrastructure (PKI)
- Enabled digital signatures, TLS/SSL encryption
- Made secure online communication possible

**Kerberos Protocol (1980s)**

- Used **tickets** for authentication
- Relied on trusted third parties to issue tickets
- Verified both user and service identity
- Precursor to modern token-based authentication (like JWT)

### Phase 7: Internet Era & Multi-Factor Authentication (1990s)

**Problem:** Simple username/password vulnerable to:

- Brute force attacks (trying many passwords)
- Dictionary attacks (trying common passwords)
- Phishing (tricking users into revealing passwords)

**Solution:** Multi-Factor Authentication (MFA)

Combines three authentication factors:

1. **Something you know** - Passwords, PINs, security questions
2. **Something you have** - Smart cards, OTP generators, hardware tokens, phones
3. **Something you are** - Biometric data: fingerprints, retina scans, facial recognition

**Example MFA flows:**

```
Login attempt:
1. Enter username & password (something you know)
2. Enter 6-digit code from authenticator app (something you have)
→ Authentication successful

OR

1. Enter username & password
2. Scan fingerprint (something you are)
→ Authentication successful
```

**Biometric Authentication Challenges:**

- False positives: System accepts unauthorized user
- False negatives: System rejects authorized user
- Template security: Biometric data theft
- Privacy concerns: Body part scanning

### Phase 8: Cloud & Distributed Systems Era (2000s-2010s)

**Drivers:**

- Cloud computing growth
- Mobile devices becoming primary client
- API-based architectures
- Microservices

**Need:** Advanced authentication frameworks that work across systems

**Solutions emerged:**

- OAuth 2.0 (delegated authorization)
- OpenID Connect (on top of OAuth 2.0)
- JWTs (stateless tokens)
- Zero Trust Architecture
- Passwordless authentication (WebAuthn)

### Phase 9: Future Directions (2020s-Present)

#### Decentralized Identity

Uses blockchain/distributed ledgers:

- User controls their own identity
- No central authority needed
- Robust security features
- Still in early experimental stages

#### Behavioral Biometrics

Authenticates based on behavior patterns:

- Typing patterns
- Mouse movement patterns
- Device usage patterns
- Gait recognition
- Still emerging technology

#### Post-Quantum Cryptography

**Problem:** Quantum computers will break current encryption

When quantum computers become common:

- RSA (used everywhere) becomes insecure
- All public key cryptography becomes vulnerable
- All passwords/data encrypted with current algorithms at risk

**Solution:** Post-quantum cryptographic algorithms

- Designed to resist quantum computer attacks
- Already being standardized by NIST
- Need implementation before quantum computers become practical
- Organizations should start preparing now

**Timeline concern:**

- Quantum computers not yet practical for breaking cryptography
- But "harvest now, decrypt later" attacks possible
- Adversaries collecting encrypted data now to decrypt later
- Need quantum-resistant algorithms in place before that happens

---

## 3. Core Components (Sessions, JWTs, Cookies)

Three critical components of modern authentication that appear in almost every system.

### A. Sessions

#### What is a Session?

A **session** is a temporary server-side context that maintains state for a specific user/client.

A session is a way for a server to remember a user across multiple HTTP requests.
Because HTTP is stateless, sessions exist to add continuity.
**Why Sessions?**

HTTP protocol is **stateless** by design:

- Each request is independent
- Server has no memory of previous requests
- Works fine for static content (reading web pages)

But modern web needs **statefulness**:

- E-commerce: Remember items in shopping cart
- Social media: Keep user logged in while navigating pages
- Applications: Maintain user context across requests

**Session creation enables:**

- Server to remember who you are
- Personalized experience
- Maintaining user context between requests

#### Session-Based Authentication Flow

```
Step 1: User Login
  Client sends: username + password
  ↓
Step 2: Validation
  Server validates credentials
  ↓
Step 3: Session Creation
  Server creates unique Session ID (random string)
  Server stores in persistent storage:
    {
      "session_id": "abc123xyz789",
      "user_id": 42,
      "username": "alice",
      "role": "admin",
      "cart_items": [...],
      "created_at": "2025-01-06T09:00:00Z",
      "expires_at": "2025-01-06T10:00:00Z"
    }
  ↓
Step 4: Send Session ID to Client
  Server sends Session ID to client in a cookie
  Client's browser stores the cookie
  ↓
Step 5: Subsequent Requests
  Client's browser automatically includes cookie in every request
  Server extracts Session ID from cookie
  Server looks up session in persistent storage
  Server identifies user and authorizes request
  ↓
Step 6: Session Expiry
  After 15 minutes (example), session expires
  Server requires new login
  New Session ID is generated
```

#### Session Storage Evolution

**File-Based Sessions (Early)**

- Stored in files on server disk
- Simple implementation
- Problems:
  - Slow disk I/O
  - Doesn't work with multiple servers
  - Lost on server restart

**Database Sessions (Middle)**

- Stored in relational database
- Persistent across server restarts
- Faster than file I/O
- Problems:
  - Database queries add latency
  - Database becomes bottleneck at scale

**Distributed In-Memory Sessions (Modern)**

- Stored in Redis or Memcached
- In-memory storage (much faster than disk)
- Distributed (shared across servers)
- Can survive server restart
- Used in production systems today

#### Session Data Structure

What gets stored with a session ID:

```
Session storage (Redis/Database):
{
  "session_id": "a7c3f9e2-1b2d-4c5e-8f9a-3b2c1d5e6f7a",
  "user_id": 42,
  "email": "alice@example.com",
  "username": "alice",
  "role": "admin",
  "permissions": ["read", "write", "delete"],
  "last_activity": "2025-01-06T09:15:23Z",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0 (Chrome)",
  "created_at": "2025-01-06T09:00:00Z",
  "expires_at": "2025-01-06T10:00:00Z"
}
```

More detailed

# 🔐 Session Storage Evolution — Deep Explanation

Sessions exist because **HTTP is stateless**, but applications are **stateful**.
The entire evolution is about answering one question better and better:

> **“Where do we store user state so it is fast, safe, and works at scale?”**

---

# 0️⃣ The Core Requirement (Before Any Storage Choice)

A session system must:

1. Identify the user across requests
2. Be fast (used on _every_ request)
3. Work with multiple servers
4. Survive failures reasonably
5. Not overload critical systems

Early systems solved **1**, later systems solved **2–5**.

---

# 1️⃣ File-Based Sessions (Early Era)

### 📌 What This Actually Looked Like

Sessions were stored as **files on the same server** running the app.

Example (conceptual):

```
/var/sessions/
 ├── sess_8f9a2c
 ├── sess_b71d3e
 ├── sess_44ac91
```

Each file contained serialized session data:

```
user_id=42
role=admin
cart_items=3
```

---

### 🧠 Why This Was the First Choice

- Servers were **single-machine**
- Traffic was low
- Disk was “good enough”
- Simplicity mattered more than scale

Framework logic:

```
Request → read file → process → write file
```

No extra systems needed.

---

### ⚠️ Deep Problems (Why This Failed)

#### ❌ 1. Disk I/O Is Slow by Nature

- Disk access = milliseconds
- Every request needs session read
- Under load → disk queue → latency spikes

Sessions are **hot data**, disk is **cold storage**.

---

#### ❌ 2. Completely Breaks with Load Balancing

Example:

```
User logs in → Server A creates session file
Next request → Load balancer sends to Server B
Server B → file not found → user logged out
```

To fix this, people tried:

- Sticky sessions ❌ (bad scalability)
- Shared network filesystem ❌ (slow & fragile)

This is where file-based sessions hit a **hard wall**.

---

#### ❌ 3. Server Restart = Session Wipeout

- Restart server
- Files deleted / lost
- All users logged out

Unacceptable for real systems.

---

### 📌 Conclusion

> File-based sessions work **only** for:

- One server
- Low traffic
- Non-critical apps

---

# 2️⃣ Database Sessions (Middle Era)

When apps became **multi-server**, engineers moved sessions to a **central database**.

---

### 📌 What Changed Architecturally

Instead of files:

```
sessions table
--------------------------------
session_id | user_id | data | expiry
```

All servers could now do:

```
SELECT * FROM sessions WHERE session_id = ?
```

---

### 🧠 Why This Was a Big Improvement

✔️ Sessions survived server restarts
✔️ All app servers could access them
✔️ Centralized and consistent
✔️ Easy to back up

This solved the **correctness problem**.

---

### ⚠️ Deep Problems (Why This Also Failed)

#### ❌ 1. Databases Are Too Slow for Sessions

Sessions are accessed:

- On **every request**
- Often multiple times per request

Databases are designed for:

- Durable data
- Complex queries
- Transactions

Not for **millions of tiny reads per second**.

---

#### ❌ 2. Database Becomes the Bottleneck

At scale:

- Every request = DB query
- DB connections exhausted
- Latency spikes
- Entire system slows down

Even with indexing:

> **Session reads drown real business queries**

---

#### ❌ 3. Sessions Pollute the Database

- Session data is:

  - Temporary
  - High churn
  - Low value

- Databases prefer:

  - Stable
  - Long-lived
  - Structured data

Mixing the two is bad design.

---

### 📌 Conclusion

> Database sessions solved _sharing_ but broke _performance_.

---

# 3️⃣ Distributed In-Memory Sessions (Modern Era)

In-memory means data is stored in RAM instead of on disk so it can be accessed extremely fast.

This is where systems **finally aligned with reality**.

---

### 📌 What This Means

Sessions are stored in **distributed in-memory stores** like:

- Redis
- Memcached

Architecture:

```
Client
  ↓
Load Balancer
  ↓
App Server
  ↓
Redis (sessions)
```

---

### 🧠 Why Memory Wins

Memory access:

- Nanoseconds locally
- Microseconds over network

Compared to:

- Disk → milliseconds
- Database → milliseconds + locks

Sessions are **hot, short-lived, frequently accessed** → perfect for memory.

---

### ⚙️ How Modern Session Flow Works

1. User logs in
2. App generates session ID
3. Session data stored in Redis
4. Session ID sent to client
5. Every request:

   - App fetches session from Redis
   - Extremely fast
   - Same for all servers

---

### ✅ Why This Is the Modern Standard

✔️ Extremely fast
✔️ Works with many servers
✔️ Survives app crashes
✔️ Centralized
✔️ Designed for high throughput

This is why:

- Netflix
- Amazon
- Large APIs
  all use **in-memory session stores**.

---

### ⚠️ Important Trade-offs (Advanced Understanding)

Redis is:

- Fast
- Not fully durable by default

So systems add:

- Replication
- Persistence
- TTL-based expiration

This accepts:

> **Small risk of session loss in exchange for massive performance gains**

Which is acceptable.

---

# 🧠 Evolution Pattern (Very Important Insight)

```
As traffic ↑
Storage moved closer to RAM
```

| Stage    | Storage      | Speed  | Scalability |
| -------- | ------------ | ------ | ----------- |
| Files    | Disk         | Slow   | ❌          |
| Database | Disk + Index | Medium | ⚠️          |
| Redis    | Memory       | Fast   | ✅          |

---

## 🧠 Final Mental Model (Lock This In)

> **Sessions are hot, temporary state.
> Hot state belongs in memory, not on disk.**

---

## 📌 Final One-Line Summary

> Session storage evolved from files → databases → distributed in-memory systems as applications needed faster access, horizontal scaling, and resilience in real-world traffic.

---

### B. JWTs (JSON Web Tokens)

#### Historical Context: Why JWTs Were Created

**Problem with Sessions in Distributed Systems (Mid-2000s)**

As systems grew globally:

1. **Memory Overhead**

   - Millions of users = millions of sessions in memory
   - Each session storing user data, cart, preferences, etc.
   - Massive storage costs

2. **Replication Across Regions**

   - Server in US, server in Europe, server in Asia
   - User in US logs in at US server
   - User requests go to Europe server
   - Session data needs to replicate across regions
   - Creates latency (network delay)
   - Creates consistency issues (which copy is authoritative?)

3. **Synchronization Complexity**
   - Keeping session data in sync across distributed systems is hard
   - Network delays cause inconsistencies
   - Expensive operations

**Solution:** Move from stateful to stateless authentication

#### What is a JWT?

JWT = JSON Web Token

A **self-contained, stateless token** that carries all information needed to authenticate a user.

**Key Innovation:** Token contains user data + cryptographic signature

No need to look up anything on server!

#### JWT Structure

A JWT has three parts, separated by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Part 1 (Header):

```json
{
  "alg": "HS256", // Algorithm used to sign
  "typ": "JWT" // Type is JWT
}
```

Part 2 (Payload):

```json
{
  "sub": "42", // Subject (user ID)
  "iat": 1516239022, // Issued at (unix timestamp)
  "exp": 1516242622, // Expiration time
  "name": "Alice", // User name (optional)
  "email": "alice@example.com", // Email (optional)
  "role": "admin", // Role (optional)
  "permissions": ["read", "write", "delete"] // Permissions (optional)
}
```

Part 3 (Signature):

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

The signature proves the token wasn't tampered with.

#### How JWT Authentication Works

```
Step 1: User Login
  Client sends: email + password
  ↓
Step 2: Validation
  Server verifies credentials
  ↓
Step 3: JWT Generation
  Server creates JWT token:
    - Header: algorithm info
    - Payload: user_id, role, permissions, expiry
    - Signature: signed with server's secret key
  ↓
Step 4: Send JWT to Client
  Server sends JWT to client
  Client stores JWT in:
    - Local storage (risky - accessible to JavaScript)
    - Session storage (lost on page refresh)
    - Cookie (secure with httpOnly flag)
  ↓
Step 5: Subsequent Requests
  Client sends JWT in Authorization header:
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ↓
Step 6: Token Verification
  Server extracts JWT from header
  Server verifies signature using its secret key
  If signature is valid and not expired:
    - Extract user_id from token
    - Authorize request
    - Send response
  If signature is invalid or expired:
    - Return 401 Unauthorized
    - Client must login again
```

#### JWT Payload Fields Explained

**Standard Claims (RFC 7519):**

| Claim | Full Name  | Purpose                    | Example                              |
| ----- | ---------- | -------------------------- | ------------------------------------ |
| `sub` | Subject    | User ID                    | `"42"` or `"user:alice@example.com"` |
| `iat` | Issued At  | When token was created     | `1516239022` (unix timestamp)        |
| `exp` | Expiration | When token expires         | `1516242622`                         |
| `aud` | Audience   | Who token is for           | `"my-app"`                           |
| `iss` | Issuer     | Who created token          | `"https://auth.example.com"`         |
| `nbf` | Not Before | Earliest token can be used | `1516238000`                         |
| `jti` | JWT ID     | Unique token identifier    | `"unique-token-id-123"`              |

**Custom Claims:**

```json
{
  "role": "admin",
  "permissions": ["read", "write", "delete"],
  "organization_id": "org_123",
  "department": "engineering"
}
```

#### JWT Advantages

1. **Statelessness**

   - No server-side session storage needed
   - Server doesn't store anything about the token
   - Reduces storage costs

2. **Scalability**

   - Works perfectly in distributed systems
   - Multiple servers, different regions
   - Each server has same secret key, can verify any JWT
   - No need to sync session data across servers

3. **Portability**

   - JWT is a string (base64 encoded)
   - Can be passed anywhere: headers, body, URL parameters
   - Works across different domains

4. **Mobile-Friendly**

   - Cookies don't work well for mobile apps
   - JWTs can be stored and sent by mobile apps
   - No browser cookie mechanism needed

5. **Microservices-Friendly**
   - One auth service issues JWT
   - Multiple microservices verify JWT with same secret
   - Services don't need to talk to each other for auth

#### JWT Disadvantages

1. **Token Revocation Problem**

Problem:

- Once JWT is issued, server can't revoke it
- Token valid until expiration
- If user's account is hacked, attacker can still use old token

Example:

```
User logs in: Get JWT (expires in 1 hour)
15 minutes later: User's account is hacked
User contacts support: "Block my account"
Support: "Your token is still valid for 45 more minutes"
Attacker: Can still use token for 45 minutes
```

Solutions:

- Short expiration times (1 hour)
- Token blacklisting (maintain list of revoked tokens)
- Use hybrid approach (see below)

2. **Data Size**

Problem:

- Payload contains user data (name, role, permissions, etc.)
- All this data is sent with every request
- Larger token = more bandwidth

Example:

```
Session ID: "abc123" (6 bytes)
JWT with full user data: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." (1000+ bytes)
```

3. **Secret Management**

Problem:

- Server must keep secret key secure
- If secret is compromised, attacker can create fake JWTs
- No way to know if JWT was issued by server or attacker

Example:

```
Server secret compromised:
  Attacker now knows secret key
  Attacker creates fake JWT: { user_id: 1, role: "admin" }
  Attacker signs it with the compromised secret
  Server verifies signature: Valid! ✓
  Attacker gains admin access
```

---

### C. Hybrid Approach: Best of Both Worlds

Combines session-based and JWT-based authentication:

```
Step 1: User Logs In
  Client sends: email + password
  ↓
Step 2: Server Issues JWT
  Server creates JWT with user data
  Server stores JWT ID in blacklist storage
  ↓
Step 3: Client Receives JWT
  Client stores JWT securely (in httpOnly cookie)
  ↓
Step 4: Subsequent Requests
  Client sends JWT
  ↓
Step 5: Server Validates JWT
  Step 5a: Verify JWT signature
    If signature invalid → 401 Unauthorized
  Step 5b: Check if JWT is blacklisted
    If blacklisted (revoked) → 401 Unauthorized
  Step 5c: Check if JWT expired
    If expired → 401 Unauthorized
  If all pass → Authorize request
```

**Benefits:**

- Stateless nature of JWT (scalability)
- Ability to revoke tokens (security)

**Tradeoff:**

- Still need to look up blacklist
- Reduces scalability benefit
- But only for actively managed revocations, not every request

#### When to Use Which?

**Question:** If hybrid approach requires blacklist lookup, why not just use sessions?

**Good Point!** Industry advice:

**Don't implement authentication yourself!**

Use third-party authentication providers:

- OAuth providers (Google, GitHub, Microsoft)
- Dedicated auth services (Auth0, Clerk, Supabase Auth, Firebase Auth)

**Why?**

- They handle all the complexity
- They handle security concerns
- They handle the tradeoffs
- You focus on business logic

**If you must implement:**

1. Use sessions for web apps (browser-based)
2. Use JWT for APIs and mobile apps (if you understand the tradeoffs)
3. Implement token blacklist for revocation needs

---

### D. Cookies

#### What is a Cookie?

A **cookie** is a small piece of data stored on the client's browser that:

- Is automatically sent with every request to the same domain
- Is managed by the browser (not JavaScript, by default)
- Has specific security properties

#### How Cookies Work

```
Step 1: Server Sets Cookie
  Server sends response header:
    Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure

Step 2: Browser Stores Cookie
  Browser automatically stores the cookie
  Saves it in browser's cookie storage

Step 3: Client Makes Subsequent Request
  Browser sees it's going to same domain
  Browser automatically includes cookie in request header:
    Cookie: session_id=abc123

Step 4: Server Reads Cookie
  Server extracts session_id from request
  Server uses it to look up session data
  Server identifies user
```

#### Cookie Attributes

```
Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict; Max-Age=3600

- session_id=abc123     : Cookie name and value
- Path=/                : Only send to routes starting with /
- HttpOnly              : JavaScript CANNOT access cookie (security)
- Secure                : Only send over HTTPS (not HTTP)
- SameSite=Strict       : Only send on same-site requests (prevents CSRF)
- Max-Age=3600          : Cookie expires in 3600 seconds (1 hour)
```

#### Cookie Security Best Practices

```
Insecure:
  Set-Cookie: session_id=abc123

Secure:
  Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```

**Why each attribute matters:**

| Attribute | Protects Against                                  |
| --------- | ------------------------------------------------- |
| HttpOnly  | XSS attacks (JavaScript accessing cookie)         |
| Secure    | Man-in-the-middle (sending over unencrypted HTTP) |
| SameSite  | CSRF attacks (cross-site request forgery)         |
| Path      | Limiting cookie scope to specific routes          |

#### Cookies in Authentication

**Session-Based (with cookies):**

```
Server issues session ID in cookie
Browser automatically sends cookie with every request
Simple: No need to manually handle token
Secure: Can use HttpOnly flag
```

**JWT-Based (alternative to cookies):**

```
Server issues JWT (returned as JSON)
Client stores JWT (in local storage or memory)
Client manually adds JWT to headers:
  Authorization: Bearer <jwt_token>

OR

Server issues JWT in cookie
Client doesn't need to do anything
Browser sends cookie automatically
```

**Most modern approach:**

- Use JWT, store in httpOnly cookie
- Browser sends automatically
- JavaScript cannot access (security)
- Looks like cookie-based, but token-based benefits

---

## 4. Authentication Methods in Detail

### Method 1: Stateful Authentication (Session-Based)

#### How It Works

```
Browser (Client)                Server
    ↓                            ↓
1. POST /login
    username: alice
    password: secret123
    ───────────────────────→
                            2. Verify credentials
                               Generate Session ID
                               Store in Redis:
                               {
                                 session_id: "abc123",
                                 user_id: 42,
                                 role: "admin",
                                 created_at: now,
                                 expires_at: now + 1h
                               }
    ←───────────────────────
3. Set-Cookie:
   session_id=abc123;
   HttpOnly; Secure

Browser stores cookie

    ↓
4. GET /api/user-profile
   Cookie: session_id=abc123
    ───────────────────────→
                            5. Extract session_id
                               Look up in Redis
                               Found: user_id=42
                               User authenticated ✓
    ←───────────────────────
6. Return user data

7. Sessions expire after 1 hour
   User needs to login again
```

#### Pros

- **Centralized Control**

  - Server controls all sessions
  - Can revoke session immediately
  - Can see all active sessions for a user
  - Can log out users from backend

- **Real-Time Information**

  - Server knows exactly who is logged in
  - Can track user activity
  - Can detect suspicious activity

- **Suited for Modern Traffic**
  - Works well for web apps (browser-based)
  - Can handle session theft gracefully

#### Cons

- **Limited Scalability**

  - Session data stored on server
  - Multiple servers need to share session storage
  - Session storage (Redis) becomes bottleneck
  - Replication across regions adds latency

- **Operational Complexity**
  - Need to maintain session store (Redis/Database)
  - Need to handle session expiry
  - Need to handle session synchronization
  - More moving parts

#### When to Use

- Web applications with browsers
- Internal tools
- Applications with strict session requirements
- When you need real-time control over user sessions

---

### Method 2: Stateless Authentication (JWT-Based)

#### How It Works

```
Client (Browser/App)           Server
    ↓                            ↓
1. POST /login
   email: alice@example.com
   password: secret123
    ───────────────────────→
                            2. Verify credentials
                               Create JWT payload:
                               {
                                 sub: "42",
                                 email: "alice@example.com",
                                 role: "admin",
                                 iat: 1234567890,
                                 exp: 1234571490  // 1 hour later
                               }
                               Sign with secret key
                               JWT = header.payload.signature
    ←───────────────────────
3. Return JWT as JSON
   {
     token: "eyJhbGciOi..."
   }

Client stores JWT in memory/localStorage

    ↓
4. GET /api/user-profile
   Authorization: Bearer eyJhbGciOi...
    ───────────────────────→
                            5. Extract JWT
                               Verify signature with secret
                               Extract sub: "42"
                               Check expiry: Not expired ✓
                               User authenticated ✓
    ←───────────────────────
6. Return user data

7. JWT expires after 1 hour
   User needs to login again
```

#### Pros

- **Excellent Scalability**

  - No server-side storage needed
  - Stateless verification
  - Works with multiple servers, multiple regions
  - No synchronization needed

- **Mobile-Friendly**

  - Works great for mobile apps
  - Cookies don't work well on mobile
  - JWT can be stored in app storage
  - Can be passed in headers

- **Microservices-Friendly**

  - One auth service issues JWT
  - Multiple services verify same JWT
  - Services don't need to communicate
  - Perfect for distributed systems

- **Cross-Domain Compatible**
  - Can be used across different domains
  - Not subject to same-origin policy (like cookies)
  - Ideal for SPAs (Single Page Applications)

#### Cons

- **Token Revocation Difficult**

  - Once issued, token valid until expiry
  - If account hacked, token still usable
  - No way to force logout without blacklist
  - Blacklist defeats scalability purpose

- **Token Size**

  - Contains user data (name, role, permissions)
  - Larger than session ID
  - Sent with every request
  - Increases bandwidth usage

- **Secret Key Management**

  - Secret must be kept secure
  - If compromised, attacker can create fake tokens
  - Need secure key rotation strategy
  - Multiple servers need same secret (risk point)

- **Data Cannot Be Updated Immediately**
  - If user's role changes, old token still valid
  - User needs to get new token
  - Changes don't take effect until new token issued
  - Can cause temporary inconsistencies

#### Token Revocation Problem in Detail

```
Scenario: User account hacked

Timeline:
  09:00 - User logs in, gets JWT (expires 10:00)
  09:15 - Attacker accesses account
  09:16 - User notices suspicious activity, requests account lock
  09:16 - Support representative locks account
  09:17 - But JWT still valid for 44 more minutes
  09:17 - Attacker uses same JWT to access account
  09:45 - Finally, JWT expires
  09:45 - Attacker can no longer access

Solution: Token Blacklist
  Keep list of revoked token IDs in Redis
  When verifying JWT:
    1. Check signature
    2. Check expiry
    3. Check if token_id in blacklist
  If in blacklist → Unauthorized

Tradeoff:
  Now we're doing Redis lookup again
  Defeats purpose of statelessness
```

#### When to Use

- Mobile applications
- APIs consumed by third parties
- Microservices architecture
- Distributed systems across regions
- Single Page Applications (SPAs)

---

### Method 3: API Key Authentication

#### What is an API Key?

An **API key** is a simple, long random string that serves as authentication credential for programmatic access.

```
API Key example:
  your_api_key...

Properties:
  - Cryptographically random
  - Long enough to be secure
  - Associated with specific user/application
  - Can have expiry date
  - Can have specific permissions
  - Can be revoked
```

#### Use Cases: API Keys Are For Machine-to-Machine Communication

**NOT for user authentication to browser!**

**Use Case 1: Third-Party Integration**

```
I'm a developer using OpenAI's API

I generate API key at OpenAI dashboard
I save it in my application:
  API_KEY=your_api_key

My application makes API calls:
  POST https://api.openai.com/v1/chat/completions
  Authorization: Bearer qwerty...
  Body: { model: "gpt-4", messages: [...] }

OpenAI verifies API key:
  - Valid?
  - Active?
  - Has quota remaining?
  - Matches rate limits?

If all pass: Process request
```

**Use Case 2: Machine-to-Machine Communication**

```
My backend server wants to use ChatGPT

User in web browser:
  1. Types: "Summarize this text for me"
  2. Submits to my backend

My backend server:
  1. Receives user's message
  2. Calls OpenAI API with my API key
  3. Gets response from OpenAI
  4. Formats response
  5. Sends to user's browser

This is machine-to-machine:
  User's browser ← My server → OpenAI server
```

#### API Key Workflow

```
Step 1: Generate API Key
  User goes to platform UI
  Clicks "Generate API Key"
  System generates: your_api_key
  User copies key
  User saves in environment variable or config

Step 2: Use API Key in Code
  const apiKey = process.env.OPENAI_API_KEY;

  const response = await fetch(
    'https://api.openai.com/v1/chat/completions',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: 'gpt-4',
        messages: [{ role: 'user', content: 'Hello' }]
      })
    }
  );

Step 3: Server Validates API Key
  Extract API key from Authorization header
  Look up in database:
    {
      api_key_id: 123,
      user_id: 42,
      key_hash: "a1b2c3d4...",
      permissions: ['read', 'write'],
      expires_at: "2025-12-31",
      rate_limit: "1000 requests/day",
      status: "active"
    }
  Verify:
    - Key is valid?
    - Key is active (not revoked)?
    - Key has not expired?
    - User has permission for this operation?
    - Rate limit not exceeded?

Step 4: Process Request
  If all checks pass: Process request
  If any check fails: Return 401 or 403
```

#### API Key Storage & Security

**NEVER store plain API key in code!**

**Secure storage methods:**

```
✗ BAD:
  const API_KEY = "your_api_key"; // In source code

✓ GOOD:
  const API_KEY = process.env.OPENAI_API_KEY; // Environment variable

✓ BETTER:
  Use .env file (gitignored):
    OPENAI_API_KEY=your_api_key...

✓ BEST:
  Use secrets management system:
    - AWS Secrets Manager
    - HashiCorp Vault
    - Kubernetes Secrets
    - 1Password / LastPass
```

#### Advantages of API Keys

1. **Simple to Generate**

   - Just click a button in UI
   - No complex flows needed

2. **Machine-to-Machine Ideal**

   - No user interaction needed
   - No login form required
   - Automated requests

3. **Easy to Manage**

   - Can generate multiple keys
   - Can revoke individual keys
   - Can set per-key permissions
   - Can set per-key expiry

4. **No Session State**
   - Stateless verification
   - Can revoke instantly with blacklist
   - No session replication needed

#### Disadvantages of API Keys

1. **Long-Lived**

   - Usually don't expire (or very long expiry)
   - If compromised, attacker has long window
   - Need rotation strategy

2. **Broad Permissions**

   - Each key often has broad access
   - Hard to limit per-request
   - If compromised, wide access granted

3. **No User Context**
   - Can't distinguish between different users
   - Can't track individual user behavior
   - Audit trail points to API key, not person

#### Comparison: API Key vs JWT

| Aspect            | API Key                              | JWT                                      |
| ----------------- | ------------------------------------ | ---------------------------------------- |
| **Use Case**      | Machine-to-machine, third-party APIs | User authentication, distributed systems |
| **Stored By**     | Application, server                  | Client/browser                           |
| **Transmission**  | Headers (automatic)                  | Headers (manual) or cookies              |
| **Server Lookup** | Required (verify key exists)         | Not required (signature verification)    |
| **Revocation**    | Immediate                            | Difficult (until expiry)                 |
| **Permissions**   | Per-key granular control             | All in token                             |
| **Data Size**     | Small (just the key)                 | Large (contains claims)                  |

---

### Method 4: OAuth 2.0

_(Covered in separate section below)_

---

## 5. Authorization Concepts & Models

Authorization is determining **what** an authenticated user can do.

### Role-Based Access Control (RBAC)

Simplest authorization model: Assign roles to users, define permissions per role.

#### Roles & Permissions Structure

```
Roles defined in system:
  - admin
  - editor
  - viewer
  - member

Permissions defined:
  - read
  - write
  - delete
  - manage_users
  - manage_settings

Role-to-Permission mapping:
  admin → [read, write, delete, manage_users, manage_settings]
  editor → [read, write, delete]
  viewer → [read]
  member → [read]
```

#### RBAC Flow

```
User logs in:
  User: alice
  Assigned role: "admin"

User makes request:
  Request: DELETE /users/123

Authorization check:
  1. Extract user's role: "admin"
  2. Look up permissions for "admin": [read, write, delete, manage_users, manage_settings]
  3. Does role have "delete" permission? YES ✓
  4. Authorization passed
  5. Process request

vs.

User bob has role: "viewer"
Request: DELETE /users/123

Authorization check:
  1. Extract user's role: "viewer"
  2. Look up permissions for "viewer": [read]
  3. Does role have "delete" permission? NO ✗
  4. Authorization failed
  5. Return 403 Forbidden
```

#### RBAC Implementation

```javascript
// Middleware for authorization
function authorize(requiredPermission) {
  return (req, res, next) => {
    const userRole = req.context.role; // Set by auth middleware

    // Look up role permissions
    const permissions = rolePermissions[userRole];

    if (!permissions.includes(requiredPermission)) {
      return res.status(403).json({
        error: "Forbidden",
        message: "User does not have required permission",
      });
    }

    next(); // Permission granted, proceed
  };
}

// Routes with authorization
app.delete("/users/:id", authorize("delete"), deleteUserHandler);
app.post("/users", authorize("write"), createUserHandler);
app.get("/users", authorize("read"), listUsersHandler);
```

#### RBAC Limitations

- Doesn't support fine-grained, attribute-based rules
- Hard to express complex conditional logic
- Can't easily implement resource-level permissions
- Scalability issues with many roles

---

## 6. OAuth 2.0 & OpenID Connect

### The OAuth 2.0 Problem OAuth 2.0 Solves

**Problem:** Third-party applications need access to your data without knowing your password.

**Example:**

```
Instagram wants to let users share photos to Facebook

Bad approach:
  1. User gives Instagram their Facebook password
  2. Instagram stores Facebook password in their database
  3. If Instagram is hacked, attackers get Facebook password
  4. User has to change Facebook password everywhere
  5. Instagram can do anything on user's Facebook account

This is insecure and violates principle of least privilege
```

**Solution:** OAuth 2.0 - Delegated Authorization

```
Good approach:
  1. User wants Instagram to access Facebook
  2. Instagram redirects to Facebook login page
  3. User logs into Facebook
  4. Facebook asks: "Do you want to let Instagram access your photos?"
  5. User clicks "Approve"
  6. Facebook gives Instagram an access token
  7. Token grants only specific permissions (read photos, not delete)
  8. Token expires after some time
  9. If Instagram is hacked, only temporary token is compromised
  10. User password is never shared with Instagram
```

### OAuth 2.0 Terminology

- **Resource Owner:** User (who owns the data)
- **Client:** Application wanting access (Instagram)
- **Authorization Server:** Issues tokens (Facebook)
- **Resource Server:** Stores user data (Facebook Photos API)

### OAuth 2.0 Authorization Code Flow (Most Common)

```
1. User clicks "Login with Facebook" on Instagram

2. Instagram redirects to Facebook:
   https://facebook.com/authorize?
     client_id=instagram_app_id
     redirect_uri=instagram.com/callback
     scope=photos,email
     state=random_string_123

3. User sees Facebook login
   Logs in with email/password

4. Facebook asks permission:
   "Instagram wants access to:
    - Your photos
    - Your email address
    Allow?"

5. User clicks "Allow"

6. Facebook redirects back to Instagram with code:
   https://instagram.com/callback?
     code=auth_code_456
     state=random_string_123

7. Instagram backend receives code
   Validates state matches
   Exchanges code for token:
   POST https://facebook.com/token
   {
     client_id: "instagram_app_id",
     client_secret: "instagram_secret_key",
     code: "auth_code_456",
     redirect_uri: "instagram.com/callback"
   }

8. Facebook returns access token:
   {
     "access_token": "token_xyz",
     "token_type": "Bearer",
     "expires_in": 3600
   }

9. Instagram uses token to access user's photos:
   GET https://facebook.com/api/photos
   Authorization: Bearer token_xyz

10. Facebook returns user's photos
    Instagram displays them

11. User now logged into Instagram
    Instagram has access to photos
    Never knew Instagram's password
```

### OpenID Connect (OIDC)

**What:** Authentication layer on top of OAuth 2.0

**Problem OAuth 2.0 Solves:** Delegated authorization
**Problem OIDC Solves:** Authentication information

```
OAuth 2.0 answers: "Can app X access user's photos?"
OIDC answers: "Who is the user?"

OAuth 2.0 gets: Access token
OIDC gets: ID token (contains user info) + Access token

ID Token (JWT):
{
  "sub": "user_123",           // User ID
  "name": "John Doe",          // User name
  "email": "john@example.com", // Email
  "email_verified": true,      // Email verified
  "picture": "https://...",    // Profile picture
  "iat": 1234567890,           // Issued at
  "exp": 1234571490,           // Expires in
  "aud": "instagram_app_id"    // Audience
}
```

---

## 7. Password Storage & Security

### Never Store Passwords in Plain Text

**Why:**

- If database compromised, all passwords exposed
- Users often reuse passwords (compromised everywhere)
- Violates data protection regulations (GDPR, HIPAA)

### Hashing

**What:** One-way function that transforms password into fixed-length string.

```
Input:  "myPassword123"
Hashing function (e.g., SHA-256)
Output: "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"

Properties:
- Same input always produces same output
- Different input produces different output
- Cannot reverse: Can't get password from hash
- Fast to compute
- Fixed length output
```

**Problem with simple hashing:**

```
Attacker has password hashes:
  alice_hash = "a1b2c3d4e5f6..."
  bob_hash = "f7g8h9i0j1k2..."

Attacker builds rainbow table (precomputed hashes):
  "password123" → "a1b2c3d4e5f6..."
  "123456" → "f7g8h9i0j1k2..."
  ...10 million combinations...

Attacker looks up alice_hash in rainbow table:
  Found: alice_hash matches "password123"

Problem: Common passwords can be cracked in seconds
```

### Salting

**Solution:** Add random data (salt) before hashing

```
Salt: "random_salt_12345"
Password: "myPassword123"

Hash = SHA256(salt + password)
     = SHA256("random_salt_12345" + "myPassword123")
     = "x1y2z3a4b5c6d7e8f9g0h1i2j3k4l5m6"

Database stores:
  user_id | password_hash | salt
  1       | x1y2z3a4...   | random_salt_12345

Login process:
  1. User enters password: "myPassword123"
  2. Look up user's salt: "random_salt_12345"
  3. Compute hash: SHA256("random_salt_12345" + "myPassword123")
  4. Compare with stored hash: x1y2z3a4...
  5. If match → Password correct

Benefits:
- Different salt for each user
- Same password produces different hashes
- Rainbow table attacks don't work
- Even if one user's password cracked, salt prevents using for others
```

### Key Derivation Functions (KDF)

**Better:** Use dedicated password hashing algorithms

```
bcrypt: purpose-built for password hashing
  - Slower than SHA256 (intentional, prevents brute force)
  - Includes salt automatically
  - Configurable work factor (how slow)

scrypt: memory-hard KDF
  - Requires significant memory
  - Makes hardware attacks expensive

PBKDF2: standard password hashing
  - Iterates hash function multiple times
  - Higher iteration count = slower = more secure

Argon2: modern, recommended
  - Won Password Hashing Competition
  - Memory-hard
  - Resistant to GPU/ASIC attacks
  - Recommended by OWASP
```

#### Recommended Password Hashing

```javascript
// Using bcrypt (recommended for most cases)
const bcrypt = require("bcrypt");

// Registering user
const password = "myPassword123";
const saltRounds = 10; // Higher = slower = more secure
const passwordHash = await bcrypt.hash(password, saltRounds);
// Store passwordHash in database

// Logging in
const userPasswordHash = getUserHashFromDB(email);
const isPasswordCorrect = await bcrypt.compare(password, userPasswordHash);
if (isPasswordCorrect) {
  // Password matches, authenticate user
}
```

### Password Requirements

**Not recommended:**

```
❌ Require numbers, symbols, uppercase, etc.
❌ Enforce password expiry
❌ Forbid password reuse

Why? These increase complexity but don't improve security.
Users write down complex passwords.
Users rotate to slightly different versions.
```

**Recommended:**

```
✓ Require minimum length (12+ characters)
✓ Prevent common passwords (use dictionary check)
✓ No arbitrary complexity rules
✓ Allow passphrases

Example good password:
  "correct horse battery staple" (long, memorable, secure)
  vs.
  "P@ssw0rd!2024" (complex, forgettable, might be reused)
```

---

## 8. Modern Challenges & Best Practices

### Challenge 1: Token Expiry vs. Revocation

**Problem:**

```
Short expiry (1 hour):
  ✓ Good: Token quickly becomes useless if leaked
  ✗ Bad: User logs out after 1 hour, needs to login again

Long expiry (30 days):
  ✓ Good: User stays logged in
  ✗ Bad: If token leaked, attacker has 30 days access

No expiry:
  ✗ Bad: Token valid forever, never expires
```

**Solution: Refresh Tokens**

```
Two-token system:

Access Token:
  - Short expiry (15 minutes)
  - Sent with every request
  - If leaked, limited damage

Refresh Token:
  - Long expiry (30 days)
  - Stored securely (httpOnly cookie)
  - Not sent with every request
  - Only used when access token expires

Workflow:
  1. User logs in
  2. Server issues: Access Token (15 min) + Refresh Token (30 days)
  3. Client uses Access Token for requests
  4. After 15 minutes, Access Token expires
  5. Client uses Refresh Token to get new Access Token
  6. Server issues new Access Token (15 min)
  7. Client continues requests
  8. After 30 days, Refresh Token expires
  9. User must login again
```

### Challenge 2: Cross-Site Request Forgery (CSRF)

**Attack:**

```
1. User logs into their bank website
2. User visits attacker's website (in another tab)
3. Attacker's website has hidden form:
   <form action="https://bank.com/transfer" method="POST">
     <input name="amount" value="1000">
     <input name="to_account" value="attacker_account">
   </form>
   <script>document.forms[0].submit();</script>

4. Form automatically submits to bank
5. User's browser includes bank's session cookie
6. Bank thinks it's legitimate request from user
7. Money transferred to attacker

Why it works:
- Browser automatically includes cookies with same-domain requests
- Server can't tell if request came from user's legitimate action
- Attacker's website makes request without user's consent
```

**Protection: CSRF Tokens**

```
Server generates unique token for each user session:

1. User requests form:
   GET /transfer-form

2. Server responds with form including token:
   <form action="/transfer" method="POST">
     <input type="hidden" name="csrf_token" value="random123">
     <input type="number" name="amount">
     <button>Transfer</button>
   </form>

3. User submits form from legitimate website:
   POST /transfer
   csrf_token: random123
   amount: 1000

4. Server verifies CSRF token:
   - Token matches user's session? YES
   - Token is fresh (not expired)? YES
   - Authorize request

5. Attacker tries same attack:
   - Attacker can't read CSRF token (same-origin policy)
   - Form submits without token
   - Server rejects request

Alternatively: SameSite Cookies
  Set-Cookie: session_id=...; SameSite=Strict

  Browser only sends cookie on same-site requests
  Attacker's cross-site request: Cookie NOT included
  Bank doesn't see session cookie
  Request rejected
```

### Challenge 3: Password Reset Security

**Vulnerable approach:**

```
1. User clicks "Forgot Password"
2. User enters email
3. Server sets new temporary password
4. Server sends password via email

Problems:
- Email sent in plain text (can be intercepted)
- Password seen by email service
- Temporary password might be weak
- No way to verify it's really the user
```

**Secure approach: Password Reset Token**

```
1. User enters email: alice@example.com

2. Server:
   - Looks up user by email
   - Generates secure random token
   - Stores token with expiry (15 minutes):
     {
       user_id: 42,
       reset_token_hash: "x1y2z3...",
       expires_at: "2025-01-06T10:15:00Z"
     }
   - Sends email with reset link:
     https://app.com/reset-password?token=abc123xyz789

3. User receives email
   Clicks link (within 15 minutes)

4. Frontend shows password reset form:
   New password: [_____________________]
   Confirm password: [_____________________]
   [Reset Password Button]

5. User submits form:
   POST /reset-password
   {
     reset_token: "abc123xyz789",
     new_password: "newPassword456"
   }

6. Server:
   - Looks up token: abc123xyz789
   - Verifies token exists and not expired
   - Hashes new password
   - Updates user's password in database
   - Deletes reset token (can't be reused)

Security benefits:
- Email only contains token, not password
- Token has short expiry (can't use old email)
- Token can only be used once
- Token verification before password change
- No temporary passwords floating around
```

### Best Practices Summary

| Practice                            | Why                                    |
| ----------------------------------- | -------------------------------------- |
| Never store passwords in plain text | Protects if database is compromised    |
| Use strong hashing (bcrypt, Argon2) | Slows down brute force attacks         |
| Use salts                           | Prevents rainbow table attacks         |
| Implement CSRF protection           | Prevents unauthorized requests         |
| Use HTTPS only                      | Encrypts credentials in transit        |
| Secure password reset tokens        | Prevents unauthorized password changes |
| Implement rate limiting             | Prevents brute force attacks           |
| Log authentication events           | Detect suspicious activity             |
| Monitor for account takeovers       | Alert users of unusual access          |
| Implement MFA                       | Adds second factor of security         |

---

## 9. Practical Recommendations

### Should You Build Your Own Authentication?

**Short answer:** No, unless learning.

**Why?**

- Security is hard to get right
- Many edge cases and attack vectors
- Requires constant updates for new threats
- Can be expensive to maintain
- Most platforms have excellent off-the-shelf solutions

### Authentication Providers (Use These!)

**Established providers:**

- **Auth0** - Comprehensive, enterprise-ready
- **Firebase Auth** - Simple, integrated with Google Cloud
- **AWS Cognito** - AWS ecosystem
- **Okta** - Enterprise focus
- **Microsoft Azure AD** - Enterprise Microsoft ecosystem

**Modern alternatives:**

- **Clerk** - Developer-friendly, modern UI
- **Supabase Auth** - Open source, PostgreSQL-based
- **NextAuth.js** - Built for Next.js applications
- **FusionAuth** - Open source option

### When to Implement Custom Authentication

**OK for learning:**

```
Personal projects, learning, prototypes
→ Implement session-based or JWT auth
→ Understand tradeoffs
→ Learn security principles
```

**NOT for production (unless expert):**

```
Production systems, user data, sensitive info
→ Use external auth provider
→ Focus on business logic
→ Let experts handle security
```

### Hybrid Approach Recommendation

For web applications:

```
Browser-based access (web app):
  → Use session-based authentication
  → Stored in httpOnly cookie
  → Server maintains session

API/Mobile access:
  → Use OAuth 2.0 or API keys
  → Stateless verification
  → Scalable across servers
```

---

## 10. Security Resources & References

### OWASP Resources (Recommended!)

OWASP CheatSheets cover authentication and security comprehensively:

**Authentication CheatSheet:**

- Password requirements best practices
- Session management
- Multi-factor authentication
- Secure coding practices

**Authorization CheatSheet:**

- Authorization patterns
- Access control implementation
- Common vulnerabilities

**Password Storage CheatSheet:**

- Password hashing algorithms
- Salting techniques
- Key derivation functions

**Overall security resource:**

- OWASP CheatSheet Series: Comprehensive for all security topics

### Additional Resources

**JWT:**

- jwt.io - JWT playground, libraries, algorithm info

**OAuth 2.0:**

- PortSwigger Web Security Academy - Free, comprehensive tutorials
- OAuth 2.0 Authorization Framework RFC 6749

**Biometrics & Advanced Topics:**

- FusionAuth blog - Educational articles on identity

**Identity Fundamentals:**

- Ping Identity - Identity concepts explained

**Hash Functions:**

- Wikipedia Hash Function - Technical background

---

## 11. Key Takeaways & Revision

### Authentication Evolution

```
Pre-Industrial → Medieval → Industrial → Digital → Modern → Future
  Implicit    →  Seals   → Passwords → Hashing → JWTs  → Decentralized
   Trust      → Tokens   → Shared    → Digital → Multi  → Quantum
               →         → Secrets   → Certs   → Factor → Safe
```

### Three Core Components

| Component | Purpose                        | Storage            |
| --------- | ------------------------------ | ------------------ |
| Session   | Maintain server-side context   | Redis/Database     |
| JWT       | Stateless authentication token | Client browser     |
| Cookie    | Automatic token transmission   | Browser cookie jar |

### Authentication Methods Comparison

| Method    | Use For            | Pros                  | Cons                |
| --------- | ------------------ | --------------------- | ------------------- |
| Session   | Web apps           | Revocable, controlled | Limited scalability |
| JWT       | APIs, mobile       | Scalable, stateless   | Hard to revoke      |
| API Key   | Machine-to-machine | Simple, powerful      | Long-lived risk     |
| OAuth 2.0 | Third-party access | Delegated, safe       | Complex flow        |

### Password Storage

```
❌ Plain text
↓
✓ SHA256 hash (bad)
↓
✓✓ SHA256 + salt (better)
↓
✓✓✓ bcrypt/Argon2 (best)
```

### Security Principles

1. **Defense in Depth:** Multiple layers of security
2. **Least Privilege:** Users get minimum necessary access
3. **Never Trust Client:** Validate everything server-side
4. **Encrypt in Transit:** HTTPS always
5. **Encrypt at Rest:** Sensitive data encrypted in database
6. **Monitor & Alert:** Track suspicious activity

### Common Vulnerabilities to Avoid

- [ ] Storing passwords in plain text
- [ ] Weak password hashing (MD5, SHA1)
- [ ] No password salt
- [ ] Hardcoded secrets in code
- [ ] No HTTPS
- [ ] Tokens too long-lived
- [ ] No rate limiting on login
- [ ] No logout functionality
- [ ] No token blacklist for revocation
- [ ] Accepting all origins (CORS misconfiguration)

---

## References & Links

**Video Source:**

- Authentication and authorization for backend engineers (Sriniously): https://www.youtube.com/watch?v=A95rliroC8Q

**Sriniously Channel & Playlist:**

- Sriniously Channel: https://www.youtube.com/channel/UCYkDx5W-v5qjkVVm1MrA1-w
- Backend from First Principles Playlist: https://www.youtube.com/playlist?list=PLui3EUkuMTPgZcV0QhQrOcwMPcBCcd_Q1

**OWASP Security Resources (Highly Recommended):**

- OWASP CheatSheet Series (All topics): https://cheatsheetseries.owasp.org/index.html
- Authentication CheatSheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Authorization CheatSheet: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- Password Storage CheatSheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- Access Control CheatSheet: https://portswigger.net/web-security/access-control

**JWT & Token-Based Auth:**

- JWT.io (playground, libraries, algorithms): https://jwt.io/
- JWT Specification (RFC 7519): https://tools.ietf.org/html/rfc7519

**OAuth 2.0 & Security:**

- PortSwigger Web Security Academy (Authentication): https://portswigger.net/web-security/authentication
- PortSwigger Web Security Academy (Access Control): https://portswigger.net/web-security/access-control
- OAuth 2.0 Authorization Framework (RFC 6749): https://tools.ietf.org/html/rfc6749

**Password Hashing:**

- Hash Function Basics (Wikipedia): https://en.wikipedia.org/wiki/Hash_function
- bcrypt Official: https://en.wikipedia.org/wiki/Bcrypt
- Argon2 Official: https://argon2.online/

**Identity & Authorization Learning:**

- FusionAuth Educational Blog: https://fusionauth.io/blog/category/education/
- Ping Identity Authorization Methods: https://www.pingidentity.com/en/resources/identity-fundamentals/authorization/authorization-methods.html

**Authentication Providers (Production Use):**

- Auth0: https://auth0.com/
- Clerk: https://clerk.com/
- Supabase Auth: https://supabase.com/auth
- Firebase Authentication: https://firebase.google.com/docs/auth
- NextAuth.js: https://next-auth.js.org/

**General Web Security:**

- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework
- OWASP Top 10 Web Vulnerabilities: https://owasp.org/Top10/
- Mozilla Developer Network Security: https://developer.mozilla.org/en-US/docs/Web/Security
