FROM node:20-alpine
RUN npm install -g supergateway @notionhq/notion-mcp-server
EXPOSE 8080
CMD ["supergateway", "--stdio","npx -y @notionhq/notion-mcp-server", "--outputTransport","sse", "--port","8080", "--cors"] 