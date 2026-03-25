[[_TOC_]]
# BDD Step Design and Scenario Structure Standards

This document defines enforceable standards for writing Given/When/Then steps in Reqnroll/SpecFlow feature files.  
These standards are supported by authoritative industry sources, including the official Cucumber documentation, and are recommended for all test automation contributors.

---

## Step Design Standards

### Atomicity
Each **Given**, **When**, and **Then** step **must** represent **one** business-relevant action, state, or verification.  
This aligns with [Cucumber's official guidance](https://cucumber.io/docs/bdd/better-gherkin/) that each step express a single intent.

### Clarity & Abstraction Level
Steps **must** describe behavior in **domain language**, not UI mechanics.  
This ensures resilience to UI changes and preserves readability for business stakeholders.

### Consistent Terminology
Domain terms **must** be used consistently across all scenarios.  
Deviations require review and explicit approval from the automation team lead.

### Symbolic Comparisons
Comparative conditions **must** use symbols (`>=`, `<=`, `=`) for precision and brevity, provided they are supported in the step definition regex.

---

## Scenario Structure Standards

### One Behavior per Scenario
Each scenario **must** validate a single business behavior.  
Bundling unrelated behaviors into one scenario is prohibited.

### Use of "And"
"And" steps **must** only continue steps of the same type (Given/When/Then) and maintain atomicity.

### Data Input Formatting
Multiple inputs **must** be expressed using parameters or Data Tables, not comma-separated lists in a single step.

### Composed Steps
If a business action requires multiple sub-actions, it **must** be represented as one high-level step in the feature file, with sub-actions encapsulated in the step definition.

---

## Examples

| Step type | Compliant Example | Non-Compliant Example |
|-----------|-------------------|-----------------------|
| **Given** | `Given user is authenticated as "Analyst"` | `Given user logs in, selects venue, opens inventory, applies filter` |
| **When** | `When user opens the "Unmatched Aliases" screen` | `When user opens "Unmatched Aliases" and selects venue and configuration` |
| **Then** | `Then external listings total count is >= 0` | `Then total >= 0 and unmatched <= total` |
| **And** | `And venue is "Oracle Park"` | `And user logs in and opens screen and selects config` |
| **Abstraction level** | `When user assigns a match to section "110" row "A"` | `When user clicks button "#assign" and selects option index 2` |
| **Data input** | `And the following filters:` *(Data Table)* | `And filters: venue=Oracle Park, config=Concert, type=Listings, ...` |
| **Composed step** | `Given user is on "Unmatched Aliases" for venue/config` | `Given user clicks menu, clicks link, waits, clicks dropdown, selects item...` |

---

## References
- [Cucumber: Writing Better Gherkin](https://cucumber.io/docs/bdd/better-gherkin/)  
- [BrowserStack: Cucumber Best Practices](https://www.browserstack.com/guide/cucumber-best-practices-for-testing)  
- [Seleniums: Cucumber Best Practices](https://www.seleniums.com/automation/notes/cucumber/best-practices/)

---

# Configuration Layering in .NET Test Framework

## How Multi-Layer Configuration Works

The test framework uses a three-layer configuration system where settings are applied in order, with later layers overriding earlier ones when key names match.

**Implementation Location:**
If you want to see the code that handles the multilayer configuration implementation, go to:  
`namespace TestHelper > public class ConfigurationReader > ConfigurationReader()` (look at the code inside the constructor method)

**Configuration Precedence:**
The last config "wins" if it shares the same key name!

1. **Base Layer** = `appsettings.json` (Global layer - settings are first set here)
2. **Local Override** = `appsettings.Development.json` (these settings "replace" the base layer settings if the key name is the same)
3. **User Secrets Override** = `secrets.json` (replaces any configs set in both base layer and local override sharing the same key name)

If each config has a different key name, then all 3 layers "combine" and no setting is overridden 🙂

---

## Adding a New Testing Configuration

### Example Scenario: Adding Browser-Based Test URL Configuration

**Goal 1: Run tests locally the same as CI/CD**
- Open Visual Studio
- Pull in the latest version of the test repository
- Open the solution file
- Build the solution until all projects succeed
- Go to the test project's base configuration (currently the `appsettings.json` file)
- Add the configuration you're needing (e.g., `"UiTestingUrl" : "https://example.com/"`)
- Save your changes

**Goal 2: Run the UI Tests on localhost locally**
- Go to the test project's development configuration (now `appsettings.Development.json`)
- Add the configuration you're needing (e.g., `"UiTestingUrl" : "https://localhost:<LocalPortNumber>"`)
- Save your changes

**Goal 3: Run the UI Test using localhost with a unique port**
- Go inside the TestHelper project's main menu by putting the cursor on top of the Test Helper project
- Right-click on the TestHelper project
- Select the "Manage User Secrets" option
- Add the configuration you're needing to customize locally inside `secrets.json` (e.g., `"UiTestingUrl:" : "https://localhost:<UniqueLocalPortNumber>"`)
- Save your changes

---

## Configuration Layer Benefits

**Advantages of this approach:**
- ✅ Base configuration works in CI/CD without modification
- ✅ Development overrides allow local testing without affecting team
- ✅ User secrets enable personal customization (unique ports, test data)
- ✅ Secrets never committed to source control (automatic .gitignore)
- ✅ Each developer can have unique settings without conflicts

**Common Use Cases:**
- Different database connection strings for local vs CI/CD
- Personal API keys for external services
- Custom port numbers for multiple solution instances
- Environment-specific feature flags
- Local debugging URLs vs deployed environments
