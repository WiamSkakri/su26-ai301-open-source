# Contribution 1: Adapter: Fireworks AI Model Provider

**Contribution Number:** 1
**Student:** Wiam Skakri
**Issue:** https://github.com/orthogonalhq/nous-core/issues/311
**Status:** Phase I — In Progress

---

## Why I Chose This Issue

I chose this issue because it's a well-scoped, self-contained feature task in an active TypeScript project (nous-core), and it comes with a clear reference implementation to model my work on. The maintainer (@atlamors) is actively mentoring multiple contributors through this same family of "provider adapter" issues, which means I can expect responsive guidance and a realistic chance of getting my work reviewed and merged within the course timeline. 

Beyond logistics, the issue matches what I want to learn. Implementing a model-provider adapter means working with a real plugin/extension architecture, an OpenAI-compatible HTTP API, and a project's testing conventions. Fireworks AI specifically exposes an OpenAI-compatible `/v1/chat/completions` API, so the maintainer has indicated it can largely reuse the shared OpenAI protocol primitive, which keeps the scope approachable while still being a meaningful end-to-end contribution.

---

## Understanding the Issue

### Problem Description
The nous-core project supports pluggable model providers via "provider leaves," but there is currently no provider leaf for Fireworks AI. As a result, users cannot use Fireworks AI as an inference backend through the standard provider surface. The task is to add a certified provider leaf for Fireworks AI that conforms to the project's current provider-adapter contract.

### Expected Behavior
A user should be able to select and use Fireworks AI as a model provider through the standard provider surface, configured as a certified provider leaf under `self/subcortex/providers/src/providers/fireworks/` (vendor directory name to be confirmed), using `ProviderDefinitionLeaf`, with the built-in provider ID derived from the `vendorKey`. Because Fireworks exposes an OpenAI-compatible API, the leaf should compose the shared OpenAI protocol (`self/subcortex/providers/src/protocols/openai-api/`) rather than hand-rolling protocol code.

### Current Behavior
No Fireworks AI provider leaf exists, so the provider is unavailable. (Historical note: the issue originally referenced an older `IModelProvider` file pattern such as `self/subcortex/providers/src/<vendor>-provider.ts` — that path is **superseded and must not be used**.)

### Affected Components
- `self/subcortex/providers/src/providers/<vendor>/` — where the new Fireworks provider leaf will live
- `self/subcortex/providers/src/protocols/openai-api/` — shared OpenAI-compatible protocol to reuse
- `self/subcortex/providers/src/providers/anthropic/` — the reference implementation for a Cloud API provider leaf
- Generated provider catalogs — **must not be hand-edited**; regenerated via the documented workflow
- Integration branch: `feat/contributor-friendly-inference-provider-surface` (confirm with maintainer before opening PR)

> **Note / open risk:** The maintainer flagged that the shared `ChatCompletionsProvider` primitive is being refactored out of `OpenAiCompatibleProvider` in issue #324 (per-provider env vars, `allowEmptyKey`, `defaultHeaders`). Picking this up may require either waiting on #324 or working around the hard-coded `OPENAI_API_KEY` env var. **Action item: confirm with @atlamors whether #311 is unblocked before starting implementation.**

---

## Reproduction Process

> This is a **feature addition**, not a bug, so "reproduction" here means confirming the provider is currently absent and that the local dev environment + test harness work, rather than reproducing a defect.

### Environment Setup
_To be completed once the repo is set up locally._
- [ ] Fork the repository
- [ ] Clone fork and check out the integration branch (`feat/contributor-friendly-inference-provider-surface`, pending maintainer confirmation)
- [ ] Install dependencies (project uses Bun — confirm exact commands from the repo's contributing/setup docs)
- [ ] Run the existing provider test suite to confirm a clean baseline
- [ ] Join the project Discord for setup help (maintainer directs contributors there)
- _Challenges faced & how I solved them:

### Steps to Reproduce (confirm provider is absent)
1. Check `self/subcortex/providers/src/providers/` for a `fireworks` directory — confirm it does not exist.
2. Attempt to select Fireworks AI as a provider through the provider surface.
3. Observed result: Fireworks AI is unavailable as a provider.

### Reproduction Evidence
- **Commit showing reproduction:** 
- **Screenshots/logs:** 
- **My findings:** 

---

## Solution Approach

### Analysis
The "root cause" is simply that no provider leaf has been authored for Fireworks AI yet. The project already has the infrastructure to support new providers (the provider-leaf contract + shared OpenAI protocol); the work is to add a correctly-structured leaf that plugs into it.

### Proposed Solution
Add a certified provider leaf for Fireworks AI under `self/subcortex/providers/src/providers/<vendor>/`, using `ProviderDefinitionLeaf`, deriving the built-in provider ID from `vendorKey`, and composing the shared OpenAI-compatible protocol since Fireworks exposes an OpenAI-compatible `/v1/chat/completions` surface. Model the structure closely on the existing **Anthropic** provider leaf. Regenerate (never hand-edit) the provider catalog via the documented workflow, and follow the provider-adapter testing checklist.

### Implementation Plan
Using the UMPIRE framework (adapted):

**Understand:** Add a Fireworks AI provider leaf so Fireworks can be used as an inference backend through nous-core's standard provider surface, conforming to the current provider-leaf contract.

**Match:** The closest existing pattern is the **Anthropic provider leaf** (`self/subcortex/providers/src/providers/anthropic/`) for overall Cloud API leaf shape. Because Fireworks is OpenAI-compatible, the **shared OpenAI protocol** (`self/subcortex/providers/src/protocols/openai-api/`) should be reused for the actual request/response handling. The four provider-adapter docs pages are the source of truth for required files, commands, and tests.

**Plan:**
1. Confirm with the maintainer that #311 is unblocked and confirm the correct target branch.
2. Read all four provider-adapter docs (quickstart, provider-leaf-anatomy, schemas-abi-reference, testing-checklist).
3. Study the Anthropic provider leaf to learn the required file structure and conventions.
4. Create the Fireworks provider leaf directory and define the `ProviderDefinitionLeaf` (vendorKey-derived ID; do **not** hand-author `wellKnownProviderId`).
5. Wire the leaf to compose the shared OpenAI-compatible protocol; supply Fireworks-specific config (base URL, auth/header handling, default model identifiers).
6. Regenerate the provider catalog using the documented workflow (no manual catalog edits).
7. Add/adjust tests per the testing checklist.
8. If I hit a protocol-level gap (e.g. custom headers, no-auth handling, streaming or tool-call differences), flag it in the issue/PR for a central fix rather than working around it locally.

**Implement:** 

**Review:** Self-review checklist —
- [ ] Follows provider-leaf contract (`ProviderDefinitionLeaf`, vendorKey-derived ID)
- [ ] Did **not** hand-author `wellKnownProviderId`
- [ ] Did **not** hand-edit generated catalogs
- [ ] Reused the OpenAI protocol instead of duplicating protocol code
- [ ] Followed the testing checklist
- [ ] PR targets the correct integration branch
- [ ] PR description links the issue with a closing keyword (e.g. `Fixes #311`)

**Evaluate:** Run the provider test suite; verify Fireworks appears and functions through the provider surface; include evidence (logs/screenshots) of manual verification in the PR.

---

## Testing Strategy

### Unit Tests
- [ ] Test case 1: Provider leaf is registered and its built-in ID is correctly derived from `vendorKey`.
- [ ] Test case 2: Leaf composes the OpenAI-compatible protocol and constructs requests to the correct Fireworks endpoint with correct auth/headers.
- [ ] Test case 3: Response parsing (and streaming, if applicable) matches expected provider-leaf behavior.

### Integration Tests
- [ ] Integration scenario 1: Selecting Fireworks via the provider surface returns a valid completion against a (mocked or real) OpenAI-compatible endpoint.
- [ ] Integration scenario 2: Generated provider catalog includes Fireworks after regeneration and passes catalog validation.

### Manual Testing


---

## Implementation Notes

### Week 1 Progress

### Week 2 Progress


### Code Changes
- **Files modified:** 
- **Key commits:** 
- **Approach decisions:** 

---

## Pull Request

**PR Link:** 

**PR Description:** 

**Maintainer Feedback:**

**Status:** Not yet submitted

---

## Learnings & Reflections

### Technical Skills Gained


### Challenges Overcome

### What I'd Do Differently Next Time


---

## Resources Used
- Provider adapter docs — quickstart: https://docs.nue.orthg.nl/docs/development/provider-adapters/quickstart
- Provider adapter docs — provider leaf anatomy: https://docs.nue.orthg.nl/docs/development/provider-adapters/provider-leaf-anatomy
- Provider adapter docs — schemas / ABI reference: https://docs.nue.orthg.nl/docs/development/provider-adapters/schemas-abi-reference
- Provider adapter docs — testing checklist: https://docs.nue.orthg.nl/docs/development/provider-adapters/testing-checklist
- Reference implementation: Anthropic provider leaf (`self/subcortex/providers/src/providers/anthropic/`)
- Shared OpenAI protocol (`self/subcortex/providers/src/protocols/openai-api/`)
- Issue thread: https://github.com/orthogonalhq/nous-core/issues/311
- Related/blocking issue #324 (ChatCompletionsProvider refactor)
- [Project Discord — add invite link]
- [Fireworks AI API docs — add link]

