# Capítulo 4 — Testes automatizados

[⬅ Anterior: Construindo e rodando a aplicação](03-rodando-aplicacao.md) · [Sumário](../README.md) · Próximo: [Capítulo 5 — TDD na prática ➡](05-tdd-na-pratica.md)

---

A aplicação roda — mas como sabemos que ela está **correta**? E como garantir que continue correta depois de futuras mudanças? Esta é a pergunta que **testes automatizados** respondem.

Neste capítulo você vai **criar** os arquivos de teste, configurar o `pytest`, e ver tudo verde pela primeira vez.

## 4.1. O que vamos criar

Adicionados ao seu repositório neste capítulo:

```
tqs-2026/
├── pyproject.toml                  ← novo (config pytest + ruff + cobertura)
├── requirements-dev.txt            ← novo (pytest, ruff, bandit)
└── tests/
    ├── __init__.py                 ← novo
    ├── test_validators.py          ← novo (testes unitários)
    └── test_app.py                 ← novo (testes de integração)
```

## 4.2. Criar `requirements-dev.txt`

Esse arquivo lista as dependências de **desenvolvimento** — ferramentas que rodam fora de produção (testes, lint, segurança).

1. Clique direito na raiz do repo → **"New File"** → nome: `requirements-dev.txt`.
2. Cole:

```text
-r requirements.txt
pytest==8.3.3
pytest-cov==5.0.0
ruff==0.6.9
bandit==1.9.4
```

3. Salve.

> A linha `-r requirements.txt` significa "inclua tudo do `requirements.txt`". Assim, instalar `requirements-dev.txt` instala Flask **mais** as ferramentas de desenvolvimento numa única chamada.

## 4.3. Instalar dependências de desenvolvimento

No terminal:

```bash
pip install -r requirements-dev.txt
```

Você verá pytest, ruff, bandit e suas dependências sendo baixados.

## 4.4. Criar `pyproject.toml`

Esse é o arquivo de **configuração centralizado** do projeto Python — pyproject.toml é o padrão moderno (PEP 518/621). Aqui vão configs do `pytest`, `ruff` e `coverage`.

1. Clique direito na raiz → **"New File"** → nome: `pyproject.toml`.
2. Cole:

```toml
[project]
name = "tqs-2026"
version = "0.1.0"
description = "Projeto-exemplo da disciplina Teste e Qualidade de Software (PC010027) - UFOPA"
authors = [{ name = "Helvecio Bezerra Leal Neto" }]
license = { text = "MIT" }
requires-python = ">=3.11"

[tool.ruff]
line-length = 100
target-version = "py311"
extend-exclude = ["docs", ".venv"]

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "UP",  # pyupgrade
    "SIM", # flake8-simplify
]

[tool.pytest.ini_options]
minversion = "8.0"
addopts = "-ra --strict-markers"
testpaths = ["tests"]
pythonpath = ["."]

[tool.coverage.run]
source = ["src"]
branch = true

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if __name__ == .__main__.:",
]
show_missing = true
skip_covered = false
```

3. Salve.

**O que cada seção faz**:

| Seção | Para que serve |
|---|---|
| `[project]` | Metadados do projeto (nome, autor, licença, versão mínima do Python) |
| `[tool.ruff]` | Config do linter/formatter (linha máx 100 chars, Python 3.11) — usado no [cap 6](06-qualidade-de-codigo.md) |
| `[tool.ruff.lint]` | Quais regras o `ruff check` aplica |
| `[tool.pytest.ini_options]` | Diz ao pytest onde achar testes e como rodar — `pythonpath = ["."]` permite o `from src.validators import ...` |
| `[tool.coverage.run]` | Mede cobertura do código em `src/`, incluindo cobertura de branch (não só linhas) |
| `[tool.coverage.report]` | Como o relatório de cobertura é exibido |

## 4.5. Criar a pasta `tests/` e os arquivos de teste

### Criar a pasta `tests/`

1. Passe o mouse sobre o nome do repositório no topo do Explorer → clique no ícone **"New Folder"**.
2. Nome: `tests` → Enter.

### Criar `tests/__init__.py` (vazio)

Mesmo papel do `src/__init__.py` — marca a pasta como pacote Python.

1. Clique direito em `tests/` → **"New File"** → nome: `__init__.py`.
2. Deixe vazio. Salve.

### Criar `tests/test_validators.py`

Os **testes unitários** das funções `validar_cpf` e `validar_email`. Mantemos a suíte propositalmente pequena (1 caso válido + 1 inválido por validador) para foco didático — a [atividade prática do capítulo 10](10-atividade-pratica.md) pede para você adicionar mais.

1. Clique direito em `tests/` → **"New File"** → nome: `test_validators.py`.
2. Cole:

```python
"""Testes unitários dos validadores de CPF e e-mail.

Mantemos a suíte propositalmente pequena: 1 caso válido + 1 caso inválido
para cada validador. O suficiente para o aluno acompanhar o ciclo TDD
sem se afogar em código de teste.

Os capítulos 5 (TDD) e 10 (atividade prática) sugerem novos testes
para cobrir casos de borda — adicione-os à medida que evoluir o projeto.
"""

from src.validators import validar_cpf, validar_email


def test_aceita_cpf_valido():
    assert validar_cpf("111.444.777-35") is True


def test_rejeita_cpf_com_digito_verificador_errado():
    assert validar_cpf("111.444.777-30") is False


def test_aceita_email_valido():
    assert validar_email("aluno@ufopa.edu.br") is True


def test_rejeita_email_sem_arroba():
    assert validar_email("semarroba.com") is False
```

3. Salve.

### Criar `tests/test_app.py`

Os **testes de integração** das rotas Flask. Aqui usamos o `test_client` do próprio Flask para simular requisições HTTP sem subir servidor de verdade — é rápido e isolado.

1. Clique direito em `tests/` → **"New File"** → nome: `test_app.py`.
2. Cole:

```python
"""Testes de integração das rotas Flask.

Mostram como exercitar a aplicação sem subir servidor de fato,
usando o test client do próprio Flask. Mantemos só dois testes
(um por rota) para foco didático.
"""

import pytest

from src.app import criar_app


@pytest.fixture
def client():
    app = criar_app()
    return app.test_client()


def test_index_retorna_formulario(client):
    resposta = client.get("/")
    assert resposta.status_code == 200
    assert "Teste e Qualidade de Software" in resposta.get_data(as_text=True)


def test_validar_retorna_resultados_corretos(client):
    resposta = client.post(
        "/validar",
        json={"cpf": "111.444.777-35", "email": "aluno@ufopa.edu.br"},
    )
    assert resposta.get_json() == {"cpf_valido": True, "email_valido": True}
```

3. Salve.

## 4.6. Rodar todos os testes pela primeira vez

```bash
pytest
```

Saída esperada:

```
collected 6 items

tests/test_app.py ..                                               [ 33%]
tests/test_validators.py ....                                      [100%]

============================== 6 passed in 0.13s ==============================
```

Um ponto `.` por teste que passou. Um `F` apareceria por teste que falhou. 🎉

## 4.7. Anatomia de um teste

Cada teste é uma função que começa com `test_`. A estrutura mínima é:

1. **Arrange** — preparar a entrada (`"111.444.777-35"`)
2. **Act** — chamar a função (`validar_cpf(...)`)
3. **Assert** — verificar o resultado (`is True`)

Nesses exemplos, o passo Arrange é tão simples que cabe na própria chamada.

## 4.8. Dois tipos de teste neste projeto

| Tipo | O que testa | Quantos |
|---|---|---|
| **Unitário** | Funções isoladas (`validar_cpf`, `validar_email`) | 4 |
| **Integração** | Rotas Flask reais (`GET /`, `POST /validar`) | 2 |

> O teste unitário não envolve nada além da função. O teste de integração usa o `test_client` do Flask para simular uma requisição HTTP completa e verificar a resposta. **Os dois são úteis** — o unitário é rápido e específico; o de integração garante que tudo "se encontra" corretamente.

A `@pytest.fixture` em `tests/test_app.py` cria uma instância isolada do app Flask para cada teste. Não sobe servidor, não abre porta — tudo acontece em memória. É **rápido** e **isolado** (um teste não interfere no outro).

## 4.9. Comandos úteis do pytest

```bash
pytest tests/test_validators.py                              # só um arquivo
pytest tests/test_validators.py::test_aceita_cpf_valido      # só um teste
pytest -v                                                    # verbose (nomes completos)
pytest -x                                                    # parar no primeiro erro
pytest -k "cpf"                                              # só testes com "cpf" no nome
```

## 4.10. Cobertura de código

**Cobertura** = % das linhas do código de produção (`src/`) que são executadas pelos testes. Quanto maior, mais difícil é uma mudança quebrar algo sem que algum teste perceba.

```bash
pytest --cov=src --cov-report=term-missing
```

Saída:

```
Name                Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------------
src/__init__.py         0      0      0      0   100%
src/app.py             15      0      0      0   100%
src/validators.py      22      4      8      4    73%   21, 26, 29, 39
---------------------------------------------------------------
TOTAL                  37      4      8      4    82%
```

A coluna `Missing` aponta as linhas **não exercitadas** pelos testes. As linhas 21, 26, 29 e 39 de `validators.py` são os **guards** (entrada não-string, CPF com todos dígitos iguais, e-mail vazio) — comportamentos que a função trata corretamente, mas que **nenhum teste atual exercita**.

> Veja só: a função é mais defensiva do que os testes — isso é um cheiro de "falta cobrir mais casos". É exatamente isso que a [atividade prática](10-atividade-pratica.md) vai te pedir.

### Relatório HTML (navegável)

```bash
pytest --cov=src --cov-report=html
python -m http.server 8000 -d htmlcov
```

O Codespace vai detectar a porta 8000 e mostrar um pop-up para abrir no navegador. Uma página abre listando os arquivos cobertos — clique em qualquer um para ver linhas cobertas (verde) e descobertas (vermelho).

Para parar o servidor HTTP: `Ctrl + C` no terminal.

## 4.11. Meta de cobertura neste projeto

Configuramos o pipeline (cap 7) para falhar se a cobertura cair abaixo de **80%**:

```bash
pytest --cov=src --cov-fail-under=80
```

Hoje estamos em **82%** — confortavelmente acima do piso, mas com **espaço para crescer**. À medida que você adiciona casos de borda (capítulo 10), a cobertura sobe.

## 4.12. Cobertura ≠ qualidade

Cobertura alta **não garante** que os testes sejam bons. É perfeitamente possível ter 100% de cobertura com testes que não verificam nada:

```python
def test_inutil():
    validar_cpf("123")     # executa a função, mas não checa o resultado
```

A cobertura sobe, mas o teste é mentira. **Mutation testing** (capítulo 11) ajuda a expor esse tipo de problema.

> **Regra prática**: cobertura **alta** é necessária mas não suficiente. Cobertura **baixa** é prova de código não testado.

## 4.13. Commit do que você fez

```bash
git add .
git commit -m "test: adiciona suite de testes com pytest e cobertura"
git push
```

---

Próximo capítulo: [Capítulo 5 — TDD na prática ➡](05-tdd-na-pratica.md)
