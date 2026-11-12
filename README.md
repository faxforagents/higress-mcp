<p align="center">
  <a href="https://faxforagents.com"><img src="faxforagents/logo.png" width="88" alt="Fax for Agents"></a>
</p>

<h1 align="center">Fax for Agents for Higress</h1>

<p align="center">The Higress MCP Marketplace contribution of <a href="https://faxforagents.com">Fax for Agents</a>, the fax API built for AI agents.</p>

<p align="center">
  <a href="https://faxforagents.com">Website</a> ·
  <a href="https://faxforagents.com/docs/quickstart/">Quickstart</a> ·
  <a href="https://faxforagents.com/docs/api/">API reference</a> ·
  <a href="https://faxforagents.com/docs/sdks/">SDKs</a> ·
  <a href="https://faxforagents.com/mcp/">MCP</a> ·
  <a href="https://app.faxforagents.com">Dashboard</a> ·
  <a href="https://faxforagents.com/pricing/">Pricing</a> ·
  <a href="https://faxforagents.com/changelog/">Changelog</a>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/licence-MIT-1f6feb" alt="MIT licence"></a>
  <a href="faxforagents/mcp-server.yaml"><img src="https://img.shields.io/badge/Higress-7%20tools-6b4fbb" alt="Seven Higress tools"></a>
</p>

Fax for Agents is a fax API, a hosted MCP server and SDKs built so AI agents can send and receive
faxes. It is made by JSB Holdings LLC and built by [Synapse](https://synapsereality.io/).

This repository holds the package prepared for the Higress MCP Marketplace. It is not submitted
upstream yet.

| Path | What it is |
|---|---|
| [`faxforagents/mcp-server.yaml`](faxforagents/mcp-server.yaml) | Maps seven REST operations of the API to Higress MCP tools |
| [`faxforagents/README.md`](faxforagents/README.md) | The English page of the contribution |
| [`faxforagents/README_ZH.md`](faxforagents/README_ZH.md) | The Chinese page |
| [`CONTRIBUTING-NOTES.md`](CONTRIBUTING-NOTES.md) | The upstream conventions, the local validation and the open submission gates |

The package sends the caller's own API key as a bearer credential on each request. It stores no key.
A test key (`ffa_test_...`) reaches the sandbox only: no carrier, no cost. Keys are created in the
[dashboard](https://app.faxforagents.com).

## Licence

MIT. See [LICENSE](LICENSE). Security reports go to hello@faxforagents.com, see
[SECURITY.md](faxforagents/SECURITY.md).
