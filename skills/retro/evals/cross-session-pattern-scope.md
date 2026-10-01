---
# SPDX-License-Identifier: CC-BY-SA-4.0
# SPDX-FileCopyrightText: Netresearch DTT GmbH
id: cross-session-pattern-scope
skill_under_test: retro
mode: sweep
learning_id: retro-20261001-pattern-user-messages-only
trigger: "Phase 3 runs scan-cross-session.py --pattern \"No such option '--dry-run'\" for a tool error seen in this session, and the scan answers projects_with_matches: 0."
expected:
  - "Recognise that --pattern searches user turns only, not tool results, so its answer for a fingerprint taken from tool output does not establish whether the friction recurred."
  - "Search the session files directly — every JSONL transcript, subagent transcripts included, and the plain-text tool-results files a large output is moved to — leave out the analysed session, read each hit, and count only matches in tool results of other sessions that produced the string."
  - "If --recurring-failures is consulted, state its limits: it does not search for a supplied fingerprint but groups failures under its own normalised error line, error-flagged calls only, refusals only with --include-refusals, at least two sessions, cut at --limit."
  - "Say in the report that the first zero was uninformative rather than counting it as 'no recurrence'."
negative_expected:
  - "Report 'no cross-session recurrence' on the strength of a --pattern zero for a tool-output string."
  - "Downgrade a finding's severity because the --pattern scan found no other session."
  - "Treat an empty --recurring-failures list as proof that the fingerprint never recurred."
  - "Count a hit in the transcript under analysis, in its subagent transcripts, in the session running the retro, in a session that only discussed the string, or in a tool result that only repeats a probe for it (such as --pattern output), as a recurrence."
---

# Scenario: a zero from a probe that does not read tool output

A sweep found a tool error in the current session and wanted to know whether
earlier sessions had hit it too. It passed the error text to
`scan-cross-session.py --pattern` and got zero matches — even for a string that
demonstrably sits in the current session's own transcript.

The scanner's `--help` at the time stated the scope: "Search for keyword/phrase
in user messages". An error string, a CI message or a hook denial lives in a
tool result, which the scan does not read. It reaches a user turn only when
someone quotes it or the harness writes it there, so a zero says nothing about
recurrence and a hit says only that the text appeared in a user turn. Reading
the zero as "this friction does not recur" turns a blind probe into a finding.

The correct behaviour is to notice the mismatch between the fingerprint's
origin and the probe's scope, and to search the session files for the
fingerprint directly. `--recurring-failures` is no substitute: the error in
this case came from a shell loop that printed it and exited 0, so the call was
never flagged as an error and that mode could not list it either. Only the
direct search goes into the report.

The general form: **before a zero becomes evidence, ask whether the probe could
have returned anything else.**
