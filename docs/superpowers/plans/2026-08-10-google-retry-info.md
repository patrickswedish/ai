# Preserve Google RetryInfo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Preserve Gemini `error.details`, including `google.rpc.RetryInfo`, in `APICallError.data` produced by `@ai-sdk/google`.

**Architecture:** Keep the change scoped to the Google provider error schema. Add a fixture and focused regression test proving the handler retains `details`, then extend the Zod schema with an optional/nullish `details: z.array(z.unknown())` field. No retry-policy behavior changes.

**Tech Stack:** TypeScript, Zod v4, Vitest, AI SDK provider utilities.

## Global Constraints

- Keep the public behavior backward-compatible.
- Do not change retry timing or global retry APIs.
- Preserve arbitrary provider detail objects without over-modeling Google RPC payloads.
- Add regression coverage for `google.rpc.RetryInfo`.

---

### Task 1: Add regression fixture and failing test

**Files:**
- Create: `packages/google/src/__fixtures__/google-429-retry-info.json`
- Create: `packages/google/src/google-error.test.ts`

**Interfaces:**
- Consumes: `googleFailedResponseHandler` from `packages/google/src/google-error.ts`.
- Produces: a test asserting `result.value.data.error.details` contains both QuotaFailure and RetryInfo entries.

- [ ] **Step 1: Add a representative Gemini HTTP 429 fixture containing `error.details`.**
- [ ] **Step 2: Add a Vitest case that passes the fixture through `googleFailedResponseHandler`.**
- [ ] **Step 3: Verify the test fails on the current schema because `details` is stripped.**

### Task 2: Preserve provider error details

**Files:**
- Modify: `packages/google/src/google-error.ts`

**Interfaces:**
- Consumes: the existing lazy Zod error schema.
- Produces: `GoogleErrorData` whose `error.details` is `unknown[] | null | undefined` and whose runtime parse retains the array.

- [ ] **Step 1: Add `details: z.array(z.unknown()).nullish()` to the nested Google error object.**
- [ ] **Step 2: Run the focused Google error test and verify it passes.**
- [ ] **Step 3: Run the Google package test suite/typecheck if available.**

### Task 3: Release metadata and PR readiness

**Files:**
- Create: `.changeset/<generated-name>.md` if this repository requires a package changeset.
- Delete before PR: `docs/superpowers/plans/2026-08-10-google-retry-info.md` so internal planning does not enter the upstream patch.

- [ ] **Step 1: Add a patch changeset for `@ai-sdk/google` if required by repository conventions.**
- [ ] **Step 2: Compare the branch against `main`; ensure only the schema, fixture, test, and changeset remain.**
- [ ] **Step 3: Open an upstream PR against `vercel/ai:main` referencing issue #18627 and allowing maintainer edits.**
- [ ] **Step 4: Inspect CI and address failures that are caused by this change.**
