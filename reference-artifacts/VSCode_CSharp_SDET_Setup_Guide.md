# VSCode Setup Guide: V6 Career Acceleration Program

**Purpose:** Configure VSCode 1.113.0 on macOS (bash shell) to replicate a Visual Studio IDE experience for C#/.NET SDET work — covering IntelliSense, solution management, integrated testing, debugging, BDD/Gherkin support, Git workflows, and general productivity.

**Environment:** macOS Tahoe 26.3.1, MacBook Pro 2019 (Intel), bash shell, Homebrew installed, Claude Code installed, npm installed.

---

## Step 1: Install .NET SDK

You need the .NET SDK before any C# extension will function properly. The SDK includes the compiler, runtime, and CLI tools.

**Which version?** Install .NET 8.0 (LTS — Long Term Support, supported through November 2026). This is the version most enterprise shops target right now and what your Mosh course uses. You can add .NET 9 or 10 later if needed — multiple SDKs coexist fine.

```bash
# Install via Homebrew (simplest on macOS)
brew install --cask dotnet-sdk

# Verify installation
dotnet --version

# See all installed SDKs and runtimes
dotnet --info
```

**Note on your Intel Mac:** Homebrew will install the x64 version automatically. This is correct for your hardware.

If `dotnet` isn't found after install, add to your `~/.bash_profile`:

```bash
export DOTNET_ROOT="/usr/local/share/dotnet"
export PATH="$PATH:$DOTNET_ROOT"
```

Then `source ~/.bash_profile`.

**Quick sanity check — create and run a test project:**

```bash
mkdir ~/dotnet-test && cd ~/dotnet-test
dotnet new console -n HelloWorld
cd HelloWorld
dotnet run
# Should print "Hello, World!"
```

Clean up after: `rm -rf ~/dotnet-test`

---

## Step 2: Core C#/.NET Extensions

### Tier 1 — Install These First (The "Visual Studio Replacement" Stack)

These extensions, installed together, give you the closest experience to Visual Studio for .NET development:

**1. C# Dev Kit** (`ms-dotnettools.csdevkit`)
- This is the big one. It bundles the C# language extension and adds solution explorer, integrated test explorer, project management, and debugging — the features you're used to from Visual Studio.
- **Licensing:** Free for individual/personal use, academic, and open-source — same terms as Visual Studio Community. You're covered.
- Installs automatically: **C#** (`ms-dotnettools.csharp`) — the base language service (Roslyn-powered IntelliSense, Go to Definition, Find All References, refactoring, syntax highlighting, debugging).
- Installs automatically: **.NET Install Tool** (`ms-dotnettools.vscode-dotnet-runtime`)

```
# Install from VSCode command palette (Cmd+Shift+P):
# > Extensions: Install Extension
# Search: "C# Dev Kit"
# Or from terminal:
code --install-extension ms-dotnettools.csdevkit
```

**2. IntelliCode for C# Dev Kit** (`ms-dotnettools.vscodeintellicode-csharp`)
- AI-assisted code completion — predicts the most likely API members based on context. This is the equivalent of Visual Studio's IntelliCode.

```
code --install-extension ms-dotnettools.vscodeintellicode-csharp
```

**3. NuGet Package Manager** (`jmrog.vscode-nuget-package-manager`)
- Search and install NuGet packages without leaving VSCode. In Visual Studio, this is the "Manage NuGet Packages" GUI. This extension gives you command palette access to add/remove packages.

```
code --install-extension jmrog.vscode-nuget-package-manager
```

---

### Tier 2 — BDD/Gherkin Support (Critical for Your SDET Workflow)

**4. Cucumber (Official)** (`CucumberOpen.cucumber-official`)
- The official Cucumber VSCode extension. Provides Gherkin syntax highlighting, auto-complete for step definitions, go-to-step-definition navigation, formatting, and outline view.
- Reqnroll is Cucumber-compatible (Gherkin syntax), so this extension works with Reqnroll `.feature` files.
- **Note:** The Reqnroll Visual Studio extension exists for Visual Studio (full IDE) but not for VSCode. The official Cucumber extension is the best option in VSCode for `.feature` file support.

```
code --install-extension CucumberOpen.cucumber-official
```

**If the Cucumber extension doesn't autocomplete your Reqnroll steps** (since it looks for Cucumber-style step definitions by default), you may also want:

**5. Reqnroll/SpecFlow Steps Definition Generator** (`RajUppadhyay.specflow-steps-definition-generator`)
- Generates step definition stubs from `.feature` files — replicates the right-click → "Generate Step Definitions" workflow from Visual Studio with SpecFlow/Reqnroll.

```
code --install-extension RajUppadhyay.specflow-steps-definition-generator
```

---

### Tier 3 — Code Quality & Refactoring

**6. Roslynator** (`josefpihrt-vscode.roslynator`)
- 500+ C# analyzers, refactorings, and code fixes. This is the equivalent of having ReSharper-lite built into your editor — catches code smells, suggests improvements, enforces patterns.

```
code --install-extension josefpihrt-vscode.roslynator
```

**7. C# Extensions** (`kreativ-software.csharpextensions`)
- Adds context menu items for creating classes, interfaces, enums, controllers from the file explorer. In Visual Studio you right-click → Add → Class. This replicates that.

```
code --install-extension kreativ-software.csharpextensions
```

---

## Step 3: Git & Version Control Extensions

You already have Git configured. These extensions bring Visual Studio-level Git integration into VSCode:

**8. GitLens** (`eamodio.gitlens`)
- The gold standard for Git in VSCode. Inline blame annotations, file/line history, commit graph visualization, comparison tools. This replaces Visual Studio's Git Changes window and then some.

```
code --install-extension eamodio.gitlens
```

**9. Git Graph** (`mhutchie.git-graph`)
- Visual commit graph — see branch topology, merge history, and navigate commits visually. Complements GitLens for when you want a visual branch map.

```
code --install-extension mhutchie.git-graph
```

---

## Step 4: Testing Extensions

C# Dev Kit includes an integrated test explorer that discovers and runs xUnit, NUnit, and MSTest tests. This covers most of your needs. For additional test workflow enhancements:

**10. .NET Core Test Explorer** (`formulahendry.dotnet-test-explorer`)
- An alternative/supplementary test explorer if you want more granular control. Try C# Dev Kit's built-in test explorer first — if it covers your needs, skip this one.

```
# Only install if C# Dev Kit's test explorer feels insufficient:
code --install-extension formulahendry.dotnet-test-explorer
```

---

## Step 5: General Productivity Extensions

**11. Thunder Client** (`rangav.vscode-thunder-client`)
- Lightweight REST API client built into VSCode. Replaces Postman for quick API testing during development. Collections, environment variables, request history — all without leaving the editor.

```
code --install-extension rangav.vscode-thunder-client
```

**12. REST Client** (`humao.rest-client`)
- Alternative to Thunder Client — lets you write `.http` files with requests and execute them inline. Some people prefer this file-based approach for version-controlling API test requests.

```
# Pick one of Thunder Client or REST Client based on preference:
code --install-extension humao.rest-client
```

**13. YAML** (`redhat.vscode-yaml`)
- YAML language support with schema validation. Essential for Azure DevOps pipeline YAML, GitHub Actions workflows, Docker Compose files, and configuration files.

```
code --install-extension redhat.vscode-yaml
```

**14. Docker** (`ms-azuretools.vscode-docker`)
- Docker file support, container management, image browsing, Docker Compose integration. You'll need this for portfolio projects involving containerized services.

```
code --install-extension ms-azuretools.vscode-docker
```

**15. Markdown All in One** (`yzhang.markdown-all-in-one`)
- Keyboard shortcuts, table of contents generation, auto preview, list editing for markdown. Useful for your reference artifacts and README work.

```
code --install-extension yzhang.markdown-all-in-one
```

**16. EditorConfig for VS Code** (`EditorConfig.EditorConfig`)
- Reads `.editorconfig` files to maintain consistent coding styles (indentation, line endings, charset) across projects. Most .NET projects include these.

```
code --install-extension EditorConfig.EditorConfig
```

---

## Step 6: VSCode Settings for C#/.NET Productivity

Open Settings JSON (`Cmd+Shift+P` → "Preferences: Open User Settings (JSON)") and add/merge these settings:

```json
{
    // Editor fundamentals
    "editor.fontSize": 14,
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true,
    "editor.bracketPairColorization.enabled": true,
    "editor.guides.bracketPairs": true,
    "editor.minimap.enabled": false,
    "editor.renderWhitespace": "boundary",
    "editor.suggestSelection": "first",
    "editor.wordWrap": "off",

    // Terminal — bash (your shell)
    "terminal.integrated.defaultProfile.osx": "bash",
    "terminal.integrated.fontSize": 13,

    // File management
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "files.trimTrailingWhitespace": true,
    "files.insertFinalNewline": true,
    "files.exclude": {
        "**/bin": true,
        "**/obj": true,
        "**/.git": true,
        "**/.DS_Store": true
    },

    // C# / .NET specific
    "dotnet.defaultSolution": "disable",
    "csharp.semanticHighlighting.enabled": true,
    "omnisharp.enableRoslynAnalyzers": true,
    "omnisharp.enableEditorConfigSupport": true,

    // Test explorer
    "dotnet-test-explorer.testProjectPath": "**/*Tests*.csproj",

    // Git
    "git.autofetch": true,
    "git.confirmSync": false,
    "git.enableSmartCommit": true,

    // Explorer
    "explorer.confirmDelete": false,
    "explorer.confirmDragAndDrop": false,
    "explorer.compactFolders": false
}
```

---

## Step 7: Keyboard Shortcuts You'll Want (Visual Studio Muscle Memory)

VSCode uses different defaults than Visual Studio. Here are the key mappings to restore your muscle memory. Open Keyboard Shortcuts JSON (`Cmd+Shift+P` → "Preferences: Open Keyboard Shortcuts (JSON)"):

| Action | Visual Studio | VSCode Default | Notes |
|---|---|---|---|
| Go to Definition | F12 | F12 | Same |
| Find All References | Shift+F12 | Shift+F12 | Same |
| Peek Definition | Alt+F12 | Opt+F12 | Same |
| Rename Symbol | F2 | F2 | Same |
| Quick Fix / Light Bulb | Ctrl+. | Cmd+. | Note: Cmd not Ctrl on Mac |
| Format Document | Ctrl+K, Ctrl+D | Shift+Opt+F | Different |
| Comment Line | Ctrl+K, Ctrl+C | Cmd+/ | Different |
| Build Solution | Ctrl+Shift+B | Cmd+Shift+B | Same concept — runs default build task |
| Integrated Terminal | — | Ctrl+` | Toggle terminal panel |
| Command Palette | — | Cmd+Shift+P | Your new best friend |
| File Search | Ctrl+, | Cmd+P | Quick open any file |
| Symbol Search | — | Cmd+T | Go to symbol in workspace |
| Solution Explorer | — | Cmd+Shift+E | Explorer sidebar |
| Test Explorer | — | Testing icon in sidebar | C# Dev Kit provides this |

**Pro tip:** If you want Visual Studio keybindings wholesale, there's an extension for that:

```
code --install-extension ms-vscode.vs-keybindings
```

This remaps VSCode shortcuts to match Visual Studio defaults. Try it — if it conflicts with macOS shortcuts, you can always uninstall it and learn VSCode's native bindings.

---

## Step 8: One-Liner Bulk Install

Run this in your terminal to install everything at once:

```bash
code --install-extension ms-dotnettools.csdevkit && \
code --install-extension ms-dotnettools.vscodeintellicode-csharp && \
code --install-extension jmrog.vscode-nuget-package-manager && \
code --install-extension CucumberOpen.cucumber-official && \
code --install-extension RajUppadhyay.specflow-steps-definition-generator && \
code --install-extension josefpihrt-vscode.roslynator && \
code --install-extension kreativ-software.csharpextensions && \
code --install-extension eamodio.gitlens && \
code --install-extension mhutchie.git-graph && \
code --install-extension rangav.vscode-thunder-client && \
code --install-extension redhat.vscode-yaml && \
code --install-extension ms-azuretools.vscode-docker && \
code --install-extension yzhang.markdown-all-in-one && \
code --install-extension EditorConfig.EditorConfig
```

After install, restart VSCode (`Cmd+Shift+P` → "Developer: Reload Window").

---

## Step 9: Verify Everything Works

1. **Open a .NET project folder** in VSCode (`code .` from terminal in project directory)
2. **C# Dev Kit should activate** — you'll see "C# Dev Kit" in the bottom status bar and a Solution Explorer in the sidebar
3. **Open a `.cs` file** — IntelliSense should work (type a class name, see completions)
4. **Open a `.feature` file** — Gherkin syntax should be highlighted
5. **Open the Test Explorer** (beaker icon in sidebar) — it should discover xUnit/NUnit tests
6. **Open the integrated terminal** (Ctrl+`) — should open bash
7. **Run `dotnet build`** from terminal — project should compile
8. **Run `dotnet test`** from terminal — tests should execute

---

## What You Now Have vs. Visual Studio

| Visual Studio Feature | VSCode Equivalent |
|---|---|
| Solution Explorer | C# Dev Kit Solution Explorer |
| IntelliSense | C# extension (Roslyn) + IntelliCode |
| Test Explorer | C# Dev Kit Test Explorer |
| NuGet Package Manager GUI | NuGet Package Manager extension (command palette) |
| Built-in debugger | VSCode debugger with C# debug adapter |
| CodeLens (references) | C# extension CodeLens |
| Refactoring menu | Roslynator + C# extension quick actions |
| Git Changes window | GitLens + Source Control sidebar |
| SpecFlow/Reqnroll integration | Cucumber Official + Steps Generator |
| Build (Ctrl+Shift+B) | Tasks → dotnet build |
| NuGet restore | Automatic on project load |

**What you won't have (and don't need for V6):**
- Visual designer surfaces (WinForms/WPF designer) — not relevant for SDET work
- Full IntelliTrace debugging — VSCode debugger covers standard breakpoint/watch/step workflows
- Architecture diagrams — not built into either IDE for your workflow
- Live Unit Testing — C# Dev Kit runs tests on demand, not continuously in background

---

## Extension Summary (16 total)

| # | Extension | Purpose | Priority |
|---|---|---|---|
| 1 | C# Dev Kit | Solution mgmt, testing, debugging | Required |
| 2 | IntelliCode for C# Dev Kit | AI code completion | Required |
| 3 | NuGet Package Manager | Package management | Required |
| 4 | Cucumber (Official) | Gherkin/BDD support | Required |
| 5 | Reqnroll Steps Generator | Step definition generation | Required |
| 6 | Roslynator | Code analysis & refactoring | Required |
| 7 | C# Extensions | Class/interface scaffolding | Recommended |
| 8 | GitLens | Git blame, history, diff | Required |
| 9 | Git Graph | Visual branch graph | Recommended |
| 10 | Thunder Client | REST API testing | Recommended |
| 11 | YAML | Pipeline/config file support | Required |
| 12 | Docker | Container management | Recommended |
| 13 | Markdown All in One | Reference artifact authoring | Recommended |
| 14 | EditorConfig | Consistent code formatting | Recommended |

---

*Reference artifact produced via MOD 4 Friction-Point Learning Protocol — V6 Career Acceleration Program, Day 1.*
*Last updated: March 26, 2026*
