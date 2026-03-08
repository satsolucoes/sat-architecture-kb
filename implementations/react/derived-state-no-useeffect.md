---
type: refactoring
title: Estado derivado não vive em useEffect — hooks e sincronização de estado
status: active
impact: high
date: 2026-02-27
tags: [react, hooks, state-management, useEffect, useMemo, useReducer, derived-state]
applies_to:
  stacks: [react, react-native]
  contexts: [state-management, hooks, performance]
outcome: validated
---

## Problema

`useEffect` usado para sincronizar ou derivar estado interno cria:

- Loops de render difíceis de rastrear
- Código que parece reativo mas esconde dependências implícitas
- Estado "fora de sincronia" em renders intermediários

## Regra

> useEffect é para side effects **externos** (chamadas de API, manipulação de DOM, timers).
> **Nunca** para transformar estado interno ou calcular valores derivados.

## Soluções por caso

### Caso 1: Valor calculável a partir de outros estados

```typescript
// ❌ Problema
useEffect(() => {
  setDerivedValue(computeFrom(stateA, stateB));
}, [stateA, stateB]);

// ✅ Solução: useMemo (valor derivado sem side effect)
const derivedValue = useMemo(
  () => computeFrom(stateA, stateB),
  [stateA, stateB]
);
```

### Caso 2: Estado complexo com múltiplas transições

```typescript
// ✅ Solução: useReducer (estado derivado vai pro reducer)
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'UPDATE_A':
      return {
        ...state,
        a: action.payload,
        derived: computeFrom(action.payload, state.b) // derivado calculado na transição
      };
  }
}
```

### Caso 3: Sincronização de estado local com props externas

```typescript
// ❌ Problema: bloqueio por ref impede atualização quando valor muda externamente
const lastIdRef = useRef<string>();
useEffect(() => {
  if (lastIdRef.current === id) return; // bloqueia atualização legítima
  lastIdRef.current = id;
  setLocalState(externalValue);
}, [id, externalValue]);

// ✅ Solução: sincronizar sempre que valores canônicos mudarem
useEffect(() => {
  setTempHeight(currentHeight);
  setTempNivel(currentNivel);
}, [currentHeight, currentNivel]); // sem bloqueio por ref
```

## Critério de decisão

*"Esse valor pode ser calculado a partir de outros estados?"*

- **Sim** → `useMemo` ou lógica no reducer
- **Não, precisa de I/O externo** → `useEffect`

## Lições aprendidas

- Remover `lastPilotiIdRef` de `usePilotiEditor` foi necessário porque o bloqueio impedia
  atualização correta dos botões após mudança de nível com recálculo automático
- Após `updatePiloti()`, ler os valores do manager (fonte canônica) e não do estado local temporário
  evitou inconsistência de UI

## Referências

- Origem: changelog-20260227 (migração de useCanvasViewport para useReducer)
- Origem: changelog-20260305 (correção de sincronização em usePilotiEditor)

### Links KB

- Relacionado: [[hooks-patterns-and-antipatterns]]
- Relacionado: [[hooks-srp-usereducer-uselatest]]
- Decisão: [[2026-02-27_hooks-separation-by-responsibility]]
