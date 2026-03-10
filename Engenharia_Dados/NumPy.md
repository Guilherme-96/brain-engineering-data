# 🔢 NumPy

> Computação numérica com arrays multidimensionais de alta performance em [[Python]].

**Relacionado:** [[Python]] | [[Pandas]] | [[Spark_PySpark]] | [[Algoritmos_e_Estruturas_de_Dados]]

**Instalação:** `pip install numpy`

---

## Criação de Arrays
```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])
matriz = np.array([[1, 2, 3], [4, 5, 6]])

# Arrays especiais
np.zeros((3, 4))              # matriz 3x4 de zeros
np.ones((2, 3))               # matriz 2x3 de uns
np.eye(3)                     # matriz identidade 3x3
np.arange(0, 10, 0.5)         # de 0 a 10 com passo 0.5
np.linspace(0, 1, 100)        # 100 pontos entre 0 e 1
np.random.rand(100)           # 100 valores aleatórios [0,1)
np.random.randn(100)          # distribuição normal
np.random.randint(0, 10, (3, 4))  # inteiros aleatórios
```

---

## Operações Vetorizadas
```python
# Muito mais rápido que loops Python!
arr = np.array([1, 2, 3, 4, 5])

arr * 2          # [2, 4, 6, 8, 10]
arr ** 2         # [1, 4, 9, 16, 25]
arr + 10         # [11, 12, 13, 14, 15]
np.sqrt(arr)
np.log(arr)
np.exp(arr)
np.abs(arr)
```

---

## Estatísticas
```python
np.mean(arr)
np.median(arr)
np.std(arr)
np.var(arr)
np.min(arr) / np.max(arr)
np.sum(arr)
np.cumsum(arr)               # soma acumulada
np.percentile(arr, [25, 50, 75])
np.corrcoef(arr1, arr2)      # correlação
```

---

## Indexação e Slicing
```python
arr = np.array([10, 20, 30, 40, 50])
arr[0]          # 10
arr[-1]         # 50
arr[1:4]        # [20, 30, 40]
arr[::2]        # [10, 30, 50] (passo 2)

# 2D
m = np.array([[1,2,3],[4,5,6],[7,8,9]])
m[1, 2]         # 6 (linha 1, coluna 2)
m[:, 0]         # [1, 4, 7] (toda coluna 0)
m[0, :]         # [1, 2, 3] (toda linha 0)

# Indexação booleana
mask = arr > 25
arr[mask]       # [30, 40, 50]
arr[arr % 2 == 0]  # pares
```

---

## Reshape e Shape
```python
arr = np.arange(12)
arr.reshape(3, 4)     # 3 linhas, 4 colunas
arr.reshape(2, -1)    # -1 = inferir automaticamente
arr.flatten()         # transforma em 1D
arr.T                 # transposta

np.concatenate([arr1, arr2], axis=0)   # empilhar verticalmente
np.concatenate([arr1, arr2], axis=1)   # empilhar horizontalmente
np.vstack([arr1, arr2])
np.hstack([arr1, arr2])
```

---

## Broadcasting
```python
# Arrays de shapes diferentes operam automaticamente
a = np.array([[1], [2], [3]])   # shape (3,1)
b = np.array([10, 20, 30])      # shape (3,)
resultado = a + b
# [[11, 21, 31],
#  [12, 22, 32],
#  [13, 23, 33]]
```

---

## Por que NumPy é mais rápido?
- Arrays armazenam elementos do **mesmo tipo** em memória contígua
- Operações implementadas em **C** internamente
- Evita o overhead do interpretador Python em cada elemento
- Ideal para milhões de operações numéricas
