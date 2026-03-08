---
type: refactoring
title: <% tp.system.prompt("Título do refactoring") %>
status: active
impact: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"], false, "Impact") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [<% tp.system.prompt("Tags (separadas por vírgula)") %>]
applies_to:
  stacks: [<% tp.system.suggester(["universal", "react", "typescript", "kotlin", "java", "angular"], ["universal", "react", "typescript", "kotlin", "java", "angular"], true, "Stacks (multi-select)") %>]
  contexts: [<% tp.system.prompt("Contextos (ex: hooks, components, domain)") %>]
outcome: <% tp.system.suggester(["validated", "partial", "failed"], ["validated", "partial", "failed"], false, "Outcome") %>
---

## Problema detectado

> Qual era o smell / problema que motivou o refactoring?

## Diagnóstico

> Como o problema foi identificado? Métricas, sintomas, contexto.

```
// código ou estrutura problemática
```

## Solução aplicada

> O que foi feito para resolver.

### Antes

```
// código antes
```

### Depois

```
// código depois
```

## Resultado

> O que melhorou concretamente? Métricas antes/depois se disponíveis.

| Métrica | Antes | Depois |
|---------|-------|--------|
|         |       |        |

## Lição abstraída

> Qual regra geral pode ser extraída desta experiência específica?

## Como detectar este problema em outros lugares

> Heurísticas, padrões de busca (rg, grep), checklist.

```bash
# comando para detectar o smell
rg "padrão" src/
```

## Cuidados e armadilhas

> O que pode dar errado ao aplicar esta solução? O que aprendemos na prática?

## Referências

- Origem:
- Relacionado: 
