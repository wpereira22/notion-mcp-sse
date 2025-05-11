FROM node:20-alpine
ENV OPENAPI_MCP_HEADERS={"Authorization":"Bearer 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","Notion-Version":"2022-06-28"}
RUN npm install -g supergateway @notionhq/notion-mcp-server
EXPOSE 8080
CMD ["supergateway", "--stdio","npx -y @notionhq/notion-mcp-server", "--outputTransport","sse", "--port","8080", "--cors"] 