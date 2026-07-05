---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-06-30
---
# JWT

## Definição

JSON Web Token: padrão para autenticação stateless. Um token assinado que carrega informações ("claims") do usuário, sem precisar manter sessão no servidor.

## Quando usar

Para autenticar usuários em uma [[REST API|REST API]] sem depender de sessões guardadas no servidor.

## Boas práticas

- Tokens de curta duração + refresh token
- Nunca guardar dados sensíveis no payload (o token é assinado, não criptografado — qualquer um pode ler o conteúdo)
- Sempre validar o token no backend a cada requisição

## Conceitos relacionados

- [[REST API|REST API]]
- [[HTTP|HTTP]]

## Veja também

- [[Documentações|Documentações]]
