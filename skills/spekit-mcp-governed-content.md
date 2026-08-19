---
name: Work with governed Spekit content over MCP
description: Connect to the Spekit MCP server, read governed GTM content with citations, and create or update Speks, topics, companies and Deal Rooms within the signed-in user's own permissions.
api: mcp/spekit-mcp.yml
endpoint: https://mcp.spekit.co/mcp
operations:
  - Get authenticated user
  - List topics
  - Get topic
  - Search content
  - List topic content
  - Get content
  - Get asset content
  - Get content style guide
  - List content templates
  - Get content template
  - List companies
  - List deal rooms
  - Create topic
  - Create content
  - Create content from template
  - Update content
  - Create company
  - Create deal room
  - Create deal room content
generated: '2026-08-14'
method: generated
source: https://help.spekit.com/hc/en-us/articles/53977808268699-Spekit-MCP-Overview-Setup
---

# Work with governed Spekit content over MCP

The Spekit MCP connector is the **only** programmatic surface that can read Spek bodies or write anything. The REST API at `api.spekit.co` is read-only analytics and shares no tools with this surface — do not expect an operation here to have a REST equivalent.

## Connect

Server URL, exactly:

```
https://mcp.spekit.co/mcp
```

One URL for every client. There is **no Client ID and no Client secret** — the server supports dynamic client registration (`https://mcp.spekit.co/register`) and you authorize with an interactive OAuth 2.0 authorization-code sign-in using the user's own Spekit login. PKCE with `S256` is required. There is no machine-to-machine path; if you are running headless with no browser, the sign-in cannot complete.

Access tokens live 24 hours; refresh tokens live 30 days and rotate on every use. On a `401 invalid_token`, clear stored tokens and re-register rather than retrying.

Client requirements to check first: Claude Team/Enterprise needs an Owner to add the connector org-wide before members connect; ChatGPT needs Developer Mode on, and write tools require Business, Enterprise or Education (Pro is read-only, Plus cannot use custom connectors); Gong needs a Tech admin.

## Everything runs as the user

Whatever the signed-in user can see and do in Spekit, these tools can do — and nothing more. Enforcement is at Spekit's backend API, not in the connector. A viewer-only user cannot create or change content no matter how the request is phrased. If a call is denied, the correct response is to report the permission gap, not to retry or work around it.

The connector cannot be scoped to particular topics or deal rooms, and there is no read-only mode independent of role.

## Reading

1. **`Get authenticated user`** first, so subsequent calls are anchored to a known identity.
2. **`Search content`** for keyword search across every Spek the user can access. Search results include the **full body** for natively authored Speks and for imported/synced file assets (PDFs, decks, external documents).
3. **`List topics`** returns top-level topics with their **direct subtopics only**. The hierarchy comes back one level at a time by design — call **`Get topic`** with a subtopic id to descend. Do not report the tree as complete after one call.
4. **`List topic content`** for the Speks filed under one topic; **`Get content`** for a single Spek's full body and its topic assignments; **`Get asset content`** for a file asset's extracted text by id.
5. **Always cite.** Ask for and return the source Spek so the answer traces back to approved content. An uncited answer defeats the point of a governed knowledge surface.

## Before authoring

- **`Get content style guide`** — Spekit's design-system rules. Read it before drafting.
- **`List content templates`** then **`Get content template`** for the body and fill instructions. Starting from the company's template keeps new content in the established layout.
- **Check what exists first.** Search and browse topics before creating, so you extend rather than duplicate the library.

## Writing

Every write tool prompts the user to confirm before it runs. That confirmation is enforced by the AI client through MCP tool annotations, not by Spekit — show the user exactly what will be created and wait.

- **`Create content`** — a new Spek filed under one or more topics.
- **`Create content from template`** — the same, starting from a company template.
- **`Update content`** — change a Spek's title, body or topic assignment. Name the exact section to change so the rest is left untouched.
- **`Create topic`** — optionally as a subtopic of an existing topic.
- **`Create company`**, then **`Create deal room`** under it (a deal room requires a parent company — create it first if it does not exist), then **`Create deal room content`** for Speks scoped to that deal.

Multi-step is expected: one request can read the approved messaging, then draft and create each piece of a launch set in turn.

## Hard limits — state these rather than attempting them

- **Nothing can be deleted.** Not Speks, topics, companies or deal rooms. Removal happens in Spekit.
- **Topics, companies and deal rooms are create-only.** They cannot be renamed or edited after creation. Only Speks are updatable.
- **An existing Spek cannot be copied into a Deal Room.** You can draft a new version based on it; the original stays where it is.
- **External Deal Room sharing cannot be enabled**, and the connector cannot even tell you whether sharing is already on. `List deal rooms` returns id, name, share link and internal link — not sharing status. Direct the user to check and enable it in Spekit.
- **No user or permission management.** That stays in Spekit admin settings.
- **Brand Studio styling is not applied** to content created or updated here. For fine-grained styling, use the AI Content Builder or the Spekit editor.

## Failure handling

A failed write returns a real, specific error and saves nothing — writes are validated server-side before anything is persisted, so there is no partial content to clean up. Read the message, fix the input, retry. Note there is **no idempotency key**: repeating a `Create content` call creates a second Spek.
