# AI SOC Assistant

An n8n-orchestrated pipeline that enriches Wazuh alerts with verified
MITRE ATT&CK and NVD/CVE context, runs them through a local LLM for
Tier-1-style triage reasoning, validates the model's output against the
evidence it was given, and raises a corresponding alert in DFIR-IRIS.

## Why
[2-3 sentences: Tier-1 analysts face high alert volume, hard to apply full
investigation rigor to every one. This assists, not replaces, judgment and helps the new analyst to learn by doing the work.]

## Architecture
[Diagram - can be a Mermaid block, see docs/architecture.md for detail]

## What it does
1. Receives Wazuh alert via webhook
2. Normalizes + extracts MITRE/CVE references
3. Enriches with verified local MITRE table + live NVD lookup
4. Builds a grounded, evidence-only prompt
5. LLM produces structured triage analysis (JSON-schema enforced)
6. Validates model output against actual evidence (catches hallucination)
7. Computes deterministic priority (independent of LLM verdict)
8. Produces analyst report (JSON + Markdown)
9. Raises a corresponding alert in DFIR-IRIS for tracking/filtering

## Status: V1 - functional, documented limitations
See [docs/decisions.md](docs/decisions.md) for known issues and reasoning.

## Test cases
- [FIM / EICAR](test-cases/fim-eicar/)
- [Log4Shell CVE-2021-44228](test-cases/log4shell-cve-2021-44228/)

## Stack
n8n · Ollama (local LLM) · NVD API · MITRE ATT&CK · DFIR-IRIS · Wazuh

## Known limitations (V1)
- Model occasionally hallucinates despite schema + grounded evidence
  (see ADR-003) - validation layer catches but doesn't block these yet
- No CVE response caching
- No automated error recovery on NVD/LLM timeout