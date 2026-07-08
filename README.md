# HexStrike Runtime Modulated — Local LLM Real-Time Web Security Scanner

A patched HexStrike build with a **local LLM runtime modulator** for authorised website security testing.

This project adds a local Ollama-powered runtime layer that watches security tools while they are running and makes real-time decisions such as:

* continue scan
* stop noisy scan
* reduce scan rate
* restart focused scan
* launch safe follow-up tool
* compare roles using supplied test tokens
* save evidence
* mark noisy output

The purpose is **real-time tool modulation**, not just running a tool and waiting for final output.

## What This Does

```text
Tool starts
↓
Live output streams line by line
↓
Output is converted into structured events
↓
Local Ollama LLM reviews each event
↓
LLM decides the next runtime action
↓
Safety policy validates the action
↓
Tool continues, stops, restarts, slows, or launches a follow-up
↓
Evidence is saved
```

## Important Safety Notice

Use this only on websites, APIs, and systems you own or are explicitly authorised to test.

This project is designed for defensive security testing and blocks unsafe automation such as:

```text
credential attacks
brute force
stealth
WAF bypass
destructive testing
exploit-chain execution
data exfiltration
persistence
lateral movement
```

## What Is Included

This repo includes:

```text
HexStrike patched server
local LLM runtime modulator
safe authorised pentest planner
real-time event parser
adaptive follow-up queue
evidence logging
role comparison support using supplied test tokens
safe API validation using non-destructive methods
```

## What Is Not Included

Ollama is **not bundled** with this repo.

Users must install Ollama separately on their own machine.

## Tested Platform

Recommended:

```text
Kali Linux
Ubuntu
Debian-based Linux
Python 3.10+
Ollama running locally
```

## Quick Install on Kali / Linux

Paste this into terminal:

```bash
sudo apt update
sudo apt install -y unzip curl git jq python3 python3-venv python3-pip

# Install Ollama locally
curl -fsSL https://ollama.com/install.sh | sh

# Start Ollama
sudo systemctl enable --now ollama || true
pgrep -x ollama >/dev/null || nohup ollama serve > ~/ollama.log 2>&1 &

# Pull local model
ollama pull qwen2.5-coder:7b

# Clone this repo
git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME

# Create Python environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install --upgrade pip setuptools wheel

if [ -f requirements.txt ]; then
  pip install -r requirements.txt
fi

pip install fastapi uvicorn pydantic pyyaml requests aiohttp
```

## Configure Local LLM Runtime Modulation

Replace `staging.example.com` with your owned staging or authorised test website.

```bash
export HEXSTRIKE_REQUIRE_LOCAL_LLM=true
export HEXSTRIKE_MODULATOR_MODEL=qwen2.5-coder:7b
export OLLAMA_URL=http://127.0.0.1:11434/api/chat

export HEXSTRIKE_ALLOWED_HOSTS=staging.example.com
export HEXSTRIKE_MODULATOR_MAX_RPS=2
export HEXSTRIKE_MODULATOR_MAX_JOBS=24
export HEXSTRIKE_MODULATOR_MAX_RUNTIME=1200
```

## Start HexStrike

```bash
source venv/bin/activate
python3 hexstrike_server.py
```

## Check Runtime Modulator Health

Open a second terminal:

```bash
curl http://127.0.0.1:8888/api/runtime-modulator/health | jq .
```

You should see local LLM / Ollama health details.

## Run Real-Time Local LLM Modulated Scan

Replace the target and allowed host with your own website.

```bash
curl -X POST http://127.0.0.1:8888/api/tools/modulated-web-scan \
  -H 'Content-Type: application/json' \
  -d '{
    "target": "https://staging.example.com",
    "profile": "web_power",
    "options": {
      "allowed_hosts": ["staging.example.com"],
      "require_local_llm": true,
      "rate_limit_rps": 2,
      "max_jobs": 24,
      "max_runtime_seconds": 1200
    }
  }' | jq .
```

## What Happens During the Scan

The runtime modulator starts safe web testing tools such as:

```text
katana
httpx
ffuf
nuclei
arjun
dalfox
ZAP-related workflows
```

Then the local LLM watches live results and decides whether to:

```text
continue
stop_tool
reduce_rate
restart_focused
launch_followup
compare_roles
api_validate
save_evidence
mark_noise
```

## Evidence Files

Results are stored in:

```text
runtime_modulator_data/
```

Typical output files:

```text
<run_id>_events.jsonl
<run_id>_decisions.jsonl
<run_id>_findings.jsonl
<run_id>_adaptive_wordlist.txt
```

Watch live LLM decisions:

```bash
tail -f runtime_modulator_data/*_decisions.jsonl
```

Watch live findings:

```bash
tail -f runtime_modulator_data/*_findings.jsonl
```

## Optional: Role Comparison

You can supply test tokens for authorised staging accounts.

Example:

```json
{
  "target": "https://staging.example.com",
  "profile": "web_power",
  "options": {
    "allowed_hosts": ["staging.example.com"],
    "require_local_llm": true,
    "rate_limit_rps": 2,
    "max_jobs": 24,
    "role_contexts": [
      {
        "name": "anonymous",
        "headers": {}
      },
      {
        "name": "customer_a",
        "headers": {
          "Authorization": "Bearer TEST_TOKEN_A"
        }
      },
      {
        "name": "customer_b",
        "headers": {
          "Authorization": "Bearer TEST_TOKEN_B"
        }
      }
    ]
  }
}
```

Role comparison uses safe, non-destructive checks and is intended to identify issues such as:

```text
IDOR
broken access control
anonymous access to restricted endpoints
role permission mistakes
```

## Safe Authorised Pentest Plan

You can generate a safe test plan with:

```bash
curl -X POST http://127.0.0.1:8888/api/tools/authorized-pentest-plan \
  -H 'Content-Type: application/json' \
  -d '{
    "target": "https://staging.example.com",
    "profile": "owned_web_full",
    "options": {
      "allowed_hosts": ["staging.example.com"],
      "rate_limit_rps": 2,
      "max_jobs": 12,
      "max_runtime_seconds": 900
    }
  }' | jq .
```

This creates a safe authorised testing plan for:

```text
attack-surface discovery
authenticated testing
access-control testing
API validation
business-logic testing
misconfiguration checks
file-upload review
JWT/session review
risk-chain mapping
reporting and retesting
```

## MCP / HexStrike AI Plugin Usage

If using the HexStrike AI plugin, call:

```text
modulated_web_scan(
  target="https://staging.example.com",
  allowed_hosts="staging.example.com",
  profile="web_power",
  require_local_llm=true,
  rate_limit_rps=2,
  max_jobs=24,
  max_runtime_seconds=1200
)
```

Suggested prompt:

```text
Run a local LLM real-time modulated web scan against my authorised staging website only.

Target:
https://staging.example.com

Allowed host:
staging.example.com

Use profile web_power.
Require local LLM.
Use low rate limit.
Do not test payment, SMS, email, delete, or destructive endpoints.
Save live events, decisions, findings, and evidence.
```

## Troubleshooting

### Ollama not running

```bash
ollama serve
```

or:

```bash
sudo systemctl restart ollama
```

### Check Ollama API

```bash
curl -s http://127.0.0.1:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5-coder:7b",
    "stream": false,
    "messages": [
      {"role": "user", "content": "Reply with OK only"}
    ]
  }' | jq .
```

### Python dependency issue

```bash
source venv/bin/activate
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
pip install fastapi uvicorn pydantic pyyaml requests aiohttp
```

### Target blocked by policy

Make sure the host is listed in:

```bash
export HEXSTRIKE_ALLOWED_HOSTS=yourdomain.com
```

and also in the API request:

```json
"allowed_hosts": ["yourdomain.com"]
```

## Recommended Use

Use on:

```text
staging websites
QA environments
owned test APIs
authorised security testing labs
bug bounty targets where automation is allowed
```

Do not use on:

```text
third-party websites without written permission
production systems without approval
payment flows
SMS/email-sending flows
delete/destructive endpoints
credential attack scenarios
```

## License

Add your chosen license here.

Recommended for open-source release:

```text
MIT License
Apache-2.0
GPL-3.0
```

## Disclaimer

This project is for authorised defensive security testing only. The maintainers are not responsible for misuse, unauthorised testing, service disruption, data loss, or legal consequences caused by improper use.
