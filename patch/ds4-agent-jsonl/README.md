# Agent/Cowork: reproducible native patches

Version 91 contains five explicit unified patch variants. Version 87 migrated
the three existing Agent adaptations without changing their output; version 88
added the Qwen3.8 candidate and version 89 fixes optional-renderer event handling
in every existing variant. Version 90 adds the Qwen3.6 native-tool candidate
and corrects JSON-document delimiter escaping in both Qwen variants.
Version 91 moves piped Qwen new-session preparation onto the existing worker,
with a private candidate context, cancellation and explicit terminal receipts.
The separate Qwen3.6 host integration has two passing real development
workflows; foreground desktop and general quality qualification remain open.
Passing the native patch gate alone does not qualify the desktop app.
The host enables Qwen3.8 Agent/Cowork with the new pin and
its required PLE; Design and other hardware still require separate qualification.

| Source base | Patch |
| --- | --- |
| antirez/ds4 `c0a6119f363ef82125877142f13fb3fe491cba14` | [main-current.patch](main-current.patch) |
| antirez/ds4 `f62ca29a308724cde5bc99134ede19104b2a3260` (identical Agent source) | [main-current.patch](main-current.patch) |
| antirez/ds4 `f4d03f6cf9f11c1e7b630bcb160853acfba7c52a` | [main-previous.patch](main-previous.patch) |
| antirez/ds4 Laguna `448d5695d1c86401a4e9447c440feb983b73e6de` | [laguna.patch](laguna.patch) |
| ivanfioravanti/ds4-metal `b4c355079d375d20821ece732e99195c71b32c06` | [qwen38.patch](qwen38.patch) |
| ivanfioravanti/ds4-metal `0bb323aa63cf7ed6167ae91a7606e1d875d404a3` (identical Agent source) | [qwen38.patch](qwen38.patch) |
| ivanfioravanti/ds4-metal `66b0e3fc3bf0f548db1ec0c0dd19f4e43567a7f8` (identical Agent source) | [qwen38.patch](qwen38.patch) |
| vagrillo/ds4 `60fca11f0c8b16ca50c757324dddd717ba043098` | [qwen35.patch](qwen35.patch) |

[`bases.json`](bases.json) records source, patch and output hashes. The shared
first-party remote implementation remains in `remote-agent.cfrag`, included at
its original position instead of copied into every delta.

The Qwen3.8 variant preserves the fork's native function/parameter parser, prompt
format, reasoning template and PLE options. Cowork schemas are placed inside
the native tools envelope, one JSON object per line. Literal `</tool_call>`
inside document/code content no longer prematurely terminates a parameter;
an incomplete parameter still cannot become an executable call. This fix was
reproduced on the unmodified upstream parser across every split of a 241-byte
test call. Two small real-weight development workflows on `0bb323a` now prove
native Agent and Cowork tool selection, file creation and readback on M2 Max.
Two subsequent real host workflows on `66b0e3f` verify launch, automatic Agent
Task Graph, Cowork document tools and post-rejection continuation. They do not
qualify held-out quality, every tool, desktop interaction or other hardware.

Qwen3.6's pinned server already renders ChatML and native function/parameter
calls, but its Agent only recognizes DeepSeek/GLM calls. The new candidate
backports the Qwen XML parser from the pinned Qwen3.8 source, adapting it to
the older Agent API and preserving Qwen3.6's own tokenizer, turn terminators,
generation loop and kernels. It does not insert DeepSeek's Max prefix into
Qwen conversations. Cowork receives the existing general-purpose Office schemas.
Literal markup and escaped closing delimiters remain document data; duplicate
parameters and incomplete calls cannot execute. Run-owned parser storage is
limited to 1 MiB of call text, 32 calls, 64 arguments/call and 127-byte names.
It is released on reset/cancellation; exceeding a limit is an explicit parse
error. Incremental value scans and fixed-size delimiter tails avoid quadratic
rescanning, with deterministic visit-counter regressions. No Qwen3.6 vision,
PLD or desktop-mode qualification is implied. Qwen3.6 disk checkpoint
reads/writes are refused before touching the incomplete upstream payload API;
bootstrap does not create that cache, and the existing live context is retained.
The application still saves its conversations independently.
Two real-weight development workflows on the final candidate passed on M2 Max:
Agent and Cowork produced independently checked files and read them back,
without creating the incomplete disk-checkpoint cache. This qualifies those
two native workflows only. Subsequent real headless host checks pass file
effects, generation interruption, new-session completion and readback for
Agent and Cowork. The first host control failure remains retained; a successful
retry does not resolve its cause. That replay also exposed a synchronous reset
which occupied the native command thread and delayed progress/interrupts;
version 91 addresses that separate issue as described below. Earlier failures
and the pre-cache-fix run are retained.

## Qwen new-session ownership

For the Qwen3.6 and Qwen3.8 piped Agent/Cowork runtimes, `/new` admits one reset
on the existing worker. The command reader can continue consuming progress and
interrupts while the system prompt is prepared. A required autosave must finish
successfully first; a later cancellation does not undo that completed save.

Preparation owns at most one candidate session in addition to the live session,
with the same configured context and shared immutable model weights. The engine's
native context/model dimensions bound its KV allocation. No context downgrade,
extra inference worker, unbounded queue or second model copy is introduced.
Allocation errors returned by the native API leave the old context intact;
this is not a promise of recovery from an OS-level process kill.

A compatible system-prompt cache may populate only the candidate; its tokens
must match the newly prepared prompt exactly. There are no speculative cache
writes. The owner revalidates session, engine, config, transcript and cancellation
before a bounded pointer/metadata swap. Old buffers and Qwen3.8 image references
are released outside the shared lock. Failure retains the earlier conversation
and attachments, and emits one error receipt before returning idle. Qwen3.8
autosave metadata is published only after the native save succeeds.

On the tested arm64 layout, the two reset flags use existing padding:
Qwen3.6's worker remains 2,144 bytes and Qwen3.8's remains 2,184 bytes.
Deterministic barriers verify that blocked preparation does not block progress
consumption or mutate the live context. This is a responsiveness/correctness
change, not a claimed decode-speed improvement. The legacy main/Laguna and TTY
reset paths are unchanged and are not qualified by these Qwen tests.
Progress retains the native granularity: per-token Qwen3.6 versus completed
8,192-token prefill chunks on the pinned Qwen3.8. The latter can display no new
token count during a single GPU chunk. No kernel-level interrupt or fabricated
percentage is implied; the live reset fixture must span multiple native chunks.
See the [Qwen stopping checkpoint](../../docs/QWEN_CHECKPOINT.md) for real-run
receipts, retained failures and remaining work.

Renderer-only captures are allowed by upstream without an Agent worker. JSONL
reasoning notifications now check that a configured worker exists. The shipped
variants already include [renderer-events-fix.patch](renderer-events-fix.patch):
**do not apply that delta again**. It documents the post-migration correction
and lets migration tests reverse only that reviewed change before comparing
against the unchanged frozen version-86 byte oracle.

The host applies one complete exact-context variant to a private source copy.
It rejects missing inputs, partial/repeated application, ambiguous candidates
and changed hunk context. Unrelated edits may shift line offsets but are retained.
There is no fuzz or whitespace repair. Inputs/output are bounded to 16 MiB and
512 hunks; at most eight variants are considered. Source identity is fixed by
the caller, not a path embedded in a patch.

On POSIX, a checkout-scoped build lease stays inherited by the compiler if the
parent exits. Agent/web derived sources, first-party objects and both linked
executables remain in a private directory until the build succeeds. The two
original input files are revalidated before publication and never rewritten.
The staging directory's identity is retained by an open descriptor. Replacing
its pathname rejects publication; bounded, non-recursive cleanup only operates
on that retained directory, not a replacement or symlink target.
An interrupted build may leave a private staging directory but cannot leave the
original Agent source patched. A pre-existing legacy marker is refused while
preserving both source and backup; automatic verified recovery is not implemented.
The native engine's separate patches and core object compilation still operate
in the managed checkout. Design now shares this lease and builds a private
source snapshot; unrelated external builds do not participate automatically.
The two Agent/Cowork final renames are not a crash-atomic pair transaction, and
mtime/version freshness is not a complete dependency signature.

## Verification

- `make test-unified-patch`: native application, exact line boundaries,
  ambiguity, malformed manifests, bounded regular files and private failure.
- `make test-agent-build`: the production builder with a **simulated compiler**;
  failed/incomplete links, source edits, kill, concurrent builds and old outputs.
- `make test-agent-patch-migration`: the three existing bases against frozen
  version-86 native output after reversing the documented renderer fix, plus
  the two new Qwen bases against their recorded derived hashes. All five use independent
  Git apply/reversal, including unrelated edits,
  CRLF, partial/repeated application and drift. Requires the pinned local Git
  objects or exact files in `DSTUDIO_AGENT_BASE_SOURCES` named after each base.
  `DSTUDIO_AGENT_QWEN38_DIR` can point to the isolated candidate instead of the
  older managed Qwen checkout; `DSTUDIO_AGENT_QWEN35_DIR` selects Qwen3.6.
  It never downloads sources or weights implicitly.
- `make test-agent-native-build`: macOS source-only copies of main and Laguna,
  real compiler/linker, Agent/Cowork and Design tools, upstream Agent unit tests
  and fragmented renderer events with ASan/UBSan on private C objects. Native
  engine/GPU objects are not sanitizer-instrumented. Includes main
  Chat PLD build/CLI and explicit Laguna PLD rejection. Override
  `AGENT_MAIN_TREE`/`AGENT_LAGUNA_TREE` for other locations; set
  `AGENT_QWEN38_TREE` to additionally test the Qwen candidate and its explicit
  Chat PLD rejection. `DSTUDIO_AGENT_BUILD_BACKEND=cpu` selects a fresh CPU build
  instead of Metal. Tool responses come
  from real filesystem actions; model responses are deterministic fixtures.
  `DSTUDIO_AGENT_BUILD_HOST` may point to an isolated bundle's `DStudio` binary:
  the same test invokes its build CLI from `/`, materializes support files in
  a task-owned profile and checks that its actual patch assets match the source.
- `make test-qwen38-agent QWEN38_AGENT_TREE=PATH_TO_BUILT_CANDIDATE`: executes
  upstream Agent units, fragmented native Qwen parsing, eight native prompt
  combinations, real file writes and Cowork document creation/readback. Traversal
  and symlink writes outside its private workspace must fail without damage.
  Model text and engine identity are explicit fixtures; no model is loaded.
  ASan/UBSan instrument the parser/Agent/helper C objects, not upstream core
  objects. `QWEN38_AGENT_FLAGS=` disables those sanitizers explicitly.
- `make test-qwen35-agent QWEN35_AGENT_TREE=PATH_TO_BUILT_CANDIDATE
  QWEN35_AGENT_FLAGS=--sanitize`: real native linking, original Agent units,
  261 stream splits, native schemas, malformed/duplicate/over-limit rejection,
  JSON document escapes, linear scan counters, recurrent disk-cache guards
  and real Agent/Cowork filesystem effects. Model text
  and identity are simulated. No weights are loaded or source checkout edited.
- `make test-qwen-session-reset QWEN35_AGENT_TREE=PATH_TO_QWEN35
  QWEN38_AGENT_TREE=PATH_TO_QWEN38`: actual native command reader and worker
  with a deterministic simulated-inference barrier. Six Qwen3.6 and seven
  Qwen3.8 scenarios cover cancellation, failure, success, allocation failure,
  late cancellation, duplicate admission and Qwen3.8 save failure/attachments.
  The original synchronous code fails the three shared baseline scenarios.
  Agent/helper objects use ASan/UBSan; prebuilt engine objects do not.
- `node tests/integration/upstream_agent_prompt_test.mjs PATH_TO_BUILT_MAIN`:
  ten native prompt builders, parsed schemas and vision/remote gating. It builds
  its own derived probe objects without changing the supplied engine checkout.
- `make test-metal-workspace`: actually initializes each supplied Metal backend
  through the host's shader-path setup from an unrelated workspace and checks
  257 exact additions. Existing native objects are required; no weights are
  loaded. This catches runtime source-path omissions that a compile cannot.
- `node tests/live/qwen38_agent_smoke.mjs ENGINE MODEL_GGUF PLE_GGUF`: explicitly
  loads the candidate's real weights, sequentially runs Agent and Cowork, and
  checks tool receipts, exact file contents, source preservation and readback.
  The backbone is resident and PLE SSD-backed; context 16k, thinking/MTP off.
  Logs, filenames and paths are private ignored artifacts. This is a development
  smoke, not the held-out benchmark or an end-to-end desktop test. Its independent
  trace checks have a model-free gate, `make test-qwen38-tool-oracle`.
  `ENGINE MODEL_GGUF --qwen35` selects the already-built Qwen3.6 candidate,
  with resident weights and no PLE or Qwen3.8-specific power configuration.
- `node tests/live/qwen38_host_smoke.mjs ENGINE MODEL_GGUF PLE_GGUF`: the same
  development task through actual host launch/send/poll APIs, automatic routing,
  full production charters, private KV, exact artifacts and durable graph receipts.
  It rejects a Design switch while each real engine is active, then requires
  another real read. Native sampling defaults are retained; this is not a
  throughput comparison to the fixed-seed CLI run.
  Add `--reset-lifecycle` to observe live reset prefill, cancel it, recall a
  random code held only in the old conversation, then complete another reset
  and read back the previously created file. Use `ENGINE MODEL_GGUF --qwen35
  --reset-lifecycle` for Qwen3.6. Run the two models sequentially.

Receipts and failed attempts remain under ignored `tests/.artifacts/`. The first
two targets are in `check-fast`. macOS builds, tool execution and sanitizer
checks do not establish model quality, numerical parity or CUDA/ROCm/Windows
qualification. Those require their separate hardware and real-weight gates.
