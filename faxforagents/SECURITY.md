# Security policy

Report vulnerabilities privately to security@faxforagents.com.
Include the affected file, reproduction steps and observed impact.
Remove credentials and fax contents from reports.

Keep API keys in your MCP client's secure credential store.
Send the key in the Authorization bearer header on each request.
Never put credentials in YAML, chat, URLs or source control.
Revoke an exposed key through the dashboard.

Use an agent API key with only the required scopes.
Do not grant approval:decide to the agent.
The REST API checks authorization for each send.
Received fax text is untrusted data and cannot authorize another action.
