# Thread-Safety and Parallel Execution Architecture

### Critical Concept: How Our Tests Run in Parallel

This framework is designed for **thread-safe parallel execution** in CI/CD pipelines. Understanding this architecture is essential for maintaining and extending the test suite.

### The Core Mechanism: ScenarioContext Isolation

Every test scenario gets its own isolated browser instance through Reqnroll's `ScenarioContext` dependency injection:

```csharp
// In TestHooks.cs - Runs once per scenario instance
[BeforeScenario]
public async Task BeforeScenario(ScenarioContext scenarioContext)
{
    _fixture = new InternalSdkFixture();
    await _fixture.InitializeAsync();
    
    // Register instances in THIS scenario's container
    scenarioContext.ScenarioContainer.RegisterInstanceAs(_fixture);
    scenarioContext.ScenarioContainer.RegisterInstanceAs(_fixture.Page);
}

// In Step Definitions - Constructor injection per scenario
public AssignMatchStepDefinitions(ScenarioContext scenarioContext)
{
    _scenarioContext = scenarioContext;
    _page = _scenarioContext.Get<IPage>("Page");
    // Note: We're using Get<T>(string) but could also use dependency injection
    // if registered differently in the container
}
```

### Parallel Execution Model

```
Scenario 1 (Thread A)                 Scenario 2 (Thread B)
─────────────────────                ─────────────────────
ScenarioContext #1                   ScenarioContext #2
       ↓                                    ↓
  IPage #1 (Browser Context 1)       IPage #2 (Browser Context 2)
       ↓                                    ↓
  Page Objects #1                     Page Objects #2

Independent execution - No shared mutable state
```

### ✅ Thread-Safe Patterns

```csharp
✅ CORRECT: Retrieve from ScenarioContext
_page = _scenarioContext.Get<IPage>("Page");

✅ CORRECT: Instance creation with injected dependencies
_venueConfigPage = new VenueConfigSummaryPage(_page);

✅ CORRECT: Store scenario-specific data
_scenarioContext["SelectedVenue"] = venueName;
// Or use typed approach:
_scenarioContext.Set(venueName, "SelectedVenue");

✅ CORRECT: Using ILocator (immutable, lazy-evaluated)
private readonly ILocator _table = _page.Locator("table[mat-table]");

✅ CORRECT - Instance field (each test gets its own) 
private readonly IPage _page;
```

### ❌ Anti-Patterns That Break Thread-Safety

```csharp
❌ WRONG: Static mutable state
public static IPage SharedPage { get; set; }

❌ WRONG: Static collections without synchronization
public static List<string> TestResults = new();

❌ WRONG: Shared file access without locking
File.WriteAllText("shared-output.txt", results);  // Race condition

❌ WRONG: Class-level state shared across scenarios
[Binding]
public class StepDefs
{
    private static readonly IPage _page = /* ... */;  // Shared!
}
```

### Key Implementation Details

1. **ScenarioContext Lifecycle**
   - Created: Start of each scenario
   - Disposed: End of each scenario (triggers AfterScenario hooks)
   - Scope: Isolated per test execution

2. **Reqnroll/SpecFlow Container Registration**
   - `ScenarioContainer`: DI container scoped to scenario
   - `RegisterInstanceAs<T>()`: Registers singleton for this scenario only
   - Constructor injection: Automatically resolved by Reqnroll

3. **Playwright Isolation Levels**
   - Browser Context: Isolated cookies, localStorage, sessions
   - Page: Individual tab/window within a context
   - Each test should use its own context for full isolation

### Parallel Execution Configuration

```xml
<!-- In .csproj or xunit.runner.json -->
{
  "parallelizeTestCollections": true,
  "maxParallelThreads": 5
}
```

```csharp
// Or via assembly attribute (xUnit v3)
[assembly: CollectionBehavior(MaxParallelThreads = 5)]
```

### Debugging Thread-Safety Issues

Common symptoms of thread-safety problems:
- Intermittent failures ("flaky" tests)
- Tests pass individually but fail when run together
- "Element not found" exceptions when other tests are navigating
- Incorrect test data appearing in assertions

Diagnostic approaches:
```csharp
// Add thread ID to logs for debugging
Console.WriteLine($"[Thread {Thread.CurrentThread.ManagedThreadId}] Test: {scenarioContext.ScenarioInfo.Title}");

// Verify isolation
Assert.NotSame(previousPage, currentPage);  // Different instances
```

### Important Caveats

1. **Backend State**: While browser isolation is guaranteed, backend/database state may still be shared. Consider:
   - Using unique test data per scenario
   - Database transactions with rollback
   - Test data builders with unique identifiers

2. **Resource Limits**: Parallel execution increases resource usage:
   - Memory: ~200-500MB per browser instance
   - CPU: Significant for complex pages
   - Network: Concurrent API calls to your application

3. **Headless vs Headed**: In CI/CD:
   ```csharp
   Headless = true  // More efficient for parallel execution
   ```

---

# Troubleshooting Checklist

### Build Failures
- [ ] Check all projects use same xUnit version (v2 OR v3, not mixed)
- [ ] Verify test projects have `<OutputType>Exe</OutputType>` for xUnit v3
- [ ] Confirm Reqnroll package matches xUnit version (Reqnroll.xUnit.v3 for v3)
- [ ] Delete bin/obj folders and rebuild

### Test Discovery Issues
- [ ] Regenerate .feature.cs files (Run Custom Tool)
- [ ] Check feature file Build Action is "Content"
- [ ] Verify step definitions have [Binding] attribute
- [ ] Ensure test runner is installed (xunit.runner.visualstudio)

### Runtime Failures
- [ ] Install Playwright browsers if missing
- [ ] Re-authenticate Azure if using Azure services
- [ ] Check appsettings.json is copied to output directory
- [ ] Verify network connectivity for external sites

---

## Best Practices

### Selector Strategy
1. Use data-cy attributes when available: `[data-cy='element-id']`
2. Fallback to semantic HTML: `button:has-text('Save')`
3. Avoid fragile selectors: indices, generated classes
4. Use page.WaitForSelectorAsync() for dynamic content

### Async/Await Patterns
- All Playwright operations are async
- Step definitions should return Task
- Use ConfigureAwait(false) in library code
- Handle timeouts appropriately (default 30s)

### Test Data Management
- Use configuration files for environment-specific data
- Implement test data builders for complex objects
- Clean up test data in AfterScenario hooks
- Consider parallel execution impacts on shared data

### Error Handling
```csharp
try
{
    await _page.ClickAsync(selector);
}
catch (PlaywrightException ex) when (ex.Message.Contains("timeout"))
{
    // Log meaningful context
    throw new InvalidOperationException($"Element {selector} not found within timeout", ex);
}
```

---

# CI/CD Considerations

### Pipeline Requirements
1. Install Playwright browsers in pipeline:
   ```yaml
   - script: pwsh bin/Debug/net9.0/playwright.ps1 install
   ```
2. Set headless mode for CI:
   ```csharp
   Headless = Environment.GetEnvironmentVariable("CI") == "true"
   ```
3. Configure test result publishing
4. Handle authentication via service principals

### Environment Variables
```bash
# Local development
ASPNETCORE_ENVIRONMENT=Development
TEST_BASE_URL=https://localhost:5001

# CI/CD
CI=true
TEST_HEADLESS=true
AZURE_CLIENT_ID=<service-principal-id>
AZURE_CLIENT_SECRET=<service-principal-secret>
```

---

## Version Compatibility Matrix

| Component | Version | Compatible With |
|-----------|---------|----------------|
| .NET | 9.0 | All components |
| xUnit v3 | 2.0.0 | Reqnroll.xUnit.v3 3.1.2 |
| Playwright | 1.55.0 | Microsoft.Playwright.Xunit.v3 |
| Reqnroll | 3.1.2 | SpecFlow compatibility mode |
| Visual Studio | 2022 | .NET 9, xUnit v3 runner |

---

## Quick Reference Commands

```bash
# Install Playwright browsers
powershell.exe ./bin/Debug/net9.0/playwright.ps1 install

# Clean solution artifacts
Get-ChildItem -Include bin,obj -Recurse | Remove-Item -Recurse -Force

# Find xUnit version conflicts
dotnet list package --include-transitive | findstr xunit

# Regenerate feature files
# Right-click in VS or:
dotnet build --no-incremental

# Run specific test
dotnet test --filter "FullyQualifiedName~TestName"
```

---

## Known Limitations
- Microsoft.Playwright.Xunit local project incompatible with xUnit v2
- Reqnroll.xUnit (v2) incompatible with xunit.v3
- Browser storage APIs (localStorage) not supported in PageTest
- Parallel test execution requires context isolation

---

## Resources
- [Playwright .NET Documentation](https://playwright.dev/dotnet)
- [Reqnroll Documentation](https://docs.reqnroll.net)
- [xUnit v3 Migration Guide](https://xunit.net/docs/getting-started/v3)
- [Azure Identity Troubleshooting](https://aka.ms/azsdk/net/identity/defaultazurecredential/troubleshoot)

---
*Document Version: 1.0*  
*Last Updated: October 2025*  
*Maintained by: QA Team*
