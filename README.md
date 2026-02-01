# Inspection Certificates

> Verifiable proof of ephemeral execution - for skills AND data

**Inspection Certificates** provide cryptographic proof that a computation happened without storing sensitive inputs. Two main use cases:

1. **Skill Verification** - Prove agent skills work correctly
2. **Data Inspection** - Prove claims about private datasets

Both use the same ephemeral execution model: load input → run computation → delete input → keep certificate.

## Quick Start

### Skill Verification (Docker)
```bash
# Verify a skill works
POST /verify
{
  "skillUrl": "https://example.com/skill.md"
}

# Certificate proves: tests passed, timing, attestation
```

📖 **[Skill Verification Docs](DEMO.md)**

### Data Inspection (GitHub Actions)
```bash
# Inspect private data
workflow_dispatch:
  inputs:
    dataset_secret: "PRIVATE_DATASET"
    inspection_prompt: "Analyze for X"

# Certificate proves: data was analyzed, result is authentic
```

📖 **[Data Inspection Quick Start](docs/data-inspection/GITHUB-ACTIONS-QUICKSTART.md)**

## Concept

**Inspection = Verify properties without full exposure**

Like a pre-buy inspection:
- Home inspection before buying (without revealing all details)
- Car inspection (without test driving for weeks)
- Dataset inspection (without seeing all records)

**Certificate = Verifiable proof**

- Public, shareable URL
- Anyone can verify
- Tamper-proof logs
- Cryptographic attestation (TEE mode)

## Use Cases

### 1. Skill Verification (Original)

**Claim:** "This Hermes skill works correctly"

**Inspection:**
- Load skill code
- Run test suite in Docker
- Generate attestation
- Delete skill code

**Certificate:**
```
✅ Hermes Skill Verified
   Duration: 2.2s
   Tests: 6/6 passed
   Attestation: 0xfb41b44...
```

### 2. Data Inspection (New)

**Claim:** "I have 5,000 support tickets about refunds"

**Inspection:**
- Load private dataset (from GitHub Secret)
- Run LLM analysis
- Generate certificate
- Delete dataset

**Certificate:**
```
✅ Dataset Inspected
   Hash: abc123...
   Analysis: "4,987 tickets, 73% mention refunds"
   Themes: shipping delays, product defects
   Data: DELETED (not accessible)
```

### 3. API Key Verification (New)

**Claim:** "I have valid API credentials"

**Inspection:**
- Load API key (secret)
- Test with API call
- Check rate limits/balance
- Delete key

**Certificate:**
```
✅ API Key Valid
   Provider: Anthropic
   Balance: $52.34
   Rate limit: Good standing
   Key value: NOT REVEALED
```

## Three Execution Backends

### 1. Docker (Current)
**Best for:** Skill verification, reproducible tests

```bash
npm install
npm start
POST /verify { "skillUrl": "..." }
```

**Isolation:** Docker containers  
**Trust:** Local Docker daemon  
**Docs:** [DEMO.md](DEMO.md)

### 2. GitHub Actions (New)
**Best for:** Data inspection, public verifiability

```bash
# One-time setup: Add secrets to repo
# Run: Trigger workflow via UI or CLI
```

**Isolation:** GitHub runners  
**Trust:** GitHub infrastructure  
**Docs:** [Data Inspection Quick Start](docs/data-inspection/GITHUB-ACTIONS-QUICKSTART.md)

### 3. dstack TEE (Future)
**Best for:** High-stakes, regulatory compliance

```bash
dstack run --tee intel-tdx analyze.py
```

**Isolation:** Intel TDX hardware  
**Trust:** CPU manufacturer  
**Docs:** [TEE_READY.md](TEE_READY.md)

## Documentation

### Core Concept
- **[INSPECTION-CERTIFICATES.md](INSPECTION-CERTIFICATES.md)** - Unifying concept
- **[PLAN.md](PLAN.md)** - Original skill verifier plan

### Skill Verification
- **[DEMO.md](DEMO.md)** - Live verification example
- **[PRODUCT_TOUR.md](PRODUCT_TOUR.md)** - Feature walkthrough
- **[TEE_READY.md](TEE_READY.md)** - TEE integration

### Data Inspection
- **[AUDITOR-GUIDE.md](docs/data-inspection/AUDITOR-GUIDE.md)** - How to verify certificates
- **[GITHUB-ACTIONS-QUICKSTART.md](docs/data-inspection/GITHUB-ACTIONS-QUICKSTART.md)** - 5-min demo
- **[ARCHITECTURE-EPHEMERAL.md](docs/data-inspection/ARCHITECTURE-EPHEMERAL.md)** - Deep dive

## API Design (Unified - Future)

```typescript
POST /inspect
{
  "type": "skill" | "data" | "credential",
  "input": {
    // For skills:
    "skillUrl"?: "https://...",
    
    // For data:
    "dataSecret"?: "PRIVATE_DATASET",
    "prompt"?: "Analyze for X",
    
    // For credentials:
    "credentialSecret"?: "API_KEY",
    "testEndpoint"?: "/api/status"
  },
  "backend": "docker" | "github-actions" | "dstack-tee"
}

Response:
{
  "certificateId": "cert_xyz",
  "result": { ... },
  "proof": {
    "attestation": "0x...",
    "timestamp": "..."
  },
  "certificate_url": "/certificate/cert_xyz"
}
```

## Installation

### Skill Verification (Docker Backend)

```bash
npm install
export DOCKER_HOST=tcp://your-docker-host:2375
npm start
```

Server runs on `http://localhost:3000`

### Data Inspection (GitHub Actions Backend)

```bash
# 1. Fork this repo
# 2. Add GitHub Secrets (dataset, API keys)
# 3. Run workflow from Actions tab
```

See [Quick Start Guide](docs/data-inspection/GITHUB-ACTIONS-QUICKSTART.md)

## Examples

### Skill Verification

```bash
curl -X POST http://localhost:3000/verify \
  -H "Content-Type: application/json" \
  -d '{
    "skillUrl": "https://raw.githubusercontent.com/user/repo/main/skill.md"
  }'
```

Response:
```json
{
  "verificationId": "ver_abc123",
  "skillName": "hermes",
  "status": "verified",
  "tests": {
    "total": 6,
    "passed": 6,
    "failed": 0
  },
  "attestation": "0xfb41b44...",
  "certificateUrl": "/verification/ver_abc123"
}
```

### Data Inspection

```bash
gh workflow run inspect-data.yml \
  -f dataset_secret="SUPPORT_TICKETS" \
  -f prompt="Count tickets mentioning 'refund'. List common themes."
```

Certificate: `https://github.com/user/repo/actions/runs/12345`

## Why "Inspection Certificates"?

**Traditional approach:**
- "Trust me, this skill works"
- "Trust me, my data is valuable"

**Inspection Certificates:**
- Verifiable proof the computation happened
- Public logs anyone can audit
- Ephemeral (no data storage risk)
- Scalable (skills, data, credentials)

**Use cases:**
- Agent skill marketplaces (prove skills work)
- Data marketplaces (prove value before purchase)
- API key verification (prove credentials valid)
- Research validation (prove results without exposing test data)

## Repository Structure

```
skill-verifier/
├── INSPECTION-CERTIFICATES.md  # Core concept
├── README.md                     # This file
│
├── DEMO.md                       # Skill verification demo
├── PLAN.md                       # Original plan
├── TEE_READY.md                  # TEE integration
│
├── docs/
│   └── data-inspection/
│       ├── AUDITOR-GUIDE.md               # How to verify
│       ├── GITHUB-ACTIONS-QUICKSTART.md   # 5-min start
│       └── ARCHITECTURE-EPHEMERAL.md      # Deep dive
│
├── .github/
│   └── workflows/
│       └── docker-publish.yml    # Skill verification CI
│
├── .github-data-inspection/
│   └── workflows/
│       └── ephemeral-execution.yml  # Data inspection workflow
│
├── examples/
│   ├── hermes-verified/          # Verified skill example
│   ├── hello-world/              # Minimal example
│   ├── node-app/                 # Node.js example
│   └── python-script/            # Python example
│
└── src/                          # Skill verifier server (TODO)
```

## Next Steps

**For Skill Verification:**
1. Read [DEMO.md](DEMO.md)
2. Try verifying a skill
3. Deploy to dstack ([TEE_READY.md](TEE_READY.md))

**For Data Inspection:**
1. Read [Quick Start](docs/data-inspection/GITHUB-ACTIONS-QUICKSTART.md)
2. Set up GitHub Actions workflow
3. Run first inspection

**For Integration:**
- See [INSPECTION-CERTIFICATES.md](INSPECTION-CERTIFICATES.md)
- Unified API coming in Phase 2

## Contributing

This repo unifies two related concepts:
- **Skill verification** (original)
- **Data inspection** (new)

Both follow the same pattern: ephemeral execution with permanent certificates.

## License

MIT

---

**Inspection Certificates: Verifiable proof without permanent storage** 🦞🔐
