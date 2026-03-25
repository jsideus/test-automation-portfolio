[[_TOC_]]
## CI/CD YAML Trigger and Resource Reference

A quick reference for Azure Pipelines YAML trigger and resource configurations.

---

### 1. Default Behavior

By default, when a YAML pipeline is created/imported, UI settings enable:

- **Continuous integration**: build on every push to any branch
- **Pull request validation**: build on every PR against any branch

*No **`trigger`** or **`pr`** blocks in YAML means UI defaults apply.*

---

### 2. Disabling Triggers Entirely

Prevents CI and PR builds for the repository containing the YAML.

```yaml
#-------------------------------------------------
# Disable any native QA‑repo triggers
#-------------------------------------------------
trigger: none   # disables all push triggers
pr: none        # disables all PR validations
```

---

### 3. Scoping to Specific Branches

Only run on pushes or PRs targeting specified branches.

```yaml
#-------------------------------------------------
# Only run on pushes to main or develop branches
#-------------------------------------------------
trigger:
  branches:
    include:
      - main       # CI on main
      - develop    # CI on develop

#-------------------------------------------------
# Only validate PRs targeting main branch
#-------------------------------------------------
pr:
  branches:
    include:
      - main
``` 

---

### 4. Multi‑Repo Triggers (Cross‑Repo)

Let a pipeline in **RepoA** trigger when **RepoB** changes.

```yaml
#-------------------------------------------------
# Wire in Catalog repo so its main branch triggers this pipeline
#-------------------------------------------------
resources:
  repositories:
    - repository: CatalogRepo        # alias for the external repo
      type: git
      name: Organization/Catalog     # "ProjectName/RepoName"
      trigger:                       # CI trigger for CatalogRepo
        branches:
          include:
            - main                  # only on Catalog/main pushes
      pr:                           # PR validation for CatalogRepo
        branches:
          include:
            - main                  # only on PRs targeting Catalog/main
```

> With this, pushes/PRs in the **Catalog** repo drive the pipeline even though the YAML lives elsewhere.

---

### 5. Gating by User or PR Author

Only execute certain stages/jobs when triggered by a specific user or PR author.

```yaml
#-------------------------------------------------
# Only run E2E stage when John (john@company.com) triggers the build
# or when he opens a PR
#-------------------------------------------------
stages:
- stage: CatalogE2E
  condition: |
    and(
      succeeded(),
      or(
        eq(variables['Build.RequestedForEmail'], 'john@company.com'),
        and(
          eq(variables['Build.Reason'], 'PullRequest'),
          eq(variables['System.PullRequest.CreatedBy'], 'john')
        )
      )
    )
  jobs: ...
```

---

### 6. UI vs YAML

- **UI Settings**: Found under **Pipelines → [pipeline] → Edit → Triggers**. Override YAML only when `Override YAML` is enabled.
- **YAML Settings**: Preferred for source‑controlled trigger configuration.

---

### 7. Example: Enhanced Commenting Style

Combine disabling QA triggers with cross‑repo wiring and clear comment blocks.

```yaml
#-------------------------------------------------
# Disable any native QA‑repo triggers
#-------------------------------------------------
trigger: none   # no CI on QA repo pushes
pr: none        # no CI on QA repo PRs

#-------------------------------------------------
# Wire in Catalog repo to trigger only on main
#-------------------------------------------------
resources:
  repositories:
    - repository: CatalogRepo         # alias for the external repo
      type: git
      name: Organization/Catalog      # "ProjectName/RepoName"
      trigger:                       # CI trigger for CatalogRepo
        branches:
          include:
            - main                  # only on Catalog/main pushes
```
