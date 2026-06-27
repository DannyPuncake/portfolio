# Requirements Analysis Case Study
### How Requirements Quality Impacts Product Outcomes

**Denys Blinov · Junior QA Engineer · 2025–2026**

---

## Overview

This case study documents three personal software projects of increasing complexity, developed using AI-assisted (vibe coding) methodology. Each project was built, tested, and refined independently on real devices (macOS, iPhone 13 Pro Max).

The primary goal was practical: building tools for personal use. The secondary outcome was a hands-on understanding of how **requirements quality directly affects product correctness, UX, and overall development effort** — a core principle of QA practice.

| Project | Platform | Requirements | Complexity | Key Finding |
|---|---|---|---|---|
| [Claude Token Tracker](#project-1-claude-token-tracker) | macOS menu bar | 3–4 sentences | 🟢 Low | Mock server instead of real API |
| [Twitch Stream Tracker](#project-2-twitch-stream-tracker) | macOS menu bar | 7 points, 1–2 sentences each | 🟡 Medium | Undocumented API auth + rate limits |
| [AMRAP.log iOS Timer](#project-3-amraplog--ios-crossfit-timer) | iOS (iPhone 13 Pro Max) | 7 sections × 3–5 sub-points (~2.5 pages A4) | 🔴 High | Concurrent timers, UX gaps, performance regression |

---

## Project 1: Claude Token Tracker

`macOS` `menu bar` `Low complexity` `3–4 sentence spec`

<!-- INSERT SCREENSHOT: macOS menu bar showing token tracker with pixel-style bars -->

### What It Does

A macOS menu bar application that displays Claude AI session and weekly token usage as pixel-style progress bars, using the claude.ai session cookie for authentication.

### Requirements Scope

Initial requirements were minimal: 3–4 sentences describing the visual output and data source. No API behaviour, no refresh interval, no error handling was specified.

### Key Findings

#### Finding 1: Unspecified API Refresh Rate Led to Rate Limit Warning

The initial implementation used a 10-second refresh interval, which immediately triggered an API rate limit warning. The minimum allowed interval was 1 minute. Final implementation was set to 3 minutes as a safe margin.

> **Root cause:** API constraints were not researched prior to writing requirements. No mention of refresh rate in the spec — it was assumed the implementation would handle it correctly.

#### Finding 2: Mock Server Created Without Explicit Requirement

The AI implementation defaulted to a mock server for API requests, meaning the tracker displayed simulated data instead of real token usage. The app appeared to work correctly but was producing no real output.

> **Root cause:** The real API endpoint was not specified in requirements. Without it, the implementation filled the gap with a mock. **A requirement must be explicit, not assumed.**

### Lessons Learned

- Even for a 3-sentence spec, technical constraints (API rate limits, authentication endpoints) must be included
- An implementation that compiles and runs is not the same as a correct implementation
- Ambiguity in requirements does not produce ambiguous output — it produces **incorrect output that looks correct**

---

## Project 2: Twitch Stream Tracker

`macOS` `menu bar` `Medium complexity` `7-point spec`

<!-- INSERT SCREENSHOT: macOS menu bar showing Twitch tracker with live channel status -->

### What It Does

A macOS status bar application that tracks up to 7 Twitch channels, displays live stream status with color-coded indicators, shows viewer counts and stream titles, and sends macOS notifications when a tracked channel goes live. Integrates with Twitch Helix API via client credentials OAuth2 flow.

### Requirements Scope

7 structured points with 1–2 sentences each. Compared to Project 1, requirements were significantly more detailed: channel slots, UI behaviour, notification logic, and stream metadata display were all specified.

### Key Findings

#### Finding 1: Third-Party API Authentication Not Anticipated

Twitch API requires OAuth2 client credentials (Client ID and Client Secret) before any data can be requested. This was not mentioned in the requirements because it was not known at requirements-writing time.

> **Root cause:** When integrating with third-party APIs, authentication requirements and developer account setup must be part of the specification, not discovered during implementation.

#### Finding 2: Rate Limit Discovered at Integration Stage

Twitch API enforces a limit of 800 requests per minute. This was discovered only after integration began. Polling interval had to be adjusted to stay within limits.

#### Finding 3: Network Permissions Blocked Outgoing Connections

In Xcode, outgoing network connections require an explicit entitlement configuration. The app built and ran without errors, but all API calls silently failed until the correct entitlement (`com.apple.security.network.client`) was added.

> **Observation:** Silent network failures are harder to detect than explicit errors. This reinforced the value of checking network logs even when the UI shows no visible error state.

### Lessons Learned

- More detailed requirements reduced functional ambiguity but created false confidence that the spec was complete
- Infrastructure requirements (auth, permissions, rate limits) are a **distinct category** that must be researched independently of feature requirements
- A working UI with broken network calls is a common failure mode — always verify data sources, not just visual output

---

## Project 3: AMRAP.log — iOS CrossFit Timer

`iOS` `iPhone 13 Pro Max` `High complexity` `~2.5 pages A4 spec`

<!-- INSERT SCREENSHOT: iOS app showing timer interface with multiple modes -->

### What It Does

A native iOS application with 5 workout timer modes (AMRAP, EMOM, For Time, Tabata, Mixed) and a workout log tab where completed sessions are recorded. Tested on a real device through daily use during actual CrossFit training sessions.

### Requirements Scope

The most detailed specification of the three projects: 7 major sections with 3–5 sub-points each, totalling approximately 2.5 pages of A4. Despite this, several significant issues were discovered during real-device testing.

### Key Findings

#### Finding 1: Concurrent Timer Execution — Functional Bug

All 5 timer modes could be started simultaneously. Each ran independently and in parallel, producing overlapping audio alerts and conflicting UI states.

> **Root cause:** The requirements defined each timer mode individually but never stated that only one mode should be active at a time. For a human developer this would be an obvious constraint. For an AI implementation without that constraint, each timer was treated as an independent, concurrent feature.

> 💡 **Key insight:** AI-assisted development requires more explicit requirements than human-led development. Constraints that are obvious by convention or UX intuition must be stated explicitly.

#### Finding 2: UX Layout Not Specified — One-Handed Usability Issue

All timer controls and settings were positioned in the upper half of the screen. During real workouts, one-handed operation of controls in the upper area is uncomfortable and error-prone.

> **Root cause:** No layout or ergonomics requirements were specified. UX constraints for the specific use context (exercise, sweaty hands, one-handed use) must be explicitly documented.

#### Finding 3: Button Size Below Minimum — Touch Target Issue

Settings buttons were smaller than expected and difficult to tap accurately during use. No minimum touch target size was specified in requirements.

> **Reference:** Apple Human Interface Guidelines recommend a minimum touch target of 44×44 points. This was not referenced in the specification.

#### Finding 4: Performance Regression in Mixed Mode

The Mixed timer mode, which contained 10 configurable parameters, exhibited noticeably slower load time compared to other modes when selected. The performance degradation was only observable on a real device, not in the simulator.

> **Root cause:** The Mixed mode requirement contained only one line describing the 10 settings. The implementation borrowed UI components from other modes to fill the gaps. The resulting view was over-complex and unoptimised. **Insufficient specification of a complex mode led to an implementation that worked functionally but degraded performance.**

### Lessons Learned

- Requirements at 2.5 pages still missed critical constraints — completeness is not measured by length
- Real-device acceptance testing revealed issues that emulators and code review would not catch (one-handed usability, button touch targets, performance feel)
- AI implementation fills specification gaps with technically valid but contextually wrong solutions. The gap between *"obvious to a human"* and *"stated in the spec"* is where most defects originate
- Each mode with significant variation in complexity should have its own dedicated requirements section, not a single-line reference

---

## Cross-Project Conclusions

### Requirements Completeness vs. Length

Across all three projects, the number of defects did not decrease proportionally with the number of requirement pages. The **type** of information mattered more than the volume. Missing constraint types — mutual exclusion, platform guidelines, API behaviour, performance expectations — caused more issues than missing feature descriptions.

### The AI Implementation Gap

Working with AI-assisted development surfaced a pattern directly relevant to QA practice: implementations are only as correct as their specifications. Where human developers apply convention, domain knowledge, and UX intuition to fill gaps, AI implementations apply technically valid defaults that may be contextually incorrect. This makes explicit, unambiguous requirements **more critical, not less**.

### Acceptance Testing on Real Devices

The most significant defects in AMRAP.log were only discovered through daily real-device use in the actual context of the application — during CrossFit workouts. Emulator-based testing or functional verification alone would not have surfaced these issues.

### QA Principles in Practice

| QA Principle | Observed in Practice |
|---|---|
| Requirements are the foundation of quality | Defects traced directly to missing or ambiguous requirements in all 3 projects |
| Test early, test in real conditions | Real-device testing on iPhone revealed UX, performance, and usability issues invisible in emulation |
| Implicit assumptions are defect risks | Concurrent timer execution and mock server bugs originated from unstated assumptions treated as obvious |
| Complexity requires proportional specification | Mixed timer mode: single-line requirement for 10-parameter UI caused functional debt and performance issues |

---

*Denys Blinov · danny.puncake@gmail.com · Junior QA Engineer*
