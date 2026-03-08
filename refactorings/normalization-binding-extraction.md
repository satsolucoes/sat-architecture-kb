---
type: refactoring
title: Extração de funções de normalização e binding por tipo — eliminando duplicação de scaling
status: active
impact: medium
date: 2026-02-23
tags: [refactoring, DRY, factory, normalization, canvas, scaling, testability]
applies_to:
  stacks: [universal]
  contexts: [factory, refactoring, canvas, transformation]
outcome: validated
---

## Problema

Lógica de scaling/normalização duplicada dentro de funções de criação de elementos.
Cada `createElement()` replicava internamente a mesma lógica de ajuste dimensional.

## Resultado da refatoração

### Antes

```
createLine()     — lógica de scaling inline
createArrow()    — lógica de scaling inline (copiada)
createDistance() — lógica de scaling inline (copiada)
createWall()     — lógica de scaling inline (copiada)
```

150+ linhas duplicadas.

### Depois

```
Normalização (pura, testável isoladamente):
  normalizeLineCanvasObjectToLength(group, totalLength, labelTop)
  normalizeArrowCanvasObjectToLength(group, totalLength, labelTop)
  normalizeDistanceCanvasObjectToLength(group, newWidth, newHeight)
  normalizeWallCanvasObjectToLength(group, newWidth)

Binding (conecta evento → normalização):
  bindLineGroupScaling(group, labelTop)
  bindArrowCanvasObjectScaling(group, labelTop)
  bindDistanceGroupScaling(group, labelTop)
  bindWallGroupScaling(group)
```

## Métricas

| Aspecto               | Antes       | Depois     |
|-----------------------|-------------|------------|
| Duplicação de scaling | 150+ linhas | ~50 linhas |
| Funções auxiliares    | 0           | 13         |
| Testabilidade         | Média       | Alta       |

## Por que separar normalização de binding

**Normalização** (pura):

- Recebe objeto + dimensões
- Retorna/modifica objeto normalizado
- Testável sem event system
- Reutilizável em outros contextos (ex: aplicar ao carregar estado salvo)

**Binding** (side effect):

- Conecta evento de scaling → normalização
- Responsabilidade única: wiring
- Não contém lógica de negócio

```typescript
// Normalização — testável isoladamente
export function normalizeLineCanvasObjectToLength(
  group: Group,
  totalLength: number,
  labelTop: number
): void {
  // lógica pura de ajuste dimensional
}

// Binding — só faz wiring
export function bindLineGroupScaling(group: Group, labelTop: number): void {
  group.on('scaling', () => {
    withScalingGuard(group, (g) =>
      normalizeLineCanvasObjectToLength(g, g.width, labelTop)
    );
  });
}
```

## Padrão genérico aplicável em qualquer stack

```
Princípio: separar "o que fazer" de "quando fazer"

normalize*(data, params) → lógica pura
bind*(target, params)    → conecta evento → normalize*
```

Em qualquer stack com event listeners ou observers:

- **Kotlin**: função pura de transformação + observer que a chama
- **Java**: método estático de normalização + listener que o invoca
- **Angular**: pipe para transformação + event binding no template

## Lições aprendidas

- Extrair normalização como funções puras permitiu testar casos edge sem instanciar o canvas
- O binding pattern tornou o código extensível: novo tipo = nova função normalize + bind
- Colocar um placeholder `" "` na criação para manter contrato de seleção/edição foi necessário
  para compatibilidade com o sistema de editores

## Referências

- Origem: refactoring-plan.md (Fase 1 — Fundação, 2026-02-23)
- Origem: changelog-20260227 (redução de duplicação de scaling)

### Links KB

- Princípio: [[pragmatic-abstraction]]
- Princípio: [[one-source-of-truth]]
- Padrão: [[reentry-guard]]
- Implementação TypeScript: [[reentry-guard-typescript]]
- Decisão: [[2026-02-24_incremental-refactoring-never-big-bang]]
