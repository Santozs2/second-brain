---
type: concept
area: Conceitos
status: stable
difficulty: beginner
id: cs-array
category: Data Structures
tags:
  - data-structures
created: 2026-07-04
updated: 2026-07-04
---
# 📦 Array (Vetor)

> Coleção de elementos do mesmo tipo em posições contíguas de memória.

---

## 📖 Definição

Um **array** é:
- **Sequencial:** Elementos em memória contígua
- **Homogêneo:** Todos mesmo tipo
- **Indexado:** Acesso por índice (0, 1, 2...)
- **Tamanho fixo:** Número de elementos definido

---

## 🧠 Funcionamento Interno

### Memória
```
Array: [10, 20, 30, 40]
Índices: [0,  1,  2,  3]

Memória:
Address  │ Valor
─────────┼──────
1000     │ 10    ← array[0]
1004     │ 20    ← array[1]
1008     │ 30    ← array[2]
1012     │ 40    ← array[3]
```

### Cálculo de Endereço
```
endereço = endereço_base + (índice × tamanho_elemento)

Para array[2] de inteiros (4 bytes):
endereço = 1000 + (2 × 4) = 1008 ✓
```

---

## 💻 Operações

### Acesso (Access) - O(1)
```python
array = [10, 20, 30, 40]
valor = array[2]  # Retorna 30
```

**Motivo O(1):** Calcula endereço direto

### Busca (Search) - O(n)
```python
def buscar(array, alvo):
    for i, valor in enumerate(array):
        if valor == alvo:
            return i
    return -1

buscar([10, 20, 30, 40], 30)  # Retorna 2
```

### Inserção (Insertion) - O(n)
```python
def inserir(array, indice, valor):
    # Precisa deslocar elementos
    array.append(None)
    for i in range(len(array) - 1, indice, -1):
        array[i] = array[i - 1]
    array[indice] = valor
    return array

array = [10, 20, 40]
inserir(array, 2, 30)  # [10, 20, 30, 40]
```

**Complexidade:**
- Inserir no início: O(n) - desloca tudo
- Inserir no meio: O(n) - desloca metade
- Inserir no fim: O(1) - sem deslocar

### Deleção (Deletion) - O(n)
```python
def deletar(array, indice):
    for i in range(indice, len(array) - 1):
        array[i] = array[i + 1]
    array.pop()
    return array

array = [10, 20, 30, 40]
deletar(array, 2)  # [10, 20, 40]
```

---

## 📊 Comparação de Operações

| Operação | Array | Linked List | Hash Table |
|----------|-------|-------------|-----------|
| Acesso | O(1) ✅ | O(n) | O(1) ✅ |
| Busca | O(n) | O(n) | O(1) ✅ |
| Inserção | O(n) | O(1) ✅ | O(1) ✅ |
| Deleção | O(n) | O(1) ✅ | O(1) ✅ |

---

## ✅ Vantagens

✅ **Acesso Rápido** - O(1)  
✅ **Uso de Memória Eficiente** - Contíguo  
✅ **Cache-Friendly** - Localidade espacial  
✅ **Simples de Usar** - Suportado nativamente  

---

## ❌ Desvantagens

❌ **Tamanho Fixo** - Não cresce dinamicamente  
❌ **Inserção/Deleção Cara** - O(n)  
❌ **Espaço Desperdiçado** - Se não preencher tudo  

---

## 🔄 Dynamic Array (Vector)

### Redimensionamento

```python
class DynamicArray:
    def __init__(self):
        self.array = [None] * 10
        self.size = 0
        self.capacity = 10
    
    def append(self, valor):
        if self.size == self.capacity:
            # Redimensiona
            self.capacity *= 2
            novo_array = [None] * self.capacity
            for i in range(self.size):
                novo_array[i] = self.array[i]
            self.array = novo_array
        
        self.array[self.size] = valor
        self.size += 1
    
    def get(self, indice):
        return self.array[indice]

# Uso
arr = DynamicArray()
for i in range(20):
    arr.append(i)
```

### Complexidade Amortizada
```
Append:
- Comum: O(1)
- Redimensionamento: O(n), mas raro
- Amortizado: O(1)

Por quê? A cada redimensionamento duplica:
Inserções: 1, 2, 4, 8, 16, 32...
Custo: 1 + 2 + 4 + 8 + 16... = O(n) para n inserções
Amortizado: O(n) / n = O(1) por inserção
```

---

## 💡 Casos Reais

### 1. Sistema de Notas (Obsidian, Notion)
```
Array de notas
[Note1, Note2, Note3, ...]
Acesso rápido por ID
```

### 2. Rankings de Jogo
```
Jogadores = [Player1, Player2, Player3, ...]
Índice = posição no ranking
Acesso O(1): ranking[5] = 6º lugar
```

### 3. Cache de Pixels (Imagem)
```
Imagem 800×600
Array linear: 480,000 pixels
Acesso: pixel[y * 800 + x] = cor
```

---

## ❓ Perguntas de Entrevista

1. **Qual complexidade de acesso em array?**
   - O(1) - cálculo de endereço direto

2. **Por que inserção é O(n)?**
   - Precisa deslocar elementos após o índice

3. **Dynamic array vs Linked List para inserção no fim?**
   - Array: O(1) amortizado, Lista: O(1) exato

4. **Como encontrar elemento em array ordenado rápido?**
   - [[cs-search-algorithms|Binary Search]] - O(log n)

5. **Array de objetos vs array de ponteiros?**
   - Objetos: mais memória, mas cache-friendly
   - Ponteiros: menos memória, mais cache-misses

---

## 📝 Exercícios

1. Implemente Dynamic Array
2. Teste inserção em posições diferentes
3. Compare tempo vs Linked List
4. Implemente busca binária
5. Encontre maior elemento - O(n)

---

## 🔗 Referências Cruzadas

- [[cs-big-o|Big O Notation]]
- [[cs-search-algorithms|Algoritmos de Busca]]
- [[cs-sorting-algorithms|Algoritmos de Ordenação]]
- [[cs-linked-list|Linked List]]
- [[cs-dynamic-programming|Programação Dinâmica]]

---

**Próximo:** [[cs-linked-list|Linked List]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐ Iniciante  
**Tempo de Leitura:** 15-20 minutos
