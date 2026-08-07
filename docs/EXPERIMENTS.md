# Experiments

This document describes the experiments conducted to validate TMM, as presented in the paper.

## LLM Comparison Results

### Per-Version Extraction Datasets (129 Docker Reports)

The 129 OpenVAS PDF reports in [`dockers/`](../dockers/) (see [docs/INVENTORY.md](INVENTORY.md) for the full container list) were extracted by all 5 LLMs across 3 pipeline iterations (V1, V2, V3), each consolidated into a per-LLM CSV dataset under `artifacts/<version>/openvas_129_dockers/` and evaluated against the same ground-truth baseline ([`baselines/native/vulnnet_scans_openvas.csv`](../baselines/native/vulnnet_scans_openvas.csv)):

| LLM      | V1 rows | V2 rows | V3 rows |
| -------- | ------- | ------- | ------- |
| DeepSeek | 13,274  | 6,706   | 6,511   |
| GPT-4    | 8,396   | 6,805   | 6,168   |
| GPT-5    | 3,528   | 5,699   | 6,432   |
| LLaMa 3  | 7,463   | 6,625   | 6,449   |
| LLaMa 4  | 10,094  | 6,585   | 6,401   |

Row counts are raw extracted entries per dataset (pre host+name matching against the baseline). The drop and convergence from V1 to V3 reflects successive fixes to the extraction/consolidation pipeline, reducing duplicate/split entries across LLMs.

Metrics for each version (and a combined V1+V2+V3 comparison) are computed with [`tools/TMM_metrics_run.py`](../tools/TMM_metrics_run.py) — see the "Consolidated Multi-LLM Metrics Report" claim in the main [README](../README.md#experiments) for the exact commands, producing `artifacts/<version>/TMM_metrics_<version>.xlsx` and `artifacts/TMM_metrics_all_versions.xlsx`.

## Running Experiments

### Metrics Report per Version (TMM_metrics_run.py)

Compute BERTScore/ROUGE-L/deterministic-field metrics for all 5 LLMs of a given version against the native baseline, and generate the consolidated XLSX report:

```bash
# V1
python tools/TMM_metrics_run.py --baseline baselines/native/vulnnet_scans_openvas.csv --versions TMMv1:artifacts/v1/openvas_129_dockers/deepseek_v1.csv:deepseek TMMv1:artifacts/v1/openvas_129_dockers/gpt4_v1.csv:gpt4 TMMv1:artifacts/v1/openvas_129_dockers/gpt5_v1.csv:gpt5 TMMv1:artifacts/v1/openvas_129_dockers/llama3_v1.csv:llama3 TMMv1:artifacts/v1/openvas_129_dockers/llama4_v1.csv:llama4 --xlsx artifacts/v1/TMM_metrics_v1.xlsx

# V2
python tools/TMM_metrics_run.py --baseline baselines/native/vulnnet_scans_openvas.csv --versions TMMv2:artifacts/v2/openvas_129_dockers/deepseek_v2.csv:deepseek TMMv2:artifacts/v2/openvas_129_dockers/gpt4_v2.csv:gpt4 TMMv2:artifacts/v2/openvas_129_dockers/gpt5_v2.csv:gpt5 TMMv2:artifacts/v2/openvas_129_dockers/llama3_v2.csv:llama3 TMMv2:artifacts/v2/openvas_129_dockers/llama4_v2.csv:llama4 --xlsx artifacts/v2/TMM_metrics_v2.xlsx

# V3
python tools/TMM_metrics_run.py --baseline baselines/native/vulnnet_scans_openvas.csv --versions TMMv3:artifacts/v3/openvas_129_dockers/deepseek_v3.csv:deepseek TMMv3:artifacts/v3/openvas_129_dockers/gpt4_v3.csv:gpt4 TMMv3:artifacts/v3/openvas_129_dockers/gpt5_v3.csv:gpt5 TMMv3:artifacts/v3/openvas_129_dockers/llama3_v3.csv:llama3 TMMv3:artifacts/v3/openvas_129_dockers/llama4_v3.csv:llama4 --xlsx artifacts/v3/TMM_metrics_v3.xlsx

# All versions combined (15 LLM x version entries in one report)
python tools/TMM_metrics_run.py --baseline baselines/native/vulnnet_scans_openvas.csv --versions TMMv1:artifacts/v1/openvas_129_dockers/deepseek_v1.csv:deepseek TMMv1:artifacts/v1/openvas_129_dockers/gpt4_v1.csv:gpt4 TMMv1:artifacts/v1/openvas_129_dockers/gpt5_v1.csv:gpt5 TMMv1:artifacts/v1/openvas_129_dockers/llama3_v1.csv:llama3 TMMv1:artifacts/v1/openvas_129_dockers/llama4_v1.csv:llama4 TMMv2:artifacts/v2/openvas_129_dockers/deepseek_v2.csv:deepseek TMMv2:artifacts/v2/openvas_129_dockers/gpt4_v2.csv:gpt4 TMMv2:artifacts/v2/openvas_129_dockers/gpt5_v2.csv:gpt5 TMMv2:artifacts/v2/openvas_129_dockers/llama3_v2.csv:llama3 TMMv2:artifacts/v2/openvas_129_dockers/llama4_v2.csv:llama4 TMMv3:artifacts/v3/openvas_129_dockers/deepseek_v3.csv:deepseek TMMv3:artifacts/v3/openvas_129_dockers/gpt4_v3.csv:gpt4 TMMv3:artifacts/v3/openvas_129_dockers/gpt5_v3.csv:gpt5 TMMv3:artifacts/v3/openvas_129_dockers/llama3_v3.csv:llama3 TMMv3:artifacts/v3/openvas_129_dockers/llama4_v3.csv:llama4 --xlsx artifacts/TMM_metrics_all_versions.xlsx
```

Each run produces sheets for Summary, Overview (global + per-version), All LLMs, one sheet per LLM, and Omission & Hallucination analysis. Close the target `.xlsx` in Excel before re-running — the script cannot overwrite a file that's open (locked on Windows).

### Full Experiment Suite

```bash
# Windows
python tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm deepseek --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm gpt4 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm gpt5 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm llama4 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm llama3 --convert csv [--allow-duplicates] [--output-dir <output_directory>]

# Linux/macOS
python3 tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm deepseek --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python3 tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm gpt4 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python3 tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm gpt5 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python3 tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm llama4 --convert csv [--allow-duplicates] [--output-dir <output_directory>]
python3 tools/batch_pdf_extractor.py --input-dir dockers --scanner openvas --llm llama3 --convert csv [--allow-duplicates] [--output-dir <output_directory>]

```

**Key Features:**

- Runs extraction for every (report, LLM, run) combination, then runs all metrics in a parallel post-pass via `tools/TMM_metrics_run.py`

**Main Parameters:**

- `--input-dir`: Directory containing paired .xlsx (baseline) and .pdf (report) files
- `--llm`: Space-separated LLMs to test (e.g., `deepseek gpt4 llama3`)
- `--scanner`: Scanner to use (`openvas` or `tenable`)
- `--metrics`: Methods to run (`bert`, `rouge`, `entity`, `schema`, `severity`, `coverage`, or `all`). Producer/consumer dependencies auto-resolved.
- `--runs-per-model`: Number of runs per combination (default: 10)
- `--allow-duplicates`: Flag to allow duplicates (recommended for OpenVAS; omit for Tenable)
- `--output-dir`: Results root directory (default: `results_runs`)
- `--metrics-workers`: Parallel workers for the post-extraction metrics pass (default: 4)
- `--skip-metrics`: Skip the metrics + aggregator post-pass
- `--checkpoint-file`: Checkpoint file to resume from

## Deduplication Strategies

### OpenVAS

- `--allow-duplicates` (**recommended**): uses custom strategy for maximum granularity
- Removes only exact duplicates (same Name, port, protocol)
- Legitimate vulnerabilities may repeat on different ports

### Tenable WAS

- Without `--allow-duplicates` (**recommended**): uses custom strategy for smart merge
- Groups instances/bases of the same type
- Consolidates arrays (URLs, description, etc.)

These strategies were designed to balance granularity and efficiency, avoiding vulnerability exceedances and respecting the structure of each scanner.
