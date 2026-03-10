# 🐍 Python

> Linguagem principal do Engenheiro de Dados. Versátil, legível e com ecossistema gigante.

**Relacionado:** [[Pandas]] | [[NumPy]] | [[Itertools]] | [[Pathlib]] | [[Requests]] | [[SQLAlchemy]] | [[Playwright]] | [[Selenium]] | [[OpenPyXL]] | [[Ambientes_Virtuais]] | [[pyenv]]

---

## Tipos de Dados
```python
# Primitivos
inteiro: int = 42
decimal: float = 3.14
texto: str = "engenharia"
booleano: bool = True
nulo: None = None

# Coleções
lista: list = [1, 2, 3]           # mutável, ordenada
tupla: tuple = (1, 2, 3)          # imutável, ordenada
dicionario: dict = {"a": 1}       # mutável, chave-valor
conjunto: set = {1, 2, 3}         # mutável, sem duplicatas
frozenset: frozenset = frozenset({1, 2})  # imutável
```

---

## Funções
```python
# Básica com type hints
def somar(a: int, b: int) -> int:
    return a + b

# Argumentos padrão
def saudacao(nome: str, prefixo: str = "Olá") -> str:
    return f"{prefixo}, {nome}!"

# *args e **kwargs
def flexivel(*args, **kwargs):
    print(args)    # tupla posicional
    print(kwargs)  # dicionário nomeado

# Lambda (função anônima)
dobrar = lambda x: x * 2
```

---

## Funções Built-in Essenciais
| Função | Uso |
|--------|-----|
| `map(fn, iter)` | Aplica função a cada elemento |
| `filter(fn, iter)` | Filtra por condição |
| `zip(a, b)` | Une iteráveis em pares |
| `enumerate(iter)` | Itera com índice |
| `sorted(iter)` | Retorna lista ordenada |
| `any(iter)` | True se algum for verdadeiro |
| `all(iter)` | True se todos forem verdadeiros |
| `isinstance(obj, tipo)` | Verifica tipo |
| `getattr(obj, 'attr')` | Acessa atributo por nome |
| `len(obj)` | Tamanho do objeto |

---

## Classes e POO
```python
class Pipeline:
    versao: str = "1.0"  # atributo de classe

    def __init__(self, nome: str, ativo: bool = True):
        self.nome = nome
        self.ativo = ativo
        self._historico: list = []

    def executar(self) -> None:
        if self.ativo:
            self._historico.append("executado")

    @staticmethod
    def validar_nome(nome: str) -> bool:
        return len(nome) > 0

    @classmethod
    def criar_inativo(cls, nome: str) -> "Pipeline":
        return cls(nome, ativo=False)

    @property
    def historico(self) -> list:
        return self._historico.copy()

    def __repr__(self) -> str:
        return f"Pipeline(nome={self.nome})"


# Herança
class PipelineETL(Pipeline):
    def __init__(self, nome: str, fonte: str, destino: str):
        super().__init__(nome)
        self.fonte = fonte
        self.destino = destino


# Dataclass — forma moderna e enxuta
from dataclasses import dataclass
from typing import Optional

@dataclass
class Configuracao:
    host: str
    porta: int = 5432
    banco: str = "default"
    senha: Optional[str] = None
```

---

## Decoradores
```python
import functools, time

def cronometrar(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        inicio = time.time()
        resultado = func(*args, **kwargs)
        print(f"{func.__name__} levou {time.time()-inicio:.4f}s")
        return resultado
    return wrapper

@cronometrar
def processar(dados: list) -> list:
    return [x * 2 for x in dados]

# Com parâmetros
def repetir(n: int):
    def decorador(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                resultado = func(*args, **kwargs)
            return resultado
        return wrapper
    return decorador

@repetir(3)
def log(msg: str): print(msg)
```

---

## Comprehensions e Generators
```python
# List
quadrados = [x**2 for x in range(10) if x % 2 == 0]

# Dict
mapa = {k: v for k, v in zip("abc", [1, 2, 3])}

# Set
unicos = {x % 3 for x in range(10)}

# Generator (lazy, eficiente em memória)
gen = (x**2 for x in range(1_000_000))

# Função geradora
def chunks(arquivo: str, tamanho: int = 1000):
    with open(arquivo) as f:
        chunk = []
        for linha in f:
            chunk.append(linha)
            if len(chunk) == tamanho:
                yield chunk
                chunk = []
        if chunk:
            yield chunk
```

---

## Tratamento de Erros
```python
class ErroDeValidacao(ValueError):
    def __init__(self, campo: str, valor):
        super().__init__(f"Campo '{campo}' inválido: {valor}")

try:
    resultado = processar(dados)
except ErroDeValidacao as e:
    print(f"Validação: {e}")
except Exception as e:
    raise  # re-lança
else:
    print("Sucesso!")
finally:
    print("Sempre executa")
```

---

## Cache e Performance
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

---

## Logging
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s"
)
logger = logging.getLogger(__name__)

logger.info("Pipeline iniciado")
logger.warning("Atenção!")
logger.error("Erro ao conectar")
logger.exception("Erro com traceback")
```
