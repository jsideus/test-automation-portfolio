[[_TOC_]]
# CI/CD Pipeline Template Hierarchy Reference

A concise reference guide to the Azure DevOps pipeline layers for a microservices project. Use this to quickly locate and understand each template, how they relate, and where build/test steps live.

---

## 1. Root Pipeline (`.pipelines/main.yml`)
- **Location**: Application repository, `.pipelines/main.yml`.
- **Purpose**: Entry point for Azure DevOps; defines triggers, pool, and extends the next layer.
- **Key snippet**:
  ```yaml
  trigger:
    - main

  pool:
    name: AKSBuildAgentPool

  extends:
    template: ci-cd.yml
  ```

---

## 2. CI/CD Template (`.pipelines/ci-cd.yml`)
- **Location**: Same `.pipelines/` folder as `main.yml`.
- **Purpose**: Defines parameters and variables (appId, projects, dotnetSdkVersion, etc.), and references the shared pipeline templates repo.
- **Key snippet**:
  ```yaml
  - template: azure-pipelines.yaml@templates
    parameters:
      appId: ${{ parameters.appId }}
      projects: ${{ parameters.projects }}
      dotnetSdkVersion: ${{ variables.dotnetSdkVersion }}
  ```

---

## 3. Shared Build & Deploy (BuildTemplates repo, `azure-pipelines.yaml`)
- **Location**: `Organization/BuildTemplates` repo, tag `v3.4.0`.
- **Purpose**: Orchestrates `Build`, `Deploy to Dev/Test/UAT/Prod`, and `Automated Testing` stages via nested templates.
- **Key stages**:
  1. **Build** uses `aks-net8-ci.yaml`
  2. **AutoTestUAT** uses `automated-testing.yaml`

Diagram:
```
┌─────────────────┐
│ .pipelines/     │
│ ├─ main.yml     │  <-- root pipeline
│ └─ ci-cd.yml    │  <-- defines parameters, calls shared
└────────┬────────┘
         │ extends ci-cd.yml
         ▼
┌─────────────────────────────────────────────────┐
│ azure-pipelines.yaml@templates                  │  <-- in BuildTemplates repo
│ ├─ stage: Build  (aks-net8-ci.yaml@templates)   │
│ ├─ stage: Deploy2Dev/Test/UAT/Prod (aks-cd.yaml)│
│ └─ stage: AutoTestUAT (automated-testing.yaml)  │
└─────────────────────────────────────────────────┘
```

---

## 4. Build Template (`aks-net8-ci.yaml`)
- **Likely path**: `Organization/BuildTemplates/aks-net8-ci.yaml`
- **Responsibilities**:
  - Install .NET SDK (from `dotnetSdkVersion` parameter)
  - `dotnet restore`, `dotnet build`, Docker image build
  - Publish build outputs to expected folder (e.g. `bin/Release/net8.0`)

> ⚠️ **Check**: Ensure build task does not pin `--framework net8.0`. Remove or update to `net9.0` where required.

---

## 5. Test Template (`automated-testing.yaml`)
- **Likely path**: `Organization/BuildTemplates/automated-testing.yaml`
- **Responsibilities**:
  - Install .NET SDK
  - `dotnet test` or `VSTest@2` against output DLLs

> ⚠️ **Check**: Confirm `testAssemblyVer2` or `searchFolder` includes `net9.0` output paths and correct `*Tests.dll` patterns.

---

## Quick Trace Strategy
1. **Open** `.pipelines/main.yml` → note the `- extends: ci-cd.yml`
2. **Open** `.pipelines/ci-cd.yml` → find `template: azure-pipelines.yaml`
3. **Open** `Organization/BuildTemplates@v3.4.0/azure-pipelines.yaml`
4. **Open** child templates:
   - `aks-net8-ci.yaml`
   - `automated-testing.yaml`
5. **Search** within those files for `UseDotNet@2`, `DotNetCoreCLI@2`, and `VSTest@2` steps

---

## Maintenance Tips
- **Centralize common logic**: if you migrate to .NET 9.0 across all services, consider renaming `aks-net8-ci.yaml` → `aks-net-ci.yaml` and updating defaults.
- **Parameterize TFMs**: use `dotnetSdkVersion` and `targetFramework` parameters rather than hard-coded `net8.0`.
- **Document** each template with a header comment listing its purpose and who owns it.
- **Diagram** updates: keep a simple ASCII or Visio diagram in your repo to reflect any new layers.

*Keep this reference handy whenever you need to trace or update your CI/CD pipelines!*
