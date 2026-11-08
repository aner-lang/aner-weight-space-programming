# Third-party notices

Third-party materials retain their original terms. This repository does not change the licensing or release status of Aner's runtime, and assigns no new blanket license to Aner-owned work.

The logos in `branding/` are the official Aner brand assets and link to [anerlang.org](https://anerlang.org).

- **MIT weight-space programming research:** Rikiya Takehi, Jayden Teoh, Alex Zhang and Omar Khattab, MIT CSAIL, *Sparks of Weight-Space Programming*, November 2, 2026. [Research report](https://rikiyat.github.io/blog/weight-space-programming/); [artifact repository license](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/LICENSE). The research repository identifies its scripts as MIT-licensed and its logs/data/networks as CC BY 4.0. This experiment reproduces their published p50c sparse intervention by copying the exact BF16 replacement bits. Discovery of that intervention is attributed to the original researchers.
- **Qwen3.5-0.8B:** [Qwen's pinned checkpoint](https://huggingface.co/Qwen/Qwen3.5-0.8B/tree/2fc06364715b967f1860aea9cf38778875588b17), Apache-2.0. Model files are downloaded separately.
- **Official GGUF export:** [ggml-org/Qwen3.5-0.8B-GGUF](https://huggingface.co/ggml-org/Qwen3.5-0.8B-GGUF/tree/807dfd9a6fadbf703207aac3bfb0b2b3040d077c), listed as Apache-2.0. Its source revision identifies the same Qwen checkpoint. This experiment changes two matrices in a separate copy.
- **llama.cpp / ggml:** [Official source and MIT license](https://github.com/ggml-org/llama.cpp/blob/5047748f114564fcf271e6a648f60e2d2eae6264/LICENSE). The native Aner runtime embeds this pinned inference dependency. Preserve its license notices when distributing runtime builds.

Attribution does not imply endorsement by MIT, the researchers, Qwen, Hugging Face or ggml-org.
