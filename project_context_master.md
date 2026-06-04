# Project-Arbitrage Master Context
_Last updated: 2026-05-07_

## 1. Document purpose

This document is the **master technical context** for Project-Arbitrage. It is written so that:

- the current developer can continue work safely,
- a future developer can join the project without losing context,
- each implementation step can be executed in the correct order,
- safety boundaries are clear before moving from scaffold to live trading,
- architectural intent is preserved as the codebase grows.

This is not only a progress note. It is a **project blueprint**, **technical operating guide**, **implementation roadmap**, **safety playbook**, and **continuation manual**.

---

## 2. Project identity

**Project name:** Project-Arbitrage  
**Primary runtime:** Python backend on FastAPI / Uvicorn  
**Current local environment:** Windows, `D:\Project-Arbitrage`  
**Current backend URL:** `http://127.0.0.1:8100`  
**Current confirmed project stage:** **Phase 3 — Step 44**  
**Current status:** backend scaffold is operational and increasingly production-shaped, but it is **not yet a live trading bot**.

---

## 3. Core project objective

Project-Arbitrage is being developed as a **safety-first arbitrage engine** with staged activation.

The goal is not merely to find price differences between DEXs. The real goal is to build a system that can eventually:

1. collect reliable on-chain market state,
2. verify freshness and safety of data,
3. discover pools and quote paths,
4. estimate whether an opportunity is real and executable,
5. simulate before execution,
6. protect funds with policy gates,
7. submit safely with private routing where appropriate,
8. persist audit history,
9. expose a dashboard and operational controls,
10. graduate from safe scaffold to paper trading, then small live trades, then controlled production automation.

This project deliberately prioritizes **correctness and capital protection** over speed of implementation.

---

## 4. Design philosophy

The entire system follows these principles:

### 4.1 Safety before profit
No route, signal, quote, or opportunity should be trusted just because it exists. Every stage must pass:
- freshness checks,
- safety metadata checks,
- policy checks,
- degraded-mode checks,
- circuit breaker protection,
- execution readiness gates,
- wallet/vault checks,
- eventually simulation and submission checks.

### 4.2 Staged activation
The backend is intentionally developed in steps. Each step:
- adds one capability,
- exposes one or more status routes,
- confirms flags via `/status`,
- validates that the new layer is active before the next layer is added.

### 4.3 Production-style scaffolding first
Many current services are structural. That is intentional. The codebase first establishes:
- service graph,
- route graph,
- policy locations,
- readiness flags,
- dependency flow,
- observability,
- auditability.

Then real implementations are filled in.

### 4.4 Kill-switch mindset
At every future stage, the system must be able to say:
- do not use this chain,
- do not trust this provider,
- do not build this opportunity,
- do not simulate,
- do not sign,
- do not broadcast.

### 4.5 Developer continuity
The project should be understandable even if the original developer stops. This document is part of that continuity system.

---

## 5. Current state summary

## 5.1 Current confirmed project stage

The project is currently at:

**Phase 3 — Step 44**  
**Backend scaffold with real pool discovery and liquidity validation active**

## 5.2 Current backend base URL

`http://127.0.0.1:8100`

## 5.3 Current confirmed routes

### Core
- `/`
- `/health`
- `/status`
- `/async-runtime-status`

### Market and configuration
- `/market-data`
- `/evm-analysis`
- `/config-readiness`

### Quote / reserve / route structure
- `/evm-quote-pipeline`
- `/evm-live-quotes`
- `/evm-reserves`
- `/evm-pools`
- `/evm-routes`
- `/evm-real-quotes`
- `/evm-real-reserves`
- `/evm-real-v3-state`
- `/evm-market-state`
- `/evm-market-state-optimized`

### Probe / RPC / websocket / runtime safety
- `/evm-probe-state`
- `/evm-ws-health`
- `/evm-circuit-health`

### Simulation / execution / submission
- `/evm-simulation-status`
- `/evm-execution-status`
- `/evm-execution-preview`
- `/evm-mev-health`
- `/evm-submission-status`

### Observability / metrics / audit
- `/observability-status`
- `/metrics-snapshot`
- `/audit/real-quotes`
- `/audit/summary`

### Ranking / readiness
- `/ranking-status`
- `/ranked-opportunities`
- `/execution-readiness-status`
- `/execution-ready-opportunities`

### Vault / wallet / balances
- `/vault-status`
- `/wallet-status`
- `/wallet-balances`
- `/token-allowances`

### DEX quote and pool readiness
- `/evm-dex-quote-status`
- `/evm-pool-liquidity-status`

## 5.4 Current confirmed capability summary

The backend can currently:
- start successfully,
- load config files,
- classify chains,
- expose policy/service readiness,
- maintain probe/circuit safety structure,
- expose wallet/vault readiness,
- expose balance/allowance structure,
- expose DEX quote readiness structure,
- expose pool liquidity readiness structure.

The backend **cannot yet**:
- fetch real profitable opportunities end-to-end,
- simulate real swap bundles accurately,
- sign and broadcast live trades,
- manage real funds safely in production,
- present a full operator UI.

---

## 6. Current technical truth

This section is intentionally direct.

### 6.1 What is real right now
These things are already materially present:
- route wiring,
- service graph,
- config loading,
- policy layering,
- readiness evaluation,
- audit snapshots,
- metrics and service flags,
- progressive safety structure.

### 6.2 What is still scaffold-level
These parts still need real chain-aware implementation:
- real DEX quotes,
- real pool discovery from factories,
- opportunity construction,
- real token value normalization with market data,
- transaction simulation,
- transaction signing,
- real execution and receipt tracking,
- production vault integration,
- production dashboard and controls.

### 6.3 Meaning of “working”
At this stage, “working” means:
- routes exist,
- services are instantiated,
- flags are true,
- structure behaves correctly,
- unsafe paths are rejected by default,
- current empty outputs are often expected.

At this stage, “working” does **not** mean:
- real trading is safe to enable.

---

## 7. Architecture overview

## 7.1 Main architectural layers

### A. Configuration layer
Purpose:
- load static YAML/JSON configuration,
- define chain metadata,
- define policies,
- define tracked tokens, dexes, addresses, thresholds.

Examples:
- chain configs,
- DEX configs,
- token whitelist,
- wallet balance policy,
- DEX quote integration policy,
- pool liquidity policy,
- execution readiness policy,
- profitability policy,
- simulation policy,
- MEV policy.

### B. Service layer
Purpose:
- implement business logic,
- transform config into decisions,
- maintain safe operational flow,
- expose reusable logic to routes.

This is the main layer of the project.

### C. Route layer
Purpose:
- expose service outputs through FastAPI,
- make validation easy via browser,
- give human-readable and machine-readable status.

### D. App state layer
Purpose:
- store service instances centrally in `AppStateContainer`,
- avoid rebuilding dependency graphs per request,
- make route handlers thin.

### E. Bootstrap layer
Purpose:
- construct services in the right order,
- wire dependencies,
- store them in app state,
- set service flags,
- finalize startup.

This mostly lives in `main.py`.

### F. App factory layer
Purpose:
- create the FastAPI app,
- register routes,
- manage lifespan startup/shutdown behavior.

This mostly lives in `api/app_factory.py`.

---

## 8. Important files and responsibilities

## 8.1 `main.py`
Responsibility:
- bootstraps the full backend,
- builds the service graph,
- sets `AppStateContainer`,
- sets metrics flags,
- defines project startup completion for the current phase.

It is the **dependency construction center**.

## 8.2 `api/app_factory.py`
Responsibility:
- creates FastAPI app,
- lists visible routes in `/`,
- registers all routers,
- manages lifespan and background startup/shutdown.

It is the **route registration and app lifecycle center**.

## 8.3 `services/app_state.py`
Responsibility:
- holds all active singleton-like service references.

It is the **runtime shared state registry**.

## 8.4 `api/routes/status.py`
Responsibility:
- gives one consolidated readiness/status view of all major layers.

It is the **high-level service activation truth source**.

## 8.5 `config/*.yaml`
Responsibility:
- determine project behavior without hard-coding every decision.

They are the **policy and metadata source**.

---

## 9. Code flow end-to-end

This section explains how the backend is intended to think.

## 9.1 Startup flow
1. Uvicorn imports `arbitrage_system.backend.main:app`
2. `main.py` calls `create_app()`
3. settings/config roots are created
4. chain registry is loaded
5. providers, policy services, and feature services are instantiated
6. service instances are written to `AppStateContainer`
7. metrics flags are set
8. background services are registered
9. FastAPI app starts
10. lifespan startup runs background startup handlers

## 9.2 Status inspection flow
When `/status` is called:
1. route reads current services from `AppStateContainer`
2. route returns boolean flags and summary objects
3. operator confirms which layer is actually active

## 9.3 Readiness flow
When readiness routes are called:
1. config policy is loaded
2. chain list is iterated
3. chain-specific readiness evaluation is performed
4. per-chain output is returned
5. current blockers become visible

## 9.4 Quote/pool pipeline target flow
The intended future flow is:

1. load chain config  
2. verify probe freshness  
3. verify RPC health / circuit state  
4. resolve DEX contract config  
5. discover pools or seeded pools  
6. validate liquidity  
7. build quote requests  
8. fetch quotes  
9. normalize value  
10. compare cross-DEX results  
11. build candidate opportunity  
12. apply profitability gate  
13. apply execution readiness gate  
14. simulate  
15. apply execution safety gate  
16. submit privately/publicly depending policy  
17. persist audit + metrics + receipts

This is the target operational pipeline.

---

## 10. Safety architecture

The project’s most important quality right now is its safety layering.

## 10.1 Probe freshness
Purpose:
- reject stale chain state,
- stop unsafe quotes from being used.

## 10.2 Quote safety metadata
Purpose:
- attach confidence/freshness/verification context to quotes.

## 10.3 Direct verify fallback
Purpose:
- cross-check suspicious results directly through provider verification.

## 10.4 Quorum verification
Purpose:
- require agreement from multiple providers when configured.

## 10.5 Degraded mode
Purpose:
- stop heavy or unsafe activity on degraded chains.

## 10.6 RPC circuit breaker
Purpose:
- stop repeatedly calling failing providers,
- protect route latency and provider abuse.

## 10.7 Fail-fast real quote logic
Purpose:
- avoid one slow chain blocking the whole route.

## 10.8 Trade value policy
Purpose:
- block obviously invalid or too-small value situations,
- normalize token value logic.

## 10.9 Per-chain execution policy
Purpose:
- enforce chain-specific thresholds such as:
  - stale age,
  - required confidence,
  - quorum rules,
  - gas/slippage guardrails.

## 10.10 Profitability policy
Purpose:
- reject opportunities that are not good enough after costs.

## 10.11 Simulation gate
Purpose:
- require pre-execution simulation before execution.

## 10.12 Execution readiness
Purpose:
- require wallet + vault + policy readiness before considering execution.

## 10.13 Submission safety
Purpose:
- enforce whether public/private submission is allowed.

## 10.14 Observability and audit
Purpose:
- every important decision should be inspectable later.

---

## 11. Current implemented steps

This section records the project progression through the latest confirmed stage.

## Step foundation
- FastAPI scaffold
- local Uvicorn backend
- `/health`
- `/status`
- `/`

## Config and chain capability layer
- chain config loading
- DEX config loading
- token whitelist loading
- execution-ready vs watch-only classification

## EVM market and route structure
- quote pipeline
- reserves
- pools
- routes
- real quote/reserve/v3 state structure
- market state and optimized market state

## Probe and runtime state
- probe state
- async runtime state
- websocket health scaffold
- background service manager
- probe supervisor
- websocket head tracker

## Safety verification layers
- probe freshness
- quote safety metadata
- direct verify fallback
- quorum verification

## Step 29 — degraded mode
- degraded mode active
- avoids unsafe work on stale/non-ready chains

## Step 30A — RPC circuit breaker
- provider state tracking
- failure/success counts
- last errors
- protected RPC call wrapper

## Step 30B / later fail-fast real quote hardening
- route timeout style behavior
- chain-level early rejection logic
- route-level responsiveness improvements

## Step 31 — trade value verification policy
- trade value policy services
- preliminary value checks

## Step 32 — per-chain execution policy
- chain-specific policy structure
- min confidence / stale / quorum / slippage direction

## Step 33 — profitability hardening
- profitability policy service
- net-profit gating scaffold

## Step 34 — pre-execution simulation gate
- simulation policy
- simulation service
- pre-execution simulation gate

## Step 35 — execution controller
- execution policy
- nonce manager scaffold
- gas manager scaffold
- transaction builder scaffold
- execution controller scaffold
- execution remains disabled by policy

## Step 36 — MEV protected submission
- MEV health
- submission status
- private submission structure
- relay placeholders

## Step 37 — observability
- observability status
- metrics snapshot
- service flags
- route counts / rejection reasons

## Step 38A — persistence and audit
- file persistence
- audit summary
- real quote audit snapshots

## Step 39 — ranking
- ranking status
- ranked opportunities
- candidate scoring
- shortlist structure

## Step 40 — execution readiness
- execution readiness policy
- wallet readiness
- vault readiness
- execution-ready opportunities route

## Step 41 — vault and secrets integration
- vault policy config
- env secret provider
- local file secret provider
- secret resolver
- wallet secret resolver
- `/vault-status`
- `/wallet-status`

## Step 42 — wallet and balance layer
- wallet balance policy
- native balance reader scaffold
- ERC20 balance reader scaffold
- allowance reader scaffold
- `/wallet-balances`
- `/token-allowances`

## Step 43 — real DEX quote integration scaffold
- DEX quote integration policy
- contract resolution service
- adapter service
- DEX quote integration summary route
- `/evm-dex-quote-status`

## Step 44 — real pool discovery and liquidity validation scaffold
- pool liquidity policy
- pool contract resolution
- pool liquidity validation
- `/evm-pool-liquidity-status`

---

## 12. Current project limitations

The following are the most important current limitations.

### 12.1 Wallet not configured
Current outputs show:
- `configured_wallet_count: 0`
- all chains `wallet_configured: false`

### 12.2 Vault not configured
Current outputs show:
- `vault_enabled: false`
- `vault_configured: false`

### 12.3 DEX quote integration not populated
Current outputs show:
- `ready_chain_count: 0`
- `total_ready_dex_count: 0`

This is because `dex_quote_integration_policy.yaml` still contains empty `dexes: []`.

### 12.4 Pool discovery/liquidity not populated
Current outputs show:
- `ready_chain_count: 0`
- `total_ready_dex_count: 0`
- `total_ready_pool_count: 0`

This is because `pool_liquidity_policy.yaml` still contains empty `dexes: []`.

### 12.5 Real opportunities are not yet being built
No route is yet combining:
- real quotes,
- real pools,
- real value normalization,
- real liquidity,
- real profitability
into actual executable arbitrage candidates.

### 12.6 Execution is still safely disabled
This is correct and intentional.

---

## 13. Known current technical findings

## 13.1 Some providers are good, some are not
Example:
- Binance BSC public endpoints and some publicnode endpoints have worked
- Ankr BSC endpoint returned unauthorized without API key

Meaning:
- provider quality and auth requirements matter,
- provider registry should later become more disciplined.

## 13.2 Empty outputs are often correct right now
Examples:
- empty ranked opportunities,
- empty execution-ready opportunities,
- zero wallet count,
- zero ready dex count,
- zero ready pool count.

These are not necessarily bugs. In many cases they reflect correct safety behavior.

## 13.3 The system is conservative by design
It prefers rejecting work rather than pretending unsafe work is valid.

---

## 14. How to code this project safely

This section is a developer operating guide.

## 14.1 Golden rule
Do **not** jump straight to “make it execute trades.”

Every new capability should follow this order:
1. config
2. service
3. app state
4. route
5. status flag
6. compile
7. clear cache
8. run backend
9. validate route outputs
10. only then move to next step

## 14.2 Standard implementation pattern
For each new step:

### 1. Create config file
If the feature is policy-driven, create `config/*.yaml`.

### 2. Create service class(es)
Write the business logic in `backend/services/`.

### 3. Create route(s)
Expose the new layer through `backend/api/routes/`.

### 4. Update `app_state.py`
Add new service placeholders.

### 5. Update `main.py`
Instantiate the new services.
Inject dependencies.
Write them to `AppStateContainer`.
Set metrics flags.

### 6. Update `app_factory.py`
Import the route.
Register the route.
Add it to `/` route list.

### 7. Update `status.py`
Expose new readiness flags.

### 8. Compile
Use `py_compile` on touched files.

### 9. Clear `__pycache__`
Always clear after heavy edits.

### 10. Run Uvicorn
Verify startup and route outputs.

## 14.3 Never skip validation
Before moving to the next setup, confirm:
- phase in `/` is correct,
- `/status` contains the new flags,
- the new route exists,
- the route output shape matches expectation,
- no literal `` `r`n `` corruption exists,
- Uvicorn startup is clean.

## 14.4 Do not patch blindly
Past errors happened because string replacement injected literal `` `r`n `` text into Python files. Safer methods:
- overwrite the full file,
- or patch using Python file editing logic rather than brittle raw PowerShell replacements.

## 14.5 Keep routes readable
Routes should remain thin. Put logic into services.

## 14.6 Keep status explicit
Every major feature should have:
- at least one route,
- at least one `status.py` flag,
- ideally one dedicated feature-status route.

## 14.7 Never enable live execution early
Do not enable:
- execution policy,
- funded wallet,
- private submission,
- real approvals,
- real swap signing
until simulation, safety gates, and dry-run testing are solid.

---

## 15. Where to stop before the next setup

This is critical.

For every step, stop after the following checklist:

### Stop checklist
- code files created/updated
- `py_compile` passes
- `__pycache__` cleared
- Uvicorn starts without syntax/import error
- `/` phase changed to the new step
- `/status` shows the new service flags
- the new route returns structured JSON
- output matches expected scaffold behavior
- only then continue to next step

If any one of those fails, **do not move to the next setup**.

---

## 16. Recommended next implementation order

The correct future order from the current Step 44 state is:

## Step 45 — Real Opportunity Builder and Candidate Generation
Purpose:
- combine DEX quote readiness,
- pool liquidity readiness,
- wallet/execution readiness,
- profitability scaffolding,
- produce actual structured candidate opportunities.

Will need:
- opportunity builder service,
- candidate object schema,
- route for real candidate generation,
- route/status output for candidate counts and blockers.

## Step 46 — Real Profit Calculation
Purpose:
- convert quote differences into actual economic decisions.

Will need:
- gas-aware cost model,
- slippage and fee model,
- token/USD pricing source,
- minimum net profit thresholds.

## Step 47 — Real Transaction Simulation
Purpose:
- simulate the actual planned route before execution.

Will need:
- `eth_call`/fork/Tenderly style simulation,
- revert reason decoding,
- allowance/balance validation,
- estimated gas and success/failure report.

## Step 48 — Transaction Builder
Purpose:
- build actual calldata for approve/swap/multicall paths.

Will need:
- router ABI definitions,
- calldata encoders,
- amountOutMin logic,
- deadlines,
- chain-specific router behavior.

## Step 49 — Execution Safety Gate
Purpose:
- final block before signing.

Will need:
- manual approval mode,
- dry-run/live mode,
- max trade size,
- max daily loss,
- kill switch,
- chain allowlist.

## Step 50 — Real Submission Layer
Purpose:
- submit safely.

Will need:
- public RPC send,
- private relay send,
- flashbots/builder integration,
- receipt tracking,
- retry/failure handling.

## Step 51+ — Frontend / DB / Deployment / Security
These steps are still required for a production system:
- operator dashboard,
- search/filter UI,
- execution controls,
- PostgreSQL persistence,
- deployment,
- auth,
- monitoring,
- backup,
- paper trading,
- small live mode,
- production mode.

---

## 17. Full roadmap from start to finish

This is the most complete roadmap view.

### Stage A — Backend foundation
- FastAPI scaffold
- health/status/root
- config detection
- chain registry
- route registration pattern

### Stage B — Market-state scaffold
- quote/reserve/pool/route routes
- market-state aggregation
- optimized market-state structure

### Stage C — Safety base
- probes
- staleness
- quote safety
- verification
- quorum
- degraded mode
- circuit breakers

### Stage D — Policy and execution guards
- trade value policy
- chain execution policy
- profitability policy
- simulation gate
- execution controller
- MEV submission structure

### Stage E — Observability and audit
- observability
- metrics
- persistence
- audit
- ranking

### Stage F — Readiness and wallet foundation
- vault readiness
- wallet readiness
- wallet balances
- token allowances

### Stage G — Real market integration
- real DEX quote integration
- pool discovery
- liquidity validation
- opportunity builder
- profit model

### Stage H — Real execution path
- real simulation
- transaction builder
- execution safety gate
- submission layer
- receipt tracking

### Stage I — Operator and production systems
- frontend dashboard
- filters/search
- execution controls
- database
- monitoring
- deployment
- security hardening

### Stage J — Trading activation
- paper trading
- small live trades
- production trading mode

---

## 18. Production-readiness truth

The project is **not** yet production-ready for real capital.

A fair assessment today is:

### Backend scaffold maturity
High for a scaffold.

### Safety architecture maturity
Good structure, but not complete.

### Real market integration maturity
Low-to-moderate; mostly scaffold.

### Execution maturity
Low; intentionally disabled.

### Frontend maturity
Minimal / not yet implemented.

### Deployment maturity
Local only.

### Security maturity
Not production-ready yet.

---

## 19. What “production-ready” will actually require

Before this system can be called production-ready, all of the following must be true:

- real wallet secrets handled securely
- production vault connected
- no private key leakage in logs or routes
- DEX contract config populated
- pool config populated
- quotes are real and decoded correctly
- opportunities are real and consistently positive after costs
- simulation is accurate
- execution disabled by default and only enabled with controls
- kill switch exists and is tested
- submission supports receipt/error tracking
- audit history is queryable
- metrics and alerts are externalized
- frontend has operator controls
- database replaces flat JSON for core history
- paper trading validates assumptions
- small live trades validate execution safety
- only then consider broader automation

---

## 20. Practical operator guidance

Until further notice:

### Safe actions
- continue scaffold development,
- add status routes,
- add config-driven policy layers,
- add simulation and validation,
- keep execution disabled.

### Unsafe actions
- enabling funded wallet secrets,
- enabling submission,
- enabling public trade broadcasts,
- using production private keys in local testing,
- skipping validation between steps.

---

## 21. Current recommended next step

From the current state, the correct next step is:

**Step 45 — Real Opportunity Builder and Candidate Generation**

Reason:
- Step 43 gave DEX quote integration structure
- Step 44 gave pool/liquidity structure
- Step 45 should combine those into candidate opportunities
- that is the correct bridge before real simulation and execution work

---

## 22. Summary for any developer taking over

If a new developer joins this project, they should understand this immediately:

1. This is a **safety-first arbitrage backend**, not yet a live bot.
2. The architecture is intentionally stepwise and route-validated.
3. Empty outputs can be correct because the system is conservative.
4. The project currently stands at **Step 44**.
5. The next correct major implementation is **Step 45**.
6. Never move to the next step before:
   - compile passes,
   - Uvicorn starts,
   - `/` phase is correct,
   - `/status` flags are correct,
   - the new route returns expected JSON.
7. Do not enable live execution until simulation, wallet security, and operator controls are complete.

---

## 23. Master short status statement

**Project-Arbitrage is currently a production-style backend scaffold with strong safety, policy, readiness, audit, and observability structure through Step 44, but it is not yet a live trading engine.**

