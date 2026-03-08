---
type: pattern
title: Reentry Guard — proteção contra recursão em callbacks de transformação
status: active
impact: medium
date: 2026-02-23
tags: [design-patterns, recursion-guard, DRY, event-handlers, callbacks]
applies_to:
  stacks: [universal]
  contexts: [canvas, animation, transformation, event-handlers, observers]
outcome: validated
---

## Problema

Callbacks que modificam o próprio objeto que disparou o evento causam recursão infinita.
A solução ingênua duplica o guard em cada função — código repetido, difícil de manter.

## Padrão: Reentry Guard genérico

### Conceito

```
1. Antes de executar: verificar se já está executando (flag no objeto)
2. Se sim: retornar imediatamente (evitar recursão)
3. Se não: setar flag → executar → limpar flag (em bloco finally)
```

O `finally` é obrigatório — sem ele, uma exception trava o guard para sempre.

### Pseudocódigo universal

```
function withReentryGuard(target, guardKey, fn):
  if target[guardKey] is true:
    return  // já está executando — sair

  target[guardKey] = true
  try:
    fn(target)
  finally:
    target[guardKey] = false  // sempre limpar, mesmo com exception
```

### Resultado: de duplicação para composição

```
// ❌ Antes: padrão duplicado em N lugares
normalizeLineScaling(obj):
  if obj.__normalizing: return
  obj.__normalizing = true
  try: normalize(obj, 'line')
  finally: obj.__normalizing = false

normalizeArrowScaling(obj):
  if obj.__normalizing: return     // duplicado
  obj.__normalizing = true         // duplicado
  try: normalize(obj, 'arrow')
  finally: obj.__normalizing = false  // duplicado

// ✅ Depois: guard centralizado, lógica específica injetada
normalizeLineScaling(obj):
  withReentryGuard(obj, '__normalizing', (o) => normalize(o, 'line'))

normalizeArrowScaling(obj):
  withReentryGuard(obj, '__normalizing', (o) => normalize(o, 'arrow'))
```

## Quando aplicar

- Callbacks que modificam o objeto que disparou o evento
- Observers/listeners que precisam ser idempotentes durante execução
- Qualquer fluxo onde reentrada causaria loop ou estado inconsistente
- Scaling, transformação, atualização de layout em sistemas de canvas

## Quando NÃO aplicar

- Operações assíncronas (a flag precisa de controle de concorrência real)
- Múltiplas instâncias concorrentes (a flag é por objeto, não global)

## Nomes de flag bons vs ruins

```
✅ __normalizingScale     — revela o que está guardando
✅ __updatingLayout       — descreve a operação protegida
✅ __processingEvent      — contexto claro

❌ __lock                 — genérico demais
❌ __busy                 — não diz o quê
❌ __flag                 — inútil
```

## Implementações por stack

- `implementations/react/` — não se aplica diretamente (React não usa este padrão)
- `implementations/typescript/reentry-guard.md` — implementação TypeScript com generics
- `implementations/kotlin/` ← (quando chegar)
- `implementations/java/` ← AtomicBoolean, synchronized

## Lições aprendidas

- O `finally` é o detalhe crítico — sem ele, qualquer exception deixa o guard travado
- O nome da flag deve descrever o que está sendo guardado, não apenas sinalizar "ocupado"
- Identificar este padrão duplicado exigiu análise cross-arquivo — vale adicionar ao code review checklist

## Referências

- Origem: refactoring-plan.md (Abstração de Padrões, 2026-02-23)
- Origem: analysis-report.md (Duplicação detectada em 4 strategies, 2026-02-27)

### Links KB

- Princípio: [[pragmatic-abstraction]]
- Relacionado: [[normalization-binding-extraction]]
- Implementação TypeScript: [[reentry-guard-typescript]]
