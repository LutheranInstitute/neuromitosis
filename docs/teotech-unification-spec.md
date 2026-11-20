# Neuromitosis + TEOTECH Unification Specification

Status: Proposed  
Destination repository: `LutheranInstitute/neuromitosis`  
Source product: `LutheranInstitute/teotech-theme`

## 1. Goal

Unify Neuromitosis and TEOTECH into one coherent product while preserving their strongest identities:

- **Neuromitosis is the engine.** It owns graphs, MCP discovery, workflow execution, portable `.disc` skills, provider abstraction, memory, security, and agent orchestration.
- **TEOTECH is the learning experience.** It owns the visual language, accessible course layouts, local WebLLM tutor, WebGPU atmosphere, Jekyll compatibility, and 9:16 mobile-first presentation.
- The combined product is an **agentic course platform**: course material can teach, call local tools, guide a learner, and package repeatable learning workflows as `.disc` skills.

TEOTECH must not become a second orchestration engine, and Neuromitosis must not absorb Jekyll publishing concerns into its Rust core.

## 2. Product Architecture

```text
TEOTECH course experience
  - Jekyll layouts/includes and teaching content
  - design tokens and accessible UI
  - WebLLM tutor and local-only mode
  - WebGPU ink background with graceful fallback
                |
                | browser contract
                v
Neuromitosis web/MCP bridge
  - capability discovery
  - typed tool and workflow calls
  - execution status and approval requests
  - learner/session context with explicit consent
                |
                | local runtime / optional remote server
                v
Neuromitosis Rust workspace
  - Canvas -> Loom -> Molt
  - Codec -> .disc
  - Providers, Memory, Tools, Channels, Security, MCP, Store
```

The integration boundary is a versioned browser contract, not direct coupling from TEOTECH JavaScript to Rust internals.

## 3. Repository Shape

Add TEOTECH as a first-party package rather than mixing its Jekyll files into the Rust crates:

```text
neuromitosis/
  crates/
  packages/
    desktop/
    teotech/
      README.md
      package.json
      _config.yml
      _includes/
      _layouts/
      assets/
        css/
        js/
      src/
        contract.ts
        bridge.ts
        tutor.ts
        fallback.ts
      themes/
        jekyll/
      examples/
        course/
  discs/
    courses/
  docs/
    teotech-unification-spec.md
```

`packages/teotech` is independently testable and consumable as a remote Jekyll theme. It may depend on public Neuromitosis browser/MCP contracts, but Rust crates must not depend on TEOTECH assets.

## 4. Source-to-Destination Migration

| `teotech-theme` source | Unified destination | Treatment |
| --- | --- | --- |
| `_includes/*` | `packages/teotech/_includes/` | Preserve initially; parameterize later |
| `_layouts/*` | `packages/teotech/_layouts/` | Preserve compatibility |
| `assets/css/teotech.css` | `packages/teotech/assets/css/` | Keep as public theme surface |
| `assets/js/ink.js` | `packages/teotech/src/ink.ts` | Port to typed module; retain WebGPU fallback |
| `assets/js/tutor.js` | `packages/teotech/src/tutor.ts` | Replace direct model assumptions with provider/bridge adapter |
| `assets/js/ui.js` | `packages/teotech/src/ui.ts` | Keep UI-only responsibilities |
| `_config.yml` | `packages/teotech/_config.yml` | Add Neuromitosis feature flags |
| `teotech-theme.gemspec` | `packages/teotech/teotech-theme.gemspec` | Preserve package identity |
| `index.md`, `demo.md`, `usar.md` | `packages/teotech/` and examples | Preserve as migrated documentation/demo |
| `LICENSE` | `packages/teotech/LICENSE` | Preserve TEOTECH license attribution |

History should be preserved where practical. Attribution must remain visible after migration.

## 5. Unified Runtime Contract

TEOTECH accesses Neuromitosis only through a small browser adapter with this conceptual shape:

```ts
interface NeuromitosisClient {
  capabilities(): Promise<Capability[]>;
  runSkill(input: RunSkillInput): Promise<RunResult>;
  chat(input: TutorInput): Promise<TutorResult>;
  stream(requestId: string, onEvent: (event: RuntimeEvent) => void): () => void;
}
```

Required event types:

- `runtime.started`
- `runtime.progress`
- `approval.required`
- `runtime.completed`
- `runtime.failed`
- `runtime.cancelled`

Every call carries a stable request identifier, schema version, and trace identifier. The browser contract must use structured errors rather than treating failures as empty or successful responses.

### Runtime modes

1. **Static mode:** the Jekyll theme and course work with no runtime. Interactive Neuromitosis controls are hidden or disabled.
2. **Local tutor mode:** the browser runs the TEOTECH WebLLM tutor locally. No learner data leaves the device.
3. **Connected mode:** the browser connects to a Neuromitosis local MCP/web bridge and can invoke approved capabilities.
4. **Embedded desktop mode:** the TEOTECH experience is rendered inside the Electron package through the same contract used by the browser.

## 6. First Unified Learning Flow

The first implementation must prove the full vertical slice without introducing unrelated features:

1. A learner opens a TEOTECH lesson.
2. The local tutor reads the rendered lesson context and can answer questions with WebLLM.
3. The learner chooses **Practice with agent**.
4. TEOTECH requests available capabilities from Neuromitosis.
5. Neuromitosis discovers and exposes an approved `.disc` learning skill.
6. The learner reviews the skill's declared tools and permissions.
7. On explicit confirmation, Loom executes the graph.
8. TEOTECH displays progress and the final result in the lesson shell.
9. Learner progress and model data remain local unless the learner explicitly enables a remote provider.

## 7. `.disc` Course Model

Add a course-oriented `.disc` profile without forking the format. A course skill should declare:

- `course.id`, `lesson.id`, and stable content revision
- learning objective and completion criteria
- optional local model/provider requirements
- tool/auth manifests, with no embedded credentials
- graph inputs, expected outputs, and schema version
- permissions and data-retention policy
- accessibility metadata where relevant

Example package location:

```text
discs/courses/reflection-v1.disc
```

The generic codec remains the sole owner of `.disc` parsing, validation, checksums, and compatibility behavior.

## 8. Security and Privacy Invariants

The merge is invalid if it violates any of these rules:

1. Local tutor mode makes no network request after its declared model/assets are cached.
2. Secrets, tokens, passwords, and model API keys never enter course content or `.disc` files.
3. Every capability with side effects requires an explicit approval boundary.
4. Lesson text and learner responses are not sent remotely by default.
5. TEOTECH can report runtime states but cannot bypass Neuromitosis security policy.
6. WebGPU is enhancement-only; failure must fall back to a usable static theme.
7. WebLLM failure must fall back to non-AI lesson content and an actionable error state.
8. The browser contract validates untrusted inputs and protocol versions.
9. A missing or incompatible Neuromitosis runtime does not break static publishing.

## 9. Design System

Treat TEOTECH as the shared visual identity for learner-facing surfaces.

Extract stable design tokens from `teotech.css` for:

- color, surface, and contrast
- type scale and condensed headings
- spacing and 9:16 rhythm
- motion and reduced-motion behavior
- focus, error, pending, success, and approval states

Electron's workflow canvas may remain optimized for graph editing, but learner-facing panels, onboarding, tutor UI, and published content should consume the TEOTECH tokens. WebGPU ink remains decorative and must not carry required meaning.

## 10. Testing and Acceptance Criteria

### Unit tests

- Theme configuration parsing
- Browser-contract version validation
- Runtime event mapping
- Tutor provider selection and fallback
- `.disc` course-profile validation

### Integration tests

- Jekyll build succeeds with the migrated theme
- Demo course renders without a Neuromitosis runtime
- Local WebLLM mode renders a learner response without remote provider calls
- Connected mode discovers and executes a no-side-effect test skill
- Approval-required actions stop and resume only after explicit approval
- Runtime, transport, model, and validation failures render explicit states

### Accessibility and resilience

- Keyboard navigation works through lesson, tutor, and approval controls
- Reduced-motion preference disables nonessential animation
- WebGPU and WebLLM are independently optional
- Text contrast and focus states meet WCAG 2.2 AA
- The static course remains complete when JavaScript is unavailable

### Compatibility

- Existing `remote_theme: LutheranInstitute/teotech-theme` users receive a documented migration path
- The migrated package can still publish a Jekyll course
- Existing `.disc` files remain valid or receive an explicit, versioned migration
- `cargo test --workspace` and the TEOTECH package checks pass in CI

## 11. Delivery Phases

### Phase 0: Contract and baseline

- Freeze the current TEOTECH assets as the visual baseline
- Remove stale Python/Rust and Midio/Neuromitosis naming ambiguity from current architecture docs
- Define the versioned browser/MCP contract
- Add CI for Jekyll build, browser tests, and Rust workspace tests

Exit gate: static TEOTECH demo builds from `packages/teotech` and protocol fixtures pass.

### Phase 1: Package migration

- Move TEOTECH layouts, includes, CSS, JavaScript, examples, and gemspec into `packages/teotech`
- Preserve licensing and attribution
- Convert JavaScript modules to typed package modules while retaining browser output
- Document the remote-theme migration

Exit gate: migrated theme matches the baseline at defined mobile and desktop viewports.

### Phase 2: Local tutor

- Adapt the WebLLM tutor behind a provider interface
- Scope tutor input to the current lesson unless the learner opts into more context
- Add loading, cache, deletion, compatibility, and failure states
- Verify no network traffic in an already-cached local conversation

Exit gate: local tutor works offline after model installation, and the course remains usable without it.

### Phase 3: Neuromitosis bridge

- Expose the browser contract from the existing MCP architecture
- Add capability discovery, event streaming, and cancellation
- Surface `approval.required` before any side effect
- Add one course-specific `.disc` example

Exit gate: a no-side-effect course skill runs end to end from the TEOTECH lesson UI.

### Phase 4: Desktop alignment

- Render the learner experience inside the existing Electron package
- Share tokens and browser components without replacing the workflow canvas
- Add a unified navigation model: Learn, Build, Run, Review

Exit gate: the same lesson and course skill work in the browser and desktop shell.

### Phase 5: Legacy repository transition

After the unified package is released and validated:

- Mark `teotech-theme` as archived or maintenance-only
- Keep its README pointing to the new package and migration guide
- Redirect documentation and demos
- Do not delete history without separate approval

## 12. Non-Goals

- Rewriting the Rust workspace in another language
- Bundling a remote model provider as a mandatory dependency
- Moving Jekyll files into Rust crates
- Replacing the graph canvas with a course website
- Publishing or archiving either repository during this specification phase
- Shipping a skill marketplace UI before the `.disc` runtime and local flows are proven

## 13. Definition of Done

The unification is complete when a learner can publish a TEOTECH course, receive local WebLLM help, optionally connect to Neuromitosis, inspect and approve a course skill, and receive visible execution results through one consistent, accessible interface. Neuromitosis remains the trusted execution engine; TEOTECH remains the learner experience and presentation layer.
