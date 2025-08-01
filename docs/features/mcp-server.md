# MCP Server

Chango MCP Server is used to expose tools like Trino tools and OpenSearch tools to LLM Agents.

<img width="900" src="../../images/mcp/mcp-server.png" />

- RBAC: Before executing tool queries, all the requested tool queries will be authenticated and authorized by authorizer.
- Trino Tools: Tool queries of trino will be executed to query data of the catalogs in Trino.
- OpenSearch Tools: Tool queries of opensearch used as embedding store will be executed to query the indices in OpenSearch.

