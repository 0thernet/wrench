---
title: x-web posts.publish confirm dispatch
description: Confirm can dispatch x-web posts.publish against the matching kernel contract, and adapter sync cannot keep a kernel-invalid manifest.
type: plan
area: x-web
status: completed
repository_scopes:
  - src/providers/x-web.ts
  - src/providers/x-web-runtime.ts
  - src/scripts/sync-bundled-adapters.ts
tags:
  - x-web
  - adapters
---

# x-web posts.publish confirm dispatch

## Outcome

`x-web` `posts.publish` confirm can cross the dispatch boundary against the matching installed contract, using fixtures. Bundled adapter sync replaces a kernel-invalid installed manifest instead of leaving it in place.

## Context

Two production confirms on 2026-08-21 failed before dispatch (`dispatchStarted: false`, `finalOrigin: null`) with:

`authenticated web API operation failed before the dispatch boundary; reason: X post dispatch failed before a response-bound result was verified`

Preview had already planned `x-web` 1.7.0 / `posts.publish@2`. The same box had earlier hosted a 1.10.0 data manifest on the 0.10.1 CLI, which made capabilities invalid until `adapter install --force` restored the bundled 1.7.0 schema.

The version-skew hypothesis explains why preview was impossible before that force-install. It does not explain the confirm failure after 1.7.0 was restored. Confirm still died inside `executePublish` after viewer binding and before `beforeRequest`, which is descriptor resolution and mutation authorization, not an X write.

`resolveUniqueXWebBundleDescriptor` treats a changed CreateTweet query ID as drift. The 0.10.1 / current-main evidence still named `hIL9XdleMYEtVXOZVbr8Bg`. The current first-party bundle uses `WXTdKnLddrQOunD6MhWi3g` in `main.7792f4fa.js`, observed 2026-08-20. Preview never resolves that live descriptor, so it can plan while confirm fails in about two seconds.

A separate installer hole remains: `syncBundledAdapters` preserved any installed snapshot the current kernel could not validate, including a future `x-web` 1.10.0 or `linkedin-web` 1.16.0 data manifest left by a newer checkout. [[notes/repository-seams|Repository seams]] keep that kernel contract in this package; a sibling checkout cannot own the installed schema.

## Scope

### In scope

- Refresh the reviewed CreateTweet query ID and source chunk.
- Return the exact pre-dispatch failure reason from `executePublish`.
- Replace kernel-invalid installed manifests with the bundled contract during adapter sync.

### Non-goals

- Live X posting.
- Video / `posts.publish@4` work from `codex/x-web-video-publish`.
- Globally linking a dirty checkout over the stable binary.

## Constraints and decisions

- Keep bounded-provider rules: no raw HTTP client, composer clicking, or cookie scraping.
- Do not print or commit cookies, tokens, or live session material.
- Query IDs stay revision evidence. Dispatch still resolves the current bundle and rejects drift instead of adopting a new ID silently.
- Fix the installer pattern for every bundled adapter, not only X.

## Plan

1. Record the current CreateTweet evidence and fixture the matching confirm path.
2. Surface query-ID drift and other pre-dispatch causes in the failed receipt.
3. Change bundled adapter sync so an installed manifest that fails kernel validation is replaced, while a still-valid user-edited manifest remains preserved.

## Verification

- Fixture confirm of `posts.publish@3` starts and verifies one CreateTweet dispatch.
- A stale CreateTweet query ID fails before dispatch with `query-ID drift` and no POST.
- Adapter sync replaces future `x-web` and `linkedin-web` contracts and keeps a valid user-edited official `x` manifest.
- `bun run check`.

## Risks and recovery

- X can rotate CreateTweet again. The receipt must name query-ID drift so the next refresh is obvious.
- Replacing an invalid installed manifest drops a newer checkout's data schema. That schema was already unusable on the current kernel.

## Execution evidence

- 2026-08-21 — Fixture confirm of `posts.publish@3` starts and verifies CreateTweet. A stale query ID fails before dispatch with `query-ID drift` and no POST (`src/providers/x-web-runtime.internal.test.ts`).
- 2026-08-21 — Adapter sync replaces future `x-web` `posts.publish@4` and `linkedin-web` `articles.draft.save@8` data manifests, and still preserves a valid user-edited official `x` 9.9.9 manifest (`src/scripts/sync-bundled-adapters.test.ts`).

## Result

Confirm died after viewer binding because the reviewed CreateTweet query ID had drifted. Version skew made preview impossible until 1.7.0 was force-installed, but it was not the confirm failure. The kernel now records `WXTdKnLddrQOunD6MhWi3g` and returns that drift in the pre-dispatch receipt. Bundled adapter sync replaces any installed manifest the current kernel cannot execute.

## Durable memory

Query-ID drift stays an exact reviewed-evidence failure in `src/providers/x-web.ts`. Kernel-owned adapter replacement lives in `src/scripts/sync-bundled-adapters.ts`. No extra maintained note was added; the executable contracts own the rule.
