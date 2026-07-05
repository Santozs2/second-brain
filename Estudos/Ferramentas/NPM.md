---
type: tech
area: Estudos
status: explorar
tecnologia: NPM
tags:
  - tech
  - estudo
  - ferramenta
created: 2026-07-05
updated: 2026-07-05
---
# NPM

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

NPM (Node Package Manager) é o gerenciador de pacotes padrão do Node.js. Instala dependências, versiona projetos via `package.json` e roda scripts de build e desenvolvimento.

## 🧠 Conceitos principais

- **`package.json`**: metadados, dependências e scripts do projeto
- **`package-lock.json`**: trava versões exatas para builds reprodutíveis
- **dependencies vs devDependencies**: runtime vs desenvolvimento
- **Versionamento semântico**: `^`, `~`, ranges (`major.minor.patch`)
- **Scripts**: `npm run <script>`, ciclo `install`/`test`/`build`
- **npx**: executa binários de pacotes sem instalar globalmente

## 💻 Exemplo

```bash
npm init -y
npm install express          # dependência
npm install -D vitest        # devDependency
npm run dev                  # roda script "dev"
npx create-vite@latest app   # executa sem instalar
```

## 🔗 Links úteis

- [Documentação oficial NPM](https://docs.npmjs.com/)

## 🔗 Veja também

- [[Node.js|Node.js]]
- [[JavaScript|JavaScript]]
- [[Webpack|Webpack]]
