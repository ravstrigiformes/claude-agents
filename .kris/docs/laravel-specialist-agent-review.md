# Laravel Specialist Agent Configuration Review

**Date:** 2025-12-26
**Reviewed By:** 4 Independent Specialist Agents
**Subject:** `.claude/agents/laravel-specialist.md` and associated skills

---

# PART 1: INITIAL INDIVIDUAL ASSESSMENTS

---

## Assessment 1: Independent Laravel Specialist

**Grade: B-**

### Executive Summary

This agent configuration represents a solid foundation for Laravel development guidance but has notable gaps in Laravel 11+ features, contains some architectural inconsistencies, and shows signs of both over-engineering in certain areas and under-engineering in others. Several industry-standard practices are absent, and some security recommendations are outdated.

### 1. STRENGTHS

#### Architectural Foundation
- **Modular Monolith Pattern**: Well-structured approach that balances separation of concerns without microservices complexity. The `app/Modules/{ModuleName}/` convention is sensible.
- **Immutability Focus**: Strong emphasis on `readonly` classes and immutable DTOs aligns with modern defensive programming.
- **SOLID Principles Coverage**: Explicit documentation of SRP, LSP, ISP, and DIP with the pragmatic note about relaxing OCP for rapid development.

#### PHP 8.2+ Modern Features
- Good coverage of constructor property promotion, enums, match expressions, named arguments, and readonly classes.
- PSR-12 enforcement with Laravel Pint is appropriate.
- Strict types declaration requirement is correct.

#### Testing Approach
- Pest framework choice is excellent for modern Laravel.
- TDD workflow (Red-Green-Refactor) is well-documented.
- Good test organization mirroring module structure.
- Proper use of `describe()` blocks for grouping.

#### Security Coverage
- Comprehensive OWASP Top 10 mapping with Laravel-specific mitigations.
- Good coverage of Sanctum token abilities.
- Signed URLs and SSRF prevention are included.
- File upload security with proper validation.

#### Eloquent Patterns
- Excellent N+1 prevention guidance with multiple approaches.
- Good query builder patterns with dynamic filtering.
- Proper soft delete handling.
- Transaction retry logic with `attempts: 5`.

### 2. GAPS AND MISSING ELEMENTS

#### Laravel 11+ Specific Features (Critical Gap)

| Missing Feature | Impact |
|----------------|--------|
| `Context` facade | No request context propagation for logging/queues |
| Deferrable operations | No `Illuminate\Support\defer()` guidance |
| Health routing | No `Route::health()` for container health checks |
| Laravel Prompts | No CLI interaction guidance with new Prompts component |
| Per-second rate limiting | Mentioned in security but not fully explained |
| `once()` helper | No memoization pattern with new helper |
| Simplified `bootstrap/app.php` | No mention of consolidated configuration |
| Laravel Reverb | No WebSocket guidance |
| Laravel Pennant | No feature flag patterns |

#### Action Pattern (Major Gap)
The description mentions "Action classes" but the skill files provide **zero guidance**.

#### Queue and Job Patterns (Major Gap)
- No job batching (`Bus::batch()`)
- No job chaining (`Bus::chain()`)
- No job middleware patterns
- No rate limiting with `RateLimited` middleware
- No unique job patterns (`ShouldBeUnique`)
- No Laravel Horizon configuration
- No failed job handling strategies

#### API Versioning (Missing Entirely)
No guidance on enterprise API versioning.

#### Repository Pattern Discussion (Absent)
The configuration enforces services but never addresses the contentious Repository pattern.

#### API Documentation (Missing)
No mention of Scribe, Scramble, or OpenAPI specification standards.

### 3. FLAWED OR OUTDATED PRACTICES

#### Security Header Issue (Outdated)
```php
// OUTDATED - X-XSS-Protection is deprecated
->header('X-XSS-Protection', '1; mode=block')
```
Modern browsers ignore this header, and it can introduce vulnerabilities.

#### Service Pattern Violates DIP
```php
// CURRENT (Violates DIP as documented)
public function __construct(
    protected ProductService $service  // Concrete class
) {}

// SHOULD BE
public function __construct(
    protected ProductServiceInterface $service  // Interface
) {}
```

#### DTO Implementation Issues
```php
// PROBLEMATIC: Using reflection for toArray() is slow
public function toArray(): array
{
    $reflection = new ReflectionClass($this);
    // ... performance cost on every call
}
```

#### Model Contains Business Logic
```php
// IN eloquent-patterns - contradicts SRP claim
public function decrementStock(int $amount): bool
{
    if ($this->quantity < $amount) {
        return false;
    }
    $this->decrement('quantity', $amount);
    return true;
}
```

### 4. OVER-ENGINEERING CONCERNS

#### DTO Proliferation
For a typical entity, the pattern requires 5+ DTOs:
- `CreateProductDto` - Reasonable
- `UpdateProductDto` - Reasonable
- `DeleteProductDto` - **Overkill** - just pass the ID
- `FindProductDto` - **Overkill** - just pass the ID
- `ListProductDto` - Reasonable for complex filtering

#### Deep Directory Nesting
```
app/Modules/{ModuleName}/Http/Controllers/api/{Entity}Controller.php
```
That's 6 levels deep.

### 5. PRIORITY RANKING

| Priority | Item | Effort | Impact |
|----------|------|--------|--------|
| **P0** | Service interface extraction | Medium | High |
| **P0** | Security header updates | Low | High |
| **P0** | Laravel 11 features | Medium | High |
| **P0** | Fix test mock example | Low | Medium |
| **P1** | Action pattern | Medium | High |
| **P1** | Queue/Job patterns | High | High |
| **P1** | API documentation | Medium | High |
| **P1** | Simplify DTOs | Low | Medium |

---

## Assessment 2: Backend Engineering Specialist

**Grade: 6.5/10**

### Executive Summary

This Laravel specialist agent configuration represents a **solid intermediate-level architecture** that covers the basics well but lacks the depth expected for truly enterprise-grade, scalable systems. It would work adequately for a startup or mid-sized application with moderate traffic, but shows significant gaps in observability, resilience, and scalability patterns.

### 1. Architecture Assessment

#### Strengths
- **Modular Monolith is a Sound Choice**: Provides clear module boundaries without network overhead
- **Separation of Concerns is Properly Implemented**: The layer separation follows established patterns

#### Critical Issues

**Issue 1: Repository Pattern Absence**
Services directly call Eloquent models, tightly coupling business logic to the ORM.

**Issue 2: Transaction Retry Logic is a Code Smell**
```php
}, attempts: 5);
```
If deadlocks are common enough to require 5 retries, the database schema or access patterns need architectural review.

**Issue 3: Business Logic Leaking into Models**
```php
public function decrementStock(int $amount): bool
```
This contradicts the stated SRP principles.

### 2. Scalability Analysis

#### Critical Gaps

**Gap 1: No Horizontal Scaling Architecture**
- No mention of database read replicas
- No session handling for multi-server
- No queue worker scaling patterns
- No database connection pooling configuration

**Gap 2: Naive Cache Invalidation**
```php
Cache::tags(['product'])->flush();
```
- Over-invalidation: Updating one product flushes ALL product cache
- No versioning
- No cache warming

**Gap 3: Offset-Based Pagination Doesn't Scale**
```json
{"meta": { "current_page": 1, "last_page": 5, "per_page": 15, "total": 75 }}
```
`LIMIT 100000, 15` requires scanning 100,015 rows.

### 3. Missing Backend Patterns (Critical)

#### Observability (Grade: F)

| Pattern | Status |
|---------|--------|
| Structured logging | Missing |
| Distributed tracing | Missing |
| Request correlation IDs | Missing |
| Metrics (Prometheus/StatsD) | Missing |
| APM integration | Missing |
| Health check endpoints | Missing |
| Error tracking (Sentry) | Not detailed |

#### Resilience Patterns (Grade: D)
- No circuit breakers
- No bulkhead isolation
- No timeouts configured
- No retry with exponential backoff
- No fallback patterns
- No feature flags

### 4. API Design Critique

**No API Versioning Strategy**

**Non-Standard Response Format**
```json
{
    "meta": {
        "context": "show|store|update|destroy|list_item"
    }
}
```
The `context` field is non-standard and confusing.

**No Idempotency Keys** for POST/PUT operations.

### 5. Priority Ranking

| Priority | Item | Impact | Effort |
|----------|------|--------|--------|
| **P0** | Observability (logging, tracing, health) | Critical | Medium |
| **P0** | API versioning | High | Low |
| **P0** | Request correlation IDs | High | Low |
| **P1** | Repository pattern abstraction | High | High |
| **P1** | Cursor-based pagination | High | Medium |
| **P1** | Cache invalidation strategy | High | Medium |
| **P1** | Circuit breakers for external calls | High | Medium |

---

## Assessment 3: PHP Language Specialist

**Grade: C+/B-**

### Executive Summary

This configuration demonstrates **solid intermediate-level PHP knowledge** but contains several issues that a senior PHP engineer would flag. While the fundamentals are correct, there are **critical anti-patterns**, **missing PHP 8.3 features**, **type system misuse**, and **performance concerns**.

### 1. CRITICAL PHP Issues

#### `new static()` on Readonly Classes is Semantically Wrong
```php
readonly class CreateProductDto extends BaseDto
{
    public static function fromApiRequest(Request $request): static
    {
        return new static(...);  // Problematic with readonly
    }
}
```

**Correct Pattern:**
```php
final readonly class CreateProductDto extends BaseDto
{
    public static function fromApiRequest(Request $request): self
    {
        return new self(...);
    }
}
```

#### Float for Monetary Values
```php
public float $price,
```
Using `float` for money causes IEEE 754 precision issues:
```php
$price = 0.1 + 0.2;  // 0.30000000000000004, NOT 0.3
```

#### Interface Method Signature Conflict
```php
interface FromApiRequestInterface
{
    public static function fromApiRequest(Request $request): static;
}

interface FromApiRequestWithIdInterface
{
    public static function fromApiRequest(Request $request, int|string $id): static;
}
```
Both interfaces define `fromApiRequest()` with DIFFERENT signatures - PHP does not support method overloading.

#### Stringly-Typed Sort Parameters (Contradicts Own Advice)
```php
public ?string $_sort = null,
public ?string $_order = null,
```
The configuration explicitly warns against "stringly-typed code" but uses strings for sort parameters.

### 2. Missing PHP 8.x Features

| Feature | Version | Impact |
|---------|---------|--------|
| `#[\Override]` | 8.3 | Compile-time error if parent method missing |
| `#[\SensitiveParameter]` | 8.2 | Prevents passwords in stack traces |
| Typed class constants | 8.3 | Type safety for constants |
| `json_validate()` | 8.3 | Efficient JSON validation |

### 3. Performance Implications

#### Reflection in Hot Path
```php
public function toArray(): array
{
    $reflection = new ReflectionClass($this);
    // Called on EVERY DTO conversion
}
```
Should cache property names per class.

#### No Mention of Opcache Configuration
Production PHP REQUIRES proper opcache configuration.

### 4. PSR Compliance Issues

#### PSR-4 Violation: Lowercase Namespace Component
```php
namespace App\Modules\Product\Http\Controllers\api;  // lowercase 'api'
```
Should be `Api` (PascalCase).

### 5. Priority Ranking

| Priority | Issue | Impact |
|----------|-------|--------|
| **P0** | Float for money values | Financial data corruption |
| **P0** | Interface signature conflict | Runtime errors |
| **P0** | `new static()` on readonly | Semantic incorrectness |
| **P1** | Missing `#[\Override]` | Refactoring safety |
| **P1** | Missing `#[\SensitiveParameter]` | Security |
| **P1** | Stringly-typed sort params | Violates own guidelines |
| **P1** | Reflection caching | Performance |

---

## Assessment 4: OOP Design Specialist

**Grade: 5/10**

### Executive Summary

This configuration represents a **pragmatic Laravel development approach** with reasonable separation of concerns, but falls significantly short of true OOP design principles. It is fundamentally a **Transaction Script architecture** dressed in object-oriented clothing.

### 1. SOLID Analysis

#### Single Responsibility Principle: PARTIALLY VIOLATED

**Services violate SRP severely:**
```php
class ProductService
{
    public function create(...) { }
    public function update(...) { }
    public function delete(...) { }
    public function find(...) { }
    public function list(...) { }
    protected function invalidateCache() { }
}
```
This class has at least 5 distinct reasons to change - a **God Service** anti-pattern.

#### Open/Closed Principle: EXPLICITLY VIOLATED
> "Open-Closed Principle is relaxed for fresh projects"

This is technical debt by design. No extension points exist.

#### Dependency Inversion Principle: FUNDAMENTALLY VIOLATED

```php
public function __construct(
    protected ProductService $service  // CONCRETE CLASS
) {}
```

The configuration claims DIP but shows concrete service classes without interfaces.

### 2. Design Patterns Assessment

| Pattern | Status | Quality |
|---------|--------|---------|
| DTO | Present | Good structure, but HTTP-coupled |
| Service Layer | Present | Actually Transaction Script |
| Repository | **MISSING** | Critical gap |
| Value Objects | **MISSING** | Primitive obsession |
| Domain Events | **MISSING** | Using Laravel events |
| Aggregates | **MISSING** | No consistency boundaries |

### 3. Anti-patterns Detected

| Anti-pattern | Severity | Description |
|--------------|----------|-------------|
| Anemic Domain Model | CRITICAL | Logic in services, data in models |
| Transaction Script | HIGH | Procedural code in class wrappers |
| Primitive Obsession | HIGH | Using primitives instead of Value Objects |
| Feature Envy | HIGH | Services heavily operate on model data |
| God Class Risk | MEDIUM | Single service handles all operations |
| Leaky Abstraction | MEDIUM | HTTP knowledge in DTOs |

### 4. Missing OOP Concepts

**Must Add (Critical):**
1. Repository Pattern
2. Value Objects (Money, Email, ProductId)
3. Service Interfaces

**Should Add (Important):**
4. Domain Events (proper)
5. Aggregate Roots
6. Specifications

### 5. Priority Ranking

| Priority | Issue | Impact | Effort |
|----------|-------|--------|--------|
| **P0** | Add Repository Pattern | Decouples from ORM | Medium |
| **P0** | Remove HTTP from DTOs | Layer separation | Low |
| **P0** | Add Service Interfaces | DIP, testability | Low |
| **P1** | Add Value Objects | Domain modeling | Medium |
| **P1** | Split God Services | SRP compliance | Medium |
| **P1** | Define Module Contracts | Module independence | Medium |

---

# PART 2: CONSENSUS REVIEWS (Cross-Assessment)

---

## Consensus Review 1: Laravel Specialist Perspective

### Consensus Analysis: Where All Specialists Agree

| Issue | Specialists | Verdict |
|-------|-------------|---------|
| DTO/Reflection performance | Laravel, PHP, OOP | **VALID** |
| API versioning missing | Laravel, Backend | **VALID** |
| Value Objects missing | PHP, OOP | **VALID** - especially for Money |
| "Relaxed OCP" normalizes debt | Laravel, OOP | **VALID** |
| Float for money | PHP | **CRITICAL BUG** |

### Conflict Resolution

#### Repository Pattern
**Backend & OOP say:** Add repository pattern (P0)
**Laravel reality:** Eloquent IS the repository

**RULING: NOT RECOMMENDED for typical Laravel apps**
Repository pattern shines when you might swap databases. Laravel apps rarely do this.

#### Service Interfaces (DIP)
**RULING: CONTEXT-DEPENDENT**
Add interfaces for:
- Services with multiple implementations
- External service integrations
- Cross-module boundaries

#### Level of DDD/OOP Rigor
**RULING: OOP specialist is grading the wrong exam**
Laravel intentionally trades OOP purity for developer velocity.

### Unified Priority List

#### P0 - CRITICAL BUGS (Fix Immediately)
1. Float for money
2. `new static()` on readonly classes
3. Interface signature conflict
4. Security headers outdated

#### P1 - HIGH PRIORITY (Next Sprint)
1. Add `#[\Override]`
2. Add `#[\SensitiveParameter]`
3. Cache reflection in BaseDto
4. Add `Route::health()`
5. Remove DTO proliferation
6. Fix stringly-typed sorts

#### P2 - MEDIUM PRIORITY (This Quarter)
1. Add Value Objects pattern
2. API versioning guidance
3. Remove HTTP from DTOs
4. Add observability basics
5. Cursor pagination guidance

#### NOT RECOMMENDED (Over-Engineering for Laravel)
- Full Repository Pattern
- Aggregates/Bounded Contexts
- Circuit Breakers/Bulkheads
- Strict DIP everywhere

### Final Grade: C+

---

## Consensus Review 2: Backend Engineering Perspective

### Production Incident Risk Matrix

#### TIER 1: Will Cause Production Incidents (Fix Immediately)

| Issue | Severity | Why It Matters |
|-------|----------|----------------|
| Float for money | CRITICAL | Financial discrepancies |
| No observability | CRITICAL | Blind debugging |
| `new static()` on readonly | HIGH | Runtime fatal errors |
| Missing `#[\SensitiveParameter]` | HIGH | Passwords in logs |
| No health endpoints | HIGH | K8s cannot manage lifecycle |

#### TIER 2: Will Cause Scaling Issues

| Issue | When It Hurts |
|-------|---------------|
| Naive cache flush | >1000 users |
| Offset pagination | >100k rows |
| 5x transaction retry | Under load |
| Reflection in hot path | High traffic |

### Practical vs Academic Assessment

```
PRODUCTION REALITY CHECK:
✓ Can you debug issues at 3 AM?           → Observability
✓ Does it survive external API failures?  → Resilience
✓ Does it handle 10x traffic?             → Scalability
✓ Are financial calculations accurate?    → Correctness

NOT:
✗ Does it follow hexagonal architecture?
✗ Are all SOLID principles satisfied?
```

### The 80/20 Rule

**FIX THESE 5 THINGS FOR 80% PRODUCTION READINESS:**
1. Replace float with proper money handling
2. Add structured logging + health endpoints
3. Fix cache invalidation strategy
4. Add `#[\SensitiveParameter]` to credentials
5. Implement cursor pagination for large datasets

**IGNORE THESE UNTIL YOU HAVE PROBLEMS:**
- Repository pattern abstraction
- Full DDD implementation
- Value objects for simple types
- Domain events

---

## Consensus Review 3: PHP Specialist Perspective

### PHP Correctness Classification

#### ACTUAL PHP BUGS/ERRORS (Must Fix)

| Issue | Severity | Explanation |
|-------|----------|-------------|
| Float for money | CRITICAL | IEEE 754 precision loss |
| Interface signature conflict | CRITICAL | Fatal error at compile time |
| PSR-4 case mismatch | HIGH | Autoloading fails on Linux |
| `new static()` on readonly | MEDIUM | Violates immutability contracts |

#### RUNTIME SAFETY ISSUES (Should Fix)

| Issue | Impact |
|-------|--------|
| Missing `#[\SensitiveParameter]` | Passwords in stack traces |
| Stringly-typed sort params | Invalid strings pass type checks |
| No exception hierarchy | Loss of error handling granularity |

#### STYLE/ARCHITECTURE PREFERENCES (Not PHP Errors)

| Issue | Reality |
|-------|---------|
| Service interfaces missing | DIP is a principle, not PHP requirement |
| No repository pattern | Both Active Record and Repository are valid |
| Action pattern missing | Organizational choice |
| Anemic Domain Model | Common in Laravel, not a PHP bug |

### The 4 Non-Negotiable Fixes

1. **Never use float for money** - Use `bcmath` or integer cents
2. **Verify interface compatibility** - Resolve signature conflicts
3. **Fix PSR-4 namespace casing** - Match filesystem case exactly
4. **Add `#[\SensitiveParameter]`** - Protect credentials in stack traces

---

## Consensus Review 4: OOP Specialist Perspective

### OOP Pragmatism: Worth Fighting vs Academic

#### CRITICAL (Non-Negotiable OOP Concerns)

| Concern | Why It Matters | Laravel Solution |
|---------|----------------|------------------|
| Float for Money | Real financial bugs | `brick/money` or Money VO |
| God Services | Untestable, unmaintainable | Action classes |
| No Service Interfaces for Externals | Can't swap gateways | Interface + binding |
| Hidden Module Coupling | Ripple effect changes | Domain events |

#### ACADEMIC (Fighting Laravel)

| Concern | Reality Check |
|---------|---------------|
| Full Repository Pattern | Eloquent IS a repository |
| Pure Domain Models | Active Record is Laravel's choice |
| Eliminating Facades | Facades are testable via `::fake()` |
| Full DDD Aggregates | Overkill for 90% of Laravel apps |

### The Right Abstraction Level

```
┌─────────────────────────────────────────────────────────────────┐
│                    RECOMMENDED ARCHITECTURE                     │
│                                                                 │
│  Controllers ───▶ Actions ───▶ Services                        │
│  (HTTP only)     (Use Cases)   (Domain Logic)                   │
│       │               │              │                          │
│       ▼               ▼              ▼                          │
│  Request DTOs    Domain DTOs    Models (Eloquent)               │
│  (HTTP aware)    (Pure PHP)                                     │
│                       │              │                          │
│                       ▼              ▼                          │
│                 Value Objects    Events                         │
│                 (Money, etc.)   (Decoupling)                    │
│                                                                 │
│  INTERFACES FOR: Payment, Mail, SMS, External APIs              │
│  CONCRETE FOR: Eloquent Models, Internal Services               │
└─────────────────────────────────────────────────────────────────┘
```

### The 80/20 Rule for OOP

**20% of changes that give 80% of improvement:**
1. Money Value Object - Prevents real bugs
2. Service Interfaces for Externals - Enables testing
3. Action Classes - Kills God Services
4. Domain Events - Eliminates coupling
5. Exception Hierarchy - Consistent errors

---

# PART 3: FINAL UNIFIED CONSENSUS

---

## Overall Configuration Grades

| Specialist | Initial Grade | After Consensus |
|------------|---------------|-----------------|
| Laravel | B- | C+ to B- |
| Backend | 6.5/10 | 6.5/10 |
| PHP | C+/B- | B- |
| OOP | 5/10 | 7/10 (context-adjusted) |

**Final Consensus Grade: C+ (with clear path to B+)**

---

## Unanimous Agreement (All 4 Specialists)

1. **Float for money is dangerous** - Must use integer cents or Money VO
2. **Observability is critically missing** - Need logging, health checks
3. **BaseDto reflection should be cached** - Performance concern
4. **"Relaxed OCP" language is problematic** - Normalizes tech debt
5. **API versioning should be addressed** - Essential for production APIs

---

## Priority Matrix (Consensus)

### IMMEDIATE (This Week)

| Issue | Action | Consensus |
|-------|--------|-----------|
| Float money | Replace with int cents or Money VO | 4/4 |
| Interface conflict | Fix or separate interfaces | 4/4 |
| PSR-4 namespace | Change `api` to `Api` | 3/4 |
| `#[\SensitiveParameter]` | Add to password params | 3/4 |
| Security headers | Remove X-XSS-Protection | 3/4 |

### SHORT-TERM (This Month)

| Issue | Action | Consensus |
|-------|--------|-----------|
| `#[\Override]` | Add to overriding methods | 3/4 |
| Readonly classes | Use `final readonly` + `new self()` | 3/4 |
| Health endpoints | Add `Route::health()` | 3/4 |
| Reflection caching | Cache in BaseDto | 3/4 |
| Stringly-typed sorts | Create enums | 3/4 |

### MEDIUM-TERM (This Quarter)

| Issue | Action | Consensus |
|-------|--------|-----------|
| Value Objects | Add Money, Email, etc. | 3/4 |
| HTTP in DTOs | Separate Request/Domain DTOs | 2/4 |
| Cursor pagination | For large datasets | 2/4 |
| Exception hierarchy | Create base + specifics | 2/4 |
| God Services | Split into Actions | 2/4 |

### CONTESTED (Context-Dependent)

| Issue | For | Against | Verdict |
|-------|-----|---------|---------|
| Repository Pattern | Backend, OOP | Laravel, PHP | Only for externals |
| Full DDD | OOP | All others | Only for complex domains |
| Interface everything | OOP | Laravel | Strategic only |

---

## Recommended Code Fixes

### Fix 1: Money Handling (CRITICAL)
```php
// BEFORE (dangerous)
public float $price;

// AFTER (safe)
use Brick\Money\Money;

final readonly class Price
{
    private function __construct(private Money $amount) {}

    public static function fromCents(int $cents): self
    {
        return new self(Money::ofMinor($cents, 'USD'));
    }
}
```

### Fix 2: Readonly Classes
```php
// BEFORE (problematic)
readonly class CreateUserDto {
    public static function from(): static { return new static(); }
}

// AFTER (correct)
final readonly class CreateUserDto {
    public static function from(): self { return new self(); }
}
```

### Fix 3: Health Endpoint
```php
Route::get('/health', function () {
    return response()->json([
        'status' => 'healthy',
        'database' => DB::connection()->getPdo() ? 'ok' : 'fail',
        'cache' => Cache::store()->get('health-check', 'ok'),
    ]);
});
```

### Fix 4: Sensitive Parameters
```php
public function authenticate(
    string $email,
    #[\SensitiveParameter] string $password
): AuthResult {
    // Password won't appear in stack traces
}
```

### Fix 5: Cache Reflection
```php
abstract readonly class BaseDto
{
    private static array $propertyCache = [];

    public function toArray(): array
    {
        $class = static::class;
        if (!isset(self::$propertyCache[$class])) {
            $reflection = new ReflectionClass($class);
            self::$propertyCache[$class] = array_map(
                fn($p) => $p->getName(),
                $reflection->getProperties(ReflectionProperty::IS_PUBLIC)
            );
        }

        $result = [];
        foreach (self::$propertyCache[$class] as $prop) {
            $result[$prop] = $this->{$prop};
        }
        return $result;
    }
}
```

---

## Final Recommendations Summary

### Do Now
1. Fix float money handling
2. Fix interface signature conflict
3. Add health check endpoint
4. Add `#[\SensitiveParameter]`
5. Update security headers

### Do Soon
1. Add `#[\Override]` attribute usage
2. Fix readonly class patterns
3. Cache BaseDto reflection
4. Create sort/filter enums
5. Add Money value object

### Do Later
1. Consider Action pattern for complex operations
2. Add cursor pagination for scale
3. Separate HTTP/Domain DTOs
4. Create exception hierarchy
5. Add structured logging

### Don't Do (Unless Needed)
1. Full repository pattern
2. Complete DDD implementation
3. Interface for every service
4. Aggregates and bounded contexts

---

*Document generated by 4 independent specialist agents with cross-review consensus.*
