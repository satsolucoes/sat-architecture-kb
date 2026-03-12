# God Hook — decomposição por domínio funcional em subpastas

## Contexto

Um hook com 500–2000+ linhas acumula lógica de múltiplos domínios funcionais — setup de biblioteca, viewport, interações de input, atalhos de teclado, clipboard, histórico, seleção, modais, toolbar, tutorial — tudo junto. Qualquer mudança num aspecto exige ler o arquivo inteiro. Testes são inviáveis. Re-renders desnecessários aparecem porque state de domínios diferentes vive no mesmo componente.

## Sinal de alerta

- Hook com mais de ~200 linhas
- Mais de 3 responsabilidades distintas num mesmo arquivo
- Parâmetros chegando a mais de 10 props
- Re-renders em interações que não deveriam afetar aquele slice de UI

## Taxonomia de decomposição

Agrupar por **domínio funcional** em subpastas, não por tipo de arquivo:

```
hooks/
├── <feature>/          ← domínio principal
│   ├── useSetup.ts                  ← inicialização e lifecycle
│   ├── useContainerLifecycle.ts     ← resize, refs de container
│   ├── useViewport.ts               ← zoom, pan, posição
│   ├── usePointerInteractions.ts    ← mouse/touch events
│   ├── useKeyboardShortcuts.ts      ← atalhos de teclado
│   ├── useHistory.ts                ← undo/redo
│   ├── useClipboard.ts              ← copy/paste
│   ├── useTools.ts                  ← ferramenta ativa
│   ├── useActions.ts                ← ações imperativas
│   └── useSelectionActions.ts       ← lógica de seleção
├── modals/
│   ├── useFloatingEditor.ts
│   ├── useObjectEditor.ts
│   └── useObjectEditorDraft.ts
├── toolbar/
│   └── useToolbarActions.ts
└── tutorial/
    ├── useTutorialFlow.ts
    └── useTutorialActions.ts
```

## Critério de corte

**Um hook = uma responsabilidade = um domínio funcional**

Perguntas para guiar o corte:
1. _"Se eu precisasse testar isso isoladamente, o que eu precisaria mockar?"_ — se a resposta for "muita coisa", ainda está grande.
2. _"Qual evento dispara esse estado?"_ — hooks que reagem a eventos diferentes devem ser separados.
3. _"Esse estado pertence à UI ou ao domínio?"_ — estado de domínio vai para o Aggregate; estado de UI fica no hook.

## Composição no componente raiz

O componente raiz consome hooks individuais e passa apenas o necessário para cada filho — não tem lógica própria:

```tsx
// FeatureRoot.tsx — composição, não lógica
const setup = useSetup({ containerRef, canvasRef });
const viewport = useViewport({ instance: setup.instance });
const history = useHistory({ instance: setup.instance });
const selection = useSelectionActions({ instance: setup.instance, onSelectionChange });

return (
  <div ref={containerRef}>
    <FeatureCanvas instance={setup.instance} />
    <FeatureOverlays viewport={viewport} />
  </div>
);
```

## Interface de saída: funções nomeadas, não callbacks genéricos

```typescript
// ✅ Interface clara — semântica explícita
export function useClipboard({ instance }: { instance: SomeLib | null }) {
  return {
    copy: () => { ... },
    paste: () => { ... },
  };
}

// ❌ Evitar — callback genérico sem semântica
export function useClipboard({ onAction }: { onAction: (type: string) => void }) { ... }
```

## Exemplo de origem

Padrão aplicado na migração v2→v3 do `rac-designer-teto`: `Canvas.tsx` (1634 linhas) e `RACEditor.tsx` (2125 linhas) foram decompostos em ~28 hooks especializados agrupados em `canvas/`, `modals/`, `toolbar/` e `tutorial/`.

## Referências

- [[god-object-detection-decomposition]]
- [[hooks-srp-usereducer-uselatest]]
- [[hooks-patterns-and-antipatterns]]

### Links KB
- [[domain-aggregate-root]] — para lógica que pertence ao domínio, não à UI
- [[hooks-separation-by-responsibility]] — decisão que fundamenta esse padrão
- [[stable-tests-before-refactoring]] — pré-condição antes de decompor
