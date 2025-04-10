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

# 🚀 TDD Final Project – Order Feature

## 🎯 Goal
Demonstrate full TDD mastery by implementing an end-to-end **Order Management** feature in ASP.NET Core using:
- ✅ Unit Testing
- ✅ Mocking & DI
- ✅ Integration Testing
- ✅ Observability (logging, health)

---

## 🧱 Project Structure

```
/OrderApi
├── Controllers/
│   └── OrdersController.cs
├── Services/
│   └── OrderService.cs
├── Repositories/
│   └── OrderRepository.cs
├── Domain/
│   └── Order.cs
├── Program.cs

/OrderApi.Tests
├── Unit/
│   └── OrderServiceTests.cs
│   └── OrdersControllerTests.cs
├── Integration/
│   └── OrderEndpointsTests.cs
```

---

## 🧪 Section 1: Unit Testing

### 1️⃣ `OrderServiceTests.cs`
```csharp
[Fact]
public void CalculateTotal_WhenCalled_ReturnsExpectedSum()
{
    var service = new OrderService(null!);
    var total = service.CalculateTotal(100, 5);
    Assert.Equal(105, total);
}
```

### 2️⃣ Testing logic with mocked repository
```csharp
[Fact]
public void GetOrder_ReturnsCorrectOrder()
{
    var mockRepo = new Mock<IOrderRepository>();
    mockRepo.Setup(x => x.GetById(1)).Returns(new Order { Id = 1 });
    var service = new OrderService(mockRepo.Object);
    var result = service.GetOrder(1);
    Assert.Equal(1, result.Id);
}
```

### 3️⃣ Test business rule logic
```csharp
[Fact]
public void IsPremiumOrder_WhenAmountOver100_ReturnsTrue()
{
    var service = new OrderService(null!);
    var isPremium = service.IsPremium(150);
    Assert.True(isPremium);
}
```

---

## 🤖 Section 2: Mocking + DI

### 1️⃣ Controller test with mocked service
```csharp
[Fact]
public async Task GetOrder_ReturnsOk()
{
    var mockService = new Mock<IOrderService>();
    mockService.Setup(x => x.GetOrder(1)).Returns(new OrderDto { Id = 1 });
    var controller = new OrdersController(mockService.Object);
    var result = controller.Get(1);
    var ok = Assert.IsType<OkObjectResult>(result);
    Assert.Equal(1, ((OrderDto)ok.Value!).Id);
}
```

### 2️⃣ Verify method call
```csharp
[Fact]
public void PlaceOrder_CallsRepositoryOnce()
{
    var mockRepo = new Mock<IOrderRepository>();
    var service = new OrderService(mockRepo.Object);
    service.PlaceOrder(new Order { Id = 2 });
    mockRepo.Verify(r => r.Save(It.Is<Order>(o => o.Id == 2)), Times.Once);
}
```

### 3️⃣ Mock failed dependency
```csharp
[Fact]
public void GetOrder_RepoThrows_LogsError()
{
    var logger = new Mock<ILogger<OrderService>>();
    var repo = new Mock<IOrderRepository>();
    repo.Setup(x => x.GetById(It.IsAny<int>())).Throws(new Exception("DB fail"));
    var service = new OrderService(repo.Object, logger.Object);

    Assert.Throws<Exception>(() => service.GetOrder(1));
    logger.Verify(l => l.Log(
        LogLevel.Error,
        It.IsAny<EventId>(),
        It.Is<It.IsAnyType>((v, _) => v.ToString()!.Contains("DB fail")),
        It.IsAny<Exception>(),
        It.IsAny<Func<It.IsAnyType, Exception?, string>>()
    ));
}
```

---

## 🔌 Section 3: Integration Testing

### Setup
```csharp
var app = new WebApplicationFactory<Program>();
var client = app.CreateClient();
```

### 1️⃣ GET /orders/{id}
```csharp
[Fact]
public async Task GetOrder_ReturnsOrder()
{
    var response = await client.GetAsync("/api/orders/1");
    response.EnsureSuccessStatusCode();
    var content = await response.Content.ReadAsStringAsync();
    Assert.Contains("\"id\":1", content);
}
```

### 2️⃣ POST /orders
```csharp
[Fact]
public async Task CreateOrder_ReturnsCreated()
{
    var order = new StringContent("{\"amount\": 100}", Encoding.UTF8, "application/json");
    var response = await client.PostAsync("/api/orders", order);
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
}
```

### 3️⃣ Health check endpoint
```csharp
[Fact]
public async Task HealthCheck_ReturnsHealthy()
{
    var response = await client.GetAsync("/health");
    response.EnsureSuccessStatusCode();
}
```

---

## 📈 Section 4: Observability

### 1️⃣ Serilog structured logging
```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs/orders.log")
    .CreateLogger();
```

### 2️⃣ Logging in controller
```csharp
_logger.LogInformation("Getting order {id}", id);
```

### 3️⃣ Health check setup
```csharp
builder.Services.AddHealthChecks();
app.MapHealthChecks("/health");
```

---

## 🧾 Summary
| Concept          | Example                         |
|------------------|----------------------------------|
| Unit Test        | `OrderServiceTests`             |
| Mocking + DI     | `OrdersControllerTests`         |
| Integration Test | `OrderEndpointsTests`           |
| Logging          | Serilog + controller injection  |
| Health           | `/health` endpoint with check   |

✅ This project demonstrates how to write scalable, test-first, observable ASP.NET Core applications with TDD discipline.



# Super Market Checkout - C# Multi-threading Simulation

## ✨ Project Overview
This project simulates a realistic **Supermarket Checkout** system using **C# multithreading**, demonstrating concurrency control, resource pooling, task cancellation, and priority handling. It focuses on thread-safe access to limited checkout counters using `SemaphoreSlim`, with features such as:

- ⏳ Auto-cancellation for long-running checkouts
- 📊 Checkout time tracking and performance reporting
- 🤔 Dynamic priority queuing (VIP customers served first)

---

## 🌐 Real-world Scenario
Imagine a supermarket with **3 checkout counters** and **10 customers**:

- Only 3 customers can check out at any given time.
- Some customers are **VIPs** who should get priority.
- Checkout might randomly take between **2–5 seconds**.
- If a customer's checkout takes longer than **4 seconds**, it is **auto-canceled**.
- At the end, we want a report of who checked out and how fast.

---

## 🔧 Core Features and Tools

| Feature                  | Implementation Tool         | Reasoning |
|--------------------------|------------------------------|-----------|
| Concurrency limit        | `SemaphoreSlim(3)`           | Controls simultaneous access to checkout counters |
| Timeout/cancellation     | `CancellationTokenSource`    | Cleanly stops long-running tasks |
| Checkout timing          | `Stopwatch`                  | Tracks how long each customer takes |
| VIP priority             | `List` with custom sorting   | Ensures VIPs are served first |

---

## 📄 Project Code (Console App)
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

class Customer
{
    public int Id { get; set; }
    public bool IsPriority { get; set; }
    public TimeSpan CheckoutDuration { get; set; }
}

class Program
{
    static SemaphoreSlim checkoutCounters = new SemaphoreSlim(3);
    static List<Customer> checkoutHistory = new List<Customer>();

    static async Task CheckoutAsync(Customer customer)
    {
        Console.WriteLine($"\U0001f6d2 Customer {customer.Id} ({(customer.IsPriority ? "VIP" : "Regular")}) is waiting...");

        var globalTimeout = TimeSpan.FromSeconds(4);
        var cts = new CancellationTokenSource(globalTimeout);

        Stopwatch stopwatch = new Stopwatch();

        try
        {
            bool entered = await checkoutCounters.WaitAsync(TimeSpan.FromSeconds(5), cts.Token);

            if (!entered)
            {
                Console.WriteLine($"\u23f1️ Customer {customer.Id} left due to long queue wait.");
                return;
            }

            stopwatch.Start();

            int checkoutTime = new Random().Next(2000, 5000); // Between 2–5 sec
            Console.WriteLine($"✅ Customer {customer.Id} started checkout ({checkoutTime}ms)...");

            await Task.Delay(checkoutTime, cts.Token); // Simulate checkout time

            stopwatch.Stop();
            customer.CheckoutDuration = stopwatch.Elapsed;

            lock (checkoutHistory)
            {
                checkoutHistory.Add(customer);
            }

            Console.WriteLine($"🏁 Customer {customer.Id} finished in {customer.CheckoutDuration.TotalSeconds:F2}s");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine($"❌ Customer {customer.Id}'s checkout was canceled (timeout).");
        }
        finally
        {
            if (checkoutCounters.CurrentCount < 3)
                checkoutCounters.Release();
        }
    }

    static async Task RunStoreAsync()
    {
        List<Customer> customers = Enumerable.Range(1, 10)
            .Select(i => new Customer
            {
                Id = i,
                IsPriority = (i % 3 == 0 || i == 7) // Customers 3, 6, 7, 9 are VIPs
            })
            .ToList();

        customers = customers
            .OrderByDescending(c => c.IsPriority)
            .ThenBy(c => c.Id)
            .ToList();

        var tasks = customers.Select(c => CheckoutAsync(c)).ToList();

        await Task.WhenAll(tasks);

        Console.WriteLine("\n📊 Checkout Report:");

        foreach (var c in checkoutHistory.OrderBy(x => x.CheckoutDuration))
        {
            Console.WriteLine($"- Customer {c.Id} ({(c.IsPriority ? "VIP" : "Reg")}) -> {c.CheckoutDuration.TotalSeconds:F2}s");
        }

        var fastest = checkoutHistory.OrderBy(x => x.CheckoutDuration).FirstOrDefault();
        if (fastest != null)
        {
            Console.WriteLine($"\n🥇 Fastest Customer: {fastest.Id} in {fastest.CheckoutDuration.TotalSeconds:F2} seconds!");
        }

        Console.WriteLine("\n🏪 Store Closed.");
    }

    static async Task Main()
    {
        await RunStoreAsync();
    }
}
```

---

## 🌍 Extensions You Can Try

1. 🤔 Add **billing logic**: charge based on time or items.
2. 🔐 Add **logging** with Serilog or NLog.
3. ✨ Convert to **Blazor UI** for interactive experience.
4. ⏳ Retry logic: failed checkouts can retry.
5. 📊 Export report to file or database.

---

## 📊 Sample Output
```
🚒 Customer 3 (VIP) is waiting...
🚒 Customer 6 (VIP) is waiting...
🚒 Customer 7 (VIP) is waiting...
✅ Customer 3 started checkout (2200ms)...
✅ Customer 6 started checkout (4300ms)...
✅ Customer 7 started checkout (3100ms)...
...
🌟 Checkout Report:
- Customer 3 (VIP) -> 2.20s
- Customer 7 (VIP) -> 3.10s

🥇 Fastest Customer: 3 in 2.20 seconds!
🏪 Store Closed.
```

---

## 🔄 Final Thoughts
This simulation demonstrates how to manage shared resources using `SemaphoreSlim`, enforce execution limits, support priority queues, and handle task cancellation gracefully in C#. It reflects many real-world scenarios like database connection limits, checkout systems, or API throttling.




