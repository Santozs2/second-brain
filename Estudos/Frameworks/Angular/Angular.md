---
type: tech
area: Estudos
status: explorar
tecnologia: Angular
tags:
  - tech
  - estudo
  - frontend
created: 2026-07-05
updated: 2026-07-05
---
# Angular

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Angular é um framework completo (baseado em TypeScript) para aplicações web de grande porte. Traz roteamento, injeção de dependência, formulários e HTTP client integrados por padrão.

## 🧠 Conceitos principais

- **Componentes** e **templates** com data binding
- **Injeção de dependência** e **Services**
- **Diretivas** (`*ngIf`, `*ngFor`) e **Pipes**
- **RxJS**: streams e observables
- **Roteamento** e **lazy loading**
- **Formulários**: template-driven e reactive forms

## 💻 Exemplo

```ts
@Component({
  selector: "app-contador",
  template: `<button (click)="inc()">Cliques: {{ contagem }}</button>`,
})
export class ContadorComponent {
  contagem = 0;
  inc() { this.contagem++; }
}
```

## 🔗 Links úteis

- [Documentação oficial Angular](https://angular.dev/)

## 📖 Aprofundar

- [[fw-angular-overview|Guia detalhado de Angular]] — components, services, routing, forms e testes

## 🔗 Veja também

- [[TypeScript|TypeScript]]
- [[React|React]]
