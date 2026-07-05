---
type: tech
area: Estudos
status: explorar
tecnologia: Node.js
tags:
  - tech
  - estudo
  - backend
created: 2026-07-05
updated: 2026-07-05
---
# Node.js

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Node.js é um runtime que executa JavaScript no servidor usando o motor V8. É orientado a eventos e non-blocking, ideal para APIs, ferramentas de linha de comando e serviços de I/O intenso.

## 🧠 Conceitos principais

- **Event loop** e I/O assíncrono não-bloqueante
- **Módulos**: CommonJS (`require`) e ESM (`import`)
- **npm**: gerenciamento de pacotes e scripts
- **Streams e Buffers**: processamento de dados em fluxo
- **Cluster / Worker Threads**: paralelismo
- **APIs nativas**: `fs`, `http`, `path`, `events`

## 💻 Exemplo

```js
import { createServer } from "node:http";

createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ ok: true }));
}).listen(3000);
```

## 🔗 Links úteis

- [Documentação oficial Node.js](https://nodejs.org/docs/latest/api/)

## 📖 Aprofundar

- [[fw-nodejs-overview|Guia detalhado de Node.js]] — async, módulos, streams, cluster e deploy

## 🔗 Veja também

- [[JavaScript|JavaScript]]
- [[Express|Express]]
- [[TypeScript|TypeScript]]
