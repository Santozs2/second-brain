---
type: tech
area: Estudos
status: aprendendo
tecnologia: Docker
tags:
  - tech
  - estudo
  - devops
created: 2026-06-30
updated: 2026-06-30
---
# Docker

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Docker permite empacotar aplicações e suas dependências em containers isolados e reproduzíveis, facilitando desenvolvimento e deploy.

## 🧠 Conceitos principais

- **Imagens vs containers**
- **Dockerfile**: receita para construir uma imagem
- **docker-compose**: orquestrar múltiplos containers (app + banco)
- **Volumes**: persistência de dados
- **Redes**: comunicação entre containers
- **Comandos básicos**: `build`, `run`, `ps`, `logs`, `exec`

## 💻 Exemplos

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: senha
```

## 🔗 Links úteis

- [Documentação oficial Docker](https://docs.docker.com/)
- [Docker Curriculum](https://docker-curriculum.com/)

## ✅ Checklist de aprendizado

- [ ] Dockerfile básico
- [ ] docker-compose com app + banco
- [ ] Volumes e persistência
- [ ] Deploy de container em produção

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[CI-CD|CI/CD]]

## 🔗 Veja também

- [[Linux|Linux]]
- [[Banco de Dados|Banco de Dados]]
