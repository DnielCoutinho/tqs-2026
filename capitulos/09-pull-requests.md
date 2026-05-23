# Capítulo 9 — Pull Requests

[⬅ Anterior: Deploy automatizado](08-deploy-automatizado.md) · [Sumário](../README.md) · Próximo: [Capítulo 10 — Atividade prática ➡](10-atividade-pratica.md)

---

Até aqui, tudo o que fizemos foi pessoal. Em **time**, o GitHub é o ponto de encontro: cada mudança vira um **Pull Request** que é discutido, revisado e mergeado. Este capítulo cria os **arquivos de governança** que vão estruturar essa colaboração, mostra o fluxo recomendado de PR, configura *branch protection* e ensina o básico de code review.

## 9.1. O que vamos criar

Adicionados ao seu repositório neste capítulo:

```
tqs-2026/
├── LICENSE                             ← novo (MIT)
└── .github/
    ├── CODEOWNERS                      ← novo
    ├── pull_request_template.md        ← novo
    └── ISSUE_TEMPLATE/                 ← nova pasta
        ├── bug_report.md               ← novo
        └── feature_request.md          ← novo
```

## 9.2. Criar `LICENSE`

A licença diz a quem usa o seu código **o que pode fazer com ele**. Sem licença, por padrão tudo é "todos os direitos reservados". Vamos usar a **MIT** — permissiva, simples, padrão da indústria.

1. Clique direito na raiz do repositório → **"New File"** → nome: `LICENSE` (tudo maiúsculo, sem extensão).
2. Cole:

```text
MIT License

Copyright (c) 2026 Helvecio Bezerra Leal Neto

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

3. Salve.

> O **copyright** acima é do autor original do material (o professor). Quando você forka, a licença MIT permite que você herde e use livremente — mas é boa prática manter o crédito original.

## 9.3. Criar `pull_request_template.md`

Esse arquivo aparece **automaticamente** preenchido na descrição quando alguém abre um PR. Garante que os autores não esqueçam de explicar a mudança ou checar os itens de qualidade.

1. Clique direito em `.github/` (a pasta foi criada no capítulo 7) → **"New File"** → nome: `pull_request_template.md`.
2. Cole:

```markdown
# Pull Request

## Descrição

<!-- Explique brevemente o que esta mudança faz e por que ela é necessária. -->

## Tipo de mudança

- [ ] Correção de bug
- [ ] Nova funcionalidade
- [ ] Refatoração (sem mudança de comportamento)
- [ ] Documentação
- [ ] Mudança no CI/CD

## Checklist de qualidade

- [ ] Os testes existentes continuam passando (`pytest`)
- [ ] Adicionei testes para a nova funcionalidade ou correção
- [ ] A cobertura de testes está em **≥ 80%**
- [ ] O lint passa sem erros (`ruff check . && ruff format --check .`)
- [ ] A análise de segurança passa (`bandit -r src/`)
- [ ] Atualizei o `README.md` se a mudança afetar o uso ou setup

## Como testar

<!-- Passos para o revisor reproduzir e validar a mudança. -->

1.
2.

## Issue relacionada

<!-- Ex.: Closes #12 -->
```

3. Salve.

## 9.4. Criar templates de issue

Quando alguém abre uma issue nova no seu repositório, o GitHub oferece uma escolha de templates. Vamos criar dois: um para bugs, outro para sugestões de funcionalidade.

### Criar a pasta `.github/ISSUE_TEMPLATE/`

1. Clique direito em `.github/` → **"New Folder"** → nome: `ISSUE_TEMPLATE` (tudo maiúsculo, sem espaços) → Enter.

### Criar `bug_report.md`

1. Clique direito em `.github/ISSUE_TEMPLATE/` → **"New File"** → nome: `bug_report.md`.
2. Cole:

```markdown
---
name: Reportar bug
about: Descreva um defeito encontrado no projeto
title: "[BUG] "
labels: bug
---

## Descrição do bug

<!-- Uma descrição clara e concisa do que está acontecendo. -->

## Como reproduzir

Passos para reproduzir o comportamento:
1. Vá para '...'
2. Execute '...'
3. Veja o erro

## Comportamento esperado

<!-- O que você esperava que acontecesse. -->

## Comportamento observado

<!-- O que de fato aconteceu. Inclua mensagens de erro completas. -->

## Ambiente

- Sistema operacional:
- Versão do Python:
- Versão do navegador (se aplicável):

## Contexto adicional

<!-- Screenshots, logs ou qualquer outra informação relevante. -->
```

3. Salve.

### Criar `feature_request.md`

1. Clique direito em `.github/ISSUE_TEMPLATE/` → **"New File"** → nome: `feature_request.md`.
2. Cole:

```markdown
---
name: Sugerir funcionalidade
about: Proponha uma melhoria ou nova funcionalidade
title: "[FEATURE] "
labels: enhancement
---

## Problema ou motivação

<!-- Qual problema esta funcionalidade resolve? Por que ela é necessária? -->

## Solução proposta

<!-- Descreva a solução que você imagina. -->

## Alternativas consideradas

<!-- Outras abordagens que você considerou e por que descartou. -->

## Critérios de aceitação

- [ ]
- [ ]
- [ ]

## Contexto adicional

<!-- Mockups, referências externas, links relacionados. -->
```

3. Salve.

> Os blocos `---` no topo são **frontmatter YAML** — metadados que o GitHub usa para criar a issue com o título, labels e descrição corretos.

## 9.5. Criar `CODEOWNERS`

Esse arquivo define **quem é automaticamente atribuído como revisor** de cada PR. Em projetos grandes, você pode dividir por pasta ("a equipe de segurança revisa tudo em `src/auth/`"). No nosso projeto vai ser simples: você revisa tudo.

1. Clique direito em `.github/` → **"New File"** → nome: `CODEOWNERS` (tudo maiúsculo, sem extensão).
2. Cole:

```text
# CODEOWNERS — define quem é atribuído automaticamente como revisor dos PRs.
#
# A regra abaixo está COMENTADA de propósito: o template não sabe qual é o seu
# usuário do GitHub. Descomente e troque SEU-USUARIO pelo seu nome de usuário
# para ativar a atribuição automática.
#
# Exemplo:
#   * @SEU-USUARIO
#
# Em projetos maiores, você pode dividir por pasta:
#   src/auth/   @time-de-seguranca
#   docs/       @time-de-conteudo
```

3. Salve.

> Por que comentado? Se você deixar `* @SEU-USUARIO` ativo sem trocar, o GitHub tenta atribuir um usuário inexistente e ignora silenciosamente. Mantendo comentado, **nenhum revisor é atribuído** até você descomentar com seu usuário real.

## 9.6. Personalizações finais antes do `push`

Agora que todos os arquivos foram criados, há **dois lugares** com o placeholder `SEU-USUARIO` que você precisa trocar pelo seu usuário real do GitHub:

| Arquivo | Linha | O que trocar |
|---|---|---|
| [`docs/index.html`](../docs/index.html) (criado no cap 8) | rodapé | `https://github.com/SEU-USUARIO/tqs-2026` → seu link real |
| [`.github/CODEOWNERS`](../.github/CODEOWNERS) (este capítulo) | comentário | Descomentar `# * @SEU-USUARIO` e trocar pelo seu usuário |

**Como encontrar todas as ocorrências no terminal do Codespace**:

```bash
grep -rn "SEU-USUARIO" .
```

Faça as substituições nos dois arquivos (clique no arquivo no Explorer, troque o texto, salve), rode o `grep` de novo para confirmar que retorna vazio.

## 9.7. Commit e push

```bash
git add LICENSE .github/
git commit -m "chore: adiciona licença MIT e governança de PRs/issues"
git push
```

🎉 **Você terminou de construir o repositório!** Da próxima vez que rodar `git status`, o resultado deve ser "nothing to commit, working tree clean".

## 9.8. Fluxo recomendado de PR

Vamos simular: "preciso adicionar um validador de CNPJ" (essa é literalmente a Tarefa 1 do capítulo 10).

### Passo 1 — Crie uma branch a partir da `main`

```bash
git checkout main
git pull                          # garante que está na versão mais recente
git checkout -b feat/validar-cnpj
```

> **Convenção**: prefixos como `feat/`, `fix/`, `docs/`, `refactor/`, `chore/` deixam claro o tipo de mudança. Não é obrigatório, mas ajuda a leitura do histórico.

### Passo 2 — Faça as mudanças seguindo TDD

```python
# tests/test_validators.py
def test_aceita_cnpj_valido():
    assert validar_cnpj("11.222.333/0001-81") is True
```

Roda → falha (função não existe). Implementa o mínimo. Refatora. (Capítulo 5.)

### Passo 3 — Garanta que tudo está verde no Codespace

```bash
ruff check . && ruff format --check .
bandit -r src/
pytest --cov=src --cov-fail-under=80
```

Se algo falhar, **conserte antes de comitar**. Você evita um round-trip com o CI.

### Passo 4 — Commit e push

```bash
git add .
git commit -m "feat: adiciona validador de CNPJ"
git push -u origin feat/validar-cnpj
```

> O `-u` (ou `--set-upstream`) só precisa ser usado no primeiro push da branch.

### Passo 5 — Abra o Pull Request

O GitHub vai mostrar um banner: **"Compare & pull request"**. Clique.

O **template de PR** que você criou na seção 9.3 aparece automaticamente no campo de descrição. Preencha:

- **Descrição**: por quê dessa mudança?
- **Tipo de mudança**: feat / fix / refactor / docs / CI
- **Checklist de qualidade**: cada item conferido
- **Como testar**: passos para o revisor reproduzir

### Passo 6 — Aguarde o CI

Em alguns minutos, o GitHub Actions vai rodar e mostrar:

- ✅ **All checks have passed** → pode pedir revisão
- ❌ **Some checks were not successful** → veja o que falhou, conserte, faça novo commit

> Cada novo commit na branch **re-dispara o CI** automaticamente.

### Passo 7 — Code review

Marque um revisor (ou aguarde alguém da equipe pegar). O revisor pode:

- **Comentar** em linhas específicas pedindo mudanças
- **Aprovar** o PR
- **Solicitar mudanças** (bloqueando o merge até serem feitas)

### Passo 8 — Merge

Quando o CI estiver verde e a revisão aprovada, clique em **Squash and merge** (recomendado para projetos pequenos — vira um único commit limpo na `main`).

### Passo 9 — Deletar a branch

O GitHub oferece um botão **"Delete branch"** logo após o merge. Use-o — branches mortas poluem.

## 9.9. Ativando *branch protection* na `main`

Para forçar que **toda mudança passe por PR + CI verde**, e ninguém (nem o admin) consiga dar push direto na `main`:

1. **Settings → Branches → Add branch protection rule**.
2. **Branch name pattern**: `main`.
3. Marque:
   - ☑ **Require a pull request before merging**
     - ☑ Require approvals: 1 (se for time)
   - ☑ **Require status checks to pass before merging**
     - ☑ Require branches to be up to date before merging
     - Selecione o job **`qualidade-e-testes`** (o nome vem de `ci.yml`)
   - ☑ **Do not allow bypassing the above settings**
4. **Create**.

> A partir daí, push direto na `main` é **bloqueado**. Toda mudança vira PR, e todo PR só mergeia com CI verde. **O processo virou portão de segurança automático.**

## 9.10. Code review básico — perguntas a se fazer

Ao revisar um PR, **passe pelas perguntas nessa ordem**:

1. **Faz o que diz que faz?** A descrição bate com o diff?
2. **Tem testes?** Eles cobrem o caminho feliz **e** os casos de borda?
3. **A cobertura caiu?** Se sim, alguma justificativa?
4. **O nome das funções/variáveis explica o que elas fazem?** Você precisaria de comentários?
5. **Há código morto, comentários esquecidos, prints de debug, TODOs vagos?**
6. **A mudança é proporcional ao problema?** Pequena correção de bug com 500 linhas refatoradas é sinal vermelho.

> **Cultura do review**: comente **o código**, não a pessoa. "Esse `if` poderia virar `dict.get(key, default)`" é diferente de "você não sabe usar `dict.get`". Diferença grande no clima do time.

## 9.11. Quando *não* mergear

Mesmo com CI verde e aprovação:

- Se a mudança gera **conflito** com `main`, peça ao autor para fazer `git merge main` (ou `git rebase main`) primeiro.
- Se está **sexta às 18h** e a mudança é grande, **espere segunda**. Mergear coisa grande às vésperas do final de semana é receita para incêndio.
- Se faltou conversa de design, peça para abrir uma **discussão** antes do PR seguir.

---

Próximo capítulo: [Capítulo 10 — Atividade prática ➡](10-atividade-pratica.md)
