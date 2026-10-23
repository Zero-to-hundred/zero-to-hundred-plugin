# Connect Zero to Hundred

Use the hosted **Streamable HTTP** endpoint:

```text
https://zerotohundred.ai/api/mcp
```

Public search, comparison and display need no account. Owned listing reads require `listings:read`; creating or editing a draft and attaching source require `listings:write`. [The tool reference](README.md#hosted-tools) describes each operation.

## Choose a connection

| What you need | Connection | Current requirement |
|---|---|---|
| Search and compare public software | Hosted HTTP MCP | No token or sign-in. |
| Read or prepare your own drafts in a local client | Published npm stdio CLI | Node.js 20 or later; your own scoped Personal Access Token (PAT). |
| Use account tools in a remote client | Hosted HTTP MCP | A scoped PAT supported securely by that client, or a pre-registered OAuth client with the exact redirect URI and an MCP resource-bound grant. |

**Fresh automatic OAuth client registration is not currently available on the hosted endpoint.** A host that requires automatic registration may fail before sign-in. An existing registered client is a separate path; the endpoint URL alone does not supply a client ID. Full authenticated seller journeys are not verified in every host. Public tools and the local PAT option do not depend on automatic registration.

## Public connection in VS Code and Copilot

Follow [VS Code's current MCP setup guide](https://code.visualstudio.com/docs/agent-customization/mcp-servers). A remote connection needs no local Node.js runtime.

1. Run **MCP: Add Server** from the Command Palette and select HTTP. Enter the endpoint above and a server ID such as `zero-to-hundred`.
2. Choose the configuration destination. Inspect and trust the endpoint only if you intend to use it. Enable the three public tools in the session; keep account tools disabled when you only want public research.
3. Ask “Search Zero to Hundred for invoice software.” Approve that public tool call when prompted. Expect listing data or a valid empty result, rather than an account-login requirement.

For the VS Code configuration format, merge this into `.vscode/mcp.json`:

```json
{
  "servers": {
    "zero-to-hundred": {
      "type": "http",
      "url": "https://zerotohundred.ai/api/mcp"
    }
  }
}
```

Other formats, including a portable workspace-root `.mcp.json`, can use a top-level `mcpServers` instead. Settings, secret prompts and forwarding differ between the Local and Copilot harnesses. Use the [configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration) for your actual session; do not rename keys or copy secret placeholders between formats without checking support.

## Connect your account with OAuth

This path requires a previously registered client. Its operator must register the **exact redirect URI the host supplies**, the intended listing scopes and the correct public/confidential client type. Sign in on the website, then open [account developer settings](https://zerotohundred.ai/account/developers) and the [OAuth documentation](https://zerotohundred.ai/docs/integrations#oauth). Account settings require sign-in and the account's access permissions. If you do not operate the host's OAuth client, ask that host or the integration operator to arrange registration; do not invent a client ID or borrow another application's credentials.

1. Configure the registered client ID using your host's supported OAuth settings. In the VS Code format, the server entry supports `"oauth": { "clientId": "<your-registered-client-id>" }`. This is a placeholder to replace with your registered ID, not a working shared client. Keep any confidential client secret in the operator's secure backend.
2. Start the host's account authorization action. It opens the Zero to Hundred website. Sign in or create an account there, check which application is requesting access, and review the requested scopes. Use `listings:read` for reads; draft and source changes require `listings:write`.
3. Complete or decline consent on the website. The host handles the authorization code and PKCE exchange and returns to the registered callback. A compliant MCP grant must use `resource=https://zerotohundred.ai/api/mcp` at authorization, exchange and refresh. A direct REST OAuth grant is not interchangeable with an MCP grant.
4. Verify the connection with `z2h_list_my_listings` before authorizing a mutation. Read access alone does not authorize creation, editing or upload. If the host cannot complete registration or authorization, use public discovery or the local PAT route; do not broaden scopes to work around the failure.

The published OAuth metadata is available at:

- `https://zerotohundred.ai/.well-known/oauth-protected-resource/api/mcp`
- `https://zerotohundred.ai/.well-known/oauth-authorization-server`

The host should manage token refresh. After expiry, revocation or lost access, reconnect through its normal authorization controls. To remove access, use your account's connected-app controls and disconnect the host. Do not paste access tokens, refresh tokens, codes or client secrets into a conversation, issue, screenshot or shared MCP configuration.

## Local CLI with an optional PAT

The official npm package is `@zerotohundred/mcp-server@0.3.2`; its executable is `z2h-mcp`. It speaks MCP over stdio, so configure it as a server in an MCP client. It is not an interactive command for buying or publishing software.

With Node.js 20 or later, a VS Code Local configuration for public search is:

```json
{
  "servers": {
    "zero-to-hundred-local": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@zerotohundred/mcp-server@0.3.2"]
    }
  }
}
```

For account access, sign in and create your own PAT in [developer settings](https://zerotohundred.ai/account/developers), selecting only the listing scopes you need. Supply it as `Z2H_API_KEY` through your client's protected credential input. The following VS Code **Local** example prompts for the value rather than committing a token:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "zero-to-hundred-pat",
      "description": "Your scoped Zero to Hundred Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "zero-to-hundred-local": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@zerotohundred/mcp-server@0.3.2"],
      "env": {
        "Z2H_API_KEY": "${input:zero-to-hundred-pat}"
      }
    }
  }
}
```

VS Code configurations requiring interactive inputs are not forwarded to Copilot harness sessions. Use a Local session for this example. Other clients need their own documented credential mechanism. Do not replace the placeholder with a literal token in the file.

The CLI uses `https://zerotohundred.ai` by default. No API-base override is needed. Public search omits account authorization even when a PAT is configured; protected tools refuse calls without a token. Revoke an unused or exposed PAT in developer settings. Never share one account's PAT as a public directory credential.

## Prepare a draft

1. Connect with the needed scopes and read your existing listings. Reuse the matching draft when appropriate.
2. Supply factual descriptions, a software category and operating status. Confirm price and source inclusion with the Builder before creation. Use the discovered schema for accepted values.
3. Authorize creation or a specific update. Choose one stable `idempotencyKey` for a new draft and reuse it on retries. Without a key, inspect owned listings before repeating an uncertain create.
4. Open the returned `editUrl` or `publishUrl`. Complete any required Terms acceptance on the website. Creating a draft does not publish it.

## Source attachment and publication

Select the exact local archive before uploading. Review its contents and exclude credentials, environment files, dependencies and unrelated private files. Connecting an account or approving a draft is not consent to upload an archive.

Read the **current ownership statement and version from the discovered upload tool** and explicitly accept it for the selected archive. Then:

1. Call `z2h_request_source_upload` with the owned draft `slug`, archive MIME `contentType`, exact `sizeBytes`, `ownershipAccepted=true` and the accepted `ownershipVersion`.
2. PUT the archive bytes to the returned `signedUrl` using its required headers. Do not forward your account bearer token to storage. Keep the signed URL and private `objectKey` out of chat and shared logs.
3. After the PUT succeeds, call `z2h_confirm_source_upload` with the returned key, matching archive metadata and the same acceptance. Inspect an uncertain outcome before retrying.
4. Finish source review, disclosure and publishing on the website. A successful confirmation attaches source; it does not publish a listing or prove a passing scan.

If the host cannot send binary archive bytes, open the draft's web edit link and attach source there. These tools cannot download another person's private source, accept legal terms, make a purchase, refund a payment or release a payout.

## Other hosts

Claude Code can validate and load this repository with `claude plugin validate --strict .` and `claude --plugin-dir .`. The portable and Claude-native manifests share the same remote endpoint and skills. A loaded package still requires the host's tool permissions and separate account setup for owned tools.

Lovable's [personal custom MCP connection](https://docs.lovable.dev/integrations/custom-mcp) supplies chat context. Use no authentication for public research where the host supports that mode. Owned workflows require the registered OAuth or secure PAT path described above. A custom connection does not make Zero to Hundred an official Lovable connector and does not automatically add an API integration to a published Lovable app.

ChatGPT, Claude and other directories apply their own eligibility and review rules. This repository is not proof of official approval. MCP App display depends on the host; a text-only client can still search and compare public results. No subscriptions or MCP Events delivery are advertised.

## Troubleshooting

| Symptom | What to do |
|---|---|
| Fresh OAuth setup cannot register a client | Automatic registration is unavailable. Arrange a registered client with the actual callback URI, use a supported personal PAT connection, or keep public access only. |
| An account tool refuses access | Check sign-in/token validity and the required scope. An expired or revoked credential needs reconnection. Public discovery does not need a token. |
| A listing cannot be read or edited | Check that it belongs to the connected account. Missing and other-owner listings are not disclosed; live listings cannot be edited by the draft-update tool. |
| Search returns no software | Try the wording a Builder might use, fewer filters, or an empty browse query. Empty results do not prove that suitable software cannot exist. |
| Cards do not appear | Read the public text results or open the canonical listing links. Inspect the host's MCP/App support and server output; do not treat assistant prose as proof that a panel rendered. |
| The server is rate-limited or unavailable | Wait before retrying. Avoid parallel retry loops; keep the same creation idempotency key if a write's outcome is uncertain. |
| A signed source URL expired or upload status is uncertain | Inspect the draft/source state. Request a replacement through the normal owned upload flow only when needed; preserve archive-specific acceptance and keep URLs private. |

For help, use [product support](https://zerotohundred.ai/support) and share the client/version, tool name, approximate time and a redacted error. Never include tokens, authorization codes, signed storage URLs or private archives. [Full integration documentation](https://zerotohundred.ai/docs/integrations) covers REST, OAuth and web prefill separately.
