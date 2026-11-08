# Primary sources and provenance

This experiment replicates the p50c Qwen3.5-0.8B intervention published with MIT's *Sparks of Weight-Space Programming*. The research brief supplied for the project served as a starting reference; implementation and claims were checked against the primary artifacts below.

| Source | Role |
|---|---|
| [MIT research report](https://rikiyat.github.io/blog/weight-space-programming/) | Rikiya Takehi, Jayden Teoh, Alex Zhang and Omar Khattab's November 2, 2026 research report |
| [Pinned Experiment 8 protocol](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/README.md) | Published constraints, evaluation settings, reported results and artifact limitations |
| [Pinned p50c manifest](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/edit/manifest.json) | Tensor names, dimensions, counts and original/edited tensor hashes |
| [Pinned sparse archive](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/edit/edit.npz) | Published indices and exact replacement BF16 bits |
| [Pinned Qwen checkpoint](https://huggingface.co/Qwen/Qwen3.5-0.8B/tree/2fc06364715b967f1860aea9cf38778875588b17) | Original full checkpoint |
| [Author-hosted edited checkpoint](https://huggingface.co/RikiyaT/Qwen3.5-0.8B-weight-space-programming) | Published reconstruction and separate evaluation; its scores are not Aner measurements |
| [Pinned official BF16 GGUF](https://huggingface.co/ggml-org/Qwen3.5-0.8B-GGUF/tree/807dfd9a6fadbf703207aac3bfb0b2b3040d077c) | Inference representation whose source revision identifies the same original Qwen checkpoint |
| [Pinned llama.cpp](https://github.com/ggml-org/llama.cpp/tree/5047748f114564fcf271e6a648f60e2d2eae6264) | Native inference dependency used through Aner |

The MIT artifact revision is `4d09ba6d6a7e891dd8063974dfe81b3d78e3494f`. The [source manifest](../source_manifest.json) pins input URLs, lengths and hashes. The [specifications](../specifications) bind the exact tensor edits for each checkpoint format. Published BF16 values are copied by their stored bits.

The MIT report primarily attributes this intervention's gains to repairing reasoning loops. Its non-looping Gemma experiment did not establish a new reasoning capability. This case therefore treats direct weight programming as experimental adaptation and keeps exact replay separate from performance claims. [MIT report](https://rikiyat.github.io/blog/weight-space-programming/)

Aner's contribution is an executable replication with native artifact validation, bounded archive import, reversible checkpoint transactions and explicit raw completion/sampling. All reported measurements in this repository come from the accompanying Aner programs. The [results](results.md) describe their restricted evaluation scope, and [third-party notices](../THIRD_PARTY_NOTICES.md) preserve attribution.
