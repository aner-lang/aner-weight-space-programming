# Reproduce the experiment with Aner

This guide targets **Aner v0.0.8** with native artifact acquisition, NPZ import, Safetensors/GGUF weight transactions and Qwen3.5 raw completion. The [recorded evidence](../evidence/current-status.json) preserves the exact runtime and source identities used for the reported measurements. Compatibility with the guide does not imply that the required runtime is already publicly available.

The experiment reproduces MIT's published p50c edit. It verifies exact checkpoint changes, runs paired inference and restores the original files. Read [results and limits](results.md) before interpreting the small pilot as a performance result.

## 1. Prepare the machine and runtime

Use an Apple Silicon Mac with a compatible macOS release and a native Aner build that includes the Metal inference backend. The supplied pilot programs explicitly require Metal, Float16 attention cache and Flash Attention. The reference host was an Apple M2 Max with **64 GiB unified memory**, running **macOS 26.7.2**. This is the observed configuration, not a measured minimum-memory requirement; smaller-memory systems and other operating systems were not qualified by this experiment.

Storage follows directly from the pinned files:

| Retained files | Payload size |
|---|---:|
| Two original checkpoints plus the sparse archive | 3,304,992,715 bytes, about 3.30 GB |
| Original, edited and restored copies of both checkpoints | 9,913,815,288 bytes, about 9.91 GB / 9.23 GiB |

The completed reference run retained about **9.93 GB** across its inputs and generated files, including patches, inverse records and logs. **15 GB free disk space** is recommended for working headroom, excluding runtime or source-build files; it is not a measured minimum. These totals describe retained file sizes, not a peak-memory requirement. Inference also needs memory for model state, attention/recurrent state and workspaces; the long pilot uses a larger context than the short pilot. Checkpoint editing itself does not require a GPU.

Follow the official [Downloads and setup](https://anerlang.org/downloads) page for the available Mac package, its supported operating systems and release checksums. Its Mac installation command is:

```sh
curl -fsSL https://anerlang.org/install.sh | sh
```

Open a new terminal and confirm the selected installation:

```sh
aner --version
aner --binary
aner doctor
```

The installer selects the currently available stable package. If that package lacks this guide's APIs, consult the download page's release or development-channel availability. A development package must be selected explicitly and must exist in the signed catalog. Installing an older runtime, an editor extension or this source repository alone does not supply the required capabilities.

Git is needed for the checkout below. Internet access to Hugging Face and the research artifact repository is needed unless all three pinned input files have already been obtained.

## 2. Obtain the source and check the APIs

```sh
git clone https://github.com/aner-lang/aner-weight-space-programming.git
cd aner-weight-space-programming
```

Keep every subsequent command in this repository root, which contains `programs/`, `specifications/`, `assets/` and `generated/`. The last two directories initially contain only placeholder files. Keep the programs and specifications unchanged when comparing against the recorded experiment.

Check all five programs before downloading model files:

```sh
aner check programs/check-helpers.aner
aner check programs/replay.aner
aner check programs/pilot-short.aner
aner check programs/pilot-long.aner
aner check programs/revert.aner
aner run programs/check-helpers.aner
```

Each source check should succeed. The helper program should print a JSON record with `"status":"PASSED"` and `"checks":11`; it loads no model and uses no network. These checks exercise the required language/API surface, including `llm.fetch_artifact`, `llm.weight_import_npz`, checkpoint operations, `with_sampling` and `complete`. They do not establish that a particular installed backend can execute Metal inference; that is checked when each pilot loads its model.

Record the actual runtime and checkout used for your run:

```sh
aner --version > generated/runtime-version.txt
aner --binary > generated/runtime-binary.txt
git rev-parse HEAD > generated/source-revision.txt
```

The pilot's fixed `development_target` field identifies the frozen experiment source. Use your saved runtime identity for the actual executable used in a new run. Preserve the repository's existing evidence as the reference record and save your own outputs under `generated/`.

## 3. Obtain the exact open model and edit archive

**The normal path is automatic:** `replay.aner` calls `llm.fetch_artifact` for each of the following three files. A missing file is downloaded over HTTPS; an existing file is accepted only after its full size and SHA-256 match. You do not need to download the whole model repository or obtain the author's already-edited model.

| Input and official pinned download | Publisher's filename | Required local path | Bytes | License |
|---|---|---|---:|---|
| [Original Qwen3.5-0.8B checkpoint](https://huggingface.co/Qwen/Qwen3.5-0.8B/resolve/2fc06364715b967f1860aea9cf38778875588b17/model.safetensors-00001-of-00001.safetensors) | `model.safetensors-00001-of-00001.safetensors` | `assets/qwen3.5-0.8b-base.safetensors` | 1,746,942,600 | Apache-2.0 |
| [Official Qwen3.5-0.8B BF16 GGUF](https://huggingface.co/ggml-org/Qwen3.5-0.8B-GGUF/resolve/807dfd9a6fadbf703207aac3bfb0b2b3040d077c/Qwen3.5-0.8B-BF16.gguf) | `Qwen3.5-0.8B-BF16.gguf` | `assets/qwen3.5-0.8b-base-bf16.gguf` | 1,557,662,496 | Apache-2.0 |
| [MIT p50c sparse archive](https://raw.githubusercontent.com/RikiyaT/lm_understand-logs/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/edit/edit.npz) | `edit.npz` | `assets/edit.npz` | 387,619 | CC BY 4.0 |

Expected SHA-256 values:

| Required local path | SHA-256 |
|---|---|
| `assets/qwen3.5-0.8b-base.safetensors` | `04b1c301231dd422b8860db31311ab2721511346a32cb1e079c4c4e5f1fe4696` |
| `assets/qwen3.5-0.8b-base-bf16.gguf` | `9a7bed4041b7975e0f71fa34670d1e9025213bc92905ac0db75d36c4fa3fa623` |
| `assets/edit.npz` | `e92a92aafd1f1068b4fd94aa0fbe081ba3ddea2047ee7bcac7161d861be299e7` |

For **manual acquisition**, open the three pinned download links, save the complete files, then place them at the required local paths above. Rename the downloaded Safetensors and GGUF files exactly as shown. Save the NPZ unchanged as a regular file; leave it compressed. A model-card page, repository pointer, quantized GGUF, different revision or newer checkpoint cannot substitute for these byte-identical inputs. The replay program still performs its full checks on manually downloaded files.

You can independently inspect the downloaded file hashes on macOS:

```sh
shasum -a 256 assets/qwen3.5-0.8b-base.safetensors
shasum -a 256 assets/qwen3.5-0.8b-base-bf16.gguf
shasum -a 256 assets/edit.npz
```

The GGUF contains the tokenizer and metadata needed by the inference backend. This experiment does not require separate tokenizer downloads. The [source manifest](../source_manifest.json) also records provenance documents; those additional documents are not input requirements for the programs. Preserve the attribution and model terms in [third-party notices](../THIRD_PARTY_NOTICES.md).

## 4. Replay the published edit

Run once into fresh output destinations:

```sh
aner run programs/replay.aner > generated/replay.jsonl
```

Wait for a successful exit before continuing. The program obtains or verifies the three inputs, independently imports the same sparse archive for each format, checks the declared edit limits, and applies the patches to separate candidate files. The originals remain intact.

The specifications bind the complete base/archive hashes, permitted tensors, shapes, original/edited tensor hashes and array member names. Import preserves the published BF16 bits. For GGUF, reversing the stored dimension notation gives the logical row-major shape; the operation does not transpose numerical values.

A complete replay produces nine JSONL receipts. Fetch receipts report `FETCHED` or `VERIFIED_EXISTING`; import and validation receipts precede the two successful applications. Both candidates must contain **130,188 actual changed entries** and match the following complete-file hashes, which the program asserts:

| Candidate | SHA-256 |
|---|---|
| `generated/qwen-safetensors/model.safetensors` | `bd5c223dc7d0ed48a6857f798a94da6ec46c40b9ff4e7bc31872dba10f943ae5` |
| `generated/qwen-gguf/model.gguf` | `f67c865e3961f13e03ac002de1b797d6806e3f80f5443926ab07a8cf1e023597` |

Each transaction directory also contains `patch.json`, `inverse.json` and `receipt.json`. Keep these together: restoration validates the complete transaction. See the [reference replay receipts](../evidence/aner-replay.jsonl) and [summary](../evidence/aner-replay-summary.json).

## 5. Run paired inference

Run the short and long pilots sequentially:

```sh
aner run programs/pilot-short.aner > generated/pilot-short.jsonl
aner run programs/pilot-long.aner > generated/pilot-long.jsonl
```

Wait for each program to finish successfully. Each loads the original GGUF and then the edited GGUF, verifies the model fingerprint, and requires actual Metal execution. The long pilot repeats two questions from the short pilot; it is not an independent holdout set.

| Setting | Short pilot | Long pilot |
|---|---|---|
| Questions per model | 8 | 2 repeated questions |
| Maximum generated tokens per answer | 2,048 | 32,768 |
| Context capacity | 4,096 | 34,816 |
| Device / attention cache | Metal / Float16 | Same |
| Flash Attention | On | Same |
| Decode / prefill CPU threads | 8 / 8 | Same |
| Batch / microbatch tokens | 512 / 512 | Same |
| Sampling | Presence penalty 1.5, temperature 0.6, top-k 20, top-p 0.95, seed 42 | Same |
| Penalty history | Generated tokens only | Same |
| Persistent prefix caching / speculation | Disabled | Same |

The application renders the full thinking prompt and sends it to raw `complete()`. It owns JSON escaping, the limited boxed-integer answer grader and paired aggregation. Each response record contains the full prompt/text, grade, finish reason, token counts, prompt/token hashes, model fingerprint and native timing/sampling metadata. Model leases close after each derived configuration and after each completed variant.

The complete short log contains 21 JSONL records; the long log contains nine. Each ends in an `aner.wsp.aner_paired_summary.v1` record. The recorded reference results are:

| Pilot | Base correct | Edited correct | Truncations per model | Base / edited generated tokens |
|---|---:|---:|---:|---:|
| Short | 1/8 | 1/8 | 7/8 each | 15,458 / 16,005 |
| Long | 2/2 | 2/2 | 0/2 each | 4,480 / 13,066 |

The short pilot has one rescue and one regression; the long pilot has neither. These are observations from the pinned measured environment, not guaranteed results across later runtimes, backends or machines. Exact checkpoint hashes are the replay gate; inference variation must be reported with its own environment and records. Neither pilot establishes an accuracy or efficiency improvement. Compare with the [short audit](../evidence/aner-short-pilot-audit.json), [long audit](../evidence/aner-long-pilot-audit.json) and [grading limits](results.md).

## 6. Restore both original checkpoints

After inference, restore into separate destinations:

```sh
aner run programs/revert.aner > generated/revert.jsonl
```

The program uses the saved inverse transactions and asserts both original complete-file hashes. It emits two restoration receipts and leaves the originals and edited candidates intact.

| Restored file | Expected SHA-256 |
|---|---|
| `generated/restored-safetensors/model.safetensors` | `04b1c301231dd422b8860db31311ab2721511346a32cb1e079c4c4e5f1fe4696` |
| `generated/restored-gguf/model.gguf` | `9a7bed4041b7975e0f71fa34670d1e9025213bc92905ac0db75d36c4fa3fa623` |

Compare with the [reference restoration receipts](../evidence/aner-revert.jsonl).

## Output map and reruns

| Location | Contents |
|---|---|
| `assets/` | Three verified original inputs |
| `generated/runtime-version.txt`, `runtime-binary.txt`, `source-revision.txt` | Your executable and checkout identities |
| `generated/p50c-safetensors.json`, `p50c-gguf.json` | Independently imported declarative patches |
| `generated/qwen-safetensors/`, `qwen-gguf/` | Edited candidates and complete transaction records |
| `generated/replay.jsonl` | Fetch, import, validation and application receipts |
| `generated/pilot-short.jsonl`, `pilot-long.jsonl` | Paired response records and summaries |
| `generated/restored-safetensors/`, `restored-gguf/` | Restored files and restoration transaction records |
| `generated/revert.jsonl` | Both restoration receipts |

The repository ignores `assets/` and `generated/` contents apart from placeholders. Keep large inputs and generated models out of source commits.

For a full rerun, retain `assets/` so the native fetch step can verify and reuse it. Move the existing `generated/` directory to a uniquely named archive outside the checkout, then recreate an empty `generated/` directory. For example, after choosing an unused archive name:

```sh
mv generated ../aner-weight-space-run-1
mkdir generated
touch generated/.gitkeep
```

Repeat the identity recording and execution steps. This preserves the prior run and gives the patch/transaction operations fresh destinations. For an inference-only rerun, keep both candidates and choose new output log filenames. Shell redirection with `>` replaces an existing log, even though the checkpoint APIs refuse to overwrite their output files.

## Troubleshooting

| Symptom | Check and resolution |
|---|---|
| `aner` is not found | Open a new terminal after installation and follow the official Downloads page's PATH instructions. Confirm the selected command with `aner --version` and `aner --binary`. |
| Unknown `llm` operation, wrong argument type or failed source check | The selected runtime must expose the v0.0.8 APIs used by the unchanged programs. Check release availability and your selected binary; installing an editor extension does not update the runtime. |
| Backend unavailable, unsupported architecture or Metal load failure | Use a compatible Apple Silicon native build with the qualified Qwen3.5/Metal backend. The programs intentionally require Metal and the specified cache configuration. A successful source check alone does not prove backend availability. |
| Memory allocation failure during the long pilot | Its context is larger and needs additional model state/workspace. Close unrelated memory-intensive work and inspect available memory. Reducing the token cap or changing placement defines a different experiment and must be reported separately. |
| HTTPS, TLS or download timeout | Check network access to the pinned publishers and available disk space; retry or use the manual-download path. Keep full size and hash verification enabled. |
| Existing artifact has the wrong size/hash | Compare the exact revision, filename and hash above. Move the mismatching file aside and obtain the pinned original; the fetch operation will not silently replace it. Avoid HTML error pages, repository pointers and quantized substitutes. |
| Archive, array-member, dtype or tensor hash rejection | Use the unchanged `edit.npz` and matching specification with its correct base format. Do not unpack/repack the archive, change tensor mappings or substitute another edit. |
| Output file or transaction directory already exists | Archive the earlier `generated/` outputs and create fresh destinations as described above. A partial failed run can leave earlier successful stages that also need preserving before retry. |
| Path or file-access failure | Run from the repository root with writable `assets/` and `generated/` parents. Use ordinary directories and regular files; artifact validation rejects symbolic-link destinations. |
| Restoration rejects the transaction | Keep the candidate, `patch.json`, `inverse.json` and `receipt.json` together and unchanged. Do not combine files from different runs. Restore to a fresh destination. |
| Inference text or grades differ from the reference | Preserve the new raw logs and runtime identity. Check model hashes, sampler settings, prompt format and backend before drawing conclusions; one small seeded pilot is not a general benchmark. |
