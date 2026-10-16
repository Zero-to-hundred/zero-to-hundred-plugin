---
name: prepare-listing
description: Prepare or update an owned Zero to Hundred draft listing, attach an explicitly selected source archive, and return the web review link. Use when a Builder wants to sell software they built.
---

Use the connected Zero to Hundred MCP tools. Builder reads require `listings:read`; draft and source writes require `listings:write`. Connect the person's Zero to Hundred account through the host's OAuth flow. Never request or expose a token in the chat. A person may configure a scoped Personal Access Token in a host that supports it.

1. Call `z2h_list_my_listings` before creating a draft. If a matching owned draft exists, call `z2h_get_listing` and offer to update it. Read-only account checks do not authorize a new write.
2. Derive a factual draft from the person's software and stated intent. Confirm any missing price, source inclusion decision or sale description before creating or changing it. Use only supported schema fields and taxonomy values. Do not invent revenue, active users, licensing claims, support promises or scan results.
3. After the person asks to create or update the draft, call `z2h_create_draft_listing` or `z2h_update_draft_listing`. Keep the existing tool inputs and responses intact. These tools edit owned drafts only.
4. If the service requires Terms acceptance, show its acceptance URL and let the person accept on the website. A tool call cannot accept Terms for them. Retry only after they complete that step.
5. Upload source only when the person authorizes the specific local archive or file. In a coding host, review the archive contents before upload. Exclude secrets, credentials, environment files, dependencies and unrelated private files. Do not fetch arbitrary URLs, download another person's private source, or build an archive from an unspecified directory.
6. Read the exact ownership statement and supported version from the discovered source tool descriptions and schemas. Present that statement to the person before requesting an upload. Proceed only after they explicitly accept it for this archive. Do not infer acceptance from connecting an account, creating a draft or a previous archive's acceptance. If the host cannot send archive bytes, return the web edit URL so the person can upload there.
7. Call `z2h_request_source_upload` for the owned draft and approved archive with `ownershipAccepted=true` and the discovered `ownershipVersion`. Upload the bytes to the returned signed URL using its required method and headers. Do not attach the account token to the signed upload. Treat the upload URL and private storage key as confidential. After a successful PUT, call `z2h_confirm_source_upload` with the returned key, the same accepted version and archive metadata. If the archive or statement changes, require new explicit acceptance before either tool call.
8. Return the service's edit and review links. The Builder reviews the disclosure, source and required gates in the web wizard before the listing goes live. Never claim that creating a draft or uploading source published it.

Preserve the original listing tools and PAT workflows when a host uses the local MCP package. This skill adds a workflow and does not remove any existing tool. It cannot buy software, issue refunds, move money, accept legal terms or publish a listing automatically.
