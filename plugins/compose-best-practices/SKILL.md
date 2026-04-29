---
name: compose-best-practices
description: Review, design, or refactor Jetpack Compose UI code according to MAD architecture and official Compose best practices. Use when the user asks about Compose规范, Compose最佳实践, Compose重组性能, 单向数据流, 状态提升, SideEffect, collectAsStateWithLifecycle, LazyColumn key, Preview coverage, NavController解耦, or requests Compose code review and quality governance.
---

# Compose Best Practices

Use this skill to review or implement Jetpack Compose code under MAD architecture. The goal is to catch code that appears functional but hides recomposition, lifecycle, testability, preview, state, or navigation problems.

## Core Review Goal

Prioritize:

- Recomposition performance: prevent unnecessary recomposition and high-frequency state reads.
- Architecture boundaries: keep Route/ViewModel/navigation concerns out of reusable UI components.
- State correctness: make ownership, persistence, and mutation paths explicit.
- Side-effect safety: run loading, navigation, listeners, Toast, and coroutine work from the correct effect or event boundary.
- Maintainability: make UI predictable, previewable, testable, and reusable across modules.

## Reference Loading

Load only the references relevant to the current task:

- `references/state-and-architecture.md`: single source of truth for unidirectional data flow, state hoisting, `remember`, observable state collections, immutable UI state, and navigation boundaries.
- `references/side-effects-and-lifecycle.md`: side effects, complete effect keys, state writes during composition, and `collectAsStateWithLifecycle`.
- `references/performance-and-lazy-layout.md`: Lazy keys, `contentType`, nested scroll containers, expensive work, `derivedStateOf`, and high-frequency state reads.
- `references/component-api-preview-accessibility.md`: `Modifier` API, component parameters, semantics, testability, and Preview coverage.

If the user asks for a full Compose review, load all four references. If the user asks about one topic, load only that topic's reference.

## Required Output

When auditing code, report issues in this order:

1. Blocker: correctness, lifecycle, crash, infinite recomposition, or severe performance risks.
2. High: architecture coupling, unstable UI state, missing Lazy keys, bad side-effect keys.
3. Medium: preview gaps, avoidable recomposition, parameter/API design issues.
4. Low: readability, naming, or optional polish.

For each issue, include:

- Location: file/function if available.
- Problem: what rule is violated.
- Risk: why it matters in Compose.
- Fix: concrete corrected code or a direct refactor plan.

## Quick Checklist

Use this checklist before approving Compose code:

- State flows down and events flow up.
- Reusable UI does not depend on ViewModel, NavController, Repository, or Android context unless unavoidable.
- Shared state is hoisted to the correct owner.
- Configuration-sensitive local state uses `rememberSaveable`.
- Collections used as state are observable or immutable.
- Side effects are not executed from composable bodies.
- Effect keys include all values that define the work lifecycle.
- No state writes happen during composition.
- Flow collection uses `collectAsStateWithLifecycle`.
- Lazy lists use stable `key`; mixed lists use `contentType`.
- Same-direction nested scroll containers have explicit size or are flattened into one Lazy layout.
- Expensive computation is moved to ViewModel/UseCase or memoized with correct keys.
- `derivedStateOf` is used only to reduce high-frequency recomposition.
- High-frequency state reads are delayed to the smallest necessary scope.
- Public composables expose `modifier: Modifier = Modifier`.
- Component APIs accept only necessary data or immutable UI state.
- UI state does not expose mutable collections or mutable holders.
- Interactive icons/images have semantics; decorative icons/images use null descriptions.
- Navigation is emitted as events and handled by Route/navigation integration.
- Core UI has meaningful Preview coverage.

## Refactor Strategy

When fixing violations:

1. Split Route and Screen first.
2. Define immutable UI state and event callbacks.
3. Move ViewModel, Flow collection, and NavController usage into Route.
4. Hoist shared local UI state to the lowest common parent.
5. Fix side effects and keys.
6. Add Lazy keys/content types and remove unbounded nested scroll.
7. Add previews for the resulting pure UI components.
