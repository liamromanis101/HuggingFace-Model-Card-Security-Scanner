# HuggingFace Model Card Security Scanner

A Python-based security and safety scanner for HuggingFace model cards. It queries the HuggingFace Hub API, analyses model card metadata and free text, and produces a structured CSV report scoring each model across a comprehensive set of security, safety, legal, and provenance checks.

This tool was created in response to a growing concern about the lack of scrutiny applied to open-source AI models before they are embedded into production systems. The findings from an initial scan — and the reasoning behind building this tool — are documented in the accompanying article:

> **[The Hidden Risks in Your AI Supply Chain: What Model Cards Reveal About the Models You Trust](#)**

---

## ⚠️ Access

**The script itself is not publicly available in this repository.**

If you would like a copy of the scanner, or if you have a specific requirement — such as a scan of a particular model type, organisation, or task category — please reach out directly:

👉 **[Request access or discuss your requirements via LinkedIn](https://www.linkedin.com/in/checkteamleader/)**

Requests for custom scan outputs will be considered on a case-by-case basis.

---

## What It Does

The scanner fetches model cards from the HuggingFace Hub and runs checks across eight risk categories, producing a CSV with one row per model and one column per check — each scored `PASS` or `FAIL` — alongside an overall risk score (0–100) and label (`LOW` / `MEDIUM` / `HIGH` / `CRITICAL`).

### Checks Performed

#### 📄 Documentation & Metadata
| Check | Description |
|---|---|
| `has_model_card` | Model has a README / model card |
| `has_model_card_metadata` | Card contains structured YAML metadata block |

#### ⚖️ Licensing
| Check | Description |
|---|---|
| `has_license` | A licence is declared |
| `license_is_permissive` | Licence is MIT, Apache 2.0, CC0, etc. |
| `license_is_restrictive` | Licence is GPL, AGPL, CC-BY-NC, etc. |
| `license_is_custom_or_unknown` | Licence is custom, gated, or unrecognised |

#### 🏛️ Provenance
| Check | Description |
|---|---|
| `from_trusted_org` | Model is from a known accountable organisation |
| `has_author_info` | An author or organisation is identified |
| `has_base_model_disclosed` | Parent/base model is explicitly named |

#### 📊 Training Data Transparency
| Check | Description |
|---|---|
| `has_dataset_disclosure` | Training datasets are named |
| `has_training_procedure` | Training methodology is described |
| `datasets_named_in_card` | At least one dataset explicitly named in metadata or text |

#### 🔐 PII & Data Licensing Risk
| Check | Description |
|---|---|
| `training_data_has_pii_risk` | Training data includes known PII-risk sources (Reddit, GitHub, StackExchange, etc.) |
| `training_data_has_pii_mitigation` | Anonymisation, de-identification, or consent language is present |
| `training_data_license_risk` | Dataset terms may conflict with model licence or commercial deployment |
| `pii_risk_datasets_named` | Named datasets that triggered PII risk flags |

#### 🕷️ Web Scraping
| Check | Description |
|---|---|
| `training_data_is_scraped` | Training data confirmed or strongly indicated to be web-scraped |
| `training_data_scrape_indirect` | Indirect signals suggest internet-scale collection |
| `training_data_scrape_compliance` | robots.txt, ToS adherence, opt-out, or domain blocklist language found |
| `scraped_datasets_named` | Named datasets identified as scraped sources |

#### 🔧 Fine-Tuning Risk
| Check | Description |
|---|---|
| `is_finetuned` | Model is a fine-tune of another model |
| `finetune_base_model_disclosed` | Base model explicitly named |
| `finetune_dataset_disclosed` | Fine-tuning dataset separately described |
| `finetune_methodology_documented` | Hyperparameters and training setup described |
| `finetune_has_post_eval` | Evaluation results reported after fine-tuning |
| `finetune_overfitting_risk` | Small, narrow, or private dataset signals detected |
| `finetune_alignment_degraded` | Explicit safety removal or alignment degradation signals |

#### 🔗 Supply Chain & Deployment Safety
| Check | Description |
|---|---|
| `uses_safe_serialisation` | Weights use safetensors / gguf / onnx format |
| `uses_unsafe_serialisation` | Weights use pickle / .bin / .pt format (arbitrary code execution risk) |
| `requires_trust_remote_code` | Model requires `trust_remote_code=True` or ships custom Python files |
| `has_vulnerable_architecture` | Architecture associated with known documented vulnerabilities |
| `training_data_is_synthetic` | Training data is AI-generated (model collapse / distillation risk) |
| `synthetic_data_licence_risk` | Synthetic data likely from a commercial provider (ToS violation risk) |
| `has_responsible_disclosure` | Contact information or security reporting mechanism documented |
| `has_versioning` | Version numbers or changelog documented |

#### 🛡️ Safety Documentation
| Check | Description |
|---|---|
| `has_safety_section` | Card contains a safety or risk section |
| `has_limitations_section` | Known limitations are documented |
| `has_bias_acknowledgement` | Bias or fairness considerations discussed |
| `has_intended_use` | Intended use cases described |
| `has_out_of_scope_use` | Out-of-scope or prohibited uses documented |
| `has_evaluation_results` | Benchmarks or evaluation results reported |

#### 🚩 Red Flags
| Check | Description |
|---|---|
| `has_redflag_language` | Card contains language associated with harmful use (weapons, malware, exploit, etc.) |
| `is_uncensored_or_unaligned` | Model explicitly described as uncensored, unaligned, or safety-removed |

#### 📈 Maintenance & Community
| Check | Description |
|---|---|
| `has_high_downloads` | Model has over 10,000 lifetime downloads |
| `recently_updated` | Model updated within the last 12 months |
| `has_community_activity` | Model has more than 5 likes |

#### 🤖 AI-Assisted Review *(optional)*
| Check | Description |
|---|---|
| `ai_safety_review_passed` | AI provider reviewed the card text and assessed the safety posture |

---

## CSV Output

Each scan produces a CSV with the following structure:

```
model_id, risk_score, risk_label, author, pipeline_tag, license, downloads, likes,
last_modified, [one column per check above], issues
```

- **`risk_score`** — 0 (lowest risk) to 100 (highest risk)
- **`risk_label`** — `LOW` / `MEDIUM` / `HIGH` / `CRITICAL`
- **`issues`** — pipe-separated plain-English descriptions of every failing check
- All check columns use `PASS` / `FAIL` — a `FAIL` always means a problem was found

---

## CLI Options

```
--limit N           Max models to scan when listing (default: 1000)
--author NAME       Restrict to a specific author or organisation
--task TASK         Filter by pipeline task (e.g. text-generation)
--model-list FILE   Scan a specific list of model IDs (one per line)
--output FILE       Output CSV file path (default: hf_security_scan.csv)
--token TOKEN       HuggingFace API token (or set HF_TOKEN env var)
--ai                Enable AI-assisted card analysis
--ai-provider NAME  AI provider: anthropic (default), openai, or gemini
--delay SECS        Base delay between models (default: 0.5s)
--max-delay SECS    Maximum adaptive backoff delay (default: 30s)
--retries N         Per-call retry attempts (default: 5)
--verbose           Enable verbose logging
```

---

## Environment Variables

| Variable | Purpose |
|---|---|
| `HF_TOKEN` | HuggingFace API token (optional for public models; recommended for large scans) |
| `ANTHROPIC_API_KEY` | Required if using `--ai --ai-provider anthropic` |
| `OPENAI_API_KEY` | Required if using `--ai --ai-provider openai` |
| `GEMINI_API_KEY` | Required if using `--ai --ai-provider gemini` |
| `AI_PROVIDER` | Default AI provider (alternative to `--ai-provider` flag) |

---

## Risk Scoring

Each check contributes weighted penalty points to the risk score. The most heavily weighted risks are:

| Risk | Score impact |
|---|---|
| No model card | +20 |
| Uncensored / unaligned | +20 |
| Red-flag language detected | +20 |
| `trust_remote_code` required | +15 |
| Fine-tuning alignment degraded | +15 |
| No licence | +12 |
| Unverified organisation | +12 |
| Unsafe serialisation only | +12 |
| PII risk with no mitigation | +18 (10 + 8) |
| Scraped data with no compliance | +18 (10 + 8) |

Positive signals (trusted org, permissive licence, high downloads, safe serialisation, post-fine-tune evaluation) reduce the score.

---

## Legal Context

The checks in this scanner are informed by an evolving international regulatory landscape that directly affects organisations deploying open-source AI models, including:

- **EU GDPR** — lawful basis requirements for training on personal data; data subject rights
- **EU AI Act** — GPAI model transparency and copyright compliance obligations (enforcement active from August 2025)
- **EU Copyright Directive** — Text and Data Mining opt-out framework
- **UK GDPR / Data Protection Act** — ICO enforcement position on web scraping for AI training
- **California AB 2013** — requires disclosure of copyrighted training materials
- **Tennessee ELVIS Act / California AB 2602** — voice and likeness rights
- **Denmark Digital Identity Law** — citizens' rights over face, voice, and body in AI-generated content
- **OpenAI / commercial provider Terms of Service** — prohibitions on using outputs to train competing models

---

## Disclaimer

This tool performs static analysis of model card text and metadata. It does not execute model weights, perform dynamic analysis, or provide legal advice. Findings indicate documentation gaps and potential risk signals — they do not constitute confirmed violations. Always consult qualified legal and security professionals before making deployment decisions based on scan results.

---

## Contact

For access to the script, custom scan requests, or to discuss findings:

**[linkedin.com/in/checkteamleader](https://www.linkedin.com/in/checkteamleader/)**
