---
name: verify-before-commit
description: Use when the user asks you to verify, check or review the change you just made with Veridu, or before committing or opening a pull request when the user has asked for Veridu checks. Sends the change to Veridu's verify_change tool, then fixes the findings that hold up and checks again.
---

# Verify a change with Veridu before committing

Veridu gives an independent review of the change you just made. Two reviewers from different vendors check it, findings that cannot be grounded in the code are dropped, and only problems both reviewers confirm can block. You send the change; you get back an outcome and concrete findings to act on.

Veridu sees only what you send. It does not read the rest of the repository, so a "pass" means "nothing found in what was sent", not "safe to merge".

## 1. Build the payload

1. Make sure new files are included in the diff: `git add -N <new files>` (intent to add; nothing is staged).
2. Get the diff against the base the change starts from, without rename detection:
   - uncommitted work: `git diff --no-renames HEAD`
   - a branch: `git diff --no-renames $(git merge-base HEAD main)`
3. `files`: the full current content of every file the diff adds or modifies, keyed by repo-relative path. You may add a few closely related unchanged files (a caller, an interface, a subclass) so the reviewers can judge the change in context.

Limits: text files only (no binaries, renames, mode changes, symlinks or submodules); diff up to 256 KB and 60 files; files up to 512 KB in total. If the change is larger, verify it in parts.

The first time in a session, tell the user which files will be sent to Veridu.

## 2. Call verify_change

Call `verify_change` with `diff`, `files` and a one- or two-sentence `intent` describing what the change is meant to do.

If the result says the review is still running, wait about 20 seconds and call `get_change` with the `change_id`. Do not call `verify_change` again for the same change.

## 3. Act on the result

- **pass**: nothing was found in what you sent. Say so, with that limit.
- **review** or **block**: read each finding against the code before changing anything. Findings marked as agreed by both reviewers are the strongest; a disputed finding means the reviewers disagreed, so check it yourself. Fix the ones that hold up, and tell the user about any you decided not to fix and why.

## 4. Check the fix

Call `verify_change` again with the new diff and `parent_change` set to the previous `change_id`. Up to two follow-ups per change are free.

Every finding has a `finding_id` that stays the same while the problem persists. A follow-up returns `followup`: which ids persist, which are new (possibly introduced by your fix), and which are no longer reported, with `code_changed` saying whether the flagged line changed. "No longer reported" is not proof of a fix: say what you changed and what the reviewers now report.

## Account and cost

The first call opens the browser to sign in to Veridu (GitHub, Google or an emailed code). Free accounts include 20 reviews a month; each new change uses one, and an identical resend returns the earlier result free. See https://veridu.io/pricing.
