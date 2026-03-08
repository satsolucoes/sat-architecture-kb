---
type: implementation
title: <% tp.system.prompt("Título da implementação") %>
status: active
impact: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"], false, "Impact") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [<% tp.system.prompt("Tags (separadas por vírgula)") %>]
applies_to:
  stacks: [<% tp.system.suggester(["react", "typescript", "kotlin", "java", "angular"], ["react", "typescript", "kotlin", "java", "angular"], true, "Stacks (multi-select)") %>]
  contexts: [<% tp.system.prompt("Contextos (ex: scaffolding, testing, architecture)") %>]
parent_pattern: <% tp.system.prompt("Padrão/decisão pai (ex: patterns/clean-architecture-layers.md)") %>
outcome: <% tp.system.suggester(["validated", "partial", "failed"], ["validated", "partial", "failed"], false, "Outcome") %>
---

## Contexto

> Para qual stack e cenário esta implementação se aplica?

## Estrutura de pastas / arquivos

```
src/
├── 
└── 
```

## Convenções de nomenclatura

| Artefato | Convenção | Extensão |
|----------|-----------|----------|
|          |           |          |

## Templates de código

### [Nome do artefato 1]

```typescript
// caminho: src/...
```

### [Nome do artefato 2]

```typescript
// caminho: src/...
```

## Restrições específicas desta stack

```
✅ Fazer:
❌ Não fazer:
```

## Checklist antes de criar novo artefato

```
□ 
□ 
□ 
```

## Diferenças em relação ao padrão base

> O que esta implementação adiciona ou adapta em relação ao padrão universal?

## Referências

- Padrão base: <% tp.system.prompt("Link para o padrão base") %>
- Origem: 
