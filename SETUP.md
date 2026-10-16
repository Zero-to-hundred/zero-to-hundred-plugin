# Platform setup

Use `https://zerotohundred.ai/api/mcp` for the hosted Streamable HTTP connection. Public tools need no credentials. Builder tools require a signed account connection with `listings:read` or `listings:write` consent. Do not put a personal token, client secret or reviewer credentials in this package.

## ChatGPT and Codex

Public submission is gated by current [OpenAI plugin guidelines](https://developers.openai.com/plugins/plugin-guidelines), which prohibit direct and indirect digital-product commerce. Zero to Hundred sells software licenses. Keep the marketplace purpose disclosed and obtain an eligibility decision before submitting this bundle. Private development testing does not establish directory eligibility.

This bundle uses the [Agent Plugins package format](https://developers.openai.com/plugins/build/plugins): root `plugin.json`, root `mcp.json`, and `skills/`. Presentation and review fields are under `extensions.com.openai`. No existing registered app ID is assumed.

The release owner uploads the complete ZIP at [OpenAI Plugins](https://platform.openai.com/plugins). Complete domain verification and the MCP connection, register the exact OAuth redirect supplied by the portal, then scan the deployed tools. Finish metadata and skills checks before review. See [submission requirements](https://developers.openai.com/plugins/deploy/submission).

The review cases in the manifest are proposed cases. A reviewer walkthrough video and a secure reviewer account are required for submission. OpenAI must confirm eligibility for this software marketplace: the current [UI commerce rules](https://developers.openai.com/plugins/build/chatgpt-ui) limit purchase flows to supported categories. This plugin has no checkout or payment tool; its listing links and seller drafts still need an accurate marketplace disclosure.

The MCP App renders public cards in a conversation or an available sidebar. Hosts without MCP Apps can use the same public tools and skills through text. The panel never receives a seller token or a private source archive.

## Claude

Claude Code discovers `.claude-plugin/plugin.json`, `.mcp.json`, and the root `skills/`. From this folder, validate and load the package with:

```sh
claude plugin validate --strict .
claude --plugin-dir .
```

Use the [Claude directory portal](https://claude.ai/directory/manage/new) to submit the hosted server as an MCP connector, then submit this plugin bundle from the same organization. The bundle's GitHub repository must be public before publication. The owner approved MIT for the listed plugin manifests/configurations and skills only; [LICENSE](LICENSE) reserves brand assets and trademarks and excludes the private application. Complete OAuth registration from the actual portal redirect and installed-host testing before submission. See [Claude plugin publishing](https://claude.com/docs/directory/publish) and the [Claude Code package reference](https://code.claude.com/docs/en/plugins-reference). Loading a local Claude Code package does not establish consumer directory approval.

For a private installed-host test, upload this ZIP in Customize > Plugins > Add > Upload plugin, then connect the server from its Connectors tab. The same remote endpoint works in chat, Cowork and Claude Code. The owner registers `https://zerotohundred.ai` as the allowed link origin, supplies a populated test account securely, and verifies every exposed tool through MCP Inspector and a custom Claude connector before making the portal's testing declarations.

## Lovable

Automatic host registration is not yet live on this endpoint. Complete KAN-646 and the actual host consent tests before claiming account linking works for a fresh installation. A pre-registered client or a supported PAT connection is a separate setup path.

Lovable uses a [custom MCP chat connector](https://docs.lovable.dev/integrations/custom-mcp). In personal connector settings, add the hosted endpoint, choose OAuth for the signed Builder workflow, and complete consent. The release owner must register the exact redirect that Lovable supplies. Use the plus menu in a chat to select the connector, search software, compare it, and prepare a draft.

Lovable's connector supplies build-time chat context. It is not a plugin store package, and connecting it does not automatically integrate MCP into a published Lovable app. Public discovery can use the no-auth connection; seller actions need the signed connection. Do not claim portable panel rendering until Lovable confirms support and the installed-host check passes.

## Seller workflow

The listing skill retains owned create, list, get, update, signed source upload and explicit source confirmation. Use the current statement and version from the discovered tool and require the Builder to accept it for the selected archive before requesting the upload. Publication, purchase, payout and refunds remain web actions. Finish with the web review link.

The local PAT package remains available for existing clients. The plugin adds no replacement tool names or automatic financial actions.

## Events

MCP Events is a separate follow-up. This release advertises no subscriptions or event delivery. A working implementation needs durable, owner-scoped subscriptions, verified callback challenges, signed webhook delivery with retry and expiry, and revocation checks. See [MCP Events](https://developers.openai.com/plugins/build/mcp-events).
