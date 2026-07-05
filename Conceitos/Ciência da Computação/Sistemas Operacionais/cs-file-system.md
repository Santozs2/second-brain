---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-file-system
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
---
# 📁 File System

> Como SO organiza arquivos em disco.

---

## 🏗️ Estrutura

### Inode (i-node)
```
Metadados:
├── Owner
├── Size
├── Timestamps
├── Block pointers (12 diretos + indiretos)
└── Permissions
```

### Diretório
```
Mapeamento: nome → inode number
/home/user/file.txt → inode 12345
```

---

## 🔗 Links

**Hard Link:** Mesmo inode, múltiplos nomes  
**Soft Link:** Aponta para outro path

---

## 📊 Alocação

### Contígua
```
Arquivo em blocos consecutivos
Rápido, mas fragmentação
```

### Linked
```
Blocos ligados por ponteiros
Sem fragmentação, acesso lento
```

### Indexado
```
Inode aponta para blocos
Melhor dos dois mundos
```

---

## 💾 Journaling

ext4, NTFS usam journals para evitar corrupção

---

**Status:** ✅ Completo
