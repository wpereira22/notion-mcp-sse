FROM node:20-alpine
RUN npm install -g supergateway @notionhq/notion-mcp-server
# Debug: Write the env var to a file
RUN echo "OPENAPI_MCP_HEADERS is: >>>$OPENAPI_MCP_HEADERS<<<" > /env_debug_output.txt
EXPOSE 8080
# Explicitly pass OPENAPI_MCP_HEADERS to the npx command within stdio
CMD supergateway --stdio "OPENAPI_MCP_HEADERS='$OPENAPI_MCP_HEADERS' npx -y @notionhq/notion-mcp-server" --outputTransport sse --port 8080 --cors --host 0.0.0.0 