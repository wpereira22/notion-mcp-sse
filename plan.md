# ✅ Deployment Checklist — Notion MCP → public SSE on Render

> Tick each box in order. When everything is checked, your Notion MCP server streams at `https://<service>.onrender.com/sse` and is usable by n8n’s **MCP Client Tool** node.

---

## 0 . Prerequisites
- [ ] Git, Docker, and Node ≥ 18 installed locally  
- [ ] Notion internal‑integration token (`ntn_xxx…`) with workspace access  
- [ ] Free Render account linked to GitHub  

## 1 . Local Notion MCP server
- [ ] `mkdir notion‑mcp && cd notion‑mcp`
- [ ] `npm init -y`
- [ ] `npm install @notionhq/notion-mcp-server`
- [ ] Test (STDIO):  
  ```bash
  OPENAPI_MCP_HEADERS='{"Authorization":"Bearer ntn_xxx","Notion-Version":"2022-06-28"}' \
  npx @notionhq/notion-mcp-server

Expect a “ready” JSON message.

2 . Wrap STDIO with Supergateway
	•	npm install -D supergateway
	•	Add NPM script in package.json:

"scripts": {
  "serve": "supergateway \
    --stdio \"npx -y @notionhq/notion-mcp-server\" \
    --outputTransport sse \
    --port 8080 --cors"
}


	•	Run: npm run serve
	•	Confirm: curl -N http://localhost:8080/sse streams ready

3 . Docker‑ize the combo
	•	Create Dockerfile:

FROM node:20-alpine
ENV OPENAPI_MCP_HEADERS='{"Authorization":"Bearer ntn_xxx","Notion-Version":"2022-06-28"}'
RUN npm install -g supergateway @notionhq/notion-mcp-server
EXPOSE 8080
CMD ["supergateway",
     "--stdio","npx -y @notionhq/notion-mcp-server",
     "--outputTransport","sse",
     "--port","8080",
     "--cors"]


	•	docker build -t notion-mcp-sse .
	•	docker run -p 8080:8080 notion-mcp-sse
	•	Re‑test /sse locally.

4 . Push to GitHub
	•	git init && git add . && git commit -m "Notion MCP SSE"
	•	git remote add origin https://github.com/<you>/notion-mcp-sse.git
	•	git push -u origin main

5 . Render Web Service
	•	New → Web Service → “Docker”
	•	Connect repo
	•	Env var OPENAPI_MCP_HEADERS = same JSON value
	•	Leave port blank (auto‑detect)
	•	Health check path /sse
	•	Click Create Web Service and wait for green deploy

6 . Validate public endpoints
	•	curl -N https://<service>.onrender.com/sse streams events
	•	Test call:

curl -X POST https://<service>.onrender.com/message \
     -H "Content-Type: application/json" \
     -d '{"jsonrpc":"2.0","id":"1","method":"tool.list"}'



7 . n8n Integration

7.1 Add MCP Client Tool node
	•	Drag MCP Client Tool node into your workflow.

7.2 Credentials
	•	Choose Bearer if your server expects Authorization: Bearer <token>.
	•	Choose Generic Header for custom auth headers.
	•	Choose None if no auth needed.

7.3 Parameters

Parameter	What to enter
SSE Endpoint	https://<service>.onrender.com/sse
Authentication	One of None / Bearer / Generic Header
Tools to Include	- All: expose every server tool. - Selected: choose specific tools via Tools to Include list. - All Except: select tools to hide via Tools to Exclude.

7.4 Send actions
	•	Add HTTP Request node (POST → https://<service>.onrender.com/message) with raw JSON body per MCP spec.
	•	Connect Request node → MCP Client Tool node; responses stream back automatically.

8 . Optional hardening
	•	Add auth guard in Supergateway (--basicUser, --basicPassword, or secret param)
	•	Enable auto‑deploys on Render
	•	Turn on Render alerts for restarts/build failures
	•	Monitor bandwidth; SSE holds long‑lived connections

⸻

All done!

