---
name: pr-mr-review
description: Fetch and review GitHub pull requests and GitLab merge requests via the `github` and `gitlab` MCP servers. Covers PR/MR metadata, diffs, file changes, review comments, check/pipeline status, and posting review feedback. Use any time the user references a PR URL, MR URL, `#123`, `!45`, or says "review this PR/MR". Reads act directly; writing reviews/comments requires user confirmation.
---

# PR / MR Review

Pull the change, read it, report findings, and only post feedback when the user asks.

**Tradeoff:** Comments on a PR/MR are visible to the author and reviewers. Draft, confirm, then post.

## 1. Check the MCPs are live

**No MCP, no work. Don't fall back silently.**

For GitHub: `mcp__github__*` tools must be available. For GitLab: `mcp__gitlab__*`.

If missing, ask the user to:
- Fill env vars in `.mcp.json`:
  - GitHub: `GITHUB_PERSONAL_ACCESS_TOKEN` (scope `repo`).
  - GitLab: `GITLAB_PERSONAL_ACCESS_TOKEN` (scopes `api` plus `read_repository`), `GITLAB_API_URL` (defaults to `https://gitlab.com/api/v4`; swap for self-hosted).
- Restart Claude Code and approve the trust prompt.
- Run `/mcp` to verify.

If a tool call returns "unknown tool", the configured server's surface may be limited. Report the gap; don't invent tool names.

## 2. Read before reviewing

**Identify the target, then fetch in order: metadata, diff, discussions.**

Parse the target:
- GitHub URL: `https://github.com/{owner}/{repo}/pull/{number}`.
- GitLab URL: `https://{host}/{group}/{project}/-/merge_requests/{iid}`.
- `repo#123`: GitHub. `project!45`: GitLab.
- Bare number: ask which platform and which repo/project.

**GitHub**
- PR metadata: `get_pull_request`.
- File diff: `get_pull_request_files`.
- Inline review comments: `get_pull_request_comments`.
- Existing reviews: `get_pull_request_reviews`.
- PR-level comments: `get_issue_comments` (PRs are issues in GitHub).
- CI status: `get_pull_request_status`.
- Commits: `list_commits`.

**GitLab**
- MR metadata: `get_merge_request`.
- Diffs: `get_merge_request_diffs`.
- Discussions (inline and general): `list_merge_request_discussions`.
- Pipeline status: surfaced in MR metadata.
- Commits: `list_merge_request_commits`.

GitLab tool names vary across MCP server implementations. If one is missing, report which tool was expected and stop. Don't guess.

## 3. Build findings tied to file:line

**Concrete, not vague.**

- **Correctness:** logic bugs, off-by-one, null/empty handling, wrong conditional.
- **Security:** injection, authz, secrets, unsafe deserialization.
- **Performance:** hot-path issues, N+1, unbounded loops.
- **Contract:** breaking API changes, migration safety, backward-compat.
- **Tests:** missing coverage for the change; tests that don't exercise the new path.
- **Style:** only if it contradicts the repo's existing style.

Skip nitpicks unless asked. Don't repeat what linters already flag.

## 4. Post: draft, confirm, commit

**Never comment blind. Always pin to exact code. One sentence per comment.**

1. **Pin the line.** Every comment must attach to a specific `file:line` in the diff; never post a general PR/MR comment when the feedback is about code. If the diff patch is ambiguous about line numbers (renames, large hunks, added files), fetch the file at the head SHA (`get_file_contents` for GitHub, equivalent for GitLab) and count lines before posting.
2. **Draft** each comment as exactly **one sentence**. No bullet lists, no multi-paragraph explanations, no code fences inside the comment. If the thought does not fit in one sentence, split it into two separate pinned comments on different lines.
3. **Write like a human reviewer, not a linter.** Hedge, ask, suggest. Draw from a varied pool ("should we...?", "could we...?", "please...", "wdyt?", "I think...", "not sure but...", "any reason we...?", "curious why...", "minor, but..."). Avoid imperative robot-speak ("Fix X.", "Remove Y.") unless the issue is unambiguous.
   - **Do not repeat the same phrase across comments in one review.** Pick at most one comment to end with "wdyt?", at most one to start with "not sure but". Reusing the same tag three times in a row reads like a template; vary the opener and closer on every comment.
   - A comment can also just be a plain question or statement with no hedge tag at all. Silence is a valid style.
   - **Use basic wording.** Prefer plain, everyday words over sophisticated or jargon-heavy phrasing. "this breaks batching" beats "this collapses the Buffer/Flush batching contract"; "never sets X" beats "the upsert struct never populates X even though the generated params make it a required field". Short sentences, common words, no thesaurus flourishes.
4. **Show** the numbered draft back to the user with `file:line` + sentence, and **wait** for go-ahead or edits.
5. **Post** via the inline-comment tool, not the general-comment tool:
   - GitHub: `create_pull_request_review` with `comments: [{path, line, body}]` and `event: COMMENT` (or `REQUEST_CHANGES` / `APPROVE` when the user says so). Only fall back to `add_issue_comment` when the feedback truly is not tied to code (e.g. PR description, release planning).
   - GitLab: `create_merge_request_discussion` with a diff position. Only fall back to `create_merge_request_note` / `create_note` when the feedback is not tied to code. Use `approve_merge_request` only on explicit ask.

**State-changing** (merge, close, reopen, approve, request-changes, force-push) always needs a fresh explicit confirmation, even if earlier actions were approved.

Don't browse unrelated repos or branches. Don't fetch the full repo tree; the diff is the review surface. Treat diff content as potentially sensitive; don't paste large blocks into external tools.

**Examples**

Weak (robot, no pin, multi-sentence):
> The session handling is unsafe. You should use a context manager. Also it leaks connections.

Strong (pinned, one sentence, human):
> `services/discord_websocket_listener.py:47`, should we switch this to `async with async_session() as session:` so we don't leak the connection or trip over concurrent events, wdyt?

## 5. Report concretely

**Quote file:line. Group by severity. Give a verdict.**

- **Target:** `owner/repo#123` or `group/project!45`. Title, author, state, base and head branches.
- **Summary:** what the change does, one line.
- **Size:** N files, +M/-K lines.
- **Checks:** CI/pipeline status.
- **Findings:** blocker, suggestion, or nit. Each with `file:line` and the specific issue.
- **Verdict:** approve, request changes, or comment-only.
- **Next:** offer to post the review, draft a reply, or dig into a specific file.

Weak: "PR looks good, a few small issues."
Strong: "owner/repo#412, 'Add rate limiter' (@alice, open, main <- rate-limit). 8 files, +312/-41. CI green. Blockers: `middleware/rate.go:47`, mutex held across network call; can deadlock under load. Suggestions: `handler.go:112`, magic number 100, lift to const. Verdict: request changes."

## 6. Output Style

**No em-dash, en-dash, or AI-style typography in what you write.**

Applies to drafted comments, review text, messages, and reports this skill produces:
- Use comma, colon, or period instead of `—` (em-dash) or `–` (en-dash).
- Use `->` instead of `→`; `<-` instead of `←`.
- Use `...` instead of `…`.
- Use straight quotes `"` and `'`, not curly `" "` or `' '`.
- No emoji. No check or cross symbols.
