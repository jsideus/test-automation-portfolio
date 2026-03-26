# Test Automation Portfolio

Sanitized technical documents from 11+ years of enterprise SDET work in C#/.NET, demonstrating Senior/Staff-level automation engineering across CI/CD architecture, distributed systems testing, BDD frameworks, and pipeline reliability.

These documents are drawn from real production work — architecture decisions, reliability analyses, debugging investigations, and framework documentation — with proprietary details removed.

---

## Documents

### CI/CD Pipeline Architecture

**[01 — CI/CD YAML Trigger Reference](01_CICD_YAML_Trigger_Reference_SANITIZED.md)**  
Reference guide for Azure DevOps YAML pipeline trigger configuration — branch filters, path filters, scheduled triggers, and pipeline resource triggers. Built as a team reference to reduce misconfiguration in complex multi-repo pipelines.

**[02 — E2E Sharded Pipeline](02_E2E_Sharded_Pipeline_SANITIZED.md)**  
Architecture and implementation of a sharded end-to-end test pipeline that achieved a 32% reduction in gate time (~$50K estimated annual ROI). Covers the sharding strategy, parallel execution model, and the trade-offs involved in splitting a monolithic test suite across pipeline stages.

**[03 — CI/CD Pipeline Template Hierarchy](03_CICD_Pipeline_Template_Hierarchy_SANITIZED.md)**  
Documentation of a layered Azure DevOps pipeline template system — base templates, extension patterns, and how template inheritance was structured to balance reusability with team-specific customization.

### Distributed Systems Observability

**[04 — ServiceControl & ServiceInsight FAQ](04_ServiceControl_ServiceInsight_FAQ_SANITIZED.md)**  
Operational FAQ for NServiceBus ServiceControl and ServiceInsight tooling — monitoring message flows, diagnosing failed messages, and understanding saga lifecycle visibility across distributed services.

**[05 — KQL Queries & ServiceControl Overview](05_KQL_Queries_ServiceControl_Overview_SANITIZED.md)**  
Kusto Query Language (KQL) patterns written in Azure for discovering operation IDs from end-to-end test executions in UAT environments, then using those IDs in Application Insights' Transaction Search to trace the full saga traversal through the distributed system — pinpointing successes, failures, and root cause detail. Transaction Search URLs were embedded directly in Jira bug tickets, giving developers immediate access to the full observability context for faster triage. This documentation was presented to the QA team and distributed as a shared reference to enable the same investigation workflow across the team.

### UI Test Framework

**[06 — UI Test Framework Documentation (Part 1)](06_UI_Test_Framework_Documentation_Part1_SANITIZED.md)**  
Foundation documentation for a Playwright-based UI test framework — project structure, page object patterns, test data management, and environment configuration for cross-browser execution. (Earlier roles used Selenium; this reflects the most recent framework architecture.)

**[07 — UI Test Framework Documentation (Part 2)](07_UI_Test_Framework_Documentation_Part2_SANITIZED.md)**  
Advanced patterns and operational guidance — custom wait strategies, network interception for test isolation, CI/CD integration points, and troubleshooting flaky UI tests in pipeline environments.

### Pipeline Reliability & BDD Standards

**[08 — Pipeline Test Reliability Analysis](08_Pipeline_Test_Reliability_Analysis_SANITIZED.md)**  
Quantitative analysis of test reliability across pipeline runs — failure categorization (infrastructure vs. test logic vs. environment), flakiness metrics, and the data-driven approach used to prioritize stability improvements.

**[09 — BDD Standards and Configuration Layering](09_BDD_Standards_and_Configuration_Layering_SANITIZED.md)**  
Team BDD standards using Reqnroll/SpecFlow — feature file conventions, step definition organization, configuration layering for multi-environment execution, and patterns for scaling BDD across a growing test suite.

---

## Reference Artifacts

The [`reference-artifacts/`](reference-artifacts/) directory contains learning reference documents produced through deliberate friction-point investigation — stopping at points of confusion, researching the underlying mechanism, and producing a reusable artifact.

---

## About

These documents represent a subset of work across healthcare, insurance, fintech, and ticketing platforms. The technical focus is backend distributed systems testing in C#/.NET with an emphasis on making test infrastructure reliable, observable, and maintainable at scale.

→ [GitHub Profile](https://github.com/jsideus) · [LinkedIn](https://www.linkedin.com/in/jeremyideus/) · heisideus@protonmail.com
