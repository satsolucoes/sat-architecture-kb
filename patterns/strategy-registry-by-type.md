---
type: pattern
title: Strategy Pattern para criação de elementos por tipo (Factory + Strategy)
status: active
impact: medium
date: 2026-02-27
tags: [design-patterns, strategy, factory, open-closed, extensibility]
applies_to:
  stacks: [universal]
  contexts: [factory, design-patterns, extensibility]
outcome: validated
---

## Problema

Switch/if-else por tipo de elemento para decidir como criar/renderizar algo.
Adicionar um novo tipo exige modificar código existente (viola Open/Closed).

## Padrão

Registry de strategies por tipo. Cada tipo tem sua própria strategy.
Adicionar tipo novo = adicionar strategy, sem tocar no código existente.

## Estrutura

```typescript
// Contrato da strategy
interface ElementStrategy {
  create(canvas: Canvas, options?: ElementOptions): CanvasElement;
}

// Registry de strategies
function createHouseViewStrategies(factories: Factories): Record<ViewType, ElementStrategy> {
  return {
    top:   new TopViewStrategy(factories),
    front: new FrontViewStrategy(factories),
    back:  new BackViewStrategy(factories),
    side1: new SideViewStrategy(factories, 'side1'),
    side2: new SideViewStrategy(factories, 'side2'),
  };
}

// Resolver por tipo (elimina o switch manual)
function getHouseViewStrategy(viewType: ViewType, factories?: Factories): ElementStrategy {
  const strategies = createHouseViewStrategies(factories);
  return strategies[viewType];
}
```

## Antes vs Depois

```typescript
// ❌ Antes: switch manual espalhado
switch (viewType) {
  case 'top':   return createHouseTop(canvas);
  case 'front': return createHouseFrontBack(canvas, 'front');
  case 'back':  return createHouseFrontBack(canvas, 'back');
  case 'side1': return createHouseSide(canvas, 'side1');
  case 'side2': return createHouseSide(canvas, 'side2');
}

// ✅ Depois: delegação ao registry
return getHouseViewStrategy(viewType, factories).create(canvas, { side });
```

## Quando aplicar

- Quando você tem 3+ variações de comportamento por tipo
- Quando adicionar novos tipos é provável
- **Não aplicar** para 2 variações simples — if/else é mais legível

## Aplicabilidade em outras stacks

- **Kotlin**: sealed classes + when expression, ou interface Strategy com registry
- **Java**: Map<Type, Strategy> com injeção de dependência
- **Angular**: providers por token

## Lições aprendidas

- A strategy recebeu `options?` opcionais para não quebrar compatibilidade com factories existentes
- O registry eliminou o switch manual em `house-view-creation.ts`

## Referências

- Origem: changelog-20260227 (House Factory Strategies)

### Links KB

- Princípio: [[pragmatic-abstraction]]
- Princípio: [[explicit-domain-rules-naming]]
- Relacionado: [[clean-architecture-layers]]
- Relacionado: [[port-adapter-persistence]]
