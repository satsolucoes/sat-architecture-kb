---
type: principle
title: <% tp.system.prompt("Título do princípio") %>
status: active
impact: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"], false, "Impact") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [<% tp.system.prompt("Tags (separadas por vírgula)") %>]
applies_to:
  stacks: [<% tp.system.suggester(["universal", "react", "typescript", "kotlin", "java", "angular"], ["universal", "react", "typescript", "kotlin", "java", "angular"], true, "Stacks (multi-select)") %>]
  contexts: [<% tp.system.prompt("Contextos (ex: architecture, testing, performance)") %>]
outcome: <% tp.system.suggester(["validated", "partial", "failed"], ["validated", "partial", "failed"], false, "Outcome") %>
---

## Problema

> Que situação ou dor este princípio endereça?

## Princípio

> Enunciado claro e direto — uma frase se possível.

## Motivação

> Por que este princípio existe? Qual o custo de ignorá-lo?

## Aplicação

### ✅ Como aplicar
- 

### ❌ Anti-padrões
- 

## Exemplos

### Correto

```
// descrever ou mostrar código
```

### Incorreto

```
// o que NÃO fazer
```

## Lições aprendidas

> Contexto real de onde este princípio surgiu ou foi validado.

## Referências

- Origem:
- Relacionado: 
