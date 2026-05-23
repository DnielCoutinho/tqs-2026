# Capítulo 7 — CI com GitHub Actions

[⬅ Anterior: Qualidade de código](06-qualidade-de-codigo.md) · [Sumário](../README.md) · Próximo: [Capítulo 8 — Deploy automatizado ➡](08-deploy-automatizado.md)

---

Rodar os testes e o lint **localmente** é ótimo — mas humanos esquecem. **CI** (*Continuous Integration*) automatiza esse processo: a cada `push` e a cada `pull_request`, um servidor neutro roda todas as verificações. Se algo falha, o PR fica vermelho e ninguém faz merge sem perceber.

## 7.1. O que é GitHub Actions?

É o serviço gratuito de CI/CD que vem embutido no GitHub. Você descreve o pipeline num arquivo YAML dentro de `.github/workflows/` e o GitHub o executa em máquinas virtuais (chamadas *runners*).

## 7.2. Criar o workflow `ci.yml`

Adicionado ao seu repositório neste capítulo:

```
tqs-2026/
└── .github/
    └── workflows/
        └── ci.yml                  ← novo
```

### Criar as pastas

1. Passe o mouse sobre o nome do repositório no Explorer → **"New Folder"** → nome: `.github` → Enter (sim, com ponto no início).
2. Clique direito em `.github/` → **"New Folder"** → nome: `workflows` → Enter.

### Criar `ci.yml`

1. Clique direito em `.github/workflows/` → **"New File"** → nome: `ci.yml`.
2. Cole:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  qualidade-e-testes:
    name: Lint, segurança e testes
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      - name: Instalar dependências
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements-dev.txt

      - name: Lint (ruff check)
        run: ruff check .

      - name: Formatação (ruff format)
        run: ruff format --check .

      - name: Análise de segurança (bandit)
        run: bandit -r src/

      - name: Testes + cobertura
        run: pytest --cov=src --cov-report=term --cov-report=xml --cov-fail-under=80

      - name: Publicar relatório de cobertura
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage.xml
```

3. Salve.

### Commit e push

```bash
git add .github/workflows/ci.yml
git commit -m "ci: adiciona pipeline de qualidade + testes"
git push
```

A partir desse push, **o workflow já está ativo** — vai rodar automaticamente neste push e em todos os próximos.

## 7.3. Dissecando o workflow

Vamos olhar bloco por bloco do `ci.yml` que você acabou de criar.

### Quando o workflow roda

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

- A cada `push` direto na `main`
- A cada `pull_request` que tenha `main` como destino

> Isso significa que, mesmo antes de mergear, o autor do PR já sabe se sua mudança passou em todas as verificações.

### Onde roda

```yaml
jobs:
  qualidade-e-testes:
    runs-on: ubuntu-latest
```

Ubuntu mais recente, gratuito para repos públicos. Você poderia rodar a mesma coisa em Windows, macOS ou múltiplas versões de Python em paralelo (matriz).

### Os passos, em ordem

```yaml
steps:
  - name: Checkout do código
    uses: actions/checkout@v4

  - name: Configurar Python 3.11
    uses: actions/setup-python@v5
    with:
      python-version: "3.11"
      cache: "pip"
```

1. **Checkout** — baixa o código do repositório para o runner.
2. **Setup Python** — instala Python 3.11 e ativa o cache do pip (acelera execuções futuras).

```yaml
  - name: Instalar dependências
    run: |
      python -m pip install --upgrade pip
      pip install -r requirements-dev.txt
```

3. **Instala as dependências** — exatamente como você faz localmente.

```yaml
  - name: Lint (ruff check)
    run: ruff check .

  - name: Formatação (ruff format)
    run: ruff format --check .

  - name: Análise de segurança (bandit)
    run: bandit -r src/

  - name: Testes + cobertura
    run: pytest --cov=src --cov-report=term --cov-report=xml --cov-fail-under=80
```

4. **Lint**, **format**, **bandit** e **testes com cobertura** — exatamente os mesmos comandos do capítulo anterior. **Qualquer um que falhar derruba o pipeline.**

```yaml
  - name: Publicar relatório de cobertura
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: coverage-report
      path: coverage.xml
```

5. **Publica** o XML de cobertura como artifact (com `if: always()`, sobe mesmo se o pipeline falhou, para você inspecionar). Você baixa pela aba **Actions** do PR.

## 7.4. Vendo o CI rodar pela primeira vez

1. Faça qualquer commit e `git push origin main`.
2. No GitHub, vá em **Actions**.
3. Você verá o workflow **CI** em amarelo (rodando), depois verde (sucesso) ou vermelho (falha).
4. Clique no run para ver os logs de cada passo.

## 7.5. Como ler logs de falha

1. Vá em **Actions** no GitHub.
2. Clique no workflow que falhou (ícone vermelho).
3. Clique no **job** (a caixinha que falhou).
4. Expanda o **passo** vermelho.
5. Leia a saída **de baixo para cima** — o erro real costuma estar nas últimas 10–20 linhas.

> **Dica de ouro**: copie a mensagem de erro **literal** e busque no Google. Em 90% dos casos, alguém já passou pelo mesmo problema.

## 7.6. Status no PR

Quando você abrir um PR (próximos capítulos), o GitHub mostra automaticamente o status do CI:

- ✅ **All checks have passed** — pode mergear com segurança
- ❌ **Some checks were not successful** — investigue antes de mergear (ou de aprovar)

Configurações de *branch protection* (capítulo 9) podem **proibir** o merge enquanto o CI estiver vermelho.

## 7.7. O ciclo virtuoso

```
1. Alguém escreve código
2. Roda testes/lint localmente (pega 90% dos problemas)
3. Faz commit/push, abre PR
4. CI roda no GitHub (pega os 10% restantes)
5. Revisor humano se concentra na lógica, não em estilo
6. Merge, deploy
```

> Sem CI, o **revisor humano** vira responsável por verificar formatação, cobertura e lint. Isso é desperdício de capital humano — ele deveria estar pensando na **arquitetura** e nos **casos de borda**, não em vírgulas.

---

Próximo capítulo: [Capítulo 8 — Deploy automatizado ➡](08-deploy-automatizado.md)
