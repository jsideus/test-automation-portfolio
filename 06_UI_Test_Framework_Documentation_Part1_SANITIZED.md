[[_TOC_]]

## Overview
This document provides comprehensive technical documentation for a UI test automation framework using Playwright, Reqnroll (SpecFlow successor), and xUnit v3 with .NET 9.

## Architecture

### Solution Structure
```
Application/
├── Application.UI/                 # Test project folder (contains Application.UI.Tests.csproj)
│   ├── Features/                   # Gherkin feature files
│   ├── StepDefinitions/            # Step definition implementations
│   ├── PageObjectModel/            # Page object classes
│   ├── Hooks/                      # Test hooks (BeforeScenario, etc.)
│   └── bin/Debug/net9.0/           # Playwright scripts location
├── Ui.Core/                        # Shared UI testing utilities
├── Ui.InternalSite/                # Internal site test configurations
├── Ui.ExternalSite/                # External site test configurations
└── Microsoft.Playwright.Xunit/     # (Removed - replaced with NuGet package Microsoft.Playwright.Xunit.v3)
```

---

# Service Principal Authentication in UI Test Framework

## Technical Explanation

A service principal is an Azure Active Directory application identity that authenticates programmatically using client credentials (ID and secret) instead of user credentials, allowing our Playwright tests to obtain OAuth tokens directly from Azure AD without any interactive login or 2FA challenges, which are then stored in the browser's localStorage to maintain authenticated sessions throughout the test execution.

## Everyday Life Analogy

Think of a service principal like a **VIP backstage pass** at a concert venue:
- Regular attendees (users) must go through the main entrance, show their ticket, and pass through security screening (username/password + 2FA)
- The backstage pass (service principal) has its own special entrance with a unique code that security already knows and trusts
- Once inside, the pass gets a wristband (OAuth token) that lets them access everything without being questioned
- Our test framework shows this wristband to the Angular app, which says "Oh, you're already verified, come right in!"

## How It Works in Our Tests

```
1. Service Principal authenticates with Azure AD using client ID/secret
   ↓
2. Receives OAuth tokens (no 2FA required)
   ↓
3. Saves tokens to browser state file (internal-user.json)
   ↓
4. New browser loads this state file with tokens in localStorage
   ↓
5. Angular app sees valid tokens and skips login entirely
```

## Why It Bypasses 2FA

- **Service principals aren't humans** - they can't receive SMS codes or use authenticator apps
- **Pre-authorized by IT** - your organization explicitly trusts this identity for test automation
- **Credentials stored securely** - client secret is kept in environment variables, not in code
- **Limited permissions** - only granted access to what tests need, reducing security risk

## Implementation in Our Framework

### Local Development
```bash
# Environment variables needed
AZURE_CLIENT_ID=<service-principal-id>
AZURE_CLIENT_SECRET=<service-principal-secret>
AZURE_TENANT_ID=<tenant-id>
```

### CI/CD Pipeline
```yaml
variables:
  - name: AZURE_CLIENT_ID
    value: $(ServicePrincipalClientId)
  - name: AZURE_CLIENT_SECRET
    value: $(ServicePrincipalSecret)
  - name: AZURE_TENANT_ID
    value: $(TenantId)
```

### Code Flow
1. `InternalSdkFixture.InitializeAsync()` - Obtains tokens using service principal
2. `StoreOAuthBrowserState()` - Saves tokens to localStorage and creates auth state file
3. `ApplicationPageTest.ContextOptions()` - Loads saved auth state into new browser context
4. Angular app reads tokens from localStorage - bypasses login completely

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Browser shows login page | Verify auth state file exists and path matches |
| 2FA prompt appears | Confirm using service principal, not user credentials |
| Tokens expired | Ensure token refresh logic is implemented |
| Works locally but not in CI/CD | Check environment variables are set correctly |

## Security Best Practices

1. **Never commit credentials** - Use environment variables or secure vaults
2. **Rotate secrets regularly** - Update service principal secrets quarterly
3. **Limit scope** - Grant minimum required permissions
4. **Separate environments** - Use different service principals for dev/test/prod
5. **Monitor usage** - Review audit logs for unusual activity

---

# Package Dependencies

### Critical Package Versions
All projects must use consistent xUnit versions to avoid type conflicts.

#### Application.UI.Tests (.csproj)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <OutputType>Exe</OutputType>  <!-- Required for xUnit v3 -->
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.13.0" />
    <PackageReference Include="Reqnroll.xUnit.v3" Version="3.1.2" />  <!-- NOT Reqnroll.xUnit -->
    <PackageReference Include="Reqnroll.SpecFlowCompatibility" Version="3.1.2" />
    <PackageReference Include="xunit.v3" Version="2.0.0" />  <!-- NOT xunit -->
    <PackageReference Include="xunit.runner.visualstudio" Version="3.0.2" />
  </ItemGroup>
</Project>
```

#### Ui.Core (.csproj)
```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Playwright.Xunit.v3" Version="1.55.0" />
  <PackageReference Include="xunit.v3.extensibility.core" Version="2.0.0" />  <!-- For libraries, not full xunit.v3 -->
</ItemGroup>
```

## Dependency Resolution Guide

### Common Issues and Solutions

#### Issue 1: Type Conflicts (TraitAttribute, IAsyncLifetime)
**Error:** `The type 'X' exists in both 'xunit.core' and 'xunit.v3.core'`

**Cause:** Mixing xUnit v2 and v3 packages in the same solution.

**Solution:**
1. Use `Reqnroll.xUnit.v3` (NOT `Reqnroll.xUnit`)
2. Use `xunit.v3` (NOT `xunit`)
3. Ensure all projects use consistent versions
4. Libraries use `xunit.v3.extensibility.core`, test projects use `xunit.v3`

#### Issue 2: Playwright Browser Not Found
**Error:** `Executable doesn't exist at ...\ms-playwright\chromium_headless_shell-1187\`

**Solution:**
```bash
# From test project directory
powershell.exe ./bin/Debug/net9.0/playwright.ps1 install
```

#### Issue 3: Azure Authentication Failure
**Error:** `DefaultAzureCredential failed to retrieve a token`

**Solution:**
Visual Studio → Tools → Options → Azure Service Authentication → Re-authenticate

#### Issue 4: xUnit v3 OutputType Requirement
**Error:** `xUnit.net v3 test projects must be executable`

**Solution:**
Add `<OutputType>Exe</OutputType>` to test project .csproj

---

# Setup Instructions

### Initial Setup
1. **Clone repository**
2. **Restore NuGet packages**
3. **Install Playwright browsers:**
   ```bash
   cd Application.UI
   dotnet build
   powershell.exe ./bin/Debug/net9.0/playwright.ps1 install
   ```
4. **Configure Azure authentication** (if using Azure Key Vault/Config)
5. **Generate feature files:** Right-click .feature files → Run Custom Tool

### After Package Updates
For routine package updates:
1. Clean solution
2. Rebuild solution

**Only if experiencing dependency conflicts:**
1. Delete all `.feature.cs` files
2. Clean solution
3. Delete `bin` and `obj` folders:
   ```powershell
   Get-ChildItem -Include bin,obj -Recurse -Force | Remove-Item -Recurse -Force
   ```
4. Rebuild solution
5. Regenerate feature files (Right-click → Run Custom Tool)

---

## Test Development Patterns

### Page Object Model Structure
```csharp
public class VenueConfigSummaryPage
{
    private readonly IPage _page;
    private const string SummaryTable = "venue-config-summary-table table";
    
    public VenueConfigSummaryPage(IPage page)
    {
        _page = page;
    }
    
    public async Task<bool> IsDisplayedAsync()
    {
        return await _page.IsVisibleAsync(SummaryTable);
    }
    
    public async Task SelectVenueAsync(string venueName)
    {
        var rowSelector = $"tr:has-text('{venueName}')";
        await _page.ClickAsync(rowSelector);
    }
}
```

### Step Definition Pattern
```csharp
[Binding]
public class NavigationSteps
{
    private readonly IPage _page;
    
    public NavigationSteps(IPage page)
    {
        _page = page;
    }
    
    [Given(@"the user navigates to ""(.*)"" screen")]
    public async Task GivenUserNavigatesToScreen(string screenName)
    {
        var page = new HomePage(_page);
        await page.NavigateToScreenAsync(screenName);
    }
}
```

### Test Hooks Configuration
```csharp
[Binding]
public class TestHooks
{
    private static IPlaywright _playwright;
    private static IBrowser _browser;
    
    [BeforeScenario]
    public async Task BeforeScenario(ScenarioContext scenarioContext)
    {
        _playwright = await Playwright.CreateAsync();
        _browser = await _playwright.Chromium.LaunchAsync(new() 
        { 
            Headless = false 
        });
        
        var context = await _browser.NewContextAsync();
        var page = await context.NewPageAsync();
        
        scenarioContext.ScenarioContainer.RegisterInstance(page);
    }
    
    [AfterScenario]
    public async Task AfterScenario()
    {
        await _browser?.CloseAsync();
        _playwright?.Dispose();
    }
}
```
