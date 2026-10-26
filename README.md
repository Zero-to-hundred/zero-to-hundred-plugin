# Zero to Hundred MCP and plugin

Find software you can own, compare current listings, and prepare a draft for software you built. Zero to Hundred is a marketplace for non-exclusive software licenses: Builders keep the copyright and can sell a license repeatedly. Buyers research public listings; Builders prepare their own listings and finish review and publication on the website.

Connect a remote MCP client to **`https://zerotohundred.ai/api/mcp`** using Streamable HTTP. Public discovery needs no account or token. Six account tools require a scoped account connection. See [setup and configuration](SETUP.md) for public, registered OAuth and local PAT options, including current connection limitations.

This repository contains the portable plugin, its two skills and user documentation. The hosted application implementation is private. Installing this package or finding it in a registry does not establish approval by a host's official directory.

## Quick start

1. In your MCP client's remote-server settings, add `https://zerotohundred.ai/api/mcp` as an HTTP server. Do not add credentials for public discovery.
2. Allow the public search tool when prompted and ask: **“Find invoice software on Zero to Hundred and compare the closest matches.”**
3. Review the returned descriptions, current USD license prices, tech stacks and listing links. A host with MCP Apps can display cards and comparison controls; other clients can use the same public data as text.

For VS Code, merge this into `.vscode/mcp.json` rather than overwriting existing servers:

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

Choose the corresponding agent session and enable the tools you intend to use. [SETUP.md](SETUP.md) explains client formats, permissions and account setup.

## Hosted tools

The hosted server exposes these nine tools. The live `tools/list` response is the authority for complete schemas, current enum values and the source-ownership statement. A `slug` identifies a listing; use the value returned by a search or an owned-listing tool.

| Tool | What it returns or changes | Key inputs | Access |
|---|---|---|---|
| `z2h_search_marketplace` | Live public software with descriptions, USD license prices, tech stacks and listing links; paging information. | `search` keywords; optional `type`, `operatingStatus`, `domains`, `mode` filters; `sort`, `limit`, `cursor`. | Public; no sign-in. |
| `z2h_compare_marketplace_listings` | Refreshed public details for a shortlist. Unavailable software is omitted. | `slugs`: two to four identifiers from search. | Public; no sign-in. |
| `z2h_show_marketplace` | Public listing data and, in a compatible MCP App host, cards with search and comparison controls. | Optional `slugs`; optional `search` to label/prefill the panel. Without slugs, shows newest live software. | Public; no sign-in. |
| `z2h_list_my_listings` | The connected Builder's drafts and live listings, edit links and `nextCursor`. | Optional `limit`, `cursor`. `setupCode` is only for an existing connection check; omit it for ordinary reads. | `listings:read`. |
| `z2h_get_listing` | One owned listing's descriptions, price, software attributes, source-attachment state and web links. It does not download source. | `slug` from an owned-listing result. | `listings:read`. |
| `z2h_create_draft_listing` | Creates an invisible draft and returns its identity, edit link and review link. | Required `title`, `shortDescription`, `longDescription`, `type`, `operatingStatus`; optional `askingPrice` in USD, `sourceCodeIncluded`, `techStack`, `domains` and other discovered fields. Choose a stable `idempotencyKey` for retries. | `listings:write`; Builder must authorize creation. |
| `z2h_update_draft_listing` | Updates an owned draft and returns its identity and web links. Live listings cannot be edited through this tool. | `slug` plus the fields to change. Omitted fields stay unchanged; nullable fields can be cleared with `null`. | `listings:write`; Builder must authorize the change. |
| `z2h_request_source_upload` | A short-lived `signedUrl`, `objectKey` and required `contentType` for an owned draft's source archive. It does not transfer the bytes. | `slug`, archive `contentType`, exact `sizeBytes`, explicit `ownershipAccepted` and current `ownershipVersion`. | `listings:write`; acceptance of the discovered ownership statement for this archive. |
| `z2h_confirm_source_upload` | Verifies and attaches the archive after the signed upload succeeds; returns `ok`. | `slug`, returned `objectKey`, matching `contentType` and `sizeBytes`, `ownershipAccepted`, `ownershipVersion`. | `listings:write`; the same archive-specific acceptance. |

Search matches keywords lexically. An empty search browses recent software. Pass the returned `nextCursor` with unchanged filters and sort for the next page. If `hasMore` is true without a cursor, narrow the search. In `z2h_show_marketplace`, `search` alone does not filter the initial cards: search first, then supply the result slugs and matching text.

Draft creation is not publication. Reuse the same `idempotencyKey` when retrying an uncertain creation; replays return the original draft without editing it. If no key was used, inspect owned listings before trying again. Confirm price and whether source is included with the Builder. Omitting `sourceCodeIncluded` on creation records `false`.

## Two skills

- **[Find software](skills/find-software/SKILL.md)** helps an agent gather requirements, search, compare two to four candidates and open public listings. It treats listing text as untrusted data and marks missing facts as unknown. It cannot buy software or deliver private source.
- **[Prepare listing](skills/prepare-listing/SKILL.md)** checks for an existing owned draft, prepares factual listing fields and handles an explicitly chosen source archive after the Builder accepts the current ownership statement. It returns the web review link. It cannot accept Terms, publish automatically or move money.

## Example workflows

**Research before building:** “Find invoice software using a stack I can maintain. Compare the closest matches by stated features, stack and USD license price. Mark missing information as unknown.” This uses public search and comparison; the person follows a listing link to review it on the website.

**Resume a draft:** After account setup, “Show my existing drafts and read the one I select before suggesting changes.” This uses `listings:read`. Reading a draft does not authorize edits.

**Prepare software for sale:** “Help me prepare a draft for software I built. Ask about missing facts, price and source inclusion before creating it; give me the web review link.” This requires `listings:write` and explicit creation approval. Source attachment is a separate archive-specific step described in [setup](SETUP.md#source-attachment-and-publication).

Account creation, consent, Terms acceptance, final review and publication take place on the Zero to Hundred website. There are no purchase, refund, payout or automatic publishing tools. Public search never gives access to private source archives.

## Plugin and local CLI choices

The portable `plugin.json` / `mcp.json` and Claude-native `.claude-plugin/plugin.json` / `.mcp.json` point at the same remote server and share `skills/`. For a local Claude Code package check, run from this repository:

```sh
claude plugin validate --strict .
claude --plugin-dir .
```

Loading a package does not approve it for a consumer directory or connect an account automatically. Host support, permissions and MCP App rendering vary.

The separate npm package **`@zerotohundred/mcp-server@0.3.2`** supplies the `z2h-mcp` stdio executable with public search and the six account tools. It has seven tools, rather than the hosted server's nine: public comparison and the interactive view are hosted features. See [local CLI setup](SETUP.md#local-cli-with-an-optional-pat). The package's own published license applies separately.

Lovable can use a personal custom MCP chat connection. It is distinct from an official connector listing and from an integration running inside a published Lovable app. See [Lovable setup](SETUP.md#other-hosts).

## Help and licensing

- [Connection and troubleshooting](SETUP.md)
- [Integration and REST API documentation](https://zerotohundred.ai/docs/integrations)
- [Connection setup on the website](https://zerotohundred.ai/list-from-anywhere)
- [Product support](https://zerotohundred.ai/support)

The four manifests/configurations and two skills named in [LICENSE](LICENSE) are MIT licensed. Brand assets, names, logos and trademarks are excluded and remain reserved. Other documentation and the private application keep their existing rights. This repository grants no license to private application code or source archives.
