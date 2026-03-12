# Separação UI → componentes de apresentação em subpasta com prefixo de escopo

## Data: 2026-03-08

## Status: active

## Contexto

Um componente raiz com 1500+ linhas mistura lógica de estado, efeitos, e rendering de dezenas de sub-elementos (toolbar, modais, tutorial, canvas, overlays). Qualquer mudança visual exige navegar num arquivo monstruoso. Smoke tests são impraticáveis porque renderizar o componente puxa tudo junto.

## Decisão

Extrair componentes de apresentação em subpasta `ui/` agrupados por área visual, com prefixo do componente pai nos orquestradores:

```
components/<feature>/ui/
├── FeatureRoot.tsx              ← layout principal (composição, sem lógica)
├── FeatureModals.tsx            ← orquestra todos os modais do feature
├── FeatureToolbar.tsx           ← composição da toolbar
├── canvas/
│   ├── FeatureCanvas.tsx        ← canvas + overlays
│   └── CanvasOverlays.tsx       ← overlays puros (sem prefixo — genérico)
├── modals/
│   ├── SettingsModal.tsx
│   ├── ConfirmDialog.tsx
│   └── editors/
│       ├── FloatingEditor.tsx
│       ├── ObjectEditor.tsx
│       └── ObjectEditorIcon.tsx
├── toolbar/
│   ├── Toolbar.tsx
│   ├── ToolbarMainMenu.tsx
│   ├── ToolbarOverflowMenu.tsx
│   └── helpers/
│       ├── toolbar-config.ts    ← config específico da toolbar
│       └── toolbar-types.ts
└── tutorial/
    ├── Tutorial.tsx
    └── TutorialBalloon.tsx
```

## Critério de corte

**Componente vai para `ui/` quando:**
- Só recebe props e renderiza — sem lógica de domínio, sem hooks de negócio
- Pode ser smoke-testado com renderização simples
- Tem responsabilidade visual única

**Fica no componente pai quando:**
- Precisa do estado dos hooks para funcionar
- É o ponto de composição de múltiplos filhos

## Convenção de naming

Prefixo com nome do feature nos componentes que **orquestram**:

```tsx
// ✅ Prefixo indica escopo — orquestrador
FeatureModals.tsx       // reúne todos os modais do Feature
FeatureCanvas.tsx       // layout do canvas dentro do Feature

// Sem prefixo — componente genérico, potencialmente reutilizável
FloatingEditor.tsx
SettingsModal.tsx
TutorialBalloon.tsx
```

## `helpers/` dentro da subpasta

Config e tipos específicos de uma área visual ficam em `helpers/` dentro da própria subpasta — não vão para `shared/` (seriam específicos demais):

```
toolbar/helpers/toolbar-config.ts   ← só a toolbar usa
toolbar/helpers/toolbar-types.ts
```

## Benefícios

- Smoke tests em componentes individuais provam renderização básica sem dependência do estado global
- Cada arquivo tem ~50–250 linhas
- PR de mudança visual toca apenas o arquivo do componente — não o root inteiro
- Componentes `ui/` que importam hooks de domínio estão no lugar errado — é um sinal de alerta claro

## Exemplo de origem

Padrão aplicado na migração v2→v3 do `rac-designer-teto`: `RACEditor.tsx` (2125 linhas) foi decomposto em ~20 componentes de apresentação agrupados em `canvas/`, `modals/`, `toolbar/` e `tutorial/`.

## Referências

- [[god-object-detection-decomposition]]
- [[god-hook-decomposition-by-domain]]
- [[hooks-separation-by-responsibility]]

### Links KB
- [[smoke-test-naming-convention]] — smoke tests aplicados a esses componentes
- [[testing-strategy-adapted-pyramid]] — onde smoke tests se encaixam na pirâmide
