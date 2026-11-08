# Results and limits

The Aner 0.0.8-dev.2 application reproduced the published sparse intervention independently in Safetensors and GGUF, changing **130,188 BF16 entries in two matrices**: 65,260 entries in text layer 15 and 64,928 in layer 19. Both complete candidates match their expected hashes; both complete restorations match their originals. [Replay summary](../evidence/aner-replay-summary.json), [replay receipts](../evidence/aner-replay.jsonl), [restoration receipts](../evidence/aner-revert.jsonl).

| Artifact | Verified complete-file SHA-256 |
|---|---|
| Safetensors candidate | `bd5c223dc7d0ed48a6857f798a94da6ec46c40b9ff4e7bc31872dba10f943ae5` |
| Safetensors restoration | `04b1c301231dd422b8860db31311ab2721511346a32cb1e079c4c4e5f1fe4696` |
| GGUF candidate | `f67c865e3961f13e03ac002de1b797d6806e3f80f5443926ab07a8cf1e023597` |
| GGUF restoration | `9a7bed4041b7975e0f71fa34670d1e9025213bc92905ac0db75d36c4fa3fa623` |

For this run, the two large originals were already present and were fully verified by the artifact operation; the sparse archive was downloaded. Import receipts bind the exact archive, specification and generated patch hashes. Each format uses the original archive directly.

## Paired inference

| Pilot | Base correct | Edited correct | Base truncated | Edited truncated |
|---|---:|---:|---:|---:|
| Eight prompts, cap 2,048 | 1/8 | 1/8 | 7/8 | 7/8 |
| Two repeated prompts, cap 32,768 | 2/2 | 2/2 | 0/2 | 0/2 |

The short pilot generated 15,458 base tokens and 16,005 edited tokens, with one rescue and one regression. The application computed its own grades and paired totals. An independent audit recomputed all 16 grades and checked recorded prompt hashes, checkpoint identities and sampling settings. [Raw observations](../evidence/aner-short-pilot.jsonl), [audit](../evidence/aner-short-pilot-audit.json).

The long follow-up generated 4,480 base tokens and 13,066 edited tokens, with no rescues or regressions. Arithmetic used 3,358 versus 3,024 tokens; algebra used 1,122 versus 10,042. All four grades, prompt hashes, checkpoint identities and sampling records passed independent audit. [Raw observations](../evidence/aner-long-pilot.jsonl), [audit](../evidence/aner-long-pilot-audit.json).

The grader examines text after the final `</think>` marker and accepts the last `\boxed{...}` containing an optional minus sign followed by a nonempty sequence of ASCII digits or commas, with surrounding Unicode whitespace trimmed. It removes commas and compares the resulting string to the expected integer. Missing accepted answers are recorded as an empty string. This is a limited integer-string grader; it does not evaluate symbolic expressions, fractions or general mathematical equivalence.

Both pilot audits record the application source hash, complete observation-file hash and measured runtime binary hash. Prompt hashes were recomputed from recorded UTF-8 prompts. Token hashes are native runtime observations; token IDs are not included for independent recomputation.

## Interpretation

The completed measurements demonstrate exact checkpoint manipulation and successful paired inference. They establish **no accuracy improvement**. The small hand-authored prompt set, one sampling seed and repeated long-pilot questions cannot establish generalization, unrelated-task preservation or controlled speed improvements. Token counts describe these outputs only.

This is a replication of a published intervention. It does not reproduce MATH-500, GSM8K or IFEval, discover a new edit, or demonstrate that direct weight programming replaces pretraining, supervised fine-tuning or LoRA. Broader independent evaluation remains necessary before making those claims.
