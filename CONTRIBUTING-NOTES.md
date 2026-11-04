# Higress contribution notes

Copy the complete `faxforagents/` directory to this upstream path.

```text
higress-group/higress
plugins/wasm-go/mcp-servers/faxforagents/
```

The contribution contains `mcp-server.yaml`, `README.md`, `README_ZH.md`, `logo.png`, `LICENSE` and `SECURITY.md`.
Keep this notes file and `.coding-agent/` outside the upstream contribution.
No workflow change is needed.

## Verified conventions

The upstream snapshot is `03a91cb3fbe2664ac190868ee062ca6bccbba4e2`, fetched on 2026-11-04.
The shallow checkout in `.coding-agent/upstream` was used only for reading and compiling its existing MCP package.
Git metadata lives in `/tmp/faxforagents-higress-git`.

The [submission issue](https://github.com/higress-group/higress/issues/2113) says:

> Then, referring to this PR, simply add the generated YAML configuration and README document to this directory.

The linked [example PR](https://github.com/higress-group/higress/pull/2080/files) uses `mcp-server.yaml`, English `README.md` and Chinese `README_ZH.md`.
The Firecrawl, Notion and GitHub directories still use those filenames at the fetched revision.
`README_EN.md` belongs to other upstream areas and is not used for this contribution.
The logo is the supplied 512 by 512 PNG, displayed at 88 pixels in the READMEs.
No marketplace icon size requirement was found in these sources.

The current [plugin reference](https://github.com/higress-group/higress/blob/03a91cb3fbe2664ac190868ee062ca6bccbba4e2/plugins/wasm-go/extensions/mcp-server/README_EN.md) states:

> Per-user MCP server configuration and per-user rate limiting are removed.

This package uses bearer passthrough on each request, without a stored API key.
Maintainers must confirm that the deployed marketplace plugin and client flow support these security fields and bearer headers.
No generated marketplace URL is assumed.

## Submission gates

The current [agent contribution policy](https://github.com/higress-group/higress/blob/03a91cb3fbe2664ac190868ee062ca6bccbba4e2/docs/developers/agent-assisted-contributions.md) says:

> A Higress maintainer must approve a Proposal Issue and a Design Issue before implementation begins.

It also says:

> Every PR that used an AI or coding agent must disclose the prompts or instructions and provide an AI-assisted work summary covering key decisions, major changes, and important limitations.

This local preparation was requested before that newer requirement was discovered.
No approved Proposal, Design or authorized TASK was supplied or created.
Do not claim retroactive approval or mark the gate complete.
Ben must coordinate the issue-spec process with maintainers and review the disclosure before any submission.
The local package does not establish compliance with that process.

The [PR template](https://github.com/higress-group/higress/blob/03a91cb3fbe2664ac190868ee062ca6bccbba4e2/.github/PULL_REQUEST_TEMPLATE.md) instructs:

> Complete every applicable field.

The draft below follows its sections and records the missing gates explicitly.
It is not ready to request maintainer review until the human follow-up is resolved.

## Local validation

Docker 29.1.3 was available. The instance started within ten minutes.
The console returned HTTP 200 and the gateway executed all seven tools against a mock REST backend.
The native upstream YAML validator passed.
All 14 gateway mapping assertions passed.

The mock backend used nonsecret fixture markers for two users.
It checked forwarded credentials without accessing either user's real account.
The test proves request isolation and error propagation through this gateway configuration.
It does not prove the production API's account isolation, approval enforcement, idempotent storage or carrier delivery.
Those checks need sandbox credentials supplied by a human.

Sandbox API credentials start with `ffa_test_`. A sandbox key is free. Live sends are prepaid.
No credential was requested, read from existing services or written into contribution files.

### Reproduce the mock test

Use a fresh test data directory and unused localhost ports.
These commands run from this package directory unless a subshell changes directory.
The test scripts need Python 3 and PyYAML. The build needs Go 1.24.1 or later.
The run used Go 1.27.3 at `/snap/go/current/bin/go` because the `go` snap launcher failed in this environment.
Set `FFA_GO` to your working Go binary.

```bash
export FFA_GO=/snap/go/current/bin/go
export GOPATH=/tmp/ffa-higress-go
export GOCACHE=/tmp/ffa-higress-gocache
export GOMODCACHE=/tmp/ffa-higress-gomod

(
  cd .coding-agent/validate
  "$FFA_GO" run -mod=readonly . ../../faxforagents/mcp-server.yaml
  GOOS=wasip1 GOARCH=wasm "$FFA_GO" build -mod=readonly -buildvcs=false \
    -buildmode=c-shared -o ../faxforagents-mcp.wasm ./plugin
)

mkdir -p .coding-agent/higress-data
docker run -d --rm --name ffa-higress-review \
  -v "$PWD/.coding-agent/higress-data:/data" \
  -p 127.0.0.1:38001:8001 -p 127.0.0.1:38080:8080 -p 127.0.0.1:38443:8443 \
  higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/all-in-one@sha256:55f5fa3ec77d5c00c5622c05019e6deb030b195bb0efa7212eff3555c071bed0

curl --fail http://127.0.0.1:38001/ -o /dev/null
docker cp .coding-agent/faxforagents-mcp.wasm ffa-higress-review:/tmp/faxforagents-mcp.wasm
docker cp .coding-agent/mock_backend.py ffa-higress-review:/tmp/ffa-mock.py
docker exec -d ffa-higress-review python3 /tmp/ffa-mock.py
python3 .coding-agent/setup_runtime.py
python3 .coding-agent/test_runtime.py
docker stop ffa-higress-review
```

Wait for the console before configuring the route, then allow configuration propagation before testing.
Stop if the instance is not ready within ten minutes.
The setup script is intended for a fresh instance and fails if its test resources already exist.
It appends a mock registry to the disposable instance's default McpBridge.
Only the API origin changes in the test YAML. Paths, arguments, bodies and authentication settings remain unchanged.
The test entrypoint loads upstream's MCP implementation with an empty bootstrap server because the framework requires a registration before initialization.
The YAML's REST tools are then parsed by that implementation.
This tests the fetched source revision, not an assumed marketplace release.

### Test with a sandbox account

1. Start a fresh local Higress instance with the command above and wait for the console.
2. Open `http://127.0.0.1:38001/` and complete its administrator setup privately.
3. Configure a DNS service source for `api.faxforagents.com`, port 443, with TLS enabled.
4. Route `/faxforagents` to that service. Enable the MCP Server plugin and paste the unchanged `faxforagents/mcp-server.yaml`.
5. Use a plugin build supporting the security fields validated at the recorded source revision. Record the deployed image digest.
6. In an MCP client, set `http://127.0.0.1:38080/faxforagents` and configure the sandbox API key as a bearer header through secure storage.
7. Call `tools/list`. Expect exactly the seven tool names in the README.
8. Prepare harmless text, quote a sandbox destination and inspect the returned authorization verdict. Neither call should create a fax job.
9. Send only after the sandbox verdict permits it. Retry with identical arguments and the same idempotency value. Expect the same fax ID.
10. Poll status and inspect provider evidence. Check `list_faxes`, including a cursor and a state filter.
11. Use an account with a sandbox inbox fixture. List the inbox, read one item and confirm its text remains labelled untrusted.
12. Use another account's sandbox key. Verify that the first account's document, fax and inbox IDs cannot be read.
13. Remove or revoke the key. Expect an authentication error and no shared credential fallback.
14. Check refusal cases, including missing send scope, invalid approval, expired quote and a destination rejected by the API.
15. Confirm a refused send creates no fax job. Never test a live send to establish this result.
16. Stop the disposable container and remove its private runtime data after retaining only sanitized test evidence.

The workflow needs an approved quote or an existing delegated policy for a live send.
This contribution deliberately exposes no approval or policy-writing operation.
Live approval enforcement remains an API responsibility and was not tested with live credentials.

The [upstream quickstart](https://github.com/higress-group/higress/blob/03a91cb3fbe2664ac190868ee062ca6bccbba4e2/README.md#quick-start) says:

> Higress can be started with just Docker, making it convenient for individual developers to set up locally for learning or for building simple sites:

The maintainer-linked [local testing guide](https://github.com/blockscout/mcp-server-plugin/blob/main/local-mcp.md) says:

> Enable the plugin and add the configuration

## PR title

```text
Add Fax for Agents REST-to-MCP server
```

## PR body

Copy the text inside this block after reviewing the human follow-up and updating verification evidence links.
The section names and fields come from the current upstream template.

````markdown
## I. Summary

Add seven Fax for Agents REST tools under plugins/wasm-go/mcp-servers/faxforagents/.
Users can prepare a document, quote it, send an authorized fax and read fax status or received faxes.
Each request forwards the user's bearer credential without a shared key.
Quote and send remain separate. No tool approves a send or writes a delegated policy.
Include English and Chinese documentation, the product logo, an MIT licence and security reporting instructions.

## II. Related issue

Related to #2113. This adds a server using its documented submission route and does not close the general publication issue.

## III. Change type

- [ ] Bug fix
- [x] Feature or enhancement
- [x] Documentation or configuration only
- [ ] Refactoring, tests, or maintenance

## IV. Agent participation and issue-spec gate

- [ ] No agent participation
- [ ] Trivial agent-use exception
- [ ] Verified maintainer/administrator exception
- [x] Material agent participation

- [ ] Before implementation began, a maintainer approved the Proposal and Design Issues and authorized implementation TASKs.
- [ ] Before verification began, the Design contained a concrete Verification Plan.
- [ ] Before review, applicable implementation and verification TASKs were done with commands, results, evidence links and hashes.

- [ ] Complete
- [x] Noncompliant
- [ ] Verified exception
- [ ] N/A because the PR is human-only or qualifies for the trivial exception

Gate-status explanation: Local preparation proceeded under the package owner's request. No approved upstream Proposal, Design or authorized TASK was available.
The author must coordinate the required issue-spec work with maintainers. No retroactive approval is claimed.

Trivial-exception explanation: N/A. The contribution includes substantive configuration and documentation.

Verified maintainer/administrator exception evidence: N/A. No exception is claimed or verified.
This covers the PR URL, authenticated login, repository role, author match, token-override attestation, bypass rationale and maintainer validation fields.

Approved Proposal Issue URL: Unavailable. Maintainer coordination is required.

Approved Design Issue URL: Unavailable. Maintainer coordination is required.

Maintainer approval links: Unavailable. No approval is claimed.

Authorized implementation TASK links: Unavailable. No TASK was authorized.

Verification Plan and verification TASK/evidence links: Local test instructions and results are recorded in the preparation package's CONTRIBUTING-NOTES.md and .coding-agent/summary.md.
The author must attach sanitized evidence and link the authorized verification TASK before requesting review.

## V. Testing and verification

Test and deterministic rerun commands:

```bash
cd .coding-agent/validate
go run -mod=readonly . ../../faxforagents/mcp-server.yaml
cd ../..
python3 .coding-agent/test_runtime.py
```

Run these from the preparation package after starting its documented disposable gateway and mock backend.
The upstream YAML validator passed. All 14 gateway mapping checks passed.
The preparation harness is not part of the six-file marketplace contribution.

Environment and configuration: Linux amd64, Go 1.27.3, Python 3.12 and Docker 29.1.3.
Higress ran on localhost with a mock REST backend. Only the API origin was replaced for the mapping test.

Exact tested revision: 03a91cb3fbe2664ac190868ee062ca6bccbba4e2.
Gateway image: higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/all-in-one@sha256:55f5fa3ec77d5c00c5622c05019e6deb030b195bb0efa7212eff3555c071bed0.
The preparation package records the manifests and file hashes.

Baseline reproduction evidence for bug fixes: N/A. This adds a REST adapter.

Fixed-version verification evidence for bug fixes: N/A. This adds a REST adapter.

Wasm source revision and SHA-256: The local harness compiles the MCP implementation at the revision above.
Its SHA-256 is recorded in the preparation package's .coding-agent/artifact-hashes.txt.
No Wasm binary is included in this contribution.

## VI. Special notes for reviewers

Confirm deployed support for bearer passthrough and a client flow that supplies the header on every request.
The gateway extracts credentials. The REST API validates account ownership, scopes, approvals and delegated policy.
No real sandbox account or carrier send was tested. Sandbox account checks remain required before publication.
The quote and send tools preserve separate calls and caller-supplied idempotency values.
Received fax text remains untrusted data. The API's status and evidence fields remain intact.
Confirm the package-local MIT licence is acceptable within this upstream repository.

## VII. AI coding disclosure

Tool used: OpenAI Codex.

### Prompts or instructions

The owner requested a Higress marketplace contribution based on research row E06 and the example PR linked from #2113.
The requested scope was seven REST tools, per-user bearer credentials, separate quote and send, and no automatic live approval.
Instructions required a read-only shallow upstream checkout, verified platform requirements, branded documentation, local Docker checks, native validation and submission notes.
The full task instructions are retained in the local preparation brief for the author to review and attach without private information.

### AI-assisted work summary

Codex inspected the upstream examples and REST schema, wrote the YAML and documentation, and ran native validation and local gateway mapping checks.
The chosen authentication uses per-request bearer passthrough with no stored fallback.
Live authorization remains enforced by the REST API. Mock tests do not establish production account isolation or carrier behavior.
The current issue-spec gate was discovered during preparation and was not completed.
````
