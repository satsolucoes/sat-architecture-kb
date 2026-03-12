# Config centralizado — constantes agrupadas por domínio semântico

## Contexto

Constantes espalhadas por múltiplos arquivos criam dependências implícitas e duplicação. Mudar um valor (cor, timing, tamanho) exige grep em vários lugares. Magic numbers aparecem sem contexto de onde vieram e o que significam.

## Decisão

Centralizar todas as constantes em `shared/config.ts`, agrupadas por domínio semântico com `as const`. Valores derivados referenciam a fonte — nunca repetem literais.

## Estrutura

```typescript
// shared/config.ts

// Grupo 1: defaults de infraestrutura
export const CANVAS_DEFAULTS = {
  width: 1300,
  height: 1300,
} as const;

// Grupo 2: estilos visuais com hierarquia de referência
export const BASE_STYLE = {
  outlineColor: '#333',
  surfaceColor: '#fff',
  strokeWidth: 1.5,
} as const;

export const ELEMENT_STYLE = {
  strokeColor: BASE_STYLE.outlineColor,      // ← referência, não duplica
  fillColor: BASE_STYLE.surfaceColor,
  selectedStrokeWidth: 4,
} as const;

export const MASTER_ELEMENT_STYLE = {
  fillColor: '#d4a574',
  strokeColor: '#8b4513',
  strokeWidth: ELEMENT_STYLE.selectedStrokeWidth,  // ← derivado
} as const;

// Grupo 3: timings — separados por clareza
export const TIMINGS = {
  tapToEditDelayMs: 300,
  debounceMs: 450,
  initPollMs: 100,
  initMaxRetries: 50,
} as const;

// Grupo 4: mensagens de feedback ao usuário
export const TOAST_MESSAGES = {
  exportSuccess: 'Exportado com sucesso!',
  invalidFile: 'Arquivo inválido.',
  itemAdded: (label: string): string => `${label} adicionado!`,
} as const;

// Grupo 5: chaves de storage
export const STORAGE_KEYS = {
  settings: 'app-settings',
  tutorialCompleted: 'app-tutorial-completed',
} as const;
```

## Re-export flat para compatibilidade com código legado

Quando código existente importa nomes flat, um arquivo de re-export evita quebrar imports sem duplicar valores:

```typescript
// shared/constants.ts — re-export plano, sem lógica
import { CANVAS_DEFAULTS, MASTER_ELEMENT_STYLE } from '@/shared/config.ts';

export const CANVAS_WIDTH = CANVAS_DEFAULTS.width;
export const MASTER_FILL_COLOR = MASTER_ELEMENT_STYLE.fillColor;
```

## Regras

1. `shared/config.ts` **não importa React, libs de UI nem nada de runtime** — apenas tipos e outros configs
2. **Valores derivados referenciam a fonte**: `strokeWidth: ELEMENT_STYLE.selectedStrokeWidth`, não `strokeWidth: 4`
3. **`as const`** em todos os objetos — habilita type narrowing nos callers
4. **Grouping semântico** > grouping alfabético: `ELEMENT_STYLE`, `MASTER_ELEMENT_STYLE` — coesos, não genéricos
5. **Mensagens de feedback** ficam no config, não inline nos handlers

## Antipadrões

```typescript
// ❌ Magic number inline
const strokeWidth = 1.5;

// ❌ Constante sem contexto
export const SW = 1.5;

// ❌ Duplicação — mesmo valor em dois lugares
export const FILL_COLOR = '#d4a574';
// ... em outro arquivo ...
export const RELATED_FILL = '#d4a574';

// ✅ Uma fonte, derivação explícita
export const MASTER_STYLE = { fillColor: '#d4a574' } as const;
export const DERIVED_STYLE = { fillColor: MASTER_STYLE.fillColor } as const;
```

## Quando usar

- Projeto com 3+ arquivos que importam a mesma constante
- Temas visuais que mudam por feature ou ambiente
- Timings que precisam de ajuste fino por dispositivo
- Mensagens de toast/erro que são testadas ou traduzidas

## Exemplo de origem

Padrão aplicado na migração v2→v3 do `rac-designer-teto`: ~50 constantes espalhadas em `lib/canvas-utils.ts` e `lib/house-manager.ts` foram consolidadas em `shared/config.ts` com hierarquia de referência.

## Referências

- [[one-source-of-truth]]
- [[explicit-domain-rules-naming]]

### Links KB
- [[clean-architecture-layers]] — `shared/` é a camada mais interna
- [[pragmatic-abstraction]] — não criar grupos por criar; grupo precisa ter coesão
