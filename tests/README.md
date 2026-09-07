# Tests: what each result actually proves

Correctness before performance. No test is accepted merely because a function
name, comment, prompt phrase or CSS declaration occurs in application source.

## Qwen integration and reset regressions

See [the Qwen checkpoint](../docs/QWEN_CHECKPOINT.md) for exact pins, model-free commands, retained failures and remaining coverage. `make test-unified-patch test-agent-build test-launch-preflight test-agent-spawn test-launch-control test-ui-launch` checks the shared patch/build and launch dependencies without loading weights. Compiler/engine responses are simulated where stated by each harness.

## Qwen real host workflows

```sh
make tests/.build/dstudio-server-test test-qwen38-tool-oracle
node tests/live/qwen38_host_smoke.mjs ENGINE MODEL_GGUF PLE_GGUF
# Qwen3.6: no PLE; additionally exercise generation interrupt and a new session:
node tests/live/qwen38_host_smoke.mjs ENGINE MODEL_GGUF --qwen35 --controls
# Separate reset lifecycle: visible prefill, cancellation with retained memory,
# then a successful reset and real tool read. Run one model at a time:
node tests/live/qwen38_host_smoke.mjs ENGINE MODEL_GGUF --qwen35 --reset-lifecycle
node tests/live/qwen38_host_smoke.mjs ENGINE MODEL_GGUF PLE_GGUF --reset-lifecycle
```

Use an already-installed `ds4-qwen38` at pin `66b0e3f`, or `ds4-qwen35` at
`60fca11f`, with the model (and Qwen3.8 PLE) resolving to its shared model
store. This runs the real headless HTTP host in a
private profile: asynchronous launch, the unmodified production Agent/Cowork
charters, structured tools, exact saved files and readback. Agent uses the
production automatic Task Graph route, including its actual durable journal.
Each mode must also remain usable for another read after a rejected Design
switch; the Agent-to-Cowork transition must replace only the test-owned process.

Runs are sequential with 16k context, thinking/MTP/DSpark off and expert
streaming off: resident backbone plus native SSD PLE for Qwen3.8, resident
weights without PLE for Qwen3.6. Sampling remains at the
production Agent defaults, **not** the fixed seed/temperature of the CLI gate.
Each workflow is bounded to 12 tool calls and 600 seconds, with bounded logs,
private KV directories and strict workspace checks. The runner refuses an
existing inference process; it does not download weights, restart the user's
app or stop unrelated engines. Requests, answers, process and binary identities,
source-install receipt and failures remain in `tests/.artifacts/qwen38-host-live/`
or `tests/.artifacts/qwen35-host-live/`.

`--controls` requires actual generated tokens before sending an interrupt,
checks the canceled task and the still-running engine, starts a new session
and requires another real tool read without changing earlier files. Its
separate deadlines are 120 seconds for generation, 15 for interruption, 600
for reset and 120 for readback; the 600-second workspace-workflow limit is
unchanged. Qwen3.6 must reject incomplete disk checkpoints before becoming
busy and must not create a checkpoint directory. Its requested power of 37
must be reported as effective native 100, without forwarding `--power`.
An early terminal reply fails this control gate; bounded follow-up status and
transcript observations are retained, not used to silently accept the failure.

The complete September 7 Qwen3.8 replay passed both workflows. Its initial Agent
attempt is retained as a failed test: the grader incorrectly rejected the
host's `.dstudio/task-graphs/` receipts. The corrected oracle permits only the
graph IDs reported by the host and checks journal identity, ordering, terminal
success, file/byte limits and symlink rejection; arbitrary hidden files remain
forbidden. Separate oracle regressions exercise valid and invalid workspaces.
This is two real development workflows, not held-out quality, numerical parity,
foreground desktop coverage or evidence for CUDA/ROCm.

The initial Qwen3.6 host control run remains failed: its workspace operations
passed, but the counting request ended before interruption could be tested.
The native Agent/Cowork CLI gate passed separately; it does not override that
host failure. The complete retry passed Agent and Cowork, including generated
tokens before interruption, canceled task receipts, new-session completion and
tool readback. It also exposed delayed native progress while the synchronous
reset ran. A generation interrupt did not qualify cancellation during reset.
The separate version-91 reset fix and gate below address that ownership path;
they do not erase or establish the cause of the initial counting failure.

`make test-qwen-session-reset QWEN35_AGENT_TREE=ENGINE35
QWEN38_AGENT_TREE=ENGINE38` executes the shipped patches and real native worker
with simulated inference and deterministic barriers: six Qwen3.6 cases and
seven Qwen3.8 cases. It checks the command reader remains available, old context
and attachments survive failure/cancellation, duplicate reset admission fails,
late cancellation prevents publication and save failure retains the old identity.
Both original synchronous baselines fail the three shared scenarios. ASan/UBSan
cover the Agent/helpers, not the already-built engine objects. The observed
arm64 worker sizes remain 2,144 and 2,184 bytes respectively; candidate session
count is bounded to one alongside the live session.

`--reset-lifecycle` uses actual weights, a random conversation-only code and a
task-owned project-memory fixture which forces a genuine prompt-cache miss.
It must observe prefill while busy, cancel within the existing 15-second
deadline, receive exactly one native terminal error, and recall the exact code
without tools. A second reset must complete, emit one success receipt and permit
a real read without changing prior files. Each reset remains bounded to 600
seconds; these development replays are not speed or held-out quality benchmarks.
The memory fixture is model-specific: the Qwen3.8 pin reports progress after
native 8,192-token chunks, so its fixture must span two chunks; Qwen3.6 reports
per token. No chunk-size/context override or numerical engine change is used.
The shared progress oracle requires unfinished work (`0 < done < total`).
The first Qwen3.8 attempt, `run-uAb5iW`, remains failed: its 4,358-token reset
completed successfully in one chunk, without an intermediate observation where
cancellation could be tested. The longer-fixture retry is separate evidence,
not a retroactive pass or an intra-GPU-kernel cancellation guarantee.

Two initial Qwen3.6 reset replays are retained as failures (`run-KaB607` and
`run-l76E5r`). Both had correct recall but exposed test-decoder defects: status
frames interleaved between answer tokens, then the reserved final-line Task
Graph receipt already hidden by the UI. The shared oracle now decodes those
transport elements for both Qwen variants before an exact-answer assertion.
Regressions at every split retain wrong codes, extra prose, repeated receipts
and malformed frames as failures; an echoed question is never an answer.
Qwen3.8's second reset attempt (`run-QQtql2`) also remains failed: cancellation
and recall were correct, but the grader included the native autosave system
line in the answer. The decoder now removes only that exact terminal shape.
A regression executes the actual UI `splitUserTurns`/`segmentAgent` functions
to verify the same text/system separation for correct and wrong answers.
See [the Qwen checkpoint](../docs/QWEN_CHECKPOINT.md) for final replay results
and the explicit desktop/quality/backend coverage gaps.


Image presets: `make test-image-pipeline` executes the production coordinator
and shell with explicitly simulated pixels. `make test-image-runtime` compares
all four presets, seven aspect ratios and every CFG step to the installed
official Ideogram/Comfy scheduler, including progress and the unchanged MAX
default; it does not generate images. `make test-http-lan` checks native HTTP
dispatch and rejects invalid presets. The Settings browser test runs in Chromium
and WebKit; the video browser and native Design interrupt tests check that the
saved preset reaches image requests without changing editing or video profiles.

The opt-in [Hermes-inspired live benchmark](live/image_preset_benchmark.py)
starts the actual installed Ideogram worker sequentially, with real weights.
It retains per-image captions, hashes, parameters, progress, failures and
wall time including model startup/shutdown. PNG sanity checks are separate from
visual prompt-adherence review. Public charts use Matplotlib; see
[image preset results](../extension/benchmarks/image-presets/README.md).

Main engine update and real DeepSeek/GLM prefill/decode comparison:
[September 5 update](../docs/DS4_MAIN_UPDATE_2026-09-05.md).
`make test-main-decode-metrics` checks timing-span parsing and refuses speed
comparisons for failed workloads or changed model/prompt identities. The live
runner is opt-in and starts one actual model at a time.

`make test-search-evidence` executes query-aware page excerpt selection and
the actual evidence-extraction request path with simulated model replies. It
checks late-page/Unicode evidence, fixed input bounds and cancellation without
publishing stale facts or issuing another call. Its loopback HTTP test verifies
real request closure and independent deadlines. It is not an LLM quality test;
live research and competitor work is tracked in
[the quality plan](../docs/SEARCH_AGENT_QUALITY_PLAN.md).

The [real page-evidence comparison](../extension/search/bench/README.md) runs
eight public fictional questions through actual Chrome and a resident native
vision model. The complete version-2 run is 3/8 before and 8/8 after, with all
latencies and original failed/grader receipts retained. It is not a full research
pipeline or competitor benchmark. `make test-search-publication` checks public
export bounds/denominators and the actual Matplotlib chart values.

`make test-remote-agent-workspace` executes the real Agent binary with simulated
model frames and actual read/write/bash tools. Absolute and relative `--chdir`
paths must affect all tools exactly once; missing directories fail before an
inference request. This catches the remote-mode regression caused by upstream
moving the local engine's `chdir` later. No weights are loaded by this gate.

`make test-web-visual-unit` executes the native same-page screenshot response
adapter. `make test-web-visual-browser` additionally requires Chrome and Python
Pillow: it compiles the browser helper against all four installed engine source
trees, reads a public fixture in isolated headless Chrome, decodes the JPEG and
checks its actual colors, retained text, exact page-tab cleanup and aggregate
fragmented-response limit. It preserves each run in a fresh ignored directory.
Neither target loads weights or establishes that a model interpreted the image
correctly. Search's simulated-model gate separately checks multimodal request
content, capability/model-switch validation, three-capture bounds, cancellation
and exclusion of image blobs from serialized chats.

Published benchmark charts use Matplotlib. Regenerate the engine, Design and
anonymous PDF charts from committed aggregates with
`python3 tests/support/publish_benchmark_charts.py`; run
`python3 tests/unit/published_benchmark_charts_test.py` to exercise actual
export validation, private-field exclusion, plotted values and PNG rendering.
These tests do not add inference or quality measurements. See the
[README results](../README.md#latest-measured-results) for their distinct scopes.

| Directory | Executes | Does not prove |
| --- | --- | --- |
| `unit/` | Production functions with controlled inputs and checked outputs | A model answered correctly |
| `browser/` | Real browser interactions, usually with simulated engine responses | Real model inference or real downloads |
| `integration/` | Tools, files, subprocesses, HTTP, build lifecycle | Model quality unless real weights are explicitly loaded |
| `live/` | Explicit network, hardware and/or real-model runs | All models, platforms or workloads are correct |
| `support/`, `fixtures/` | Shared harnesses and controlled input data | Independent test results |

## Clean installation and real inference

```sh
make test-setup-live                  # Real GitHub downloads + builds: main, Laguna, Qwen3.8, Qwen3.6
make test-first-launch-e2e            # Headless .app + real WebKit UI + fresh network engine installation
make test-inference-live              # Real resident Metal: installed DeepSeek + Laguna
make test-inference-live ENGINES=qwen  # Requires downloaded Qwen base + PLE
make test-engine-acceptance           # Fresh builds AND real inference for all four engines
make test-qwen-chat-live              # Actual DStudio launch + Chat HTTP proxy, real Qwen
make benchmark-qwen-decode            # Native generation tok/s, three exact-output checks
```

The setup gate calls DStudio's production headless installer, using the same
archive installer and runtime builders as app setup. It starts with no engine
directory, downloads pinned source archives over HTTPS, builds the executables,
executes their help command, and checks that optional engines share the model
store. It does **not** simulate a browser onboarding click. Existing user
checkouts, models, preferences and running processes are not replaced or stopped.

`test-first-launch-e2e` instead relocates the signed `.app`, starts its real
binary from `/` with an empty `DS4UI_DATA_DIR`, and uses headless WebKit to
operate the first-run installation controls. `DS4UI_TEST_MODE` is **not** set.
It clicks Install, chooses the optional models, and checks real setup responses,
pinned revisions, compiled executable startup, automatic checkout selection,
shared model storage and discovery after reload. It also reverses/reapplies the
complete six-patch main runtime stack on a private source copy and requires exact
file preservation. An existing engine-port listener (or a test-owned sentinel)
must survive installation and test shutdown.

Only `/api/model/download` is intentionally refused at the browser boundary,
**after** actual optional-engine setup; weights are not downloaded and inference
is not counted as tested by this gate. Model-start attempts are also blocked.
No setup/build/catalog response is simulated. This is headless application/UI
coverage, not Finder double-click, native title-bar, or native file-picker QA.
Evidence and screenshots are retained in `tests/.artifacts/first-launch-*/`.

To follow a successful setup with real loading of every supported installed GGUF:

```sh
node tests/live/installed_models_e2e.mjs tests/.artifacts/first-launch-<successful-run> ds4/gguf
```

This heavyweight gate requires the existing inference engine to be stopped
explicitly. It uses the freshly built runtimes and links their empty task-owned
GGUF directory to existing weights, without copying or moving them. Models are
loaded **one at a time**, with 8k context, DSpark off, resident Qwen/Laguna and
SSD expert streaming for main DeepSeek/GLM models. Qwen3.8 keeps its native SSD
PLE. Each model must become ready, return exact arithmetic and JSON extraction
answers through DStudio, then answer a checked prompt through the real Chat UI,
including completed SSE and visible rendered text. No response is mocked.
Auxiliary GGUFs and unsupported models are listed separately; missing weights,
timeouts, truncated answers and failed checks are not passes. This does not
qualify all quantizations, maximum context, vision, Agent or Cowork behavior.

The inference gate starts a real `ds4-server`, waits for its live model catalog,
and checks arithmetic, structured extraction, ordering, Unicode, multi-turn
recall, longer-context lookup, code reasoning, malformed-request recovery and
actual SSE completion. Expected answers are independently checked, not supplied
by a model judge. Missing weights, a failed build, truncation and wrong answers
fail the run. No nonempty-answer-only pass criteria.
The tool check requires a real model-generated function call and correct use of
a controlled tool result; it does not claim autonomous execution of an Agent.

The Qwen Chat gate uses an isolated DStudio data directory, invokes the actual
launch API, rejects reuse of an unrelated engine, and sends the same checked
tasks through DStudio's Chat proxy. It does not simulate a browser click.

Each unique run retains requests, responses, failures, exact install receipts,
binary SHA-256, model path/size/mtime, load time and response time under
`tests/.artifacts/engine-acceptance/`. No model content is committed. Timings
describe this run, not a universal speed benchmark. The acceptance checks are
not full-logit comparisons against BF16/CPU, and do not establish general model
quality, tool-use quality, CUDA parity or exhaustive context-boundary correctness.

Qwen3.8 has experimental **Chat, Agent and Cowork** integration on macOS Metal
with the new engine pin. Its Design adapter is not implemented. Qwen3.6 now has
an experimental **Agent/Cowork host integration**, in
addition to Chat. Design and forced expert SSD streaming remain rejected
before stopping the current runtime. Only Qwen3.8 needs the SSD-backed PLE file. Qwen3.6 uses the
31.8 GB Q6_K_XL file, without PLE or expert SSD streaming. Its disk KV checkpoint
path is disabled until the fork can serialize its complete recurrent state.

For a fresh Qwen3.6 source/build check (including its primary-store dependency):

```sh
node tests/live/engine_acceptance.mjs --setup --engines main,qwen35
make test-engine-setup-unit test-qwen35-download
# Use the empty-model fresh-install path printed by the setup run:
node tests/integration/qwen35_setup_http_test.mjs path/from-setup-output/fresh-install
# Explicit, heavyweight; requires installed Qwen3.6 weights:
node tests/live/engine_acceptance.mjs --infer --engines qwen35 --via-app
```

The setup gate downloads into an empty private installation and now executes
both Qwen structured runtimes as well as the native binaries. The HTTP gate
also repeats the real CLI installation, runs Agent/Cowork `--help`, verifies
source and binary preservation, checks the declared capability and requires
the shared weights directory to remain empty. It does not run inference.
The updated fresh and repeated Qwen3.6 installation gates passed on September 7.
Qwen3.8 throughput and answer results must not be attributed to Qwen3.6.

The September 7 quality campaign added a native Qwen3.6 inference baseline:
11/12 development answer/protocol checks passed; its Python filtering answer
was incorrect (`22` instead of independently executed `16`). The original
private receipt is `engine-acceptance/run-m8zF5Z/`; this is not held-out quality
qualification. The same run revealed incorrect DeepSeek aliases in the native
model catalog, a separate defect now addressed by
[`ds4-qwen35-catalog`](../patch/ds4-qwen35-catalog/README.md). Fresh and repeated
Qwen3.6 setup apply the patch before building. Existing checkouts are not
silently rewritten at every Chat launch; run engine setup to upgrade them.

`make test-qwen35-catalog QWEN35_DIR=/path/to/source` copies the native server
source, tests the reversible patch lifecycle and compiles its actual HTTP
catalog serializer. Metadata is controlled and no weights are loaded. Real
Qwen acceptance separately verifies that `/v1/models` names Qwen correctly;
that metadata check does not replace or alter the answer checks.

## Local regression suite

### Vision encoder with SSD streaming (real Metal)

```sh
make test-vision-streaming-live
# Optional existing source/encoder locations:
make test-vision-streaming-live VISION_DS4_DIR=/path/to/ds4 VISION_ENCODER=/path/to/encoder.gguf
```

This explicit, bounded regression clones the pinned local Git source into an
ignored test directory and maps only the installed 933 MB DeepSeek Vision-Exp
encoder. It does not launch a language model, restart the app or touch downloads.
Real Metal kernels encode a synthetic image and route text/image token IDs before
and after three alternating language-weight span replacements. The unpatched
build must reproduce six mapping failures; the patched build must preserve the
exact encoder and routing outputs on all six checks. Baseline output hashes must
also match between the separate unpatched/patched builds. Patch apply/restore is
checked twice, with exact tracked-source restoration.

Logs, engine revision and the run receipt remain under
`tests/.artifacts/vision-stream-*/`. Missing hardware/weights or a failure is not
counted as a pass. This verifies the reproduced memory-mapping regression, not
full PDF comprehension, end-to-end LLM inference or BF16 reference equivalence.
The native fix takes effect only when a rebuilt engine is started; this test
cannot update an already-running process.

### Small SSD prefill batches (real Metal, 128k context)

```sh
make test-ssd-prefill-batch-live
# Optional existing checkout (default installed Vision-Exp weights under gguf/):
make test-ssd-prefill-batch-live SSD_TEST_DS4_DIR=/path/to/ds4
# Or specify existing GGUF and encoder explicitly:
node tests/live/ssd_prefill_batch_test.mjs /path/to/ds4 /path/to/model.gguf /path/to/encoder.gguf
```

This sequential test reproduces the DStudio GLM/M2 port's DeepSeek regression:
the old port rejects a batch with more than eight distinct experts, although
each token selects only six. It builds clean, task-owned engine sources and
checks that the production migration fixes both the real Metal kernel and the
real first transformer layer. Nothing restarts the app, copies/downloads weights
or changes user preferences. Each layer process has a 60-second deadline and
loads layer 0 only, with 131072 context, SSD on and a 256-expert cache (about
4.17 GiB planned GPU allocation on the tested Flash model).

- Four synthetic GPU batches check every result against an **exact CPU oracle**:
  12, 30 and 384 distinct experts, plus GLM's eight experts per token. Invalid
  per-token counts 0/9 and resource count 385 remain rejected.
- Six real-weight cases (1, 2, 139, 760, 761 and 1024 tokens) check **every output
  float bit-for-bit against upstream without the GLM/M2 port**. This covers the
  reported 139-token failure and both sides of the selected-address boundary.
- The legacy binaries must reproduce the specific kernel/layer failures; a
  timeout or missing dependency cannot satisfy the negative test. Existing
  GLM top-8/cache numerical tests run as well.

Logs, patch/engine identity, raw layer outputs and hashes are retained in
ignored `tests/.artifacts/ssd-prefill-*/`. This is a numerical layer regression,
**not** full-model inference, PDF comprehension, a 128k-token input test, BF16
equivalence or a speed benchmark. The context allocation is 128k; each tested
input contains at most 1024 tokens.

`make test-glm53-m2max-patch` also exercises fresh apply/restore and installed
legacy migration, with dry checks, partial-state refusal and unrelated-edit
preservation. Native readiness and the PLD builder tests verify that Metal-only
source changes invalidate old binaries. Frontend behavior tests verify that a
generic prefill failure is no longer mislabeled as an out-of-memory diagnosis.

### Model-free checks

For an explicit real-embedding benchmark of every PDF in a supplied directory,
see [PDF library benchmark](../docs/PDF_LIBRARY_BENCHMARK.md). It retains private
source-grounded questions, per-file cold/warm times, retrieval/evidence failures
and matplotlib charts in ignored artifacts. This tests retrieval, not answers
from a generative model.

For complete PDF reading, run `make test-pdf-complete` (Poppler and Playwright
WebKit required). This executes the native reader on actual PDF fixtures and
compares all text, page by page, to independent Poppler extraction. It covers
uneven page lengths, warm/changed inputs, sparse/scanned/oversized fallbacks,
native images, mixed attachment preparation and full Chat upload with actual
source highlights. Only the browser's engine response is simulated; no model
is loaded. Reports/screenshots are kept in ignored `tests/.artifacts/pdf-complete-*`.
See [the reading design and limits](../docs/PDF_READING.md).

For PDF evidence, run `make test-pdf-evidence`, also with
`DSTUDIO_TEST_BROWSER=webkit`. This uses real synthetic PDFs, the native HTTP
endpoint and Poppler to verify passage matching, render geometry and clickable
links. Repeated labels retain all distinct passages in a modal chooser; no
unattached source or ambiguous calculation is silently accepted. Choosing a
different page aborts the old request and a delayed reply cannot replace the
current image/highlights. Invalid/truncated metadata is hidden with an honest
warning. The full-app attachment browser test separately verifies streaming and
chat reload with repeated labels (simulated model answer, no inference).

`make check-fast` runs local functional/browser/integration tests without a
large language model. `make test-cowork`, `make test-frontend-unit`,
`make test-pdf-evidence` and `make test-pld` select narrower suites. Stateful test
doubles remain useful for failure/recovery coverage but are labeled as such.
`make test-video-open-weight` is separate: it compiles and runs a small real
Metal attention-equivalence probe and requires the installed H3 checkout.

Settings and model-switch regressions can also run in WebKit (the browser engine
used by the macOS app):

```sh
node tests/browser/ui_settings_redesign_playwright_test.mjs
DSTUDIO_TEST_BROWSER=webkit node tests/browser/ui_settings_redesign_playwright_test.mjs
```

Both require the corresponding Playwright browser installed. They click Done
with invalid fields in hidden panes and exercise Qwen/DeepSeek/GLM selection,
SSD preferences, cancellation, duplicate clicks, delayed preparation, stale
readiness, progress and launch errors. Background download progress and paused
state remain in Settings while the composer retains the model name, including
after a reload; these display-only interactions must issue no engine/download
mutations. Filtering/refreshing the model list also preserves the selected
download target; the confirmed request must name that exact model, not the first
option in the refreshed list. Loading visibility is hit-tested in the browser. The launcher
responses are simulated; no weights are loaded and these
tests do not measure inference. Screenshots go under `tests/.artifacts/`.

The composer model picker has its own browser regression:

```sh
node tests/browser/ui_model_picker_playwright_test.mjs
DSTUDIO_TEST_BROWSER=webkit node tests/browser/ui_model_picker_playwright_test.mjs
```

It searches installed models by name/quantization, excludes manual engine-branch
choices and support files, checks keyboard navigation and viewport fit in both
themes, and verifies that selecting Qwen chooses its matching engine before
launching. Catalog delays/errors and background download polling must preserve
search input and focus. The launcher is simulated; this is not real inference.
Screenshots and the request receipt go under `tests/.artifacts/model-picker/`.

The old source-pattern contract files were removed. Their useful parser, LAN,
Markdown and browser checks were retained as behavioral tests. Build-failure
injection tests cover cleanup and source preservation, not compiler correctness;
real first-run builds provide that separate evidence.

Original Design packs and native agent quality have separate checks:
`make test-design-originals` exercises the actual catalog and rendered components;
`tests/live/design_originals_comparison.mjs` launches the real native agent.
`tests/support/design_comparison_audit.mjs` operates completed generated pages;
`tests/support/design_comparison_report.mjs` summarizes two full audits, retaining
failures and rejecting mismatched models, prompts, settings or audit revisions.
`make test-design-comparison-report` tests this accounting with synthetic
receipts; it is not an inference or aesthetic-quality test.
`node tests/live/design_generation_limit_test.mjs` explicitly loads real DS4
weights and deliberately limits each round to one token. It checks three bounded
continuations, an honest incomplete status, return to the input loop and prior
file preservation. This is runtime fault injection, not a useful-answer or model
quality test. Run it sequentially after other model tests. An optional captured
comparison directory selects its frozen executable/source to reproduce the old
failure; that failing receipt is retained rather than counted as passing.
`make test-design-tool-recovery` executes the native loop with deliberately
truncated simulated model frames and verifies exact file preservation/retry
results. `make test-design-archive-build` compiles a real local source archive
without engine Git metadata and checks rebuilds after source changes. Neither
test loads a model; the archive test does not download its fixture.
See [Design systems and regression coverage](../docs/DESIGN_SYSTEMS.md).
Successful previews and prompt-string checks are not counted as model quality.

Actual-product comparison helpers live under `tests/support/product_*`;
`tests/live/product_quality_pilot.mjs --run` requires the pinned OpenWork and
OpenDesign checkouts and their installed runtime tools. It starts the actual
servers, not a generic standalone OpenCode task, and routes their inference to
one shared local model. Run it sequentially. Independent code/document and
Chromium workflow audits reopen the saved outputs. The artifact-serving and
semantic grader regressions are model-free:

```sh
node tests/unit/product_artifact_server_test.mjs
node tests/unit/product_design_grader_test.mjs
```

`make test-search-evidence` also covers whole-loop work budgets and actual HTTP
stream cancellation. Its model/page responses are simulated. In contrast,
`node tests/live/research_pipeline_benchmark.mjs --run` loads real weights and
executes full Search/Research against public websites, including the final
answer. Each invocation creates a fresh ignored receipt directory and preserves
failures. Review every answer against the fixed primary-source expectations
before publishing a quality score; completion and citations alone are not a pass.

Focused regressions exercise URL-heavy page excerpts, overlapping windows,
Unicode offsets, preserved original language/length constraints, evidence-ID
selection and cancellation during synthesis. They run production functions
with synthetic source/model data, not real-model quality measurements.
`research_answer_review_test.mjs` exercises omitted counter-evidence, bounded
writer/reviewer corrections, actual word counts (including exclusive bounds),
malformed/unknown review IDs, transport failure and cancellation. Additional
regressions preserve valid multi-envelope JSON comparisons and correct an
invented evidence gap without deleting the general rule that answers it.
The distinct
`research_reply_delivery_test.mjs` executes the production assistant-reply
lifecycle: exact reviewed text reaches the rendered and persisted message,
failed review stays incomplete, and ordinary Chat/Search still stream normally.
These are simulated model/UI dependencies, not a desktop WebKit E2E run.

For a new measured replay, `--before COMMIT` records the intended historical
runtime; `--variants after` keeps all four questions without re-running that
historical version. The live runner follows each runtime's actual final-answer
handoff, including direct delivery of reviewed reports in the new version.
Independent answer review remains mandatory; the model review is not the grader.

The complete [pipeline report](../extension/search/bench/PIPELINE.md) separates
live answer reviews from those deterministic checks. Its publisher requires
every question and a review bound to each unchanged raw row. After review,
`node tests/support/check_research_quality.mjs REFERENCE_JSON CURRENT_JSON`
rejects any lost previously demonstrated requirement, even if the aggregate
score or latency improves. The current receipt must contain all four after
cases; a partial retry cannot satisfy this gate.

`make test-product-comparison-publication` also runs a real headless Chromium
layout regression on the untouched published website. It detects wrapped radio
labels entering the indicator column at 390/1440 px, tests an independent good
layout, and verifies the original artifact hash. A passing regression detector
does not mean the archived design passed: its visual defects remain visible.

See the [real-run report](../docs/ENGINE_ACCEPTANCE.md) for actual failures as well
as successes. Qwen native generation throughput is reported separately from
DStudio Chat latency and from the small cross-engine acceptance battery.
