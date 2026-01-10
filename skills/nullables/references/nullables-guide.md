# Nullables and A-Frame Architecture Guide

## Overview

This guide covers James Shore's Nullables pattern and A-Frame architecture, enabling fast, reliable testing without mocks through sociable, state-based tests.

## A-Frame Architecture

A-Frame separates code into three distinct layers:

### 1. Logic Layer (Business Logic & Value Objects)

Pure, algorithmic code with no external dependencies. Contains business rules, calculations, and data transformations.

**Characteristics:**
- No network calls, file I/O, or external system interactions
- Fast, deterministic, purely in-memory operations
- Can be tested directly without any special setup

**Value Objects:**
- Immutable by default - once created, state doesn't change
- Mutable value objects are acceptable - can be instantiated with `.create()` and modified with methods, but methods only perform algorithmic in-memory operations
- Operations requiring network/infrastructure should be handled by infrastructure code that takes/returns value objects

### 2. Infrastructure Layer

Code that directly interfaces with the outside world (network, filesystem, databases, external APIs).

**Two types:**
- **Infrastructure Wrappers**: Low-level code that directly interfaces with external systems (HTTP clients, database connectors, filesystem operations)
- **High-level Infrastructure**: Code that depends on infrastructure wrappers (API clients, repositories, service clients)

**All infrastructure classes should implement the Nullables pattern** (see below).

### 3. Application Layer

High-level orchestration code that coordinates infrastructure and logic layers.

**Characteristics:**
- Orchestrates infrastructure calls and logic operations
- Implements business workflows
- Uses "Logic Sandwiches" pattern: read from infrastructure → process with logic → write to infrastructure
- Also implements Nullables pattern

## The Nullables Pattern

Nullables are production code with an "off switch" - they function normally except external interactions can be disabled for testing.

### Core Implementation

Every application and infrastructure class implements **two static factory methods**:

```javascript
class ApiClient {
  static create(config) {
    // Production instance with real external calls
    const httpClient = HttpClient.create(config.apiUrl);
    return new ApiClient(httpClient);
  }

  static createNull(config = {}) {
    // Test instance with nulled external calls
    const httpClient = HttpClient.createNull();
    return new ApiClient(httpClient);
  }

  constructor(httpClient) {
    this._httpClient = httpClient;
  }

  async fetchUser(userId) {
    return await this._httpClient.get(`/users/${userId}`);
  }
}
```

### Key Principles

**1. Static Factory Methods Over Constructors**
- Always use `.create()` for production instances
- Always use `.createNull()` for test instances
- Never call constructors directly from outside the class

**2. Minimal Arguments with Reasonable Defaults**
- `.create()` should accept minimal arguments and provide reasonable production-ready defaults
- `.createNull()` should work with no arguments or minimal configuration

**3. Dependency Injection Through Factories**
- `.create()` calls `.create()` on dependencies
- `.createNull()` calls `.createNull()` on dependencies
- Dependencies are passed to the constructor

**4. Infrastructure Wrapper Pattern**

Low-level infrastructure wrappers use "Embedded Stubs" - they stub third-party libraries, not application code:

```javascript
class HttpClient {
  static create() {
    return new HttpClient(realHttp); // Real HTTP library (e.g., Node.js http module)
  }

  static createNull() {
    return new HttpClient(new StubbedHttp()); // Stubbed HTTP library
  }

  constructor(http) {
    this._http = http;
  }

  async request(options) {
    const response = await this._http.request(options);
    return response.body;
  }
}

// StubbedHttp - embedded stub that mimics the third-party HTTP library
class StubbedHttp {
  async request(options) {
    // Returns a canned response with default values
    return {
      status: 200,
      headers: {},
      body: {}
    };
  }
}
```

## Testing Approach

### Test Philosophy

**Narrow, Sociable, State-Based Tests:**
- **Narrow**: Each test focuses on a specific function or behavior
- **Sociable**: Tests use real dependencies (via `.createNull()`), not mocks
- **State-Based**: Tests verify outputs and observable state, not interaction patterns

### Unit Tests

```javascript
describe('ApiClient', () => {
  it('fetches user data', async () => {
    // Use createNull for fast, reliable tests
    const client = ApiClient.createNull();

    // Configure the response
    client.httpClient.stubResponse('/users/123', {
      id: 123,
      name: 'Alice'
    });

    // Test the behavior
    const user = await client.fetchUser(123);

    // Verify state/output
    expect(user.name).toBe('Alice');
  });
});
```

**Characteristics:**
- Use `.createNull()` instances exclusively
- Configure responses using nullable infrastructure
- Verify outputs and state changes
- Fast execution (no real external calls)
- Sociable (hit real logic and nullable infrastructure)

### Integration Tests

A **small number** of narrow integration tests verify live infrastructure:

```javascript
describe('HttpClient (Integration)', () => {
  it('makes real HTTP requests', async () => {
    // Use .create() for live tests
    const client = HttpClient.create('https://api.example.com');

    // Hit real API
    const response = await client.get('/health');

    // Verify it works
    expect(response.status).toBe('ok');
  });
});
```

**Characteristics:**
- Use `.create()` instances to test live infrastructure
- Small number of tests (smoke tests)
- Verify infrastructure wrappers work with real systems
- May be slower, may require test environment setup

### Configurable Responses

Configure responses from the **dependency's externally-visible behavior**, not its implementation. Use named, optional parameters so tests only configure what they care about.

```javascript
class LoginClient {
  static create(config) {
    const httpClient = HttpClient.create(config.authUrl);
    return new LoginClient(httpClient);
  }

  static createNull({
    email = "user@example.com",
    emailVerified = true,
    userId = "test-user-id",
    loginSucceeds = true,
  } = {}) {
    // Configuration focuses on behavior (login success, user data)
    // NOT on implementation (HTTP status codes, JSON structure)
    return new LoginClient(
      HttpClient.createNull(),
      { email, emailVerified, userId, loginSucceeds }
    );
  }

  constructor(httpClient, config = {}) {
    this._httpClient = httpClient;
    this._config = config;
  }

  async login(credentials) {
    if (this._config.loginSucceeds === false) {
      throw new Error("Login failed");
    }
    
    // In production, this would call httpClient and parse response
    // In tests, it returns configured behavior
    return {
      email: this._config.email,
      emailVerified: this._config.emailVerified,
      userId: this._config.userId,
    };
  }
}

// In tests - configure only what matters for this scenario
const client = LoginClient.createNull({ emailVerified: false });
// Other parameters use sensible defaults
```

**Key principles:**
- Define responses at the behavior level (user data, success/failure)
- NOT at the implementation level (HTTP status codes, JSON structure)
- Use named, optional parameters with sensible defaults
- Tests configure only what they care about
- Support both single values and arrays for sequential responses

### Output Tracking

Track what infrastructure would send externally using the `track[OutputType]()` pattern. Output tracking records **behavior** (what was written), not function calls.

```javascript
const OUTPUT_EVENT = "email-sent";

class EmailService {
  static createNull() {
    return new EmailService(new EventEmitter(), new NullMailer());
  }

  constructor(emitter, mailer) {
    this._emitter = emitter;
    this._mailer = mailer;
  }

  // Returns an OutputTracker that listens for email events
  trackSentEmails() {
    return OutputTracker.create(this._emitter, OUTPUT_EVENT);
  }

  async sendEmail(to, subject, body) {
    await this._mailer.send(to, subject, body);
    // Emit the behavior that occurred
    this._emitter.emit(OUTPUT_EVENT, { to, subject, body });
  }
}

// In tests
const emailService = EmailService.createNull();
const tracker = emailService.trackSentEmails();

await emailService.sendEmail('user@example.com', 'Hello', 'World');

const sent = tracker.data;
expect(sent[0].to).toBe('user@example.com');
```

**Key principles:**
- Use `track[OutputType]()` naming convention
- Returns an `OutputTracker` instance (not raw data)
- Emits events representing the behavior that occurred
- Works with both real and null instances

## Event Emitters for State-Based Tests

Use event emitters to observe state changes in objects:

```javascript
class UserAccount extends EventEmitter {
  static create(userId) {
    return new UserAccount(userId);
  }

  static createNull(userId = 'test-user') {
    return new UserAccount(userId);
  }

  constructor(userId) {
    super();
    this._userId = userId;
    this._balance = 0;
  }

  deposit(amount) {
    this._balance += amount;
    this.emit('balance-changed', {
      userId: this._userId,
      newBalance: this._balance
    });
  }

  getBalance() {
    return this._balance;
  }
}

// In tests
const account = UserAccount.createNull('user-123');
const events = [];

account.on('balance-changed', (event) => {
  events.push(event);
});

account.deposit(100);

expect(events[0].newBalance).toBe(100);
expect(account.getBalance()).toBe(100);
```

## Implementation Patterns

### Pattern: Logic Sandwich

Application code follows: **Read → Process → Write**

```javascript
class TransferMoneyService {
  static create() {
    const accountRepo = AccountRepository.create();
    const transferLogic = TransferLogic.create();
    return new TransferMoneyService(accountRepo, transferLogic);
  }

  static createNull() {
    const accountRepo = AccountRepository.createNull();
    const transferLogic = TransferLogic.create(); // Logic is pure
    return new TransferMoneyService(accountRepo, transferLogic);
  }

  constructor(accountRepo, transferLogic) {
    this._accountRepo = accountRepo;
    this._transferLogic = transferLogic;
  }

  async transfer(fromId, toId, amount) {
    // Read (infrastructure)
    const fromAccount = await this._accountRepo.get(fromId);
    const toAccount = await this._accountRepo.get(toId);

    // Process (logic)
    const result = this._transferLogic.execute(fromAccount, toAccount, amount);

    // Write (infrastructure)
    await this._accountRepo.save(result.fromAccount);
    await this._accountRepo.save(result.toAccount);

    return result;
  }
}
```

### Pattern: Cascading Nullables

When `.createNull()` is called on application code, it cascades down through all dependencies:

```javascript
class Application {
  static create(config) {
    const apiClient = ApiClient.create(config.apiUrl);
    const database = Database.create(config.dbUrl);
    const emailService = EmailService.create(config.smtpConfig);
    return new Application(apiClient, database, emailService);
  }

  static createNull() {
    // All dependencies are also nulled
    const apiClient = ApiClient.createNull();
    const database = Database.createNull();
    const emailService = EmailService.createNull();
    return new Application(apiClient, database, emailService);
  }

  // ...
}
```

### Pattern: Reasonable Defaults

```javascript
class NotificationService {
  static create(config = {}) {
    // Reasonable production defaults
    const emailService = config.emailService ?? EmailService.create();
    const smsService = config.smsService ?? SmsService.create();
    const retryCount = config.retryCount ?? 3;
    const timeout = config.timeout ?? 5000;

    return new NotificationService(emailService, smsService, retryCount, timeout);
  }

  static createNull() {
    // Minimal setup for tests
    return new NotificationService(
      EmailService.createNull(),
      SmsService.createNull(),
      1, // Single attempt in tests
      100 // Short timeout in tests
    );
  }

  // ...
}
```

## Quick Reference Checklist

When implementing Nullables and A-Frame:

- [ ] Separate code into Logic, Infrastructure, and Application layers
- [ ] Logic layer has no external dependencies
- [ ] All infrastructure wrappers wrap third-party code, not application code
- [ ] Every class has a static `.create()` method
- [ ] Every application and infrastructure class has a static `.createNull()` method
- [ ] `.create()` methods accept minimal arguments with reasonable defaults
- [ ] `.create()` calls `.create()` on dependencies
- [ ] `.createNull()` calls `.createNull()` on dependencies
- [ ] Dependencies are passed to constructors
- [ ] Unit tests use `.createNull()` instances exclusively
- [ ] Integration tests (few in number) use `.create()` instances
- [ ] Tests are narrow, sociable, and state-based
- [ ] No mocks - use configurable responses on nullable infrastructure
- [ ] Event emitters expose state changes for testing
- [ ] Output tracking captures what would be sent externally
