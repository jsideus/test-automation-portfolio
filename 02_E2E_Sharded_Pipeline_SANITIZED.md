[[_TOC_]]
# E2E Sharded Pipeline — Engineer Reference & Critical Review  
*Updated: 2025-06-09*

**Scope** – This doc explains _how_ the MVP YAML works **and** captures a frank assessment of what it already does well and what could be tightened next.

---

## 🚀 Executive summary

| Aspect | Result |
|--------|--------|
| **Wall‑clock gate** | 13 m 30 s ➜ **≈ 9 m** (‑32 %) |
| **Coverage** | Catalog suite of E2E tests |
| **Scalability ceiling** | Raise `shardCount`; no redesign |
| **Determinism** | Namespaced `TEST_RUN_ID` + per‑shard filter |
| **Cost trade‑off** | +13 agent‑minutes (3× agents) ↔ ~4 min dev wait saved/run |

You can safely call this *value‑accretive* and production‑worthy.

---

## 1. Pipeline flow (high‑level)

```mermaid
flowchart LR
    C[Checkout repo] --> L[NServiceBus licence]
    L --> S[Install .NET 9 SDK]
    S --> R[Cache/Restore/Build]
    R --> V[VS Test Platform]
    V --> E[Export shard vars]
    E --> F[Compute test filter]
    F --> T[dotnet test (shard)]
    T --> P[Publish TRX & merge]
```

All steps run **inside one job** that Azure DevOps automatically fans‑out to `N = shardCount` agents.

---

## 2. Parameter cheat‑sheet

| Parameter | Default | Why change it? |
|-----------|---------|----------------|
| `shardCount` | **3** | >15 E2E tests? Bump to 4+. |
| `testFilter` | `"Catalog"` | Temporary subset runs (e.g., smoke only). |
| `adoServiceConnection` | `IaC‑Deployment‑Test` | Non‑default sub or environment. |

---

## 3. Critical evaluation

### ✅ Strengths
1. **Predictable sharding** – simple modulo, no external DB needed.  
2. **Isolation guard‑rails** – `TEST_RUN_ID` prevents Redis/DB collisions.  
3. **NuGet caching** – `Cache@2` saves 2‑5 min on warm agents.  
4. **Fail‑fast yet full logs** – `continueOnError: true` keeps sibling shards alive; final TRX merge flips build red on any failure.  
5. **Single SDK install** – avoids 6/8 SDK redundancy of legacy gate.

### 🔍 Improvement opportunities
| Priority | Issue | Recommendation |
|----------|-------|----------------|
| **High** | `$allTests` list is **static** → easy to forget new tests. | Read the **associated test cases** from the Azure DevOps **Test Suite** at runtime and shard dynamically. |
| **Med** | Workload skew – some tests may run > others. | Record historical durations → weighted balancer. |
| **Med** | Agent cost ~2× | Offer `serialFallback=true` toggle for feature branches. |
| **Low** | `CoreProjectPath` variable never used | Delete or wire into `dotnet test` path. |
| **Low** | SDK 9.0.x is preview today (2025‑06‑09) | Pin exact patch once GA hits or stick to LTS 8.0 if prod uses it. |
| **Low** | VisualStudioTestPlatformInstaller runs every shard | Install once via caching or omit on MS‑hosted agents where vstest is pre‑baked. |

---

## 4. Sharding algorithm deep‑dive

```powershell
# Round‑robin distribution
if (($i % $shardCount) -eq ($shardIndex - 1)) {
    $thisShardTests += $allTests[$i]
}
```

* **1‑based shard index** matches Azure DevOps' `$SYSTEM_JOBPOSITIONINPHASE`.  
* Edge case of empty shard emits dummy filter so the job exits green immediately.

---

## 5. Local repro recipe

```powershell
# Re‑run only shard‑2 tests locally
$env:TestShard  = 2
$env:TestShards = 3
.\scripts\get-shard-filter.ps1
dotnet test src/Company/Company.Features/Company.Features.csproj `
  --filter "$env:TestFilter" --logger trx
```

Open the TRX in VS or VS Code's Test Explorer for debugging.

---

## 6. Scaling guidelines

| Suite size | `shardCount` | Expected gate time* |
|------------|-------------|---------------------|
| 1‑15 tests | 3 | ≤ 10 min |
| 16‑30 | 4 | ≤ 11 min |
| 31‑50 | 5 | ≤ 12 min |

\*Assumes per‑test runtime ≈ current mix & high NuGet cache hit.

---

## 7. Future roadmap

1. **Auto‑balancer** – pick tests for each shard based on prior runtimes.  
2. **Flake quarantine lane** – auto‑retry flaky tests once, tag, alert.  
3. **Teams bot** – post shard summary & gate delta to team channel.  
4. **Cache warm‑prime** – keep a small self‑hosted agent with pre‑restored packages to hit sub‑5 min gates.

---

## 8. FAQ

**Q: Does parallelism risk race conditions in the Sagas?**  
A: No—each shard gets a unique `TestRunId` that namespaces queues & DB docs.

**Q: How do I add a new test?**  
A: Add FQN to `$allTests` (until auto‑discovery arrives) *and* commit code; no YAML edits unless you need more shards.

**Q: Can we run smoke‑only?**  
A: Set `testFilter: "Category=Smoke"` in pipeline parameters.

---

*Maintainer: @team • PRs welcome!*
