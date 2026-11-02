# Specification: Enable Native Chat UI for Antigravity (AGY) on Mobile Companion (iOS / Android)

- **Date**: 2026-11-02
- **Author**: repires
- **Status**: Draft / In-Progress
- **Target Upstream Repo**: `stablyai/orca`
- **Issue**: https://github.com/stablyai/orca/issues/37299 (#37299)

---

## 1. Problem Statement & Motivation

### Background
Orca Companion (mobile app for iOS and Android) connects to an Orca host (running on macOS / Linux / Windows / Unraid) and allows developers to interact with AI agents either via a PTY terminal view or via a high-level **Native Chat UI**.

Antigravity (`antigravity` / AGY) is fully supported on the Orca desktop host with an Agent Client Protocol (ACP) dialect (`src/main/acp/acp-dialects/antigravity-dialect.ts`), launch specification (`ANTIGRAVITY_LAUNCH_SPEC`), and runtime management.

### The Bug / User-Reported Symptom
When a user on mobile Orca (iPhone) opens or launches a new Antigravity session (either via "+ New Tab" or when navigating to an active Antigravity session), the mobile app renders the raw terminal/CLI session rather than the interactive mobile Native Chat UI. In contrast, sessions using Claude, Codex, Grok, OpenCode, etc. open directly into the sleek Native Chat UI with transcript streaming, rich message bubbles, diff inspections, and prompt input controls.

### Root Cause Analysis
Mobile Orca determines whether a tab can be displayed in Native Chat via `resolveMobileNativeChat` in `mobile/src/session/native-chat/mobile-native-chat-eligibility.ts`.

1. **Native Chat Supported Agents List (`src/shared/native-chat/native-chat-agent-support.ts`)**:
   `NATIVE_CHAT_SUPPORTED_AGENT_LIST` lists agents eligible for native chat:
   ```typescript
   export const NATIVE_CHAT_SUPPORTED_AGENT_LIST: readonly TuiAgent[] = [
     'claude',
     'openclaude',
     'codex',
     'grok',
     'omp',
     'opencode',
     'opencode2',
     'zcode'
   ]
   ```
   `'antigravity'` is omitted from this allowlist. Because `isNativeChatSupportedAgent('antigravity')` returns `false`, `isNativeChatAgentAvailable('antigravity', ...)` returns `false`.
   Consequently, `resolveMobileNativeChat` returns `null`, causing `canShowMobileNativeChat` to return `false` on the mobile device.

2. **Mobile Launch Routing (`mobile/src/session/new-tab-agent-host-launch.ts` & `src/shared/structured-native-chat-launch-route.ts`)**:
   When launching a new agent tab from mobile, `resolveStructuredNativeChatSupport` in `src/shared/structured-native-chat-launch-route.ts` is consulted. While Antigravity supports structured agent sessions over ACP, its eligibility in mobile tabs requires:
   - Inclusion in the shared native chat agent list (`NATIVE_CHAT_SUPPORTED_AGENT_LIST`).
   - Appropriate handling in transcript resolution / ACP session handling if connecting to an ongoing or structured session.

---

## 2. Goals & Non-Goals

### Goals
- Enable mobile Orca users (iOS / Android) to interact with Antigravity (`antigravity`) sessions using the Native Chat UI.
- Maintain backward compatibility with terminal PTY fallback if native chat is not supported on older hosts.
- Add comprehensive test coverage in unit tests verifying `antigravity` resolves as native-chat-eligible on mobile.
- Follow upstream contribution standards (`CONTRIBUTING.md`, PR template, issue format).

### Non-Goals
- Changing the desktop ACP binary packaging or execution mechanism.
- Modifying non-Antigravity agent launch behaviors.

---

## 3. Detailed Technical Design & Changes

### 3.1. Shared Native Chat Support List
File: `src/shared/native-chat/native-chat-agent-support.ts`
- Add `'antigravity'` to `NATIVE_CHAT_SUPPORTED_AGENT_LIST`.
- Ensure type definitions (`TuiAgent`) include `'antigravity'` if not already present.

### 3.2. Mobile Native Chat Eligibility & Launch Options
Files:
- `mobile/src/session/native-chat/mobile-native-chat-eligibility.ts`
- `mobile/src/session/mobile-new-tab-agent-options.ts`
- Verify that `resolveMobileNativeChat` properly recognizes `antigravity` tabs and binds to transcripts (either structured ACP sessions or local transcript logs).

### 3.3. Test Updates
Files:
- `src/shared/native-chat/native-chat-agent-support.test.ts`
- `mobile/src/session/native-chat/mobile-native-chat-eligibility.test.ts`
- Add test assertions verifying `isNativeChatSupportedAgent('antigravity') === true`.
- Add test assertions verifying `resolveMobileNativeChat` with an `antigravity` tab produces a valid `MobileNativeChatResolution`.

---

## 4. Implementation Steps & Milestones

1. **Spec Document Creation** (`SPEC-MOBILE-AGY-CHAT.md`): Defined requirements and architecture (completed).
2. **Issue Filing**: File formal bug report on `stablyai/orca` with reproduction steps, environment details, and expected behavior.
3. **Branch Creation**: Create `fix/mobile-antigravity-chat-ui` in the fork.
4. **Code Implementation**:
   - Update `NATIVE_CHAT_SUPPORTED_AGENT_LIST`.
   - Update mobile launch and eligibility logic as needed.
   - Update unit tests.
5. **Verification**: Run unit and integration tests (`pnpm test`).
6. **PR Submission**: Open Pull Request to `stablyai/orca:main` adhering to PR template and linking the filed issue.
