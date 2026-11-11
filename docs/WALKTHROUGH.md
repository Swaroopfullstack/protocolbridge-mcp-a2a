# ProtocolBridge walkthrough

## What to demonstrate

Actual MCP host/client/server interactions and an actual remote-agent A2A service. Both use the attributed public corpus; neither exposes arbitrary local paths or user URLs.

### MCP stdio and agent use

```powershell
uv run portfolio demo mcp
uv run portfolio demo mcp-agent
uv run portfolio mcp --transport stdio
```

The demo launches a real subprocess, initializes a ClientSession, discovers tools, profiles the Iris CSV and loads a bounded NIST PDF page window. A separate call adapts discovered schemas into LangChain StructuredTools and runs create_agent with actual tool observations. Offline tool selection uses a declared scripted model. Stdio stdout is reserved for protocol messages; diagnostics go to stderr.

Tools: catalog, profile_public_csv, load_public_pdf and search_public_pdf. The catalog://public resource provides metadata independently from callable tools. Source arguments are typed catalog IDs. PDF windows are at most three pages; search returns at most five hits. The actual public PDF is parsed and searched in Chroma.

### Authenticated Streamable HTTP

Set a random MCP_HTTP_TOKEN of at least 24 characters, then:

```powershell
uv run portfolio mcp --transport http
```

Endpoint: http://127.0.0.1:8010/mcp. Supply Authorization: Bearer YOUR_TOKEN. MCP's host/origin checks stay enabled. A missing credential returns 401. Legacy SSE is a different transport and is not configured by changing this URL. A production reverse proxy must preserve host/origin requirements; set an explicit approved host policy for your deployment.

### Actual A2A service

```powershell
uv run --extra adk portfolio a2a
```

In another terminal with the same protocol credential:

```powershell
uv run --extra adk portfolio a2a --client
```

The service publishes /.well-known/agent-card.json and JSON-RPC through the official ADK to_a2a adapter. It uses durable SQLite-backed ADK sessions and A2A tasks. The official SDK client discovers the card, validates the advertised local endpoint, sends a Message and consumes typed response/task events. Authentication is enforced by ASGI middleware, including discovery. This one credential represents a public-catalog operator; private upload namespaces are never exposed.

Read [protocol.py](../src/portfolio/protocol.py) and [a2a_project.py](../src/portfolio/a2a_project.py). The lockfile tests MCP 1.30 and A2A SDK 1.3/protocol 1.0 APIs; do not paste older A2A 0.3 examples into the new protobuf SDK.

### Limits and extension exercise

MCP capabilities are tools/resources, while A2A requests concern agent tasks/messages/artifacts. Neither protocol is a durable general job queue. The SDK adapter provides protocol task handling; this short PDF agent does not implement a long-running custom cancellation workflow. Test cancel/retry/recovery explicitly before extending it to long jobs. For a remote registry, use an administrator-maintained HTTPS allowlist and verified credentials; never follow arbitrary agent-card URLs received from a prompt.
