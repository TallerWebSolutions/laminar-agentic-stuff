# Demand templates

Templates for work item title + body. Apply during step 5 of "Create demand".

## Title

Pattern: `[Quem] + [Onde/Quando] + [O que] (+ [Para que])`

| Slot | Source | Rule |
|------|--------|------|
| Quem | Project persona | Mandatory. Reuse existing persona names. |
| Onde/Quando | Screen, route, or persona/system routine | Mandatory. Tie to story map user step/activity. |
| O que | Expected behavior | Mandatory. Always present tense. |
| Para que | Goal/motivation | Optional. Only when previous slots feel meaningless without it. |

Feature and bug share the same title pattern. Never prefix with type — type is a structured field.

### Examples

**1.** `Ao enviar email em massa, CS preenche quem receberá o e-mail como cópia (cc)`
- Quem: CS · Onde/Quando: ao enviar email em massa · O que: preenche quem receberá email como cópia · Para que: —

**2.** `Colaboradores cadastrados pela Empresa no onboarding recebem acesso à Central`
- Quem: Colaboradores · Onde/Quando: cadastrados pela Empresa no onboarding · O que: recebem acesso à Central · Para que: —

**3.** `CS adiciona e remove permissões de login (ativa e desativa) das Empresas individualmente ou em massa, para que elas não acessem mais a central`
- Quem: CS · Onde/Quando: listagem das Empresas · O que: ativa e desativa Empresas individualmente ou em massa · Para que: não acessem mais a central

### Anti-patterns

- `Bug: login quebrado` → type prefix + missing slots
- `Implementar API de export` → no Quem, no Onde/Quando, infinitive instead of present
- `Melhorias na tela X` → no O que

## Body structure

Demand description body has these sections, in order. Problema and Critérios de aceite are mandatory; Apoio is optional (omit entirely if empty).

### Problema

One sentence max. Always present tense. Even for future features: describe the existing problem the feature solves. Problem only — no solution, no scope, no design notes.

Examples:
- ✅ `CS não consegue adicionar destinatários em cópia ao enviar email em massa.`
- ✅ `Colaboradores cadastrados via onboarding não conseguem acessar a Central.`
- ❌ `Vamos adicionar suporte a CC no envio de email em massa.` (solution, not problem)
- ❌ `Empresas pediram para conseguir ativar/desativar logins em massa para melhorar a gestão de acesso.` (mistura problema + solução + motivo, e tem mais de uma frase)

### Apoio

Links provided by the user: reference threads, docs, designs, related work items. Bullets only.

Skip section entirely if user provided none — don't fabricate placeholders.

### Critérios de aceite

**Required.** Gherkin only. No free-text AC, no checklist AC. **Keywords must match the demand's language** — if the demand body is pt-BR, write Gherkin in pt-BR; if English, write Gherkin in English. Never mix.

**pt-BR template** (use when demand is written in pt-BR):

```
Funcionalidade: <nome da funcionalidade>
  Como <persona>
  Quero <capacidade>
  Para que <resultado>

  Contexto:
    Dado <pré-condição compartilhada>

  Cenário: <comportamento observável>
    Dado <estado>
    Quando <evento>
    Então <resultado>
    E <resultado adicional>
```

**English template** (use when demand is written in English):

```
Feature: <feature name>
  As a <persona>
  I want <capability>
  So that <outcome>

  Background:
    Given <shared precondition>

  Scenario: <observable behavior>
    Given <state>
    When <event>
    Then <outcome>
    And <additional outcome>
```

Keyword map EN ↔ PT: `Feature`/`Funcionalidade`, `Background`/`Contexto`, `Scenario`/`Cenário`, `Scenario Outline`/`Esquema do Cenário`, `Examples`/`Exemplos`, `Given`/`Dado`, `When`/`Quando`, `Then`/`Então`, `And`/`E`, `But`/`Mas`.

Rules:
- One `Funcionalidade`/`Feature` per work item.
- One `Cenário`/`Scenario` per observable behavior; split when Então/Then-chain mixes unrelated outcomes.
- `Contexto`/`Background` only when ≥2 scenarios share preconditions.
- Bug AC uses same template; `Cenário`/`Scenario` describes broken behavior reproducibly. Root cause goes in Problema if known.
