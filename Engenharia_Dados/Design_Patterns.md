# 🎨 Design Patterns

> Soluções reutilizáveis para problemas recorrentes. O vocabulário comum entre engenheiros sênior.

**Relacionado:** [[Python]] | [[Airflow]] | [[FastAPI]] | [[SQLAlchemy]] | [[Algoritmos_e_Estruturas_de_Dados]]

> **Seu projeto:** [github.com/Guilherme-96/Design-Patterns](https://github.com/Guilherme-96/Design-Patterns)

---

## Por que Design Patterns?

Patterns não são regras rígidas — são **vocabulário compartilhado**.

Quando você diz "isso é um Singleton", todos os engenheiros entendem imediatamente. Isso economiza horas de explicação e evita que cada um reinvente a roda de forma diferente.

Os 23 patterns do GoF (Gang of Four) se dividem em três categorias:

```
Criacionais  → Como criar objetos
Estruturais  → Como compor estruturas
Comportamentais → Como objetos se comunicam
```

---

## Criacionais

### Singleton — Uma única instância
```python
class Configuracao:
    _instancia = None
    
    def __new__(cls):
        if cls._instancia is None:
            cls._instancia = super().__new__(cls)
            cls._instancia._carregar_config()
        return cls._instancia
    
    def _carregar_config(self):
        from dotenv import load_dotenv
        import os
        load_dotenv()
        self.db_url = os.getenv("DATABASE_URL")
        self.debug = os.getenv("DEBUG", "false").lower() == "true"

# Sempre a mesma instância!
c1 = Configuracao()
c2 = Configuracao()
assert c1 is c2  # True

# Em Python moderno, prefira módulos (já são singletons por natureza)
# ou Pydantic Settings
```

**Uso em dados:** pool de conexão com banco, cliente S3, configurações globais.

---

### Factory Method — Delegar criação de objetos
```python
from abc import ABC, abstractmethod
import pandas as pd

class LeitorDados(ABC):
    @abstractmethod
    def ler(self, caminho: str) -> pd.DataFrame:
        pass

class LeitorCSV(LeitorDados):
    def ler(self, caminho: str) -> pd.DataFrame:
        return pd.read_csv(caminho)

class LeitorParquet(LeitorDados):
    def ler(self, caminho: str) -> pd.DataFrame:
        return pd.read_parquet(caminho)

class LeitorExcel(LeitorDados):
    def ler(self, caminho: str) -> pd.DataFrame:
        return pd.read_excel(caminho)

class LeitorJSON(LeitorDados):
    def ler(self, caminho: str) -> pd.DataFrame:
        return pd.read_json(caminho)

# Factory — cria o objeto correto com base no tipo
def criar_leitor(formato: str) -> LeitorDados:
    leitores = {
        "csv": LeitorCSV,
        "parquet": LeitorParquet,
        "xlsx": LeitorExcel,
        "json": LeitorJSON,
    }
    LeitorClass = leitores.get(formato.lower())
    if not LeitorClass:
        raise ValueError(f"Formato não suportado: {formato}")
    return LeitorClass()

# Uso — o cliente não sabe qual classe está criando
extensao = "arquivo.csv".split(".")[-1]
leitor = criar_leitor(extensao)
df = leitor.ler("arquivo.csv")
```

---

### Builder — Construção complexa passo a passo
```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Pipeline:
    nome: str
    fonte: str
    destino: str
    transformacoes: list = field(default_factory=list)
    retries: int = 3
    notificar_slack: bool = False
    particionar_por: Optional[str] = None

class PipelineBuilder:
    def __init__(self, nome: str):
        self._pipeline = {"nome": nome}
    
    def de(self, fonte: str) -> "PipelineBuilder":
        self._pipeline["fonte"] = fonte
        return self
    
    def para(self, destino: str) -> "PipelineBuilder":
        self._pipeline["destino"] = destino
        return self
    
    def com_transformacao(self, fn) -> "PipelineBuilder":
        self._pipeline.setdefault("transformacoes", []).append(fn)
        return self
    
    def com_retries(self, n: int) -> "PipelineBuilder":
        self._pipeline["retries"] = n
        return self
    
    def notificar_slack(self) -> "PipelineBuilder":
        self._pipeline["notificar_slack"] = True
        return self
    
    def build(self) -> Pipeline:
        return Pipeline(**self._pipeline)

# Uso fluente (method chaining)
pipeline = (PipelineBuilder("vendas_diarias")
    .de("postgresql://localhost/raw")
    .para("s3://bucket/gold/")
    .com_transformacao(limpar_nulos)
    .com_transformacao(calcular_metricas)
    .com_retries(5)
    .notificar_slack()
    .build()
)
```

---

## Estruturais

### Decorator — Adicionar comportamento sem modificar a classe
```python
import functools
import time
import logging

# Em Python, decoradores são nativos!
def log_execucao(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logging.info(f"Iniciando: {func.__name__}")
        inicio = time.time()
        try:
            resultado = func(*args, **kwargs)
            logging.info(f"Concluído: {func.__name__} ({time.time()-inicio:.2f}s)")
            return resultado
        except Exception as e:
            logging.error(f"Erro em {func.__name__}: {e}")
            raise
    return wrapper

def retry(max_tentativas: int = 3, delay: float = 1.0):
    def decorador(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for tentativa in range(max_tentativas):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if tentativa == max_tentativas - 1:
                        raise
                    time.sleep(delay * (2 ** tentativa))  # backoff exponencial
        return wrapper
    return decorador

@log_execucao
@retry(max_tentativas=3, delay=2.0)
def extrair_dados_api(url: str) -> dict:
    import requests
    return requests.get(url, timeout=10).json()
```

### Adapter — Compatibilizar interfaces incompatíveis
```python
# Problema: diferentes fontes de dados têm interfaces diferentes

class AdapterDadosMongoDB:
    """Adapta MongoDB para a interface padrão de DataSource."""
    
    def __init__(self, colecao):
        self.colecao = colecao
    
    def ler(self, filtros: dict = {}) -> pd.DataFrame:
        dados = list(self.colecao.find(filtros, {"_id": 0}))
        return pd.DataFrame(dados)

class AdapterDadosAPI:
    """Adapta uma API REST para a interface padrão."""
    
    def __init__(self, url_base: str, token: str):
        self.url_base = url_base
        self.headers = {"Authorization": f"Bearer {token}"}
    
    def ler(self, filtros: dict = {}) -> pd.DataFrame:
        import requests
        r = requests.get(f"{self.url_base}/dados", 
                        params=filtros, headers=self.headers)
        return pd.DataFrame(r.json()["results"])

# Agora o pipeline trata tudo da mesma forma!
def processar(fonte):  # aceita qualquer adaptador
    df = fonte.ler({"ativo": True})
    return transformar(df)
```

---

## Comportamentais

### Observer — Notificar múltiplos assinantes
```python
from typing import Protocol, Callable

class EventoObserver(Protocol):
    def notificar(self, evento: str, dados: dict) -> None: ...

class PipelineEventBus:
    """Barramento de eventos do pipeline."""
    
    def __init__(self):
        self._assinantes: dict[str, list[Callable]] = {}
    
    def assinar(self, evento: str, callback: Callable):
        self._assinantes.setdefault(evento, []).append(callback)
    
    def publicar(self, evento: str, dados: dict):
        for callback in self._assinantes.get(evento, []):
            try:
                callback(dados)
            except Exception as e:
                print(f"Erro no observer {callback.__name__}: {e}")

bus = PipelineEventBus()

# Assinantes independentes
bus.assinar("pipeline_falhou", lambda d: notificar_slack(d))
bus.assinar("pipeline_falhou", lambda d: salvar_log_erro(d))
bus.assinar("pipeline_concluido", lambda d: atualizar_dashboard(d))
bus.assinar("dados_invalidos", lambda d: enviar_email_time(d))

# Produtor não sabe quem está ouvindo
bus.publicar("pipeline_falhou", {"pipeline": "vendas", "erro": "timeout"})
```

### Strategy — Trocar algoritmos em tempo de execução
```python
from abc import ABC, abstractmethod

class EstrategiaCompressao(ABC):
    @abstractmethod
    def comprimir(self, df: pd.DataFrame, caminho: str): pass

class CompressaoSnappy(EstrategiaCompressao):
    def comprimir(self, df, caminho):
        df.to_parquet(caminho, compression="snappy")  # rápido

class CompressaoGzip(EstrategiaCompressao):
    def comprimir(self, df, caminho):
        df.to_parquet(caminho, compression="gzip")   # menor arquivo

class CompressaoZstd(EstrategiaCompressao):
    def comprimir(self, df, caminho):
        df.to_parquet(caminho, compression="zstd")   # melhor ratio/velocidade

class EscritorDados:
    def __init__(self, estrategia: EstrategiaCompressao):
        self._estrategia = estrategia
    
    def trocar_estrategia(self, estrategia: EstrategiaCompressao):
        self._estrategia = estrategia
    
    def salvar(self, df: pd.DataFrame, caminho: str):
        self._estrategia.comprimir(df, caminho)

# Trocar estratégia sem mudar o cliente
escritor = EscritorDados(CompressaoSnappy())
escritor.salvar(df, "rapido.parquet")

escritor.trocar_estrategia(CompressaoGzip())
escritor.salvar(df, "compacto.parquet")
```

---

## Patterns Específicos para Dados

### Repository Pattern — Abstrair acesso a dados
```python
from abc import ABC, abstractmethod
from typing import Optional

class RepositorioPedidos(ABC):
    @abstractmethod
    def buscar_por_id(self, id: int) -> Optional[dict]: pass
    
    @abstractmethod
    def buscar_todos(self, filtros: dict = {}) -> list[dict]: pass
    
    @abstractmethod
    def salvar(self, pedido: dict) -> int: pass

class RepositorioPedidosSQL(RepositorioPedidos):
    def __init__(self, engine):
        self.engine = engine
    
    def buscar_por_id(self, id: int):
        return pd.read_sql(f"SELECT * FROM pedidos WHERE id = {id}", self.engine).to_dict('records')[0]
    
    def buscar_todos(self, filtros={}):
        query = "SELECT * FROM pedidos WHERE 1=1"
        for k, v in filtros.items():
            query += f" AND {k} = '{v}'"
        return pd.read_sql(query, self.engine).to_dict('records')
    
    def salvar(self, pedido):
        # INSERT ...
        pass

# Fácil de trocar implementação (SQL → MongoDB → API) sem mudar a lógica de negócio
repo = RepositorioPedidosSQL(engine)
pedido = repo.buscar_por_id(123)
```
