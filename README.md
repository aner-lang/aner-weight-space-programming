<a href="https://anerlang.org#gh-dark-mode-only">
  <img src="branding/aner-logo-dark.svg#gh-dark-mode-only" alt="Aner programming language" width="300">
</a>
<a href="https://anerlang.org#gh-light-mode-only">
  <img src="branding/aner-logo.svg#gh-light-mode-only" alt="Aner programming language" width="300">
</a>

# Replicating MIT weight-space programming with Aner

This case study replicates MIT's published direct-weight intervention in Qwen3.5-0.8B using and improving Aner. The experiment programs are written in Aner. New native module capabilities support verified artifact downloads, exact sparse weight import, reversible checkpoint edits and explicit model sampling.

**Exact replay and restoration passed for Safetensors and GGUF.** Both edited checkpoints contain the published 130,188 changed BF16 values, match their expected complete-file hashes, and restore to the original hashes. The eight-question short pilot scored **1/8 for both base and edited models**, with seven truncated answers each. A longer follow-up on two repeated questions scored **2/2 for both**, without truncation. These measurements demonstrate successful execution and establish no accuracy gain.

See the [results and limits](docs/results.md), [replay evidence](evidence/aner-replay-summary.json), [short-pilot audit](evidence/aner-short-pilot-audit.json), [long-pilot audit](evidence/aner-long-pilot-audit.json) and [current status](evidence/current-status.json).

## Run the experiment

**Compatibility target: Aner v0.0.8.** Use a runtime with native artifact downloads, NPZ import, reversible Safetensors/GGUF edits and Qwen3.5 raw completion through `aner.llm`. Later versions must expose these same APIs and support the inference configuration below. Exact build identities remain in the [recorded evidence](evidence/current-status.json).

The included inference pilots require an Apple Silicon Mac and the native Metal backend. They were tested on an **Apple M2 Max with 64 GiB unified memory, running macOS 26.7.2**; minimum RAM has not been established. The editing operations themselves do not require a GPU. The three input downloads total **3.305 GB**. Allow about **10 GB** for the complete replay and restoration data; **15 GB free disk space** is recommended for working headroom, excluding runtime or source-build files.

### 1. Install and check Aner

Use the official [Aner Downloads and setup](https://anerlang.org/downloads) page to obtain a compatible runtime. The standard Mac installer is:

```sh
curl -fsSL https://anerlang.org/install.sh | sh
```

Open a new terminal, then check the selected executable:

```sh
aner --version
aner --binary
aner doctor
```

The installer supplies the available stable release, which may precede this experiment's required capabilities. This repository does not include a runtime package. Check the Downloads page for availability and complete the source checks below before downloading the model. If those APIs are unavailable, obtain a compatible v0.0.8 release or development build before continuing.

### 2. Obtain and check the experiment

```sh
git clone https://github.com/aner-lang/aner-weight-space-programming.git
cd aner-weight-space-programming
aner check programs/check-helpers.aner
aner check programs/replay.aner
aner check programs/pilot-short.aner
aner check programs/pilot-long.aner
aner check programs/revert.aner
aner run programs/check-helpers.aner
```

Run all subsequent commands from this repository root. All five source checks must succeed; the helper program reports `"status":"PASSED"` and `"checks":11` without loading a model. Record your actual runtime and checkout for comparison with the reference evidence:

```sh
aner --version > generated/runtime-version.txt
aner --binary > generated/runtime-binary.txt
git rev-parse HEAD > generated/source-revision.txt
```

### 3. Download the open model and replay the edit

**The Aner replay program downloads and verifies all three required files automatically:**

```sh
aner run programs/replay.aner > generated/replay.jsonl
```

| Pinned input download | Required local path | Download size |
|---|---|---:|
| [Qwen3.5-0.8B Safetensors checkpoint](https://huggingface.co/Qwen/Qwen3.5-0.8B/resolve/2fc06364715b967f1860aea9cf38778875588b17/model.safetensors-00001-of-00001.safetensors) | `assets/qwen3.5-0.8b-base.safetensors` | 1,746,942,600 bytes |
| [Official Qwen3.5-0.8B BF16 GGUF](https://huggingface.co/ggml-org/Qwen3.5-0.8B-GGUF/resolve/807dfd9a6fadbf703207aac3bfb0b2b3040d077c/Qwen3.5-0.8B-BF16.gguf) | `assets/qwen3.5-0.8b-base-bf16.gguf` | 1,557,662,496 bytes |
| [MIT p50c sparse edit archive](https://raw.githubusercontent.com/RikiyaT/lm_understand-logs/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/edit/edit.npz) | `assets/edit.npz` | 387,619 bytes |

For manual downloads, save those exact files at the listed paths, renaming the two checkpoints accordingly and leaving `edit.npz` compressed. The program verifies existing files by complete size and SHA-256 before use. Different revisions or quantized GGUF files cannot substitute for these inputs. The GGUF already contains the tokenizer; no separate tokenizer download or model conversion is needed.

Wait for a successful exit before proceeding. Replay creates edited copies in `generated/qwen-safetensors/` and `generated/qwen-gguf/`, with inverse patches and receipts. It asserts the complete candidate hashes and the declared edit constraints. Full input and expected output hashes are listed in the [reproduction guide](docs/reproducibility.md) and [artifact manifest](source_manifest.json).

### 4. Run the paired inference pilots

Run each command to completion before starting the next:

```sh
aner run programs/pilot-short.aner > generated/pilot-short.jsonl
aner run programs/pilot-long.aner > generated/pilot-long.jsonl
```

Both programs run the original and edited GGUF in sequence. The short pilot uses eight questions and a 2,048-token answer cap; the long pilot repeats two questions with a 32,768-token cap. Each log includes full responses, grades, model fingerprints, sampling settings and a final paired summary. Inspect the summaries with:

```sh
tail -n 1 generated/pilot-short.jsonl
tail -n 1 generated/pilot-long.jsonl
```

The reference scores are **1/8 → 1/8** for the short pilot and **2/2 → 2/2** for the long pilot. Inference can vary with runtime, backend and hardware; preserve your own logs and environment. These small pilots do not establish a reasoning or efficiency improvement.

### 5. Restore and verify the originals

```sh
aner run programs/revert.aner > generated/revert.jsonl
```

Restoration uses the saved inverse transactions, writes separate restored checkpoints and asserts both complete original hashes. Keep the candidate files, patches and receipts together. Use fresh output destinations for another full run; archive previous outputs first. See the [complete guide](docs/reproducibility.md) for the output map, exact inference settings, safe reruns and troubleshooting.

## Experiment programs

| Program | Purpose |
|---|---|
| [replay.aner](programs/replay.aner) | Fetch pinned originals, independently import the published patch for both checkpoint formats, apply it and assert complete-file hashes |
| [revert.aner](programs/revert.aner) | Restore both complete original checkpoints |
| [pilot-short.aner](programs/pilot-short.aner) | Run eight paired prompts with a 2,048-token output cap, grade answers and report paired totals |
| [pilot-long.aner](programs/pilot-long.aner) | Repeat two prompts with a 32,768-token output cap |
| [check-helpers.aner](programs/check-helpers.aner) | Check JSON escaping and the integer-answer grader |

## What this demonstrates

Direct weight programming is an experimental gradient-free adaptation approach alongside established LLM training methods. Here MIT discovered the intervention; Aner reproduces its exact stored changes. This experiment does not search for a new edit or reproduce MIT's full benchmark evaluation.

The published intervention targeted 131,072 positions; the released sparse archive contains 130,188 positions whose bits changed. Exact replay establishes implementation fidelity. Claims about improved reasoning, generalization or training efficiency require broader evaluation.

[Primary sources](docs/provenance.md) · [Artifact identities](source_manifest.json) · [Attribution](THIRD_PARTY_NOTICES.md)

Replication of MIT research using and improving Aner.

Alfonso G. Bastias, Ph.D.
