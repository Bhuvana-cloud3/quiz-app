# CI/CD Workflow Architecture

## Workflow Diagram

```
GitHub Event (Push/PR to main/develop)
         ↓
    ┌────────────────────────────────────┐
    │    Build Job (Multi-Node Test)    │
    │  Tests on: 16.x, 18.x, 20.x      │
    └────────────────────────────────────┘
         ↓
    ├─ Checkout code
    ├─ Setup Node.js + npm cache
    ├─ Install dependencies
    ├─ Run linting (if configured)
    ├─ Build app (if configured)
    ├─ Run tests
    ├─ Generate code coverage
    └─ Upload artifacts:
       ├─ test-results-node-{version}
       ├─ code-coverage-report
       ├─ build-artifacts-node-{version}
       └─ test-summary-node-{version}
         ↓
    ┌────────────────────────────────────┐
    │   Code Quality Job                 │
    │   (Coverage Threshold Check)       │
    └────────────────────────────────────┘
         ↓
    ├─ Download build artifacts
    ├─ Verify coverage thresholds
    └─ Generate quality report
         ↓
    ┌────────────────────────────────────┐
    │   Publish Artifacts Job            │
    │   (Consolidation & Storage)        │
    └────────────────────────────────────┘
         ↓
    ├─ Download all artifacts
    ├─ Create artifact manifest
    ├─ Consolidate all results
    └─ Upload: all-ci-artifacts
         ↓
    ┌────────────────────────────────────┐
    │   Notification Job                 │
    │   (Summary & Status Report)        │
    └────────────────────────────────────┘
         ↓
    ├─ Generate workflow summary
    ├─ Update GitHub Actions summary
    └─ Report final status
         ↓
    ✅ Workflow Complete
```

## Artifact Flow

```
Test Execution
    ├─ Test Results (.log, .json)
    │   └─ test-results-node-{version}
    │
    ├─ Code Coverage Reports
    │   ├─ coverage/index.html          → code-coverage-report
    │   ├─ coverage/lcov.info
    │   ├─ coverage/lcov-report/
    │   └─ coverage-final.json
    │
    ├─ Build Artifacts (if any)
    │   ├─ dist/
    │   ├─ build/
    │   └─ lib/
    │       → build-artifacts-node-{version}
    │
    └─ Test Summary
        └─ test-summary-node-{version}
            └─ all-ci-artifacts
```

## Node Version Matrix

```
┌─────────────────────────────────────────┐
│  Build Job Matrix Testing               │
├─────────────────────────────────────────┤
│ ✓ Node.js 16.x (LTS)                    │
│ ✓ Node.js 18.x (LTS - Current)          │
│ ✓ Node.js 20.x (Latest)                 │
└─────────────────────────────────────────┘
   Each version gets full test suite
   Coverage reports published for 18.x
```

## Coverage Report Formats

```
Coverage Output Formats:
├─ text          → Console output (summary)
├─ text-summary  → Detailed console stats
├─ html          → Interactive HTML report
│   └─ coverage/index.html
├─ lcov          → LCOV format for CI tools
│   └─ coverage/lcov.info
└─ json          → Machine-readable format
    └─ coverage/coverage-final.json
```

## Artifact Retention Policy

```
All Artifacts:
└─ Retention: 30 days
   ├─ test-results-node-*
   ├─ code-coverage-report
   ├─ build-artifacts-node-*
   ├─ test-summary-node-*
   └─ all-ci-artifacts

Access: GitHub Actions → Run Details → Artifacts
```

## Coverage Thresholds

```
Global Minimum Coverage Requirements:
├─ Statements: 70% ✓ (Current: 98.47%)
├─ Branches:   70% ✓ (Current: 90.9%)
├─ Functions:  70% ✓ (Current: 100%)
└─ Lines:      70% ✓ (Current: 98.43%)

Build fails if any threshold not met!
```

## Integration Points

```
GitHub Actions CI Pipeline
    ↓
├─→ Codecov Integration
│   └─ Automatic coverage upload
│   └─ Coverage trends tracking
│   └─ Badge generation
│
├─→ GitHub Artifacts Storage
│   └─ 30-day retention
│   └─ Download from Actions tab
│
└─→ PR Status Checks
    └─ Block merge if tests fail
    └─ Show status in PR checks
```

## Workflow Triggers

```
Triggers:
├─ Push to 'main' branch      → Runs workflow
├─ Push to 'develop' branch   → Runs workflow
├─ PR to 'main' branch        → Runs workflow
└─ PR to 'develop' branch     → Runs workflow

Other Branches:
└─ No workflow execution
```

## Performance Metrics

```
Typical Execution Times:
├─ npm install:           ~15-20s (with cache: ~5s)
├─ Running tests:         ~2-5s
├─ Coverage generation:   ~1-2s
└─ Total per job:         ~25-30s (per Node version)

Total for all versions (3):  ~75-90s
Full pipeline with jobs:     ~120-150s (~2-2.5 min)
```

## Status Check Integration

```
GitHub PR Status Checks:
├─ Build: Node 16.x   [passing/failing]
├─ Build: Node 18.x   [passing/failing]
├─ Build: Node 20.x   [passing/failing]
├─ Code Quality       [passing/failing]
├─ Publish Artifacts  [passing/failing]
└─ Notification       [passing/failing]

PR can be merged only if all required checks pass
```

## Security & Secrets

```
Environment Variables Used:
├─ CODECOV_TOKEN (optional)
│   └─ For enhanced Codecov features
│
└─ GitHub Token (automatic)
    └─ Provided by GitHub Actions
    └─ No manual configuration needed
```

## Error Handling

```
Build Failure Scenarios:
├─ Test fails                 → Stops, artifacts uploaded
├─ Coverage below threshold   → Stops pipeline
├─ Dependency install fails   → Stops at install step
├─ Build fails               → Stops, shows error logs
└─ Coverage generation fails → Stops, shows error logs

All failures visible in:
└─ GitHub Actions run logs
└─ Workflow summary
```

## Local Development Alignment

```
Local Testing        →  CI Testing
├─ npm test          →  npm test
├─ npm run test:coverage → npm run test:coverage
└─ Coverage files    →  Coverage artifacts

Keep in sync for consistency!
```

---

**Created:** January 27, 2026
