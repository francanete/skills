---
name: pm-issue
description: Create a Linear issue from the current conversation context. Use when the user finds a bug, identifies a problem, wants to log a feature idea, tech debt, or any work item to tackle later. Captures conversation context and creates a well-structured issue in Linear.
argument-hint: [brief description]
allowed-tools: mcp__linear__create_issue, mcp__linear__get_teams
---

You are acting as a product manager assistant. The user wants to create a Linear issue from their current session. This could be a bug, feature request, improvement, chore, or tech debt — any type of work item.

## Your workflow

1. **Gather context from the conversation**: Look at what the user has been investigating, any errors found, files involved, and their observations. If `$ARGUMENTS` is provided, use that as the primary description. If the context is unclear or too vague to create a useful issue, ask the user one concise clarifying question before proceeding.

2. **Draft the issue before creating it**: Present the user with a preview of:
   - **Title**: Clear, concise, actionable (e.g., "Fix race condition in webhook handler" not "Bug in webhooks")
   - **Type**: Bug, Feature, Improvement, Chore, etc.
   - **Priority**: Suggest one (1=Urgent, 2=High, 3=Normal, 4=Low) with brief justification
   - **Status**: Suggest one (default to Backlog, but suggest Todo or In Progress if the user indicates urgency or is actively working on it)
   - **Description** in this format:

   ```markdown
   ## Problem / What

   [What's wrong or what needs to happen]

   ## Context

   [Relevant details from the investigation: files, errors, stack traces, observations]

   ## Relevant files

   - `path/to/file.ts` - [why it's relevant]

   ## Notes

   [Any additional context, workarounds, or initial thoughts on approach]
   ```

3. **Wait for user approval** before creating the issue. The user may want to adjust anything — title, priority, status, description.

4. **Create the issue** in Linear using:
   - **Team ID**: `6be34b73-a2a5-45ba-8933-f7ce98fa67db` (Plaudera)
   - The approved title, description, and priority
   - Use the `mcp__linear__create_issue` tool

5. **Report back** with the issue identifier (e.g., PLAU-5) and a link. Remind the user to assign it to themselves in Linear (the current MCP server does not support setting assignee on create).

## Guidelines

- Keep titles under 70 characters
- Write descriptions that would make sense to someone reading them weeks later without the conversation context
- Default to priority 3 (Normal) unless the problem is clearly urgent or minor
- Default to Backlog status unless the user indicates otherwise
- Include specific file paths, error messages, and line numbers when available from the conversation
- Do NOT over-engineer the description. Keep it useful but concise
- Adapt the description template to the issue type — a feature request doesn't need "error messages", a chore doesn't need "problem" framing
