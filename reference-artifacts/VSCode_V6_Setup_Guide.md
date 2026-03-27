# VSCode → Visual Studio IDE Experience: Complete V6 Setup Guide

**Machine:** MacBook Pro 2019 (Intel) · macOS Tahoe 26.3.1 · bash shell  
**Goal:** Replicate a Visual Studio 2026-class C#/.NET SDET development environment in VSCode 1.113.0  
**Date:** March 26, 2026

---

## Phase 1: .NET SDK Installation

You need the .NET SDK before any C# extensions will work properly. C# Dev Kit will prompt you to install it, but doing it via Homebrew gives you cleaner version management.

## Install via Homebrew (recommended if brew is installed)

```bash
# Install .NET 9.0 SDK (current LTS-adjacent, used in enterprise development)
brew install dotnet-sdk

# Verify installation
dotnet --version
dotnet --info
```

**Which version?** .NET 9.0 is the current production release (SDK 9.0.312 as of March 2026). .NET 10.0 just shipped as Standard Term Support. For V6 portfolio work and interview targets, .NET 9.0 is the safe bet — it's what Datadog-class companies are running in production. You can always install 10 later side-by-side.

### Trust the dev HTTPS certificate (needed for any web API work)

```bash
dotnet dev-certs https --trust
```

**Why this matters & what it does:** When you are developing web APIs or any service that uses HTTPS locally ( V6 portfolio work — WireMock.NET, ASP.NET test hosts, etc.), your browser and HTTP clients need to trust the SSL certificate your local dev server presents. By default, .NET generates a self-signed certificate for `localhost`, but macOS doesn't trust self-signed certs — so you'd get security warnings on every request, which becomes a blocker during development.

This command tells macOS to add that self-signed dev certificate to your login keychain as a trusted certificate. Under the hood, it calls `security add-trusted-cert`, which is the macOS system utility that manages your keychain trust store. It will prompt for your macOS password to authorize the keychain modification.

The net result: when you run a local API (`dotnet run` on an ASP.NET project) and hit `https://localhost:5001`, it just works without certificate errors. One-time setup, never needs to be repeated unless you explicitly remove the cert.

**Note:** The first time you run *any* `dotnet` command on a fresh SDK install, .NET prints a one-time welcome banner (telemetry notice, getting-started links). This is not a response to the dev-certs command — it's a first-run splash screen that never appears again.

---

## Phase 2: Core Extensions — One Command Install

This installs everything in one shot.

```bash
# ═══════════════════════════════════════════════════════════
# TIER 1: C#/.NET Development (Visual Studio parity)
# ═══════════════════════════════════════════════════════════

# C# Dev Kit — THE core extension. Includes Solution Explorer,
# integrated test runner, IntelliSense, debugging, project management.
# Free for individual/personal use (same license as VS Community).
# This auto-installs the base C# extension and .NET Install Tool.
code --install-extension ms-dotnettools.csdevkit

# IntelliCode for C# Dev Kit — AI-powered completion suggestions
# ranked by patterns from thousands of open-source repos
code --install-extension ms-dotnettools.vscodeintellicode-csharp

# NuGet Package Manager — search/install NuGet packages from the
# command palette (Ctrl+Shift+P → "NuGet"). Replaces VS's built-in
# NuGet GUI. Essential for adding Reqnroll, xUnit, Playwright, etc.
code --install-extension jmrog.vscode-nuget-package-manager

# Roslynator — 500+ analyzers, refactorings, and code fixes.
# This is the VS equivalent of having ReSharper-lite built in.
# Catches code smells, suggests improvements, enforces patterns.
code --install-extension josefpihrt-vscode.roslynator

# C# Extensions (JosKreativ) — adds "New C# Class", "New Interface"
# to the right-click context menu. Small but eliminates friction
# you'll feel immediately coming from Visual Studio.
code --install-extension kreativ-software.csharpextensions


# ═══════════════════════════════════════════════════════════
# TIER 2: Testing & BDD (Your SDET core workflow)
# ═══════════════════════════════════════════════════════════

# Reqnroll/SpecFlow Step Definition Generator — generates step
# definitions from .feature files with a right-click, just like
# the Visual Studio Reqnroll extension. Supports both Reqnroll
# and SpecFlow projects.
code --install-extension RajUppadhyay.specflow-steps-definition-generator

# Cucumber (Gherkin) Full Support — syntax highlighting, auto-complete,
# and formatting for .feature files. Makes Gherkin editing in VSCode
# feel like the Visual Studio Reqnroll extension.
code --install-extension alexkrechik.cucumberautocomplete

# Playwright Test for VSCode — Microsoft's official extension.
# Run/debug Playwright tests from the sidebar, record tests with
# Codegen, pick locators, trace viewer integration. Essential for
# your UI test framework portfolio work.
code --install-extension ms-playwright.playwright


# ═══════════════════════════════════════════════════════════
# TIER 3: Git & Version Control (Visual Studio Team Explorer parity)
# ═══════════════════════════════════════════════════════════

# GitLens — inline blame, file history, commit graph, branch
# comparison. This is your Team Explorer + Source Control Explorer
# replacement. The free tier covers everything you need for V6.
code --install-extension eamodio.gitlens


# ═══════════════════════════════════════════════════════════
# TIER 4: API Testing & REST (replaces Postman/VS REST tools)
# ═══════════════════════════════════════════════════════════

# Thunder Client — lightweight REST client built into VSCode.
# Collections, environments, variables, scripting. Replaces
# switching to Postman for API testing during development.
code --install-extension rangav.vscode-thunder-client

# REST Client — send HTTP requests from .http files directly
# in the editor. Great for documenting API calls alongside code.
# Complementary to Thunder Client (Thunder = GUI, REST Client = file-based).
code --install-extension humao.rest-client


# ═══════════════════════════════════════════════════════════
# TIER 5: Productivity & Code Quality
# ═══════════════════════════════════════════════════════════

# Error Lens — shows errors and warnings inline at the end of
# the line, not just as squiggly underlines. Dramatically faster
# feedback loop when coding.
code --install-extension usernamehw.errorlens

# EditorConfig — reads .editorconfig files for consistent formatting
# across projects. Standard in any enterprise .NET codebase.
code --install-extension EditorConfig.EditorConfig

# Material Icon Theme — clear, distinguishable file icons.
# Makes the Explorer panel actually usable when you have .cs,
# .feature, .json, .csproj, .sln files all in one project.
code --install-extension PKief.material-icon-theme

# Code Spell Checker — catches typos in code, comments, and strings.
# Surprisingly useful for BDD feature files where business language
# matters.
code --install-extension streetsidesoftware.code-spell-checker

# YAML — syntax support and validation. You'll need this for
# GitHub Actions workflows, Azure DevOps pipeline YAML, and
# Docker Compose files.
code --install-extension redhat.vscode-yaml

# Docker — manage containers, images, Compose files from VSCode.
# Needed when you get to WireMock.NET and service virtualization
# portfolio work.
code --install-extension ms-azuretools.vscode-docker

# Markdown All in One — preview, TOC generation, formatting.
# Useful for your reference artifacts and README work.
code --install-extension yzhang.markdown-all-in-one
```

### Verify installation

```bash
code --list-extensions | grep -E "csdevkit|gitlens|errorlens|thunder|roslynator|playwright"
```

You should see all the key extensions listed.

---

## Phase 3: VSCode Settings for Visual Studio Muscle Memory

Open Settings JSON: `Cmd + Shift + P` → type "Preferences: Open User Settings (JSON)"

Paste this configuration block. Each setting is annotated with the Visual Studio behavior it replicates.

```json
{
    // ─── Editor Behavior (VS-like) ───────────────────────────
    "editor.fontSize": 14,
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.wordWrap": "off",
    "editor.minimap.enabled": false,
    "editor.renderWhitespace": "selection",
    "editor.bracketPairColorization.enabled": true,
    "editor.guides.bracketPairs": "active",
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true,
    "editor.suggestSelection": "first",
    "editor.acceptSuggestionOnCommitCharacter": true,
    "editor.inlineSuggest.enabled": true,

    // ─── Files & Explorer ────────────────────────────────────
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "files.exclude": {
        "**/bin": true,
        "**/obj": true,
        "**/.vs": true,
        "**/node_modules": true
    },
    "explorer.confirmDelete": false,
    "explorer.confirmDragAndDrop": false,

    // ─── Terminal ────────────────────────────────────────────
    // Matches your bash shell preference from Mosh Git course
    "terminal.integrated.defaultProfile.osx": "bash",
    "terminal.integrated.fontSize": 13,

    // ─── C# / .NET Specific ─────────────────────────────────
    // Format on type gives you the VS auto-formatting feel
    "csharp.format.enable": true,
    "dotnet.defaultSolution": "disable",

    // ─── Git ─────────────────────────────────────────────────
    "git.autofetch": true,
    "git.confirmSync": false,
    "git.enableSmartCommit": true,
    "gitlens.currentLine.enabled": true,

    // ─── Theme & Icons ───────────────────────────────────────
    "workbench.iconTheme": "material-icon-theme",

    // ─── Testing ─────────────────────────────────────────────
    // Auto-discover tests when you open a project
    "testing.automaticallyOpenPeekView": "failureVisible",

    // ─── Cucumber/Gherkin (for .feature files) ───────────────
    "cucumberautocomplete.steps": [
        "**/*.cs"
    ],
    "cucumberautocomplete.syncfeatures": "**/*.feature",

    // ─── Error Lens ──────────────────────────────────────────
    "errorLens.enabledDiagnosticLevels": [
        "error",
        "warning",
        "info"
    ]
}
```

---

## Phase 4: Keyboard Shortcuts — Visual Studio Muscle Memory

Open Keyboard Shortcuts JSON: `Cmd + Shift + P` → "Preferences: Open Keyboard Shortcuts (JSON)"

These remap common VS shortcuts to their VSCode equivalents:

```json
[
    // Build Solution (VS: Ctrl+Shift+B → VSCode: same)
    // Already default in VSCode, no remap needed

    // Go to Definition (VS: F12 → VSCode: F12)
    // Already default, no remap needed

    // Find All References (VS: Shift+F12 → VSCode: Shift+F12)
    // Already default, no remap needed

    // Rename Symbol (VS: F2 → VSCode: F2)
    // Already default, no remap needed

    // Run Tests (VS: Ctrl+R, T → remap to Cmd+R, T)
    {
        "key": "cmd+r t",
        "command": "testing.runAll"
    },

    // Debug Tests (VS: Ctrl+R, Ctrl+T → remap)
    {
        "key": "cmd+r cmd+t",
        "command": "testing.debugAll"
    },

    // Toggle Terminal (VS: Ctrl+` → VSCode: Ctrl+`)
    // Already default, no remap needed

    // Quick Fix / Light Bulb (VS: Ctrl+. → VSCode: Cmd+.)
    // Already default on macOS

    // Navigate Back (VS: Ctrl+- → VSCode: Ctrl+-)
    // Already default, no remap needed

    // Solution Explorer equivalent (open Explorer sidebar)
    {
        "key": "cmd+shift+l",
        "command": "workbench.view.explorer"
    }
]
```

---

## Phase 5: First Project Verification

Run this sequence to confirm everything works end-to-end:

```bash
# Create a test project to verify the full toolchain
mkdir -p ~/v6-workspace/verification-test && cd ~/v6-workspace/verification-test

# Create a new xUnit test project (your primary test framework)
dotnet new xunit -n VerificationTests
cd VerificationTests

# Add Reqnroll (your BDD framework)
dotnet add package Reqnroll.xUnit

# Open in VSCode
code .
```

**What you should see when VSCode opens:**
1. C# Dev Kit activates — look for "C#" in the bottom status bar
2. Solution Explorer appears in the sidebar (from C# Dev Kit)
3. The .cs test file shows syntax highlighting and IntelliSense
4. Test Explorer (beaker icon in sidebar) discovers the default test
5. You can click the green play button next to the test to run it

If all five check out, your environment is production-ready for V6.

---

## Phase 6: Sign In to C# Dev Kit

C# Dev Kit requires a Microsoft sign-in for full functionality. For individual/personal use, it's **free** — same terms as Visual Studio Community Edition.

1. `Cmd + Shift + P` → "C# Dev Kit: Sign In"
2. Sign in with any Microsoft account (your personal one is fine)
3. This unlocks Solution Explorer, integrated test runner, and project management features

---

## What You Now Have vs. Visual Studio

| Visual Studio Feature | VSCode Equivalent |
|---|---|
| Solution Explorer | C# Dev Kit Solution Explorer |
| IntelliSense | C# extension + IntelliCode |
| Test Explorer | C# Dev Kit Test Explorer + Playwright extension |
| NuGet Package Manager | NuGet Package Manager extension + `dotnet add package` CLI |
| Refactoring tools | Roslynator (500+ analyzers) |
| Git/Team Explorer | GitLens + built-in Source Control |
| Debugging | Built-in .NET debugger (breakpoints, watch, call stack) |
| SpecFlow/Reqnroll integration | Step Definition Generator + Cucumber auto-complete |
| REST API testing | Thunder Client + REST Client |
| Build (Ctrl+Shift+B) | `dotnet build` task (auto-configured) |
| Error List | Error Lens (inline) + Problems panel |

---

## What's NOT Here (and why)

- **GitHub Copilot** — You have Claude Code. Adding Copilot creates competing AI suggestions and costs $10/mo. Skip it unless you specifically want to evaluate it later.
- **ReSharper** — Roslynator covers the code analysis gap. ReSharper is Windows/VS-only anyway.
- **Live Share** — Useful for pair programming but not needed for solo V6 work. Add later if a portfolio project calls for it.
- **Azure DevOps extension** — Your V6 portfolio targets GitHub Actions. The YAML extension covers pipeline file editing. Add the Azure extension only if you get a role that uses ADO.

---

## Quick Reference: Daily Commands

```bash
# Create new project types
dotnet new console -n MyApp           # Console app
dotnet new xunit -n MyTests           # xUnit test project
dotnet new classlib -n MyLib          # Class library
dotnet new sln -n MySolution          # Solution file

# Add project to solution
dotnet sln add MyTests/MyTests.csproj

# Add NuGet packages (your common ones)
dotnet add package Reqnroll.xUnit     # BDD framework
dotnet add package FluentAssertions   # Better assertions
dotnet add package Moq                # Mocking
dotnet add package WireMock.Net       # Service virtualization
dotnet add package Microsoft.Playwright  # UI testing
dotnet add package Refit              # REST client generation

# Build and test
dotnet build
dotnet test
dotnet test --filter "Category=Smoke"

# Run with watch (auto-rebuild on save)
dotnet watch run
dotnet watch test
```

---

*This document is a V6 reference artifact. Produced March 26, 2026 via friction-point investigation (MOD 4).*
