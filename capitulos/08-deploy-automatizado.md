# Capítulo 8 — Deploy automatizado

[⬅ Anterior: CI com GitHub Actions](07-ci-github-actions.md) · [Sumário](../README.md) · Próximo: [Capítulo 9 — Pull Requests ➡](09-pull-requests.md)

---

Testes verdes não bastam — alguém precisa **usar** a aplicação. Vamos automatizar o **deploy** para que toda mudança em `main` apareça publicada na web em poucos minutos, sem ninguém ter que rodar comando algum.

## 8.1. Por que GitHub Pages?

| Vantagem | Detalhe |
|---|---|
| **Gratuito** | Para repositórios públicos, sem custo |
| **Integrado ao GitHub** | Workflow oficial pronto, sem cadastro em outro serviço |
| **Suficiente para o demo** | Hospeda HTML+CSS+JS, que é o suficiente para o frontend do nosso validador |

**Limitação importante**: GitHub Pages só serve **arquivos estáticos**. O Flask **não roda lá**. Por isso a lógica do validador também existe em JavaScript (capítulo 1) — para o demo público funcionar sem servidor.

## 8.2. O que vamos criar

Adicionados ao seu repositório neste capítulo:

```
tqs-2026/
├── .github/
│   └── workflows/
│       └── deploy.yml              ← novo
└── docs/
    ├── index.html                  ← novo (formulário e UI)
    ├── style.css                   ← novo (paleta UFOPA)
    └── validators.js               ← novo (espelho JS de src/validators.py)
```

## 8.3. Criar a pasta `docs/`

A pasta `docs/` é uma escolha de convenção: muitos projetos usam a branch `gh-pages` em vez disso, mas pasta na própria branch é mais simples.

1. Passe o mouse sobre o nome do repositório no Explorer → clique no ícone **"New Folder"**.
2. Nome: `docs` → Enter.

## 8.4. Criar `docs/style.css`

1. Clique direito em `docs/` → **"New File"** → nome: `style.css`.
2. Cole:

```css
:root {
    --azul: #006699;
    --azul-escuro: #004d73;
    --vermelho: #cc3300;
    --verde: #008040;
    --cinza-claro: #f5f5f5;
    --cinza-borda: #ccc;
}

* { box-sizing: border-box; }

body {
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    max-width: 720px;
    margin: 2rem auto;
    padding: 1.5rem;
    line-height: 1.6;
    color: #222;
    background: #fafafa;
}

header h1 {
    color: var(--azul);
    margin-bottom: 0.25rem;
}

.subtitulo {
    color: #666;
    margin-top: 0;
}

.aviso {
    background: var(--cinza-claro);
    border-left: 4px solid var(--azul);
    padding: 0.75rem 1rem;
    margin: 1.5rem 0;
    border-radius: 4px;
    font-size: 0.95rem;
}

form {
    display: grid;
    gap: 1rem;
    margin-top: 1.5rem;
    background: white;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}

label {
    font-weight: 600;
    display: block;
    margin-bottom: 0.25rem;
}

input {
    width: 100%;
    padding: 0.6rem;
    font-size: 1rem;
    border: 1px solid var(--cinza-borda);
    border-radius: 4px;
}

input:focus {
    outline: none;
    border-color: var(--azul);
    box-shadow: 0 0 0 2px rgba(0,102,153,0.2);
}

button {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    font-weight: 600;
    background: var(--azul);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.2s;
}

button:hover { background: var(--azul-escuro); }

.resultado {
    margin-top: 0.75rem;
    padding: 0.75rem 1rem;
    border-radius: 4px;
    font-weight: 600;
}

.valido { background: #e0f5e8; color: var(--verde); }
.invalido { background: #fbe5e0; color: var(--vermelho); }

footer {
    margin-top: 3rem;
    padding-top: 1rem;
    border-top: 1px solid #eee;
    font-size: 0.85rem;
    color: #666;
    text-align: center;
}

footer a { color: var(--azul); }
```

3. Salve.

## 8.5. Criar `docs/validators.js`

Esse é o **espelho JavaScript** das funções `validar_cpf` e `validar_email` do Python. Sem ele o demo do Pages não consegue validar nada (Flask não roda no GitHub Pages).

1. Clique direito em `docs/` → **"New File"** → nome: `validators.js`.
2. Cole:

```javascript
// Porte JavaScript dos validadores em src/validators.py.
// A versão principal é o Python — este arquivo existe apenas para
// permitir um demo interativo no GitHub Pages (que só hospeda
// conteúdo estático). Qualquer mudança de regra deve ser feita
// primeiro no Python (com testes), e depois replicada aqui.

const REGEX_EMAIL = /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;

function calcularDigitoVerificador(digitos, pesoInicial) {
    let soma = 0;
    for (let i = 0; i < digitos.length; i++) {
        soma += parseInt(digitos[i], 10) * (pesoInicial - i);
    }
    const resto = (soma * 10) % 11;
    return resto === 10 ? 0 : resto;
}

export function validarCpf(cpf) {
    if (typeof cpf !== "string") return false;

    const apenasDigitos = cpf.replace(/[.\-\s]/g, "");

    if (apenasDigitos.length !== 11 || !/^\d{11}$/.test(apenasDigitos)) return false;
    if (new Set(apenasDigitos).size === 1) return false;

    const primeiro = calcularDigitoVerificador(apenasDigitos.slice(0, 9), 10);
    const segundo = calcularDigitoVerificador(apenasDigitos.slice(0, 10), 11);

    return apenasDigitos[9] === String(primeiro) && apenasDigitos[10] === String(segundo);
}

export function validarEmail(email) {
    if (typeof email !== "string" || email.length === 0) return false;
    return REGEX_EMAIL.test(email);
}
```

3. Salve.

## 8.6. Criar `docs/index.html`

A página principal do demo público. Contém o formulário e o JavaScript que chama os validadores localmente (sem servidor).

1. Clique direito em `docs/` → **"New File"** → nome: `index.html`.
2. Cole:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TQS 2026 - Validador CPF/Email</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>Teste e Qualidade de Software - PC010027</h1>
        <p class="subtitulo">Validador de CPF e e-mail &middot; UFOPA &middot; 2026</p>
    </header>

    <div class="aviso">
        Esta página é a versão estática hospedada no GitHub Pages.
        A lógica de validação está implementada em JavaScript apenas para
        permitir o demo interativo. A versão principal, testada com pytest,
        está em <code>src/validators.py</code>.
    </div>

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

    <!-- TODO: substitua SEU-USUARIO pelo seu nome de usuário no GitHub -->
    <footer>
        Projeto-exemplo da disciplina &middot;
        <a href="https://github.com/SEU-USUARIO/tqs-2026">código-fonte no GitHub</a>
    </footer>

    <script type="module">
        import { validarCpf, validarEmail } from "./validators.js";

        document.getElementById("formulario").addEventListener("submit", (e) => {
            e.preventDefault();
            const cpf = document.getElementById("cpf").value;
            const email = document.getElementById("email").value;
            const cpfValido = validarCpf(cpf);
            const emailValido = validarEmail(email);
            const div = document.getElementById("resultado");
            div.innerHTML = `
                <div class="resultado ${cpfValido ? "valido" : "invalido"}">
                    CPF: ${cpfValido ? "válido" : "inválido"}
                </div>
                <div class="resultado ${emailValido ? "valido" : "invalido"}">
                    E-mail: ${emailValido ? "válido" : "inválido"}
                </div>
            `;
        });
    </script>
</body>
</html>
```

3. Salve.

> ⚠️ Antes de fazer o push final, **lembre de trocar `SEU-USUARIO`** no rodapé pela sua conta do GitHub. O capítulo 9 vai consolidar todas as personalizações do tipo num único checklist.

## 8.7. Criar o workflow `deploy.yml`

Esse workflow vai automatizar a publicação no GitHub Pages a cada push na `main`.

1. Clique direito em `.github/workflows/` (a pasta foi criada no capítulo 7) → **"New File"** → nome: `deploy.yml`.
2. Cole:

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    name: Preparar artefato
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar Pages
        uses: actions/configure-pages@v5

      - name: Upload do diretório docs/
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs

  deploy:
    name: Publicar no GitHub Pages
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy
        id: deployment
        uses: actions/deploy-pages@v4
```

3. Salve.

## 8.8. Dissecando o `deploy.yml`

### Quando dispara

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

- A cada `push` na `main` (após merge de PR, por exemplo)
- Manualmente via botão **"Run workflow"** na aba Actions (`workflow_dispatch`)

### Permissões necessárias

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

Sem essas permissões, o GitHub não autoriza a publicação no Pages. Esse é o "Login Sem Senha" entre GitHub Actions e GitHub Pages, usando *OIDC*.

### Concorrência

```yaml
concurrency:
  group: pages
  cancel-in-progress: false
```

Garante que **só um deploy roda por vez**. Se dois pushes acontecerem em sequência, o segundo espera o primeiro terminar — evita estado inconsistente no Pages.

### Os dois jobs

**Job 1 — build**: empacota a pasta `docs/` num artifact.
**Job 2 — deploy**: publica o artifact no Pages e devolve a URL final.

## 8.9. Commit e push

```bash
git add docs/ .github/workflows/deploy.yml
git commit -m "feat: site estático para GitHub Pages + workflow de deploy"
git push
```

## 8.10. Ativar o GitHub Pages no repositório

**Uma configuração manual** (uma vez só):

1. No GitHub, abra seu repositório no navegador.
2. Clique em **Settings** (aba superior).
3. No menu lateral esquerdo, clique em **Pages**.
4. Em **Build and deployment → Source**, escolha **GitHub Actions** (em vez do padrão "Deploy from a branch").
5. A página salva automaticamente.

A partir desse momento, o próximo push para `main` dispara o `deploy.yml` e publica o site.

## 8.11. Confirmando que está no ar

Volte na aba **Actions** do seu repositório e veja o workflow **"Deploy GitHub Pages"** rodando. Quando ficar verde (~1-2 min), uma URL aparece no resultado do job `deploy`: algo como

```
https://SEU-USUARIO.github.io/tqs-2026/
```

Abra a URL no navegador. Você deve ver:

- O título "Teste e Qualidade de Software - PC010027"
- O formulário com CPF e e-mail
- A faixa explicando que essa é a versão estática

Teste com `111.444.777-35` (válido) e `111.444.777-30` (inválido) — a validação acontece **inteiramente no navegador**, sem comunicação com servidor.

> Na primeira vez pode demorar alguns minutos para o DNS propagar. Tome um café.

## 8.12. Encadeando CI e deploy (opcional)

No estado atual, o deploy roda **mesmo se o CI estiver vermelho**. Para fazer o deploy **depender** do CI, mude o gatilho:

```yaml
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]

jobs:
  build:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
```

Não fizemos isso aqui para manter os dois workflows independentes — fica como exercício para times maduros.

## 8.13. Outras opções de deploy

Se quiser explorar além de GitHub Pages (por exemplo, para deployar o **Flask** de verdade):

| Serviço | Free tier? | Suporta Python? | Observação |
|---|---|---|---|
| [Render](https://render.com/) | ✅ | ✅ | Deploy a partir do GitHub via webhook |
| [Railway](https://railway.app/) | Limitado | ✅ | Excelente DX |
| [Fly.io](https://fly.io/) | ✅ (com limites) | ✅ | Requer Dockerfile |
| [Vercel](https://vercel.com/) | ✅ | Via serverless | Foco em Node.js mas suporta Python |

Esses ficam como leitura adicional.

---

Próximo capítulo: [Capítulo 9 — Pull Requests ➡](09-pull-requests.md)
