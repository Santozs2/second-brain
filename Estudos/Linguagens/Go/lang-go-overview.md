---
type: tech
area: Estudos
status: explorar
id: lang-go-overview
category: Go
tecnologia: Go
tags:
  - tech
  - estudo
  - backend
created: 2026-07-05
updated: 2026-07-05
---
# Go

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Go (Golang) é uma linguagem compilada, estaticamente tipada e criada no Google. Foca em simplicidade, compilação rápida e concorrência nativa — muito usada em backends, CLIs e infraestrutura (Docker, Kubernetes são escritos em Go).

## 🧠 Conceitos principais

- **Tipagem estática** com inferência (`:=`)
- **Concorrência**: goroutines (`go f()`) e channels
- **Structs e interfaces** (composição, sem herança)
- **Tratamento de erros** explícito (`if err != nil`)
- **Ponteiros** sem aritmética
- **Ferramentas embutidas**: `go build`, `go test`, `go fmt`, módulos

## 💻 Exemplo

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go func(n int) {
			defer wg.Done()
			fmt.Println("goroutine", n)
		}(i)
	}
	wg.Wait()
}
```

## 🔗 Links úteis

- [Documentação oficial Go](https://go.dev/doc/)
- [A Tour of Go](https://go.dev/tour/)

## ✅ Checklist de aprendizado

- [ ] Sintaxe e tipos
- [ ] Structs e interfaces
- [ ] Goroutines e channels
- [ ] Tratamento de erros
- [ ] Módulos e testes

## 🔗 Veja também

- [[Python|Python]]
- [[JavaScript|JavaScript]]
- [[Docker|Docker]]
