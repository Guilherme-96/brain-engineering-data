# 🔁 Itertools

> Ferramentas para iteração eficiente. Biblioteca padrão do [[Python]] — sem instalação.

**Relacionado:** [[Python]] | [[Pandas]] | [[Algoritmos_e_Estruturas_de_Dados]]

---

## Encadeamento
```python
import itertools

# chain — encadeia múltiplos iteráveis em um
list(itertools.chain([1,2], [3,4], [5]))
# [1, 2, 3, 4, 5]

# chain.from_iterable — achata lista de listas
listas = [[1,2], [3,4], [5,6]]
list(itertools.chain.from_iterable(listas))
# [1, 2, 3, 4, 5, 6]
```

---

## Combinatórias
```python
# product — produto cartesiano (como loop aninhado)
list(itertools.product("AB", [1, 2]))
# [('A',1), ('A',2), ('B',1), ('B',2)]

# combinations — combinações sem repetição
list(itertools.combinations([1, 2, 3], 2))
# [(1,2), (1,3), (2,3)]

# combinations_with_replacement — com repetição
list(itertools.combinations_with_replacement([1,2,3], 2))
# [(1,1), (1,2), (1,3), (2,2), (2,3), (3,3)]

# permutations — todas as ordens possíveis
list(itertools.permutations([1, 2, 3], 2))
# [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]
```

---

## Filtragem e Corte
```python
# islice — fatia iterável lazy (sem carregar na memória)
primeiros_5 = list(itertools.islice(range(1_000_000), 5))
# [0, 1, 2, 3, 4]

# takewhile — pega enquanto condição for True
list(itertools.takewhile(lambda x: x < 5, [1,2,3,4,5,6]))
# [1, 2, 3, 4]

# dropwhile — descarta enquanto condição for True
list(itertools.dropwhile(lambda x: x < 5, [1,2,3,4,5,6]))
# [5, 6]

# filterfalse — filtra o que NÃO atende a condição
list(itertools.filterfalse(lambda x: x%2==0, range(10)))
# [1, 3, 5, 7, 9]
```

---

## Agrupamento
```python
# groupby — agrupa elementos consecutivos iguais
# IMPORTANTE: a lista deve estar ordenada antes!
dados = sorted([("A",1), ("B",2), ("A",3)], key=lambda x: x[0])

for chave, grupo in itertools.groupby(dados, key=lambda x: x[0]):
    print(chave, list(grupo))
# A [('A', 1), ('A', 3)]
# B [('B', 2)]
```

---

## Infinitos
```python
# count — contador infinito
counter = itertools.count(start=1, step=2)
# 1, 3, 5, 7, 9...

# cycle — cicla infinitamente pela sequência
ciclico = itertools.cycle(["jan", "fev", "mar"])
# jan, fev, mar, jan, fev, mar...

# repeat — repete um valor
repetido = list(itertools.repeat(42, times=3))
# [42, 42, 42]
```

---

## Casos de Uso em Engenharia de Dados
```python
# Processar arquivos em batches sem usar memória excessiva
def processar_em_batches(iteravel, tamanho_batch):
    it = iter(iteravel)
    while True:
        batch = list(itertools.islice(it, tamanho_batch))
        if not batch:
            break
        yield batch

# Gerar combinações para teste de parâmetros (grid search)
params = {
    "batch_size": [32, 64, 128],
    "lr": [0.001, 0.01],
    "epochs": [10, 20]
}
for combo in itertools.product(*params.values()):
    config = dict(zip(params.keys(), combo))
    print(config)
```
