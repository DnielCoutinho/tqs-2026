# Capítulo 2 — Preparando o ambiente

[⬅ Anterior: Visão geral](01-visao-geral.md) · [Sumário](../README.md) · Próximo: [Capítulo 3 — Rodando a aplicação ➡](03-rodando-aplicacao.md)

---

Nesta disciplina vamos trabalhar **inteiramente pelo navegador**, usando o **GitHub Codespaces** — um ambiente Linux completo, com Python, Git e VS Code, rodando na nuvem da Microsoft. Você não vai instalar nada na sua máquina.

> 💡 **E se eu quiser usar outra forma?** Você poderia trabalhar **localmente** (instalando Python, Git e uma IDE) ou usar o **github.dev** (VS Code no navegador, sem terminal e sem execução). Ambas são possíveis, mas não vamos cobrir nesta disciplina. **Padronizar em Codespaces** evita problemas de "no meu computador funciona" e garante que todos os alunos têm o mesmo ambiente.

## 2.1. Pré-requisitos

Para acompanhar esta aula você precisa apenas de:

- Uma conta no [GitHub](https://github.com/signup) (gratuita)
- Um navegador moderno (Chrome, Edge, Firefox, Safari)
- Conexão razoável com a internet

E é só. Nenhum software instalado, nenhum Python, nenhum Git, nenhum SO específico.

> O free tier do GitHub Codespaces oferece **120 horas-core/mês** — mais que suficiente para a disciplina inteira.

## 2.2. Criar o repositório no GitHub

Acesse <https://github.com/new> e preencha o formulário:

1. **Repository name**: `tqs-2026` (ou outro nome de sua escolha).
2. **Description** (opcional): "Projeto da disciplina Teste e Qualidade de Software".
3. **Visibility**: marque **Public** — isso é necessário para o GitHub Pages funcionar no plano gratuito.
4. **Initialize this repository with**: marque **Add a README file** — assim o repositório nasce inicializado e você consegue editar pela web imediatamente.
5. **Add .gitignore** e **Choose a license**: deixe ambos em "None" — vamos criar manualmente para entender cada arquivo.
6. Clique no botão verde **Create repository**.

Você será redirecionado para a página do seu novo repositório, com apenas o `README.md` dentro.

## 2.3. Criar o arquivo `.devcontainer/devcontainer.json`

Esse é o arquivo que diz ao Codespaces **qual ambiente preparar para você**: versão do Python, extensões do VS Code, dependências a instalar automaticamente, portas a expor.

### Passo a passo (interface web do GitHub)

1. Na página do seu repositório, clique no botão **"Add file"** (perto do canto superior direito da lista de arquivos) → **"Create new file"**.

2. No campo de nome de arquivo (no topo), digite **exatamente**:

   ```
   .devcontainer/devcontainer.json
   ```

   > A `/` cria automaticamente a pasta `.devcontainer/`. O ponto no início faz parte do nome (não é separador).

3. Na área de edição (campo grande abaixo), **cole** o conteúdo completo a seguir:

   ```json
   {
       "name": "TQS 2026 - Teste e Qualidade de Software",
       "image": "mcr.microsoft.com/devcontainers/python:1-3.11",
       "postCreateCommand": "pip install -r requirements-dev.txt || true",
       "customizations": {
           "vscode": {
               "extensions": [
                   "ms-python.python",
                   "ms-python.vscode-pylance",
                   "charliermarsh.ruff",
                   "github.vscode-pull-request-github"
               ],
               "settings": {
                   "python.defaultInterpreterPath": "/usr/local/bin/python",
                   "python.testing.pytestEnabled": true,
                   "python.testing.unittestEnabled": false,
                   "editor.formatOnSave": true,
                   "[python]": {
                       "editor.defaultFormatter": "charliermarsh.ruff"
                   }
               }
           }
       },
       "forwardPorts": [5000],
       "portsAttributes": {
           "5000": {
               "label": "Flask app",
               "onAutoForward": "notify"
           }
       }
   }
   ```

4. Role a página para baixo até a seção **"Commit changes"**.

5. Preencha:
   - **Commit message**: `chore: adiciona devcontainer.json para Codespaces`
   - **Extended description** (opcional): pode deixar vazio
   - **Commit directly to the `main` branch** — deixe selecionado

6. Clique no botão verde **"Commit changes"**.

### O que cada linha do arquivo significa

| Linha | Para que serve |
|---|---|
| `"image": ".../python:1-3.11"` | Imagem Docker oficial com Python 3.11 pré-instalado |
| `"postCreateCommand"` | Roda automaticamente após o Codespace ser criado — aqui instala as dependências do projeto |
| `"extensions"` | Extensões do VS Code instaladas automaticamente (Python, Pylance, Ruff, GitHub PRs) |
| `"settings"` | Configurações do editor: testes via pytest, formatar ao salvar usando Ruff |
| `"forwardPorts": [5000]` | Quando o Flask rodar na porta 5000, o Codespace expõe uma URL pública temporária |

> O `|| true` no `postCreateCommand` é proposital: se ainda não houver `requirements-dev.txt` no repositório (caso você esteja começando do zero), o comando falha graciosamente em vez de travar o setup.

## 2.4. Abrir o Codespace

Com o `devcontainer.json` commitado, agora você pode subir o ambiente.

1. Na página principal do repositório, clique no botão verde **"Code"**.
2. Selecione a aba **"Codespaces"** (ao lado de "Local").
3. Clique em **"Create codespace on main"**.

Uma nova aba abre. Aguarde aproximadamente **1 a 2 minutos** na primeira vez (o GitHub cria o container, instala as extensões e roda o `postCreateCommand`). Você verá um **VS Code completo rodando no navegador**.

### Verificar que tudo está pronto

Abra o terminal: tecle **Ctrl + `** (Control + crase) — ou menu **Terminal → New Terminal**. Um terminal abre na parte inferior. Digite:

```bash
python --version
```

Deve responder algo como `Python 3.11.x`. Se aparecer, o ambiente está pronto.

## 2.5. Trabalhando com Codespaces no dia a dia

### Como parar e retomar

- **Para parar** (e economizar cota): vá em <https://github.com/codespaces>, encontre o Codespace na lista e clique nos três pontinhos (`···`) → **"Stop codespace"**. Isso preserva todos os seus arquivos.
- **Para retomar**: na mesma lista, clique no nome do Codespace. Ele acorda em ~10 segundos, exatamente como você deixou (arquivos abertos, terminal, tudo).
- Codespace **parado não consome cota** — só ocupa um pouco de storage (também limitado e generoso).

### Atalhos úteis do VS Code

| Atalho | O que faz |
|---|---|
| `Ctrl + `\` | Abre/foca o terminal |
| `Ctrl + Shift + P` | Paleta de comandos (qualquer ação) |
| `Ctrl + P` | Abrir arquivo pelo nome |
| `Ctrl + Shift + F` | Busca global no projeto |
| `Ctrl + S` | Salvar (formato automático ativa aqui) |

### O que NÃO mudou entre Codespaces e máquina local

Os comandos de terminal são **exatamente os mesmos** que você usaria em uma máquina Linux. Todos os exemplos nos próximos capítulos rodam aqui sem ajuste nenhum:

```bash
pytest                              # roda testes
flask --app src.app run             # sobe o Flask
ruff check .                        # lint
git checkout -b feat/nova-feature   # cria branch
```

## 2.6. Como criar arquivos e pastas no Codespace

Você vai criar muitos arquivos nos próximos capítulos. **Faça sempre pela interface do VS Code** — é mais simples e visual do que comandos `mkdir`/`touch` no terminal.

### Criar uma pasta

1. No painel **EXPLORER** (lado esquerdo do VS Code, ícone de dois arquivos sobrepostos), passe o mouse sobre o nome do repositório no topo. Aparecem 4 ícones.
2. Clique no ícone **"New Folder"** (ícone de pasta com `+`).
3. Digite o nome (ex: `src`) e tecle **Enter**.

> Para criar uma subpasta, clique direito numa pasta existente → **"New Folder"**.

### Criar um arquivo

1. Clique direito na pasta onde você quer criar (ou na raiz, no topo do explorer).
2. Selecione **"New File"**.
3. Digite o nome completo, **incluindo a extensão** (ex: `validators.py`).
4. Tecle **Enter**. O arquivo abre vazio no editor central.
5. Cole o conteúdo do código que o livro vai te dar.
6. Salve com **`Ctrl + S`** (o ponto bolado no nome da aba desaparece quando salva).

### Atalho útil

Você pode digitar o **caminho completo** ao criar um arquivo. Por exemplo, com o explorador na raiz, ao clicar em "New File" e digitar `src/templates/index.html`, o VS Code cria as pastas `src/` e `src/templates/` automaticamente se elas não existirem.

> 💡 **Formato automático ao salvar**: como nosso `devcontainer.json` já configurou `editor.formatOnSave: true` com o Ruff, todo arquivo `.py` é formatado automaticamente ao salvar. Você não precisa se preocupar com indentação ou estilo — basta colar o código e salvar.

## 2.7. Próximo passo: construir o projeto

Você está com um Codespace aberto, um terminal funcionando e um repositório quase vazio (só `README.md` e `.devcontainer/devcontainer.json`).

A partir daqui você tem **dois caminhos**:

1. **Forkar o repositório-modelo do professor** (caminho ideal para quem só quer rodar) — você herda todos os arquivos prontos e pode pular para o [capítulo 10](10-atividade-pratica.md) direto. Quando for forkar, lembre de configurar o Codespace no seu fork e ajustar os placeholders `SEU-USUARIO` (instruções no fim do capítulo 9).

2. **Construir cada arquivo seguindo os próximos capítulos** (caminho desta disciplina) — você cria todas as pastas e arquivos pela interface do VS Code, copia o código que o livro fornece, e entende exatamente para que cada peça serve.

**Vamos pelo caminho 2.** Cada um dos próximos capítulos (3 a 9) vai te pedir para criar arquivos específicos, sempre com o código pronto. Ao final do capítulo 9, você terá reproduzido o repositório completo.

---

Próximo capítulo: [Capítulo 3 — Rodando a aplicação ➡](03-rodando-aplicacao.md)
