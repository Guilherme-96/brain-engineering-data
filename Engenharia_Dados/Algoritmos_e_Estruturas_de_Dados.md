# 🧮 Algoritmos e Estruturas de Dados

> Fundamentos de Ciência da Computação. Essencial para escrever código eficiente e resolver problemas complexos.

**Relacionado:** [[Python]] | [[NumPy]] | [[Pandas]] | [[Spark_PySpark]] | [[Itertools]]

---

## Notação Big O — Complexidade

### O que é?
Mede como o tempo de execução (ou uso de memória) cresce conforme o tamanho da entrada `n`.

### Tabela de Complexidades
| Notação | Nome | 10 elementos | 100 elementos | 1.000 elementos |
|---------|------|:---:|:---:|:---:|
| O(1) | Constante | 1 | 1 | 1 |
| O(log n) | Logarítmica | 3 | 7 | 10 |
| O(n) | Linear | 10 | 100 | 1.000 |
| O(n log n) | Log-linear | 30 | 700 | 10.000 |
| O(n²) | Quadrática | 100 | 10.000 | 1.000.000 |
| O(2ⁿ) | Exponencial | 1.024 | 10^30 | 10^301 |

### Exemplos Práticos
```python
# O(1) — tempo constante (independe do tamanho)
def primeiro(lista: list):
    return lista[0]

dicionario = {"a": 1, "b": 2}
dicionario["a"]       # O(1) — hash map

# O(log n) — divide o problema pela metade a cada passo
def busca_binaria(lista: list, alvo: int) -> int:
    esq, dir = 0, len(lista) - 1
    while esq <= dir:
        meio = (esq + dir) // 2
        if lista[meio] == alvo:
            return meio
        elif lista[meio] < alvo:
            esq = meio + 1
        else:
            dir = meio - 1
    return -1

# O(n) — percorre cada elemento uma vez
def busca_linear(lista: list, alvo):
    for item in lista:
        if item == alvo:
            return True
    return False

# O(n log n) — ordenação eficiente
lista.sort()             # TimSort — O(n log n)
sorted(lista)

# O(n²) — loop aninhado — evitar para n grande!
def tem_duplicata_lento(lista):
    for i in range(len(lista)):
        for j in range(i+1, len(lista)):
            if lista[i] == lista[j]:
                return True
    return False

# O(n) — versão otimizada usando set
def tem_duplicata_rapido(lista):
    return len(lista) != len(set(lista))
```

---

## Estruturas de Dados Principais

### Lista / Array — O(1) acesso, O(n) busca
```python
lista = [1, 2, 3, 4, 5]
lista.append(6)          # O(1) amortizado
lista.insert(0, 0)       # O(n) — desloca elementos
lista.pop()              # O(1) — remove do final
lista.pop(0)             # O(n) — remove do início
lista[2]                 # O(1) — acesso por índice
2 in lista               # O(n) — busca linear
```

### Dicionário (Hash Map) — O(1) médio
```python
from collections import defaultdict, Counter, OrderedDict

d = {}
d["chave"] = "valor"     # O(1) inserção
d["chave"]               # O(1) acesso
"chave" in d             # O(1) busca
del d["chave"]           # O(1) remoção

# defaultdict — sem KeyError para chaves novas
contagem = defaultdict(int)
for item in "banana":
    contagem[item] += 1  # não precisa verificar se existe

# Counter — conta ocorrências automaticamente
from collections import Counter
c = Counter("banana")    # {'a': 3, 'n': 2, 'b': 1}
c.most_common(2)         # [('a', 3), ('n', 2)]
```

### Set (Conjunto) — O(1) busca, sem duplicatas
```python
visitados = set()
visitados.add("url1")    # O(1)
"url1" in visitados      # O(1) — MUITO mais rápido que lista!
visitados.discard("url1")# O(1) — sem erro se não existir

# Operações de conjunto
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
a & b    # {3, 4}      — intersecção
a | b    # {1,2,3,4,5,6} — união
a - b    # {1, 2}      — diferença
a ^ b    # {1,2,5,6}   — diferença simétrica
```

### Deque — O(1) inserção/remoção em ambos os lados
```python
from collections import deque

fila = deque()
fila.append(1)          # inserir no final — O(1)
fila.appendleft(0)      # inserir no início — O(1)
fila.pop()              # remover do final — O(1)
fila.popleft()          # remover do início — O(1)
# Use deque em vez de lista quando precisar de fifo!
```

### Heap / Fila de Prioridade — O(log n) inserção
```python
import heapq

# Min heap (menor elemento sempre no topo)
heap = [3, 1, 4, 1, 5, 9]
heapq.heapify(heap)              # O(n) — transforma in-place
heapq.heappush(heap, 2)          # O(log n)
menor = heapq.heappop(heap)      # O(log n)

# Top N elementos maiores/menores
maiores = heapq.nlargest(3, heap)
menores = heapq.nsmallest(3, heap)

# Max heap (Python só tem min heap nativamente)
# Truque: inserir valores negativos
heapq.heappush(heap, -valor)
maior = -heapq.heappop(heap)
```

---

## Algoritmos de Ordenação

```python
# Python usa TimSort (híbrido MergeSort + InsertionSort) — O(n log n)
lista.sort()              # in-place
sorted(lista)             # retorna nova lista
sorted(lista, key=lambda x: x[1], reverse=True)  # com critério

# Busca binária (lista deve estar ordenada!)
import bisect
lista = sorted([1, 3, 5, 7, 9])
bisect.bisect_left(lista, 5)    # índice onde 5 se encaixaria
bisect.insort(lista, 6)         # insere mantendo ordenação
```

---

## Técnicas de Otimização

### 1. Prefira Set para buscas repetidas
```python
lista_grande = list(range(1_000_000))
conjunto = set(lista_grande)

# Ruim: O(n) por busca
999_999 in lista_grande   # ~0.1s

# Bom: O(1) por busca
999_999 in conjunto       # ~0.000001s
```

### 2. Use generators para dados grandes
```python
# Carrega TUDO na memória
dados = [linha.strip() for linha in open("gigante.txt")]

# Processa linha por linha — eficiente!
def processar_linhas(arquivo: str):
    with open(arquivo) as f:
        for linha in f:
            yield linha.strip()
```

### 3. Prefira comprehensions a loops + append
```python
# Lento
resultado = []
for x in range(10000):
    if x % 2 == 0:
        resultado.append(x ** 2)

# Rápido (~2x mais rápido)
resultado = [x**2 for x in range(10000) if x % 2 == 0]

# Mais rápido ainda (NumPy vetorizado, ~100x mais rápido)
import numpy as np
arr = np.arange(10000)
resultado = arr[arr % 2 == 0] ** 2
```

### 4. Cache com lru_cache
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n: int) -> int:
    if n < 2: return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(100)    # O(n) com cache vs O(2^n) sem cache
```

### 5. String concatenation — use join
```python
# Ruim: O(n²) — cria nova string a cada iteração
resultado = ""
for palavra in palavras:
    resultado += palavra + " "

# Bom: O(n)
resultado = " ".join(palavras)
```

---

## Recursão vs Iteração
```python
# Recursão — elegante mas pode estourar stack (Python ~1000 calls)
def fatorial_recursivo(n: int) -> int:
    if n <= 1: return 1
    return n * fatorial_recursivo(n - 1)

# Iterativo — mais eficiente em Python
def fatorial_iterativo(n: int) -> int:
    resultado = 1
    for i in range(2, n + 1):
        resultado *= i
    return resultado

# Aumentar limite de recursão (não recomendado)
import sys
sys.setrecursionlimit(10000)

# Tailcall optimization manual com generator
# Ou usar sys.setrecursionlimit apenas se necessário
```

---

## Padrões de Algoritmos para Dados

### Sliding Window
```python
def media_movel(dados: list, janela: int) -> list:
    """Calcula média móvel com sliding window — O(n)."""
    soma = sum(dados[:janela])
    medias = [soma / janela]
    
    for i in range(janela, len(dados)):
        soma += dados[i] - dados[i - janela]
        medias.append(soma / janela)
    
    return medias
```

### Two Pointers
```python
def soma_alvo(lista: list, alvo: int) -> tuple:
    """Encontra dois números que somam o alvo — O(n) com lista ordenada."""
    esq, dir = 0, len(lista) - 1
    while esq < dir:
        soma = lista[esq] + lista[dir]
        if soma == alvo:
            return (lista[esq], lista[dir])
        elif soma < alvo:
            esq += 1
        else:
            dir -= 1
    return None
```
