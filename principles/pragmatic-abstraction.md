---
type: principle
title: Abstrações com ganho real (Pragmatic Abstraction)
status: active
impact: high
date: 2026-02-27
tags: [clean-architecture, SOLID, abstraction, pragmatism, design]
applies_to:
  stacks: [universal]
  contexts: [architecture, design, refactoring]
outcome: validated
---

## Problema

Criar abstrações (Ports, Aggregates, Strategies, Adapters) por hábito ou por seguir Clean Architecture
"ao pé da letra" resulta em camadas anêmicas, mais código para manter e zero ganho real.

## Princípio

Só introduza uma abstração quando ela tiver ganho imediato e concreto:

- Reduz duplicação real de código
- Facilita extensão futura com custo baixo
- Clarifica responsabilidades que estavam misturadas

## Critérios de decisão (antes de criar qualquer abstração)

1. **Inventário**: já existe algo que resolve isso?
2. **Decisão**: reutilizar vs extrair comum vs criar novo
3. **Justificativa**: qual ganho concreto essa abstração entrega agora?

## Exemplos de aplicação

### ✅ Abstração com ganho real

`HousePersistencePort` + `InMemoryHousePersistenceAdapter`:

- Desacopla domínio de mecanismo de persistência
- Facilita troca futura (localStorage → API) sem tocar no domínio
- Custo: baixo. Ganho: preparação real para evolução futura.

### ❌ Abstração sem ganho

`HouseAggregate` removido porque:

- Adicionava indireção sem comportamento próprio
- O `HouseManager` já cumpria o papel
- Custo: manutenção extra. Ganho: nenhum imediato.

### ❌ Anti-padrão: encadeamento sem ganho

```typescript
// ❌ Evitar: cadeia que não reduz código nem clareza
function getMinHeight(nivel: number) {
  return calculateBase(nivel);
}
function calculateBase(nivel: number) {
  return applyFactor(nivel);
}
function applyFactor(nivel: number) {
  return nivel * 3;
}

// ✅ Preferir: direto quando não há divergência de regra
function getMinimumPilotiHeightForNivel(nivel: number) {
  return nivel * 3;
}
```

## Lições aprendidas

- Remover `HouseAggregate` simplificou o código sem perda de clareza
- `HousePersistencePort` valeu porque o ganho de desacoplamento era concreto e imediato
- Wrappers em cadeia sem divergência de regra são ruído, não arquitetura

## Referências

- Decisão original: changelog-20260227 (Discussão arquitetural e decisões finais)
- Anti-padrão registrado: changelog-20260303 (diretriz de simplicidade)

### Links KB

- Relacionado: [[explicit-domain-rules-naming]]
- Relacionado: [[mandatory-decision-flow]]
- Relacionado: [[one-source-of-truth]]
- Padrão: [[clean-architecture-layers]]
