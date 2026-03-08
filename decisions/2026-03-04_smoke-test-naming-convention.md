---
type: decision
title: Nomenclatura de smoke tests por arquivo-alvo
status: active
impact: medium
date: 2026-03-04
tags: [testing, smoke-tests, naming, conventions, maintainability]
applies_to:
  stacks: [universal]
  contexts: [testing, conventions, ci-cd]
outcome: validated
---

## Decisão

O nome do smoke test deve refletir exatamente o arquivo-alvo testado.

## Regra de nomenclatura

```
<arquivo-alvo>.smoke.test.ts  →  testa  →  <arquivo-alvo>.ts
```

### Exemplos

```
house-auto-stairs.smoke.test.ts      →  house-auto-stairs.ts
house-manager.smoke.test.ts          →  house-manager.ts
terrain-volume.smoke.test.ts         →  terrain-volume.ts
```

## Quando consolidar

Quando dois smoke tests cobrem o mesmo arquivo-alvo, consolidar em um só:

```
# Antes (colisão)
house-auto-stairs.smoke.test.ts
house-auto-stairs-settings.smoke.test.ts  # mesmo alvo: house-auto-stairs.ts

# Depois (consolidado)
house-auto-stairs.smoke.test.ts  # cobre ambos os cenários
```

## Checklist de validação (por rodada)

1. `npx vitest run smoke.test` — todos os smoke tests
2. `npm run lint` — sem erros (warnings conhecidos documentados)
3. `npm run test:e2e` — fluxos críticos
4. `npx tsc --noEmit` — tipagem limpa

## Critério mínimo de aceitação

- Smoke tests: 100% passando
- Lint: sem erros (warnings preexistentes documentados e conhecidos)
- E2E: 100% passando
- TypeScript: sem erros novos introduzidos

## Lições aprendidas

- Nomenclatura inconsistente dificultava saber o que cada teste cobria
- Consolidação eliminou duplicação e colisão de nomes
- `describe` principal nomeado igual ao arquivo-alvo facilita `grep` e navegação

## Referências

- Origem: changelog-20260304 (padronização dos nomes dos describe)
