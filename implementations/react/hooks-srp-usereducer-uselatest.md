---
type: refactoring
title: Hooks com SRP — useReducer para estado relacionado, useLatest para sincronizar refs
status: active
impact: high
date: 2026-02-27
tags: [react, hooks, SRP, useReducer, useEffect, refs, state-management, performance]
applies_to:
  stacks: [react, react-native]
  contexts: [hooks, state-management, performance, architecture]
outcome: validated
---

## Problema 1: Múltiplos useState para um conceito único

Quando vários `useState`s representam facetas de um mesmo conceito (ex: estado de viewport),
qualquer transição que atualiza múltiplos valores dispara múltiplos renders e espelha a
mesma lógica em vários lugares.

### Sinal de alerta

```typescript
// 8 useState para um único conceito: viewport
const [zoom, setZoom] = useState(1);
const [viewportX, setViewportX] = useState(0);
const [viewportY, setViewportY] = useState(0);
const [containerSize, setContainerSize] = useState({ w: 0, h: 0 });
const [isPanning, setIsPanning] = useState(false);
const [isPinching, setIsPinching] = useState(false);
const [isSingleFingerPanning, setIsSingleFingerPanning] = useState(false);
const [lastPanPoint, setLastPanPoint] = useState<Point | null>(null);
```

### Solução: useReducer

```typescript
interface ViewportState {
  zoom: number;
  x: number;
  y: number;
  containerSize: { w: number; h: number };
  isPanning: boolean;
  isPinching: boolean;
  isSingleFingerPanning: boolean;
  lastPanPoint: Point | null;
}

type ViewportAction =
  | { type: 'SET_ZOOM'; payload: number }
  | { type: 'START_PAN'; payload: Point }
  | { type: 'UPDATE_PAN'; payload: Point }
  | { type: 'END_PAN' }
  | { type: 'START_PINCH' }
  | { type: 'END_PINCH' }
  | { type: 'RESIZE'; payload: { w: number; h: number } };

function viewportReducer(state: ViewportState, action: ViewportAction): ViewportState {
  switch (action.type) {
    case 'START_PAN':
      return { ...state, isPanning: true, lastPanPoint: action.payload };
    case 'UPDATE_PAN':
      return {
        ...state,
        x: state.x + (action.payload.x - (state.lastPanPoint?.x ?? 0)),
        y: state.y + (action.payload.y - (state.lastPanPoint?.y ?? 0)),
        lastPanPoint: action.payload,
      };
    case 'END_PAN':
      return { ...state, isPanning: false, lastPanPoint: null };
    // ...
  }
}

// Hook resultante — um único dispatch, transições atômicas
const [viewport, dispatchViewport] = useReducer(viewportReducer, initialViewportState);
```

**Vantagem**: transições atômicas (ex: `START_PAN` atualiza `isPanning` e `lastPanPoint` num único render).

---

## Problema 2: Múltiplos useEffect apenas para sincronizar props com refs

Padrão verboso onde cada prop tem seu próprio `useEffect` só para `ref.current = prop`.

### Sinal de alerta

```typescript
// 9 useEffects — 6 deles fazem apenas ref.current = prop
useEffect(() => { onClickRef.current = onClick; }, [onClick]);
useEffect(() => { onHoverRef.current = onHover; }, [onHover]);
useEffect(() => { onSelectRef.current = onSelect; }, [onSelect]);
// ... mais 3 iguais
```

### Solução: useLatest helper

```typescript
// Helper universal — sintetiza o padrão
function useLatest<T>(value: T): React.RefObject<T> {
  const ref = useRef(value);
  useEffect(() => {
    ref.current = value;
  }); // sem array de deps — atualiza em todo render (proposital)
  return ref;
}

// Uso — elimina os 6 useEffects repetitivos
const onClickRef = useLatest(onClick);
const onHoverRef = useLatest(onHover);
const onSelectRef = useLatest(onSelect);
```

**Resultado**: 6 `useEffect`s → 3 linhas. Mesma semântica, zero ruído.

---

## Critérios para decidir

| Situação                                                        | Solução                       |
|-----------------------------------------------------------------|-------------------------------|
| 3+ estados que transitam juntos                                 | `useReducer`                  |
| 2 estados independentes simples                                 | `useState` (ok)               |
| Props que precisam estar disponíveis em callbacks sem re-render | `useLatest` / `useRef`        |
| Estado derivado de outros estados                               | `useMemo` (nunca `useEffect`) |

## Threshold de alerta para hooks

| Métrica             | Alerta            |
|---------------------|-------------------|
| `useState` count    | > 5 no mesmo hook |
| `useEffect` count   | > 3 no mesmo hook |
| `useCallback` count | > 8 no mesmo hook |
| Linhas do hook      | > 150             |

Acima dos thresholds: avaliar extração ou consolidação.

## Lições aprendidas

- `useCanvasViewport` com 8 `useState`s era candidato claro para `useReducer`
- `useContraventamentoRefs` com 9 `useEffect`s tinha 6 puramente de sincronização — `useLatest` resolve
- O `useReducer` facilita testes: você testa o reducer puro separadamente dos efeitos colaterais

## Referências

- Origem: analysis-report.md (Hooks SRP, 2026-02-27)
- Origem: refactoring-plan.md (Fase 3, 2026-02-27)
