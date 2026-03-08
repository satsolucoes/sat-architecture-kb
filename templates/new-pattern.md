---
type: pattern
title: <% tp.system.prompt("Título do padrão") %>
status: active
impact: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"], false, "Impact") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [<% tp.system.prompt("Tags (separadas por vírgula)") %>]
applies_to:
  stacks: [<% tp.system.suggester(["universal", "react", "typescript", "kotlin", "java", "angular"], ["universal", "react", "typescript", "kotlin", "java", "angular"], true, "Stacks (multi-select)") %>]
  contexts: [<% tp.system.prompt("Contextos (ex: architecture, canvas, event-handlers)") %>]
outcome: <% tp.system.suggester(["validated", "partial", "failed"], ["validated", "partial", "failed"], false, "Outcome") %>
---

## Problema

> Que problema recorrente este padrão resolve?

## Solução

> Descrição da solução — estrutura, participantes, colaborações.

## Estrutura

```
// diagrama ou pseudocódigo da estrutura
```

## Quando aplicar
- 

## Quando NÃO aplicar
- 

## Implementação

### ❌ Antes (sem o padrão)

```
// código ou pseudocódigo
```

### ✅ Depois (com o padrão)

```
// código ou pseudocódigo
```

## Consequências

> Benefícios e trade-offs de usar este padrão.

## Implementações por stack

- `implementations/react/` ←
- `implementations/kotlin/` ← (futuro)

## Referências

- Origem:
- Relacionado: 
