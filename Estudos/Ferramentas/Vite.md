---
type: tech
area: Estudos
status: aprendendo
tecnologia: Vite
tags:
  - tech
  - estudo
  - frontend
  - ferramenta
created: 2026-08-24
updated: 2026-08-24
---
# ⚡ Vite

> [!tip] Status
> 🟢 Em uso

## 📝 Resumo

Build tool de frontend criada por Evan You (o autor do [[Vue|Vue]]). Resolve o problema que tornava [[Webpack|Webpack]] lento: em desenvolvimento, **não empacota nada** — serve os módulos direto pelo `import` nativo do navegador (ESM).

```
Webpack (dev):  empacota TUDO  →  depois serve      → inicialização lenta
Vite (dev):     serve direto   →  transforma sob demanda → inicialização instantânea
```

Em produção, aí sim, empacota — com **Rollup**, otimizado.

## 🧠 Conceitos principais

- **ESM nativo em dev** — o navegador resolve os imports; o servidor só transforma o arquivo pedido
- **esbuild para pré-bundling** — dependências de `node_modules` são pré-processadas em Go, ordens de magnitude mais rápido que ferramentas em JS
- **HMR granular** — ao salvar, só o módulo alterado é substituído; o estado da aplicação sobrevive
- **Rollup no build** — *tree shaking*, divisão de código e minificação
- **Config mínima** — funciona sem arquivo de configuração na maioria dos casos

## 💻 Uso

```bash
npm create vite@latest meu-app -- --template react-ts
cd meu-app && npm install && npm run dev
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      // Evita CORS em dev: /api vai para o backend
      "/api": { target: "http://localhost:8000", changeOrigin: true },
    },
  },
});
```

## 🔑 Variáveis de ambiente

```env
# .env — só o prefixo VITE_ é exposto ao cliente
VITE_API_URL=http://localhost:8000/api
```

```ts
const api = import.meta.env.VITE_API_URL;
```

> [!warning] Tudo com prefixo `VITE_` vai para o bundle público
> O valor é embutido no JavaScript entregue ao navegador. **Nunca coloque chave de API secreta ali** — qualquer pessoa lê no DevTools. Segredo fica no backend, sempre.

## 📜 Scripts

| Comando | O que faz |
|---|---|
| `npm run dev` | Servidor de desenvolvimento com HMR |
| `npm run build` | Build de produção em `dist/` |
| `npm run preview` | Serve o build local, para conferir antes de publicar |

## ⚖️ Vite × Webpack

| | Vite | Webpack |
|---|---|---|
| Início do dev server | Instantâneo | Cresce com o projeto |
| HMR | Constante | Degrada com o tamanho |
| Configuração | Mínima | Extensa |
| Ecossistema de plugins | Bom | Maior e mais maduro |
| Casos exóticos de build | 🟡 | ✅ |

## ⚠️ Pontos de atenção

- Comportamento de dev (ESM) e de produção (Rollup) **não são idênticos** — sempre teste o build antes de publicar
- Bibliotecas antigas em CommonJS podem exigir configuração de `optimizeDeps`
- `import.meta.env` é específico do Vite; código compartilhado com Node precisa de cuidado

## 🔗 Conceitos relacionados

- [[Webpack|Webpack]] · [[Babel|Babel]] · [[NPM|NPM]]
- [[JavaScript|JavaScript]] · [[TypeScript|TypeScript]]

## Veja também

- [[Estudos|📚 Estudos]]
- [[Documentações|Documentações]]
