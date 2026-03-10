# 🤖 Selenium

> Automação de browsers. Amplamente usado, muito conteúdo na comunidade. Ver também [[Playwright]].

**Relacionado:** [[Python]] | [[Playwright]] | [[Requests]] | [[Pandas]]

**Instalação:** `pip install selenium webdriver-manager`

---

## Setup
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service

options = Options()
options.add_argument("--headless")           # sem janela
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--window-size=1920,1080")

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)
driver.implicitly_wait(10)                   # espera global em segundos
```

---

## Localizar Elementos
```python
from selenium.webdriver.common.by import By

# Por ID
driver.find_element(By.ID, "meu-id")

# Por CSS
driver.find_element(By.CSS_SELECTOR, ".classe")
driver.find_element(By.CSS_SELECTOR, "table tbody tr td:nth-child(2)")

# Por XPath
driver.find_element(By.XPATH, "//div[@class='dados']//span")
driver.find_element(By.XPATH, "//button[text()='Entrar']")

# Por nome/tag
driver.find_element(By.NAME, "usuario")
driver.find_element(By.TAG_NAME, "h1")

# Múltiplos elementos (retorna lista)
linhas = driver.find_elements(By.CSS_SELECTOR, "table tbody tr")
```

---

## Interações
```python
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains

elemento = driver.find_element(By.ID, "campo")

# Clique e digitação
elemento.click()
elemento.send_keys("texto")
elemento.send_keys(Keys.ENTER)
elemento.clear()

# Submit de formulário
elemento.submit()

# Scroll
driver.execute_script("window.scrollTo(0, document.body.scrollHeight)")
driver.execute_script("arguments[0].scrollIntoView();", elemento)

# Hover
actions = ActionChains(driver)
actions.move_to_element(elemento).perform()
```

---

## Esperas Explícitas (Melhor Prática)
```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, timeout=15)

# Aguardar elemento aparecer
elemento = wait.until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".dados"))
)

# Aguardar elemento ser clicável
botao = wait.until(
    EC.element_to_be_clickable((By.ID, "submit"))
)

# Aguardar texto aparecer
wait.until(EC.text_to_be_present_in_element((By.ID, "status"), "Concluído"))

# Aguardar URL mudar
wait.until(EC.url_contains("/dashboard"))
```

---

## Extraindo Dados de Tabela
```python
def extrair_tabela(driver, seletor_tabela: str) -> list[dict]:
    tabela = driver.find_element(By.CSS_SELECTOR, seletor_tabela)
    
    headers = [th.text for th in tabela.find_elements(By.TAG_NAME, "th")]
    
    dados = []
    for tr in tabela.find_elements(By.CSS_SELECTOR, "tbody tr"):
        colunas = [td.text for td in tr.find_elements(By.TAG_NAME, "td")]
        if colunas:
            dados.append(dict(zip(headers, colunas)))
    
    return dados
```

---

## Boas Práticas
```python
# Sempre usar try/finally para garantir fechamento
try:
    driver.get("https://exemplo.com")
    # ... automação
finally:
    driver.quit()    # fecha o browser e o processo

# Ou usando context manager customizado
from contextlib import contextmanager

@contextmanager
def browser(headless=True):
    options = Options()
    if headless:
        options.add_argument("--headless")
    driver = webdriver.Chrome(options=options)
    try:
        yield driver
    finally:
        driver.quit()

with browser() as driver:
    driver.get("https://exemplo.com")
```
