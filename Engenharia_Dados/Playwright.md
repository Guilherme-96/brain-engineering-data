# 🎭 Playwright

> Automação de browsers moderna para web scraping e testes. Mais robusto que [[Selenium]].

**Relacionado:** [[Python]] | [[Selenium]] | [[Requests]] | [[Pandas]]

**Instalação:**
```bash
pip install playwright
playwright install          # baixa os browsers (Chromium, Firefox, WebKit)
playwright install chromium # só o Chromium (menor)
```

---

## Modo Síncrono
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    navegador = p.chromium.launch(headless=True)  # False para ver o browser
    pagina = navegador.new_page()
    
    pagina.goto("https://exemplo.com")
    pagina.wait_for_load_state("networkidle")
    
    # Extrair texto
    titulo = pagina.title()
    texto = pagina.locator("h1").inner_text()
    
    navegador.close()
```

---

## Modo Assíncrono (Recomendado para pipelines)
```python
import asyncio
from playwright.async_api import async_playwright

async def extrair_dados():
    async with async_playwright() as p:
        navegador = await p.chromium.launch(headless=True)
        pagina = await navegador.new_page()
        
        await pagina.goto("https://exemplo.com")
        dados = await pagina.evaluate("() => document.title")
        
        await navegador.close()
        return dados

asyncio.run(extrair_dados())
```

---

## Seletores
```python
# CSS (mais comum)
pagina.locator(".classe-css")
pagina.locator("#id-elemento")
pagina.locator("table tbody tr")
pagina.locator("input[name='usuario']")

# XPath
pagina.locator("xpath=//div[@class='dados']//span")

# Texto
pagina.get_by_text("Entrar")
pagina.get_by_label("E-mail")
pagina.get_by_placeholder("Digite seu CPF")
pagina.get_by_role("button", name="Confirmar")
```

---

## Interações
```python
# Navegação
pagina.goto("https://exemplo.com")
pagina.go_back()
pagina.reload()

# Cliques e formulários
pagina.click("#botao")
pagina.fill("#campo-usuario", "meu_usuario")
pagina.press("#campo-senha", "Enter")
pagina.select_option("#select-estado", "SP")
pagina.check("#checkbox-aceito")
pagina.uncheck("#checkbox-aceito")

# Hover e drag
pagina.hover(".menu-item")
pagina.drag_and_drop("#origem", "#destino")

# Upload de arquivo
pagina.set_input_files("input[type=file]", "arquivo.csv")
```

---

## Esperas (Evitar Race Conditions)
```python
# Aguardar elemento aparecer
pagina.wait_for_selector(".tabela-carregada")

# Aguardar navegação completar
with pagina.expect_navigation():
    pagina.click("#botao-submit")

# Aguardar requisição de rede
with pagina.expect_response("**/api/dados") as response_info:
    pagina.click("#carregar")
resposta = response_info.value

# Aguardar estado da página
pagina.wait_for_load_state("networkidle")   # sem requisições pendentes
pagina.wait_for_load_state("domcontentloaded")
```

---

## Extraindo Dados de Tabelas
```python
def extrair_tabela(pagina, seletor: str) -> list[dict]:
    """Extrai uma tabela HTML como lista de dicionários."""
    tabela = pagina.locator(seletor)
    
    # Cabeçalhos
    headers = [
        th.inner_text() 
        for th in tabela.locator("thead th").all()
    ]
    
    # Linhas
    dados = []
    for tr in tabela.locator("tbody tr").all():
        colunas = [td.inner_text() for td in tr.locator("td").all()]
        dados.append(dict(zip(headers, colunas)))
    
    return dados
```

---

## Cookies, LocalStorage e Screenshots
```python
# Salvar e restaurar sessão (login uma vez)
pagina.context.storage_state(path="sessao.json")
# próxima vez:
contexto = navegador.new_context(storage_state="sessao.json")

# Screenshot
pagina.screenshot(path="captura.png")
pagina.screenshot(path="pagina-completa.png", full_page=True)
pagina.locator(".elemento").screenshot(path="elemento.png")

# PDF
pagina.pdf(path="pagina.pdf")
```

---

## Playwright vs Selenium
| | Playwright | Selenium |
|--|--|--|
| Velocidade | ⚡ Mais rápido | Mais lento |
| API | Moderna, async | Mais verbosa |
| Waits automáticos | ✅ Sim | Manual |
| Suporte a browsers | Chromium, Firefox, WebKit | Chrome, Firefox, Edge |
| Instalação | `playwright install` | Precisa de driver separado |
