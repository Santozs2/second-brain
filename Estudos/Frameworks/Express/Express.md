---
type: tech
area: Estudos
status: explorar
tecnologia: Express
tags:
  - tech
  - estudo
  - backend
created: 2026-07-05
updated: 2026-07-05
---
# Express

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Express é um framework web minimalista para Node.js. Fornece roteamento, middlewares e utilitários de requisição/resposta para construir APIs e aplicações web com pouca cerimônia.

## 🧠 Conceitos principais

- **Roteamento**: `app.get/post/put/delete`, parâmetros e query
- **Middlewares**: funções `(req, res, next)` encadeadas
- **Tratamento de erros**: middleware de 4 argumentos
- **Autenticação**: JWT, sessions, Passport
- **Integração com banco**: Prisma, Sequelize, Mongoose

## 💻 Exemplo

```js
import express from "express";
const app = express();

app.use(express.json());

app.get("/usuarios/:id", (req, res) => {
  res.json({ id: req.params.id });
});

app.listen(3000);
```

## 🔗 Links úteis

- [Documentação oficial Express](https://expressjs.com/pt-br/)

## 📖 Aprofundar

- [[fw-express-overview|Guia detalhado de Express]] — routing, middleware, auth, database e testes

## 🔗 Veja também

- [[Node.js|Node.js]]
- [[JavaScript|JavaScript]]
- [[REST API|REST API]]
