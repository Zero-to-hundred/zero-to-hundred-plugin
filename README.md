# Zero to Hundred plugin

This package connects public software discovery and owned draft preparation to the hosted MCP server at `https://zerotohundred.ai/api/mcp`. The portable manifest supports ChatGPT and Codex. The Claude manifest and `.mcp.json` use Claude's native package layout. Both hosts discover the same `skills/` folder.

Public search and comparison need no token. Draft reads and writes use a connected account with the relevant listing scopes. The interactive MCP App displays only public software. The Builder completes publication in the web wizard.

Provider registration, installed-host testing and directory approval are covered in [Setup](SETUP.md).

For a local Claude Code package check, run these from this folder:

```sh
claude plugin validate --strict .
claude --plugin-dir .
```

The first command checks the package. The second starts a session with the plugin and may ask the person to connect their account. Neither command publishes the plugin.

Lovable uses a personal custom MCP chat connector. Its setup is documented in the distribution guide. This package does not claim a Lovable plugin directory listing.

The four manifests/configurations and two skills listed in [LICENSE](LICENSE) are MIT licensed. Brand assets, names, logos and trademarks are excluded and remain reserved. Other bundle documentation and the private application keep their existing rights. The MIT declaration grants no rights to the hosted application or private source archives.
