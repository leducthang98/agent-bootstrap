---
name: atlassian
description: Work with Jira and Confluence via the `atlassian` MCP server (mcp-atlassian). Covers Jira issues, JQL search, sprints, boards, epics, versions, components, worklogs, transitions, comments, links, watchers, attachments, service desk queues, proforma forms, plus Confluence pages, CQL search, space trees, comments, labels, attachments, page history. Use any time the user mentions a ticket key, JQL, sprint, epic, worklog, transition, Confluence page, space, or CQL. Reads act directly; writes require user confirmation.
---

# Atlassian

One skill for Jira and Confluence, read and write.

**Tradeoff:** Reads are cheap: act directly. Writes are visible to the team: draft, confirm, then commit.

## 1. Check the MCP is live

**No MCP, no work. Don't fall back silently.**

If `mcp__atlassian__jira_*` or `mcp__atlassian__confluence_*` tools aren't available, stop and ask the user to:
- Fill env vars in `.mcp.json` (`JIRA_URL`, `JIRA_USERNAME`, `JIRA_API_TOKEN`, `CONFLUENCE_URL`, `CONFLUENCE_USERNAME`, `CONFLUENCE_API_TOKEN`).
- Restart Claude Code and approve the trust prompt.
- Run `/mcp` to verify.

Never guess ticket or page content from a key or title alone.

## 2. Read before acting

**Pick the narrowest tool. Fetch only what the task needs.**

**Jira: issues**
- One issue: `jira_get_issue`; dates: `jira_get_issue_dates`; SLA: `jira_get_issue_sla`; images: `jira_get_issue_images`.
- Filtered list: `jira_search` (JQL); by project: `jira_get_project_issues`.
- History: `jira_batch_get_changelogs`.
- Linked PRs/branches: `jira_get_issue_development_info` (one) or `jira_get_issues_development_info` (many).
- Watchers: `jira_get_issue_watchers`.
- Worklogs: `jira_get_worklog`.
- Proforma forms: `jira_get_issue_proforma_forms`, `jira_get_proforma_form_details`.
- Transitions available: `jira_get_transitions`.
- Attachments: `jira_download_attachments`.

**Jira: projects / agile / service desk**
- Projects: `jira_get_all_projects`; components: `jira_get_project_components`; versions: `jira_get_project_versions`.
- Boards: `jira_get_agile_boards`; board issues: `jira_get_board_issues`.
- Sprints: `jira_get_sprints_from_board`; sprint issues: `jira_get_sprint_issues`.
- Service desk: `jira_get_service_desk_for_project`, `jira_get_service_desk_queues`, `jira_get_queue_issues`.
- Fields and options: `jira_search_fields`, `jira_get_field_options`.
- Link types: `jira_get_link_types`.
- User: `jira_get_user_profile`.

**Confluence**
- One page: `confluence_get_page`; history: `confluence_get_page_history`; diff: `confluence_get_page_diff`; images: `confluence_get_page_images`; views: `confluence_get_page_views`.
- Search: `confluence_search` (CQL); users: `confluence_search_user`.
- Structure: `confluence_get_space_page_tree`, `confluence_get_page_children`.
- Comments: `confluence_get_comments`.
- Labels: `confluence_get_labels`.
- Attachments: `confluence_get_attachments`, `confluence_download_attachment`, `confluence_download_content_attachments`.

Prefer IDs over titles when both are available.

## 3. Write: draft, confirm, commit

**Never write blind.**

1. **Gather** required fields. Ask the user for anything ambiguous, don't invent values.
2. **Draft** the full payload and show it back:
   ```
   Project: ABC          Space: PLAT
   Type: Bug             Parent: "Auth v2 Design"
   Summary: ...          Title: ...
   Description: ...      Body: ...
   ```
3. **Wait** for explicit go-ahead. No "I'll go ahead and...".
4. **Commit** via the right tool:

**Jira writes**
- Create: `jira_create_issue`, `jira_batch_create_issues`, `jira_create_sprint`, `jira_create_version`, `jira_batch_create_versions`, `jira_create_issue_link`, `jira_create_remote_issue_link`.
- Update: `jira_update_issue`, `jira_update_sprint`, `jira_update_proforma_form_answers`, `jira_edit_comment`.
- Add: `jira_add_comment`, `jira_add_watcher`, `jira_add_worklog`, `jira_add_issues_to_sprint`, `jira_link_to_epic`.
- Transition: `jira_transition_issue` (check `jira_get_transitions` first).
- **Destructive:** `jira_delete_issue`, `jira_remove_issue_link`, `jira_remove_watcher`. Always a fresh confirmation.

**Confluence writes**
- Create: `confluence_create_page`.
- Update: `confluence_update_page`, `confluence_add_comment`, `confluence_reply_to_comment`, `confluence_add_label`.
- Move: `confluence_move_page`.
- Attachments: `confluence_upload_attachment`, `confluence_upload_attachments`.
- **Destructive:** `confluence_delete_page`, `confluence_delete_attachment`. Always a fresh confirmation.

## 4. Stay in scope

**Touch only what was asked.**

- Don't touch tickets or pages the user didn't name or scope via search.
- Don't chain writes ("while I'm here, I also..."). One action per approval.
- For bulk ops (`batch_create_*`), confirm the count and a sample row before firing.
- Treat ticket/page content as potentially sensitive. Don't paste large chunks into external tools.

## 5. Report concretely

**Quote fields and headings. Don't paraphrase away the detail.**

**Read:**
- `KEY: title` (status, assignee, priority), or `Page title` (space, updated, author).
- One-line summary, then relevant sections with quotes.
- Offer the natural next step.

**Write:**
- `Created KEY`, `Updated KEY`, or `Transitioned KEY: In Progress -> Done`. Quote the summary or title.
- URL returned by the API.
- Flag any field the server dropped, defaulted, or rewrote.

Weak: "ticket created."
Strong: "Created ABC-482, 'Fix login 500 on + email aliases' (Bug, P2, @thang). Linked to epic ABC-410. URL: .../browse/ABC-482."

## 6. Output Style

**No em-dash, en-dash, or AI-style typography in what you write.**

Applies to drafted tickets, pages, comments, messages, and reports this skill produces:
- Use comma, colon, or period instead of `—` (em-dash) or `–` (en-dash).
- Use `->` instead of `→`; `<-` instead of `←`.
- Use `...` instead of `…`.
- Use straight quotes `"` and `'`, not curly `" "` or `' '`.
- No emoji. No check or cross symbols.
