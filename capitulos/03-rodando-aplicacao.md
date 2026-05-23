# Capítulo 3 — Construindo e rodando a aplicação

[⬅ Anterior: Preparando o ambiente](02-preparando-ambiente.md) · [Sumário](../README.md) · Próximo: [Capítulo 4 — Testes automatizados ➡](04-testes-automatizados.md)

---

Neste capítulo você vai **criar** todos os arquivos necessários para a aplicação Flask e rodá-la pela primeira vez. Cada arquivo é criado pela interface do VS Code (revisão no [capítulo 2.6](02-preparando-ambiente.md#26-como-criar-arquivos-e-pastas-no-codespace)) e o conteúdo vem todo pronto para você copiar.

## 3.1. O que vamos criar

Ao final deste capítulo, seu repositório terá:

```
tqs-2026/
├── .devcontainer/devcontainer.json      (já criado)
├── .gitignore                            ← novo
├── README.md                             (já criado)
├── requirements.txt                      ← novo
└── src/
    ├── __init__.py                       ← novo
    ├── validators.py                     ← novo
    ├── app.py                            ← novo
    └── templates/
        └── index.html                    ← novo
```

## 3.2. Criar `.gitignore`

Esse arquivo diz ao Git para **ignorar** arquivos gerados automaticamente (cache, build, ambiente virtual) que não devem entrar no histórico.

1. Clique direito na raiz do repositório no Explorer → **"New File"**.
2. Nome: `.gitignore` (com o ponto no início).
3. Cole:

```gitignore
# Byte-compilados / cache
__pycache__/
*.py[cod]
*$py.class
*.so

# Ambientes virtuais
.venv/
venv/
env/
ENV/

# Distribuição / build
build/
dist/
*.egg-info/
*.egg
.eggs/

# Testes e cobertura
.pytest_cache/
.coverage
.coverage.*
htmlcov/
coverage.xml
.tox/
.nox/

# Lint / type checking
.ruff_cache/
.mypy_cache/

# Editores
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# Variáveis de ambiente
.env
.env.local
```

Salve com **`Ctrl + S`**.

## 3.3. Criar `requirements.txt`

Lista as dependências de **produção** da aplicação. Neste projeto só precisamos do Flask.

1. Clique direito na raiz → **"New File"** → nome: `requirements.txt`.
2. Cole:

```text
Flask==3.0.3
```

3. Salve.

## 3.4. Instalar as dependências

No terminal do Codespace (atalho **`Ctrl + ``**):

```bash
pip install -r requirements.txt
```

Você verá algumas linhas de download e termina com `Successfully installed Flask-3.0.3 ...`.

## 3.5. Criar a pasta `src/` e os arquivos Python

A pasta `src/` vai conter todo o **código de produção** da aplicação.

### Criar a pasta `src/`

1. Passe o mouse sobre o nome do repositório no topo do Explorer → clique no ícone **"New Folder"**.
2. Nome: `src` → Enter.

### Criar `src/__init__.py` (vazio)

Esse arquivo, mesmo vazio, sinaliza ao Python que `src/` é um **pacote** (importável via `import src...`).

1. Clique direito na pasta `src/` → **"New File"** → nome: `__init__.py`.
2. **Não cole nada** — deixe vazio. Salve.

### Criar `src/validators.py`

Essa é a **versão principal da lógica de validação** — sem dependência de framework, só funções puras Python. É o coração do projeto.

1. Clique direito na pasta `src/` → **"New File"** → nome: `validators.py`.
2. Cole:

```python
"""Validadores de CPF e e-mail.

Lógica de negócio pura, sem dependência de framework web.
Pensada para ser exercitada via TDD na disciplina PC010027 (UFOPA).
"""

import re

_REGEX_EMAIL = re.compile(r"^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$")


def _calcular_digito_verificador(digitos: str, peso_inicial: int) -> int:
    pesos = range(peso_inicial, 1, -1)
    soma = sum(int(d) * peso for d, peso in zip(digitos, pesos, strict=True))
    resto = (soma * 10) % 11
    return 0 if resto == 10 else resto


def validar_cpf(cpf: str | None) -> bool:
    if not isinstance(cpf, str):
        return False

    apenas_digitos = re.sub(r"[.\-\s]", "", cpf)

    if len(apenas_digitos) != 11 or not apenas_digitos.isdigit():
        return False

    if len(set(apenas_digitos)) == 1:
        return False

    primeiro = _calcular_digito_verificador(apenas_digitos[:9], peso_inicial=10)
    segundo = _calcular_digito_verificador(apenas_digitos[:10], peso_inicial=11)

    return apenas_digitos[9] == str(primeiro) and apenas_digitos[10] == str(segundo)


def validar_email(email: str | None) -> bool:
    if not isinstance(email, str) or not email:
        return False
    return _REGEX_EMAIL.match(email) is not None
```

3. Salve. Repare que o Ruff formatou automaticamente — todo arquivo `.py` é formatado ao salvar (configurado no `devcontainer.json`).

> **Não vamos explicar linha-a-linha agora.** O [capítulo 5](05-tdd-na-pratica.md) refaz essa função passo a passo via TDD, mostrando como cada parte nasce de um teste que falhou.

### Criar `src/app.py`

Aqui ficam as **rotas Flask** que usam os validadores.

1. Clique direito na pasta `src/` → **"New File"** → nome: `app.py`.
2. Cole:

```python
"""Aplicação Flask que expõe os validadores via formulário web.

Para rodar localmente:
    flask --app src.app run
"""

from flask import Flask, jsonify, render_template, request

from src.validators import validar_cpf, validar_email


def criar_app() -> Flask:
    app = Flask(__name__)

    @app.get("/")
    def index():
        return render_template("index.html")

    @app.post("/validar")
    def validar():
        dados = request.get_json(silent=True) or request.form
        cpf = dados.get("cpf", "")
        email = dados.get("email", "")
        return jsonify(
            cpf_valido=validar_cpf(cpf),
            email_valido=validar_email(email),
        )

    return app


app = criar_app()


if __name__ == "__main__":
    app.run()
```

3. Salve.

**O que esse arquivo faz**:

- `criar_app()` é uma **factory function** — cria e retorna uma instância do Flask. Esse padrão facilita testar (cada teste cria sua própria instância) e isolar configuração.
- Rota `GET /` renderiza o template HTML com o formulário.
- Rota `POST /validar` recebe `{cpf, email}` (JSON ou form-data), chama os validadores e devolve `{cpf_valido, email_valido}` como JSON.

### Criar a subpasta `src/templates/` e `index.html`

O Flask procura templates HTML por padrão numa pasta chamada `templates/` dentro do app.

1. Clique direito na pasta `src/` → **"New Folder"** → nome: `templates`.
2. Clique direito na pasta `src/templates/` → **"New File"** → nome: `index.html`.
3. Cole:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TQS 2026 - Validador CPF/Email</title>
    <style>
        :root {
            --azul: #006699;
            --vermelho: #cc3300;
            --verde: #008040;
        }
        * { box-sizing: border-box; }
        body {
            font-family: system-ui, -apple-system, sans-serif;
            max-width: 640px;
            margin: 2rem auto;
            padding: 1rem;
            line-height: 1.5;
            color: #222;
        }
        h1 { color: var(--azul); }
        .subtitulo { color: #666; margin-top: -0.5rem; }
        form { display: grid; gap: 1rem; margin-top: 2rem; }
        label { font-weight: 600; }
        input {
            width: 100%;
            padding: 0.5rem;
            font-size: 1rem;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            background: var(--azul);
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover { background: #004d73; }
        .resultado { margin-top: 1.5rem; padding: 1rem; border-radius: 4px; }
        .valido { background: #e0f5e8; color: var(--verde); }
        .invalido { background: #fbe5e0; color: var(--vermelho); }
    </style>
</head>
<body>
    <h1>Teste e Qualidade de Software - PC010027</h1>
    <p class="subtitulo">Demo: validador de CPF e e-mail (UFOPA)</p>

    <form id="formulario">
        <div>
            <label for="cpf">CPF</label>
            <input type="text" id="cpf" name="cpf" placeholder="000.000.000-00" required>
        </div>
        <div>
            <label for="email">E-mail</label>
            <input type="email" id="email" name="email" placeholder="aluno@ufopa.edu.br" required>
        </div>
        <button type="submit">Validar</button>
    </form>

    <div id="resultado"></div>

    <script>
        document.getElementById("formulario").addEventListener("submit", async (e) => {
            e.preventDefault();
            const cpf = document.getElementById("cpf").value;
            const email = document.getElementById("email").value;
            const resposta = await fetch("/validar", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ cpf, email }),
            });
            const dados = await resposta.json();
            const div = document.getElementById("resultado");
            div.innerHTML = `
                <div class="resultado ${dados.cpf_valido ? "valido" : "invalido"}">
                    CPF: ${dados.cpf_valido ? "válido" : "inválido"}
                </div>
                <div class="resultado ${dados.email_valido ? "valido" : "invalido"}">
                    E-mail: ${dados.email_valido ? "válido" : "inválido"}
                </div>
            `;
        });
    </script>
</body>
</html>
```

4. Salve.

## 3.6. Subir o servidor Flask

Com tudo no lugar, no terminal:

```bash
flask --app src.app run
```

Você verá:

```
 * Serving Flask app 'src.app'
 * Running on http://127.0.0.1:5000
```

O Codespace detecta a porta 5000 e exibe um pop-up no canto inferior direito do VS Code: **"Your application running on port 5000 is available"** com botões **"Open in Browser"** e **"Make Public"**. Clique em **"Open in Browser"** — uma nova aba abre com uma URL temporária (algo como `https://seu-codespace-5000.app.github.dev`) servindo seu Flask.

Você verá o formulário com dois campos (CPF e e-mail) e um botão "Validar".

> Se perdeu o pop-up, veja a aba **PORTS** no painel inferior do VS Code (ao lado de TERMINAL): a porta 5000 aparece lá com um ícone de globo para abrir no navegador.

## 3.7. Testar manualmente

Experimente:

- **CPF válido**: `111.444.777-35` → deve aparecer "CPF: válido" em verde
- **CPF inválido**: `111.444.777-30` (último dígito errado) → "CPF: inválido"
- **E-mail válido**: `aluno@ufopa.edu.br` → "E-mail: válido"
- **E-mail inválido**: `aluno-ufopa.edu.br` (sem `@`) → "E-mail: inválido"

## 3.8. O que está acontecendo por trás?

```
Navegador                  Flask                     Python
───────                    ─────                     ──────
Usuário preenche
campo e clica          ──► POST /validar
                                                ──► validar_cpf("111.444.777-35")
                                                ──► validar_email("aluno@ufopa.edu.br")
                                                ◄── {True, True}
                       ◄── { "cpf_valido": true,
                             "email_valido": true }
JS atualiza o DOM
mostrando "válido"
```

Três arquivos estão envolvidos:

| Arquivo | Papel |
|---|---|
| [`src/app.py`](../src/app.py) | Define as rotas Flask `GET /` (renderiza o HTML) e `POST /validar` (recebe JSON e chama os validadores) |
| [`src/validators.py`](../src/validators.py) | Funções puras `validar_cpf` e `validar_email` — sem dependência de Flask, fáceis de testar isoladamente |
| [`src/templates/index.html`](../src/templates/index.html) | Template Jinja2 com o formulário e o JavaScript que chama `/validar` |

> **Observação**: `validators.py` **não importa Flask**. Essa separação é proposital: a lógica de negócio fica isolada do framework, o que torna os testes mais simples e rápidos. Essa é uma das ideias centrais de arquiteturas como *hexagonal*, *clean architecture* e *ports & adapters*.

## 3.9. Parar o servidor

Pressione `Ctrl + C` no terminal onde o Flask está rodando.

## 3.10. Modo desenvolvimento (auto-reload)

Durante o desenvolvimento, queremos que o servidor recarregue automaticamente quando salvamos um arquivo:

```bash
flask --app src.app run --debug
```

> **Atenção**: nunca rode com `--debug` em produção. Ele expõe o Werkzeug Debugger, que permite executar código arbitrário no servidor. O **bandit** (que veremos no capítulo 6) detecta isso como vulnerabilidade.

## 3.11. Commit do que você fez

No terminal:

```bash
git add .
git commit -m "feat: aplicação Flask com validadores de CPF e email"
git push
```

Você acabou de fazer seu **primeiro commit funcional** do projeto. No próximo capítulo vamos adicionar testes automatizados.

---

Próximo capítulo: [Capítulo 4 — Testes automatizados ➡](04-testes-automatizados.md)
