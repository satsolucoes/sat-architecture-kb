---
type: decision
title: <% tp.system.prompt("Título da decisão") %>
status: <% tp.system.suggester(["active", "deprecated", "experimental"], ["active", "deprecated", "experimental"], false, "Status") %>
impact: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"], false, "Impact") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [<% tp.system.prompt("Tags (separadas por vírgula)") %>]
applies_to:
  stacks: [<% tp.system.suggester(["universal", "react", "typescript", "kotlin", "java", "angular"], ["universal", "react", "typescript", "kotlin", "java", "angular"], true, "Stacks (multi-select)") %>]
  contexts: [<% tp.system.prompt("Contextos (ex: architecture, testing, CI-CD)") %>]
outcome: <% tp.system.suggester(["validated", "partial", "failed"], ["validated", "partial", "failed"], false, "Outcome") %>
---

## Contexto

> Qual era a situação que exigiu uma decisão? Quais forças estavam em jogo?

## Decisão

> O que foi decidido? Enunciado claro e direto.

## Motivação

> Por que esta opção foi escolhida em vez das alternativas?

## Alternativas consideradas

### Opção A — [nome]

- Prós:
- Contras:

### Opção B — [nome]

- Prós:
- Contras:

## Consequências

### Positivas
- 

### Negativas / Trade-offs
- 

## Regras derivadas

> Que regras práticas surgem desta decisão?

```
✅ Fazer:
❌ Não fazer:
```

## Como reverter (se aplicável)

> Passos para desfazer esta decisão caso necessário.

## Referências

- Origem:
- Relacionado: 
