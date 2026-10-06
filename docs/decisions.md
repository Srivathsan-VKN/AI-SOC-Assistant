## ADR-001: JSON Schema enforcement over prompt-only instructions
**Problem:** Gemma4Defense-2B emitted its own CVE/CWE/CVSS schema instead of
the required triage schema, ignoring prompt instructions.
**Decision:** Switched Ollama's `format` param from `"json"` to a full
JSON Schema with `additionalProperties: false`.
**Result:** Structural schema violations became impossible; model-level
hallucination (fabricated CVE IDs) still occurred and is caught by the
validation layer instead (see ADR-003).

## ADR-002: Context loss across HTTP Request node boundary
**Problem:** Priority scoring and CVE/MITRE validation were silently wrong -
`rule.level` and `evidence_package` were `undefined` downstream of the
Ollama LLM Call node.
**Root cause:** n8n's HTTP Request node replaces `$json` with the response
body; it doesn't merge with incoming data.
**Fix:** Parse LLM Output node restores prior context via
`$('Build LLM Prompt').item.json`, matching the pattern already used in
Extract NVD Context.

## ADR-003: Model reliability findings (open issue)
Across three test runs, the fine-tuned model (Gemma4Defense-2B) exhibited
severe hallucination even with grounded evidence and schema constraints -
including labeling a CVSS 10.0 RCE (Log4Shell) as FALSE_POSITIVE at 0%
confidence with zero supporting evidence. Flagged as a known limitation;
model swap to a general-purpose instruct model is planned but not yet
implemented in V1.

## ADR-004: Model reproduces few-shot example wording instead of reasoning
**Observation:** On the FIM/EICAR test case, the model's output used near-
verbatim phrasing from the system prompt's worked example ("scheduled backup
job touched the file," "antivirus scan updated metadata") despite the actual
alert (eicar.com.txt, specific mtime delta) differing from the example.
**Significance:** Unlike the hallucination cases (ADR-003), this failure is
schema-valid and plausible-looking, making it harder to catch via automated
validation. It strengthens, rather than contradicts, the case for a model
swap - the issue isn't shape or factual grounding alone, it's whether the
model reasons over the specific evidence at all.