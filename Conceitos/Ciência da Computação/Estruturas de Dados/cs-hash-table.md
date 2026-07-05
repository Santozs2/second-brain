---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-hash-table
category: Data Structures
tags:
  - data-structures
created: 2026-07-05
updated: 2026-07-05
---
# #️⃣ Hash Table (Tabela Hash)

> Estrutura O(1) que mapeia chaves para valores usando hash.

---

## 📖 Definição

**Hash Table**:
- **Função hash:** Transforma chave → índice
- **Colisões:** Múltiplas chaves mesmo índice
- **Load factor:** α = n / m (n itens, m slots)
- **Tempo médio:** O(1) inserção/busca/deleção

---

## 🧠 Funcionamento

```
Chave "João" 
  ↓
hash("João") = 12345
  ↓
12345 % table_size = 3
  ↓
table[3] = Valor
```

---

## 💥 Resolução de Colisões

### 1. Chaining (Encadeamento)
```python
# Múltiplas chaves na mesma posição (linked list)
class HashTable:
    def __init__(self, size):
        self.size = size
        self.table = [[] for _ in range(size)]
    
    def insert(self, key, value):
        index = hash(key) % self.size
        for i, (k, v) in enumerate(self.table[index]):
            if k == key:
                self.table[index][i] = (key, value)
                return
        self.table[index].append((key, value))
    
    def search(self, key):
        index = hash(key) % self.size
        for k, v in self.table[index]:
            if k == key:
                return v
        return None
```

### 2. Open Addressing (Linear Probing)
```python
# Se colisão, tenta próxima posição
def insert(self, key, value):
    index = hash(key) % self.size
    while self.table[index] is not None:
        index = (index + 1) % self.size  # Próxima posição
    self.table[index] = (key, value)
```

### 3. Quadratic Probing
```python
# Próxima posição: (hash + i²) % size
index = (hash(key) + i**2) % self.size
```

---

## 📊 Complexidade

| Operação | Médio | Pior |
|----------|-------|------|
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Search | O(1) | O(n) |

Pior caso: Todas chaves colidem (raramente acontece)

---

## 🎯 Load Factor & Rehashing

```
α = número de itens / número de slots

- α < 0.5: Muito espaço vazio (desperdício)
- α = 0.7: Ideal
- α > 0.9: Muitas colisões (rehash necessário)

Ao rehashing:
- Cria nova tabela (2x tamanho)
- Re-insere todos items
- Tempo: O(n)
```

---

## 💻 Python Built-in

```python
# Dict é hash table
d = {'name': 'João', 'age': 30}
d['name']  # O(1)

# Set é hash table sem valores
s = {1, 2, 3}
2 in s  # O(1)
```

---

## 🎯 Casos de Uso

✅ **Cache (memcached)**  
✅ **Índices de BD**  
✅ **Deduplicação**  
✅ **Contagem de frequência**  
✅ **Lookup em O(1)**

---

## ❓ Entrevista

1. Implementar hash table com chaining
2. Detectar itens duplicados (O(n) time, O(1) space?)
3. Two sum (encontrar pares que somam alvo)
4. Anagrams (agrupar strings anagrama)
5. LRU Cache

---

## 🔗 Relacionado

- [[cs-array|Array]]
- [[cs-linked-list|Linked List]]

---

**Status:** ✅ Completo
