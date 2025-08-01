# Connect Chango MCP Server

Chango MCP Server will expose tools like trino and opensearch tools to LLM Agents. 
This shows how to connect Chango MCP Server by mcp client.


## Connect with MCP Client in Java

There is Java MCP SDK to connect MCP Server. 
Tools of Chango MCP Server will be exposed through NGINX Proxy to support HTTPS(TLS) with self-signed certificate by default.

You need to construct the following client connecting through TLS with self-signed certificate.
```agsl
import io.modelcontextprotocol.client.McpClient;
import io.modelcontextprotocol.client.McpSyncClient;
import io.modelcontextprotocol.client.transport.HttpClientSseClientTransport;
import io.modelcontextprotocol.spec.McpClientTransport;
import io.modelcontextprotocol.spec.McpSchema;
import javax.net.ssl.*;
import java.net.http.HttpClient;
import java.security.cert.X509Certificate;
import java.net.URI;
import java.net.http.HttpRequest;
import java.time.Duration;
import java.util.Map;

...

    private static HttpClient.Builder createUnsafeHttpClient() throws Exception {
        // Trust manager that does not validate certificate chains
        TrustManager[] trustAllCerts = new TrustManager[]{
                new X509TrustManager() {
                    public void checkClientTrusted(X509Certificate[] chain, String authType) { }
                    public void checkServerTrusted(X509Certificate[] chain, String authType) { }
                    public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
                }
        };

        // Install the all-trusting trust manager
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null, trustAllCerts, new java.security.SecureRandom());

        // Disable hostname verification
        SSLParameters sslParams = sslContext.getDefaultSSLParameters();
        sslParams.setEndpointIdentificationAlgorithm(null);  // disables hostname verification

        // Build HttpClient
        return HttpClient.newBuilder()
                .sslContext(sslContext)
                .sslParameters(sslParams)
                .connectTimeout(Duration.ofSeconds(30))
                .version(HttpClient.Version.HTTP_1_1);
    }
```


And create mcp client with this transport client for self-signed certificate.  In the following, 
`accessToken` which is api key issued by Chango is used to call tools of Chango MCP Server.
```agsl
        String url = "https://cp-2.chango.private:443"; 

        // access token to mcp server.
        String accessToken = "...";

        McpClientTransport transport = HttpClientSseClientTransport.builder(url)
                .clientBuilder(createUnsafeHttpClient()) // Http client for self-signed certificate.
                .requestBuilder(HttpRequest.newBuilder()
                        .uri(URI.create(url))
                        .header("Authorization", "Bearer " + accessToken))
                .build();

        McpSyncClient client = McpClient.sync(transport)
                .requestTimeout(Duration.ofSeconds(30))
                .build();

        // Initialize connection
        client.initialize();

        // List all tools provided by Chango MCP Server.
        McpSchema.ListToolsResult toolsList = client.listTools();
        LOG.info("Available Tools = " + toolsList);
        toolsList.tools().stream().forEach(tool -> {
            LOG.info("Tool: " + tool.name() + ", description: " + tool.description() + ", schema: " + tool.inputSchema());
        });

        // For example, call a tool like listing catalogs of trino.
        McpSchema.CallToolResult listCatalogsResult =
                client.callTool(new McpSchema.CallToolRequest("listCatalogsWithTrino",
                Map.of()));
                
        ...
```



