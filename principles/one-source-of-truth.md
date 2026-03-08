---
type: principle
title: One Source of Truth — constantes, estilos e configurações
status: active
impact: high
date: 2026-02-27
tags: [clean-code, constants, configuration, DRY, maintainability]
applies_to:
  stacks: [universal]
  contexts: [architecture, configuration, styling, constants]
outcome: validated
---

## Problema

Valores "chumbados" espalhados pelo código (magic numbers, strings literais, estilos duplicados)
causam inconsistências difíceis de rastrear e retrabalho em toda mudança de configuração.

## Princípio

Todo valor que pode mudar ou se repetir tem **um único ponto de definição**.
Qualquer lugar que precise do valor **referencia a constante**, nunca declara de novo.

## Categorias de aplicação

### 1. Constantes numéricas de domínio

```typescript
// ✅ Centralizado em shared/constants.ts
export const NUMERIC_EPSILON = 1e-4; // tolerância para comparações de ponto flutuante

// ❌ Espalhado pelo código
if (Math.abs(a - b) < 0.0001) { ... } // em 5 arquivos diferentes
```

### 2. Dimensões estruturais

```typescript
// ✅ Centralizado em house-dimensions.ts
export const HOUSE_DIMENSIONS = {
  piloti: { widthMt3: 0.15 },
  terrain: { rachaoMinCm: 13 }
};

// ❌ Valores repetidos em canvas, 3D e domínio
```

### 3. Estilos de UI

```typescript
// ✅ Objeto unificado
export const CANVAS_ELEMENT_STYLE = {
  fontSize: 15,
  strokeWidth: 1,
  strokeColor: { piloti: '#333', label: '#000' }
};

// ❌ Evolução problemática: CANVAS_STYLE_COLORS + CANVAS_TEXT_DEFAULTS separados
```

### 4. Configurações de feature flags

```typescript
// ✅ Centralizado em shared/config.ts com defaults explícitos
export const APP_SETTINGS_DEFAULTS = {
  showStairsOnTopView: false,
  openEditorsAtFixedPosition: false,
  disableDrawModeAfterFreehand: false
};
```

## Processo de aplicação

1. Antes de criar uma constante, buscar se já existe
2. Se existe mas está em lugar errado, mover e atualizar referências
3. Nomear pelo **propósito semântico**, não pelo valor (`NUMERIC_EPSILON`, não `SMALL_NUMBER`)

## Lições aprendidas

- `NUMERIC_EPSILON` eliminou 5 ocorrências de `0.0001` literal espalhadas
- Unificação de estilos em `CANVAS_ELEMENT_STYLE` eliminou inconsistências visuais entre vistas
- Renomear `Hex` → `Color` em sufixos de constantes de cor melhorou clareza sem custo

## Referências

- Origem: changelog-20260227 (padronização de código e constantes)
- Origem: changelog-20260303 (padronização de tolerância numérica)
