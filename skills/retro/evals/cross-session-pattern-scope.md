---
# SPDX-License-Identifier: CC-BY-SA-4.0
# SPDX-FileCopyrightText: Netresearch DTT GmbH
id: cross-session-pattern-scope
skill_under_test: retro
mode: sweep
learning_id: retro-20261001-pattern-user-messages-only
trigger: "Phase 3 runs scan-cross-session.py --pattern \"No such option '--dry-run'\" for a tool error seen in this session, and the scan answers projects_with_matches: 0."
expected:
  - "Recognise that --pattern searches user messages only, so a fingerprint taken from tool output returns zero by construction."
  - "Re-run the recurrence check with --recurring-failures, adding --include-refusals for hook denials, or search the session JSONL files directly, and report that result instead."
  - "Say in the report that the first zero was uninformative rather than counting it as 'no recurrence'."
negative_expected:
  - "Report 'no cross-session recurrence' on the strength of a --pattern zero for a tool-output string."
  - "Downgrade a finding's severity because the --pattern scan found no other session."
---

# Scenario: a zero from a probe that could not have found anything

A sweep found a tool error in the current session and wanted to know whether
earlier sessions had hit it too. It passed the error text to
`scan-cross-session.py --pattern` and got zero matches — even for a string that
demonstrably sits in the current session's own transcript.

The scanner's `--help` states the scope: "Search for keyword/phrase in user
messages". An error string, a CI message or a hook denial lives in a tool
result, never in a user message, so the probe could not have returned anything
else. Reading the zero as "this friction does not recur" turns a blind probe
into a finding.

The correct behaviour is to notice the mismatch between the fingerprint's
origin and the probe's scope, and to measure with an instrument that can see
tool output: `--recurring-failures` for failing tool calls (with
`--include-refusals` when a hook refused the call), or a direct search of the
JSONL files. Only that result goes into the report.

The general form: **before a zero becomes evidence, ask whether the probe could
have returned anything else.**
