---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-08-24
---
# JWT

## Definição

JSON Web Token: padrão para autenticação stateless. Um token assinado que carrega informações ("claims") do usuário, sem precisar manter sessão no servidor.

## Anatomia

Três partes em Base64URL, separadas por ponto:

```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiIxMjMiLCJleHAiOjE3...} . dBjftJeZ4CVP-mB92K
└─── HEADER ────────┘   └────── PAYLOAD ────────────┘   └─── SIGNATURE ───┘
```

```json
// Header — qual algoritmo assinou
{ "alg": "HS256", "typ": "JWT" }

// Payload — os claims
{ "sub": "123", "exp": 1735689600, "iat": 1735686000, "org_id": 7 }

// Signature — HMAC(header + "." + payload, segredo)
```

## Claims padrão

| Claim | Significado |
|---|---|
| `sub` | Subject — quem é o usuário |
| `exp` | Expiration — timestamp de expiração |
| `iat` | Issued At — quando foi emitido |
| `nbf` | Not Before — não válido antes de |
| `iss` | Issuer — quem emitiu |
| `aud` | Audience — para quem se destina |
| `jti` | JWT ID — identificador único (permite revogação) |

## Quando usar

Para autenticar usuários em uma [[REST API|REST API]] sem depender de sessões guardadas no servidor.

**Quando NÃO usar:** aplicação web tradicional com servidor único, onde sessão em cookie é mais simples e mais segura. JWT existe para resolver escala horizontal e clientes heterogêneos — se você não tem esse problema, ele adiciona complexidade sem benefício.

## Boas práticas

- Tokens de curta duração + refresh token
- **Nunca guardar dados sensíveis no payload** (o token é assinado, não criptografado — qualquer um pode ler o conteúdo)
- Sempre validar o token no backend a cada requisição
- Sempre HTTPS — token interceptado é acesso concedido
- Validar `exp`, `iss` e `aud`, não só a assinatura

> [!warning] Assinado ≠ criptografado
> Qualquer pessoa pode colar o token em jwt.io e ler o payload inteiro. A assinatura garante que ele **não foi alterado**, não que seja secreto. CPF, e-mail e permissões detalhadas não pertencem ali.

## 🚨 Os ataques clássicos

### `alg: none`
Bibliotecas antigas aceitavam um token com algoritmo `none` e assinatura vazia — permitindo forjar qualquer payload.

### Confusão de algoritmo (RS256 → HS256)
O atacante troca o algoritmo para HMAC e usa a **chave pública** (que é pública) como segredo. Se a biblioteca decide o algoritmo pelo header, ela valida com sucesso.

```python
# ❌ Confia no header do token
jwt.decode(token, chave)

# ✅ Exige o algoritmo esperado
jwt.decode(token, chave, algorithms=["HS256"], audience="minha-api")
```

> [!important] Nunca deixe o token dizer como ele deve ser validado
> O algoritmo é decisão do servidor, não informação do cliente. Sempre passe `algorithms=[...]` explicitamente.

## O problema da revogação

JWT é **stateless** — e é por isso que não dá para deslogar de verdade.

```
Usuário clica "sair" → o token continua válido até expirar
Conta comprometida  → o token continua válido até expirar
Permissão revogada  → o token antigo ainda carrega a permissão antiga
```

Mitigações, em ordem de custo:

| Solução | Custo |
|---|---|
| Access token curto (5–15 min) | 🟢 baixo — a janela de exposição fica pequena |
| Lista de revogação por `jti` | 🟡 reintroduz estado (Redis) |
| Rotação de refresh token | 🟡 detecta reuso de token roubado |
| Trocar o segredo | 🔴 invalida todos os usuários |

## Onde armazenar no cliente

| Local | XSS | CSRF | Comentário |
|---|:---:|:---:|---|
| `localStorage` | 🔴 vulnerável | 🟢 imune | Simples; qualquer script lê |
| Cookie `HttpOnly` + `SameSite` | 🟢 protegido | 🟡 mitigável | Mais seguro; exige config |
| Memória (variável JS) | 🟡 | 🟢 | Perde no refresh da página |

> [!tip] O padrão mais seguro na prática
> Access token curto **em memória**, refresh token em cookie `HttpOnly; Secure; SameSite=Strict`. O XSS não alcança nenhum dos dois de forma persistente, e o refresh sobrevive ao recarregamento da página.

Ver [[cs-xss|🚨 XSS]] e [[cs-csrf|🎣 CSRF]].

## Conceitos relacionados

- [[REST API|REST API]] · [[HTTP|HTTP]] · [[cs-authentication|🔑 Autenticação]]
- [[cs-hash|🔐 Hash]] · [[cs-xss|🚨 XSS]] · [[cs-csrf|🎣 CSRF]] · [[cs-ssl-tls|🔒 SSL/TLS]]

## Veja também

- [[Documentações|Documentações]]
