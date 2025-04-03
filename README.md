# developerkeynotes
Some regular stuff a developer will face in their daily jobl


# 🧠 Super Charge a Slow API Response

## 🎯 Goal
Turn a sluggish ASP.NET Core Web API into a fast, scalable, and responsive API even when it relies on slow external services.

---

## 📊 Step-by-Step Investigation: Where's the Lag?

### 1. Measure Actual Latency
- **Tool**: Postman, Fiddler, Swagger UI
- **Why**: Establish baseline response time

### 2. Enable Profiling
- **Tool**: Application Insights, MiniProfiler, Serilog + Stopwatch
- **Code**:
  ```csharp
  var sw = Stopwatch.StartNew();
  var result = await _service.GetDataAsync();
  sw.Stop();
  _logger.LogInformation("Service call took {ms}ms", sw.ElapsedMilliseconds);
  ```

### 3. Log External Call Times
- Wrap external service calls in Stopwatch and log duration

### 4. Inspect External Service
- Use Postman/curl directly on external API
- Measure response latency independent of your API

### 5. Profile the Full Pipeline
- Analyze EF Core (if applicable), controller logic, serialization time

---

## 🚀 Solution: Code + Explanation + Best Practices

### ✅ 1. Use Async/Await Everywhere
- **Why**: Frees up threads, supports high throughput
- **Watch out**: Avoid `.Result` or `.Wait()`

### ✅ 2. Use HttpClientFactory with Polly
- **Code**:
  ```csharp
  services.AddHttpClient<IExternalService, ExternalService>()
      .SetHandlerLifetime(TimeSpan.FromMinutes(5))
      .AddPolicyHandler(Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(3)))
      .AddPolicyHandler(HttpPolicyExtensions
         .HandleTransientHttpError()
         .WaitAndRetryAsync(3, retry => TimeSpan.FromMilliseconds(300 * retry))
      );
  ```
- **Why**: Prevents socket exhaustion, adds retry + timeout
- **Best Practice**: Tune retry counts and backoff strategies

### ✅ 3. Add In-Memory Caching
- **Code**:
  ```csharp
  if (_cache.TryGetValue(id, out ProductDto cached)) return cached;
  var result = await _client.GetAsync($"/products/{id}");
  _cache.Set(id, result, TimeSpan.FromMinutes(10));
  ```
- **Why**: Reduces redundant calls
- **Watch out**: Invalidate cache carefully for stale data

### ✅ 4. Parallelize Multiple Calls
- **Code**:
  ```csharp
  var task1 = _svc.GetProduct();
  var task2 = _svc.GetReviews();
  await Task.WhenAll(task1, task2);
  ```
- **Why**: Saves time if services are independent

### ✅ 5. Compress Responses
- **Configure in Program.cs**:
  ```csharp
  services.AddResponseCompression();
  app.UseResponseCompression();
  ```
- **Why**: Speeds up response over the wire

### ✅ 6. Use Middleware to Log Slow Requests
- **Code**:
  ```csharp
  public class SlowRequestLogger
  {
      public async Task Invoke(HttpContext context)
      {
          var sw = Stopwatch.StartNew();
          await _next(context);
          sw.Stop();
          if (sw.ElapsedMilliseconds > 1000)
              _logger.LogWarning($"Slow request: {context.Request.Path} took {sw.ElapsedMilliseconds}ms");
      }
  }
  ```
- **Why**: Detect and monitor performance spikes

---

## ✅ Summary Best Practices Checklist

| Area              | Best Practice                                      |
|-------------------|----------------------------------------------------|
| Async             | Always use async/await, avoid blocking calls       |
| Resilience        | Use Polly: retry + timeout + circuit breaker       |
| Performance       | Cache repeat calls (MemoryCache/Redis)             |
| Throughput        | Use IHttpClientFactory, avoid creating HttpClient  |
| Logging           | Add slow request middleware + Serilog structured logs |
| UX                | Return fast, load heavy data in background         |

---

## 🧪 Extra Recommendations

- Use distributed caching for scalability
- Pre-fetch data if API is predictable
- Use pagination and filtering to reduce payload size
- Use DTOs, not full entities, for leaner responses

---

## 📦 Result
With these changes, your API will:
- Respond faster even when external services are slow
- Handle high load with less CPU/thread pressure
- Be observable, testable, and production-ready

# 📘 Project Document: TDD Development – Mastering Testing & Observability in ASP.NET Core

## 🎯 Objective
Learn and master TDD (Test-Driven Development), unit testing, integration testing, and observability for features like repositories with multiple dependencies in ASP.NET Core.

---

## 🧭 Course Roadmap

| Week | Focus Area                                 | Outcome                                                |
|------|---------------------------------------------|--------------------------------------------------------|
| 1️⃣   | TDD Basics + Unit Testing Fundamentals      | Understand red-green-refactor, test structure          |
| 2️⃣   | Mocking + Dependency Injection in Testing   | Master Moq/NSubstitute + mocking services/repos        |
| 3️⃣   | Testing ASP.NET Core Controllers            | Unit test controllers with DI dependencies             |
| 4️⃣   | Testing Business Logic + Services           | Test services independently of DB or web               |
| 5️⃣   | Testing Repositories (EF Core, external)    | Unit test with in-memory DB or fakes                   |
| 6️⃣   | Integration Testing + In-Memory Web Server  | Test API endpoints + HTTP pipeline with real data      |
| 7️⃣   | Observability (Logging, Metrics, Tracing)   | Add and verify structured logs, health checks, metrics |
| 8️⃣   | Final Project: TDD for Order Feature        | Build full feature with TDD from scratch               |

---

# 📘 Project Document: TDD Development – Mastering Testing & Observability in ASP.NET Core

## 🎯 Objective
Learn and master TDD (Test-Driven Development), unit testing, integration testing, and observability for features like repositories with multiple dependencies in ASP.NET Core.

---

## 🧭 Course Roadmap with Step-by-Step Implementation

### Week 1: TDD + Unit Testing Fundamentals

#### ✅ What to Learn
- Red → Green → Refactor cycle
- What is a unit test?
- Using `xUnit` test framework

#### 🛠 How to Implement
1. **Create Test Project**:
   ```bash
   dotnet new xunit -n MyApp.Tests
   dotnet add reference ../MyApp.Core
   ```

2. **Add a Class to Test** (in `MyApp.Core`):
   ```csharp
   public class Calculator
   {
       public int Add(int a, int b) => a + b;
   }
   ```

3. **Write a Failing Test First** (in `MyApp.Tests`):
   ```csharp
   public class CalculatorTests
   {
       [Fact]
       public void Add_TwoNumbers_ReturnsSum()
       {
           var calc = new Calculator();
           var result = calc.Add(2, 2);
           Assert.Equal(5, result); // Wrong on purpose
       }
   }
   ```

4. **Make It Pass**:
   ```csharp
   Assert.Equal(4, result); // Corrected
   ```

#### 💡 Best Practices
- Write minimal code to pass the test
- Refactor after test passes

#### ⚠️ Watch Out
- Avoid testing implementation details
- Tests should not depend on order of execution

#### 🔗 Related Topics
- AAA pattern (Arrange-Act-Assert)
- Naming conventions for tests

---

### Week 2: Mocking & Dependency Injection

#### ✅ What to Learn
- How to isolate code with mocks
- Using `Moq` to fake dependencies

#### 🛠 How to Implement
1. **Create Interface + Class**:
   ```csharp
   public interface IOrderRepository
   {
       Order GetById(int id);
   }

   public class OrderService
   {
       private readonly IOrderRepository _repo;
       public OrderService(IOrderRepository repo) => _repo = repo;
       public Order Get(int id) => _repo.GetById(id);
   }
   ```

2. **Test with Moq**:
   ```csharp
   var mock = new Mock<IOrderRepository>();
   mock.Setup(r => r.GetById(1)).Returns(new Order { Id = 1 });
   var service = new OrderService(mock.Object);
   var result = service.Get(1);
   Assert.Equal(1, result.Id);
   ```

#### 💡 Best Practices
- Use `Setup` for expected behavior
- Verify with `mock.Verify()` if needed

#### ⚠️ Watch Out
- Don’t overuse mocks for everything
- Avoid mocking concrete classes

#### 🔗 Related Topics
- Stub vs Mock vs Fake
- AutoFixture for auto-data generation

---

### Week 3: Testing ASP.NET Core Controllers

#### ✅ What to Learn
- Unit testing controllers that use DI

#### 🛠 How to Implement
1. **Controller**:
   ```csharp
   public class OrdersController : ControllerBase
   {
       private readonly IOrderService _service;
       public OrdersController(IOrderService service) => _service = service;

       [HttpGet("{id}")]
       public IActionResult Get(int id)
       {
           var order = _service.GetById(id);
           return Ok(order);
       }
   }
   ```

2. **Test**:
   ```csharp
   var mock = new Mock<IOrderService>();
   mock.Setup(s => s.GetById(1)).Returns(new OrderDto { Id = 1 });
   var controller = new OrdersController(mock.Object);
   var result = controller.Get(1);
   var ok = Assert.IsType<OkObjectResult>(result);
   Assert.Equal(1, ((OrderDto)ok.Value).Id);
   ```

#### 💡 Best Practices
- Test response types and content
- Use `ObjectResult` and `StatusCodeResult` types

#### ⚠️ Watch Out
- Avoid testing routing/DI — that’s for integration tests

#### 🔗 Related Topics
- ActionResult<T>
- FluentAssertions for readability

---

### Week 4: Testing Services

#### ✅ What to Learn
- How to test services in isolation from controllers/repos

#### 🛠 How to Implement
1. **Service**:
   ```csharp
   public class DiscountService
   {
       public decimal ApplyDiscount(decimal price, decimal discountPercent)
       {
           return price - (price * discountPercent / 100);
       }
   }
   ```

2. **Test**:
   ```csharp
   [Fact]
   public void ApplyDiscount_ValidInput_ReturnsDiscountedPrice()
   {
       var service = new DiscountService();
       var result = service.ApplyDiscount(100, 10);
       Assert.Equal(90, result);
   }
   ```

---

### Week 5: Testing Repositories (EF Core)

#### ✅ What to Learn
- Use `InMemoryDatabase` to test EF logic

#### 🛠 How to Implement
1. **Setup DbContext**:
   ```csharp
   var options = new DbContextOptionsBuilder<AppDbContext>()
       .UseInMemoryDatabase("TestDb")
       .Options;

   var context = new AppDbContext(options);
   context.Orders.Add(new Order { Id = 1 });
   context.SaveChanges();
   ```

2. **Test Repository**:
   ```csharp
   var repo = new OrderRepository(context);
   var order = repo.GetById(1);
   Assert.NotNull(order);
   ```

---

### Week 6: Integration Testing

#### ✅ What to Learn
- Use `WebApplicationFactory` to test entire HTTP pipeline

#### 🛠 How to Implement
```csharp
var factory = new WebApplicationFactory<Program>();
var client = factory.CreateClient();
var response = await client.GetAsync("/api/orders/1");
response.EnsureSuccessStatusCode();
```

---

### Week 7: Observability

#### ✅ What to Learn
- Logging, metrics, and health checks

#### 🛠 How to Implement
1. **Add Serilog**:
   ```csharp
   Log.Logger = new LoggerConfiguration()
       .WriteTo.Console()
       .CreateLogger();
   ```
2. **Health Check Endpoint**:
   ```csharp
   services.AddHealthChecks();
   app.MapHealthChecks("/health");
   ```

---

### Week 8: Final Project – Order Feature via TDD

#### ✅ Step-by-Step
1. Write failing test: `GetById_ShouldReturnOrder`
2. Implement only enough logic to pass
3. Add repo mock + controller test
4. Add integration test for `/api/orders/{id}`
5. Add structured logs in controller + service
6. Add `/health` and ensure logs/metrics emit properly

---

## 🎓 Outcome
- You’ll be confident writing testable ASP.NET Core code
- You’ll have real-world examples for repositories, services, and controllers
- You’ll understand TDD from red → green → refactor
- You’ll build observable, test-driven, maintainable software 💪

