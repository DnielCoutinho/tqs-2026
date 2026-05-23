# Capítulo 5 — TDD na prática

[⬅ Anterior: Testes automatizados](04-testes-automatizados.md) · [Sumário](../README.md) · Próximo: [Capítulo 6 — Qualidade de código ➡](06-qualidade-de-codigo.md)

---

Sabemos rodar testes que já existem. Agora vamos aprender a **escrever testes antes do código** — esse é o coração do TDD.

> 📜 **Capítulo retrospectivo.** Nos capítulos 3 e 4 você já criou o `src/validators.py` com a versão final e o `tests/test_validators.py` com a suíte completa. Aqui o livro **conta a história**: como um desenvolvedor disciplinado teria chegado àquele código fazendo TDD de verdade. Você pode acompanhar lendo e entendendo — ou recriar os passos numa branch de estudo. **A oportunidade de praticar TDD com as próprias mãos está no [capítulo 10](10-atividade-pratica.md)**, ao adicionar CNPJ e telefone.

## 5.1. O que é TDD?

**TDD** = *Test-Driven Development* (desenvolvimento guiado por testes).

A ideia parece contraintuitiva: você escreve o teste de uma funcionalidade **antes** de implementá-la. O ciclo é sempre:

1. 🔴 **RED** — escreva um teste que **falha** (porque o código ainda não existe).
2. 🟢 **GREEN** — escreva a **menor quantidade de código** possível para o teste passar.
3. 🔵 **REFACTOR** — melhore o código sem mudar o comportamento; os testes continuam verdes.

E repete. **Sempre nessa ordem.**

> **Por que essa ordem?**
> - Escrever o teste primeiro força você a pensar na **interface** (como a função vai ser usada) antes da **implementação** (como ela vai funcionar).
> - Ver o teste falhar **antes** de implementar prova que o teste de fato testa algo. Já vi muito teste que passa mesmo com o código quebrado.
> - O passo "refactor" só é seguro porque os testes funcionam como uma **rede de segurança**: você refatora; rodando os testes, sabe imediatamente se quebrou algo.

## 5.2. Walkthrough completo: criando `validar_cpf` via TDD

Vamos refazer a implementação de [`validar_cpf`](../src/validators.py) passo a passo, escrevendo cada teste **antes** do código que o faz passar.

### 🔴 Ciclo 1 — Aceitar um CPF válido

**RED.** Crie [`tests/test_validators.py`](../tests/test_validators.py) com:

```python
from src.validators import validar_cpf

def test_aceita_cpf_valido():
    assert validar_cpf("111.444.777-35") is True
```

Rode `pytest`. **Esperamos a falha**:

```
E   ModuleNotFoundError: No module named 'src.validators'
```

Isso é **bom**. O teste está testando algo (a existência do módulo). Se passasse de primeira, seria suspeito.

**GREEN.** Crie [`src/validators.py`](../src/validators.py) com a implementação mais boba possível:

```python
def validar_cpf(cpf: str) -> bool:
    return True
```

Rode `pytest`. **Passa**.

Sim, é "trapaça" — mas é **proposital**. O próximo teste vai expor a trapaça.

### 🔴 Ciclo 2 — Rejeitar tamanho inválido

**RED.** Adicione:

```python
def test_rejeita_cpf_com_tamanho_invalido():
    assert validar_cpf("123") is False
```

Roda → **falha** (a função sempre retorna `True`).

**GREEN.** Mínimo para passar:

```python
def validar_cpf(cpf: str) -> bool:
    apenas_digitos = cpf.replace(".", "").replace("-", "")
    if len(apenas_digitos) != 11:
        return False
    return True
```

Roda → **passa**.

### 🔴 Ciclo 3 — Rejeitar todos os dígitos iguais

**RED.**

```python
def test_rejeita_cpf_com_todos_digitos_iguais():
    assert validar_cpf("11111111111") is False
```

Roda → **falha** (`"11111111111"` tem 11 dígitos, então passa pelo check anterior).

**GREEN.**

```python
def validar_cpf(cpf: str) -> bool:
    apenas_digitos = cpf.replace(".", "").replace("-", "")
    if len(apenas_digitos) != 11:
        return False
    if len(set(apenas_digitos)) == 1:
        return False
    return True
```

### 🔴 Ciclo 4 — Validar dígitos verificadores

**RED.**

```python
def test_rejeita_cpf_com_digito_verificador_errado():
    assert validar_cpf("111.444.777-30") is False
```

Roda → **falha**. Agora **não tem como trapacear**: precisa implementar o algoritmo dos dígitos verificadores de verdade.

**GREEN.** O algoritmo do CPF: cada um dos 9 primeiros dígitos é multiplicado por pesos decrescentes (10, 9, 8, ..., 2). A soma dos produtos vezes 10 dividido por 11 dá o resto que é o primeiro dígito verificador (0 se for 10). Mesma lógica com pesos de 11 a 2 para o segundo dígito.

```python
def validar_cpf(cpf: str) -> bool:
    apenas_digitos = cpf.replace(".", "").replace("-", "")
    if len(apenas_digitos) != 11 or not apenas_digitos.isdigit():
        return False
    if len(set(apenas_digitos)) == 1:
        return False

    # primeiro dígito verificador
    soma = sum(int(apenas_digitos[i]) * (10 - i) for i in range(9))
    primeiro = (soma * 10) % 11
    if primeiro == 10:
        primeiro = 0

    # segundo dígito verificador
    soma = sum(int(apenas_digitos[i]) * (11 - i) for i in range(10))
    segundo = (soma * 10) % 11
    if segundo == 10:
        segundo = 0

    return apenas_digitos[9] == str(primeiro) and apenas_digitos[10] == str(segundo)
```

Roda → **todos os testes passam**.

### 🔵 REFACTOR

Olhando o resultado, vemos que o cálculo dos dois dígitos é **muito parecido**. Vamos extrair uma função auxiliar:

```python
def _calcular_digito_verificador(digitos: str, peso_inicial: int) -> int:
    pesos = range(peso_inicial, 1, -1)
    soma = sum(int(d) * peso for d, peso in zip(digitos, pesos, strict=True))
    resto = (soma * 10) % 11
    return 0 if resto == 10 else resto


def validar_cpf(cpf: str) -> bool:
    apenas_digitos = cpf.replace(".", "").replace("-", "")
    if len(apenas_digitos) != 11 or not apenas_digitos.isdigit():
        return False
    if len(set(apenas_digitos)) == 1:
        return False

    primeiro = _calcular_digito_verificador(apenas_digitos[:9], peso_inicial=10)
    segundo = _calcular_digito_verificador(apenas_digitos[:10], peso_inicial=11)
    return apenas_digitos[9] == str(primeiro) and apenas_digitos[10] == str(segundo)
```

Roda os testes de novo: **todos passam**. O refator foi seguro porque a rede de segurança já estava montada.

## 5.3. Princípio para levar consigo

> O teste é a **rede de segurança** que permite refatorar (e adicionar features) sem medo.

Cada vez que você bate em um bug em produção, pergunte: **"qual teste eu deveria ter escrito que teria pegado isso?"** Escreva esse teste primeiro (vai falhar — confirmando que o bug existe). Depois conserte o código. Esse hábito sozinho já melhora muito a qualidade do código.

---

Próximo capítulo: [Capítulo 6 — Qualidade de código ➡](06-qualidade-de-codigo.md)
