---
type: pattern
title: useLatest — sincronização de props com refs sem múltiplos useEffects
status: active
impact: medium
date: 2026-02-27
tags: [react, hooks, useEffect, refs, performance, SRP]
applies_to:
  stacks: [react, react-native]
  contexts: [hooks, state-management, performance]
outcome: validated
---

## Problema

Sincronizar múltiplas props com refs usando um `useEffect` por prop cria:

- Hooks verbosos com responsabilidade única trivial
- Violação de SRP (múltiplos effects para o mesmo conceito)
- Re-renders desnecessários em alguns cenários

## Anti-padrão identificado

```typescript
// ❌ 6 useEffects fazendo apenas ref.current = prop
function useContraventamentoRefs(props: Props) {
  const pilotiRef = useRef(props.piloti);
  const canvasRef = useRef(props.canvas);
  const viewRef = useRef(props.view);
  const modeRef = useRef(props.mode);
  const onSelectRef = useRef(props.onSelect);
  const onCancelRef = useRef(props.onCancel);

  useEffect(() => { pilotiRef.current = props.piloti; }, [props.piloti]);
  useEffect(() => { canvasRef.current = props.canvas; }, [props.canvas]);
  useEffect(() => { viewRef.current = props.view; }, [props.view]);
  useEffect(() => { modeRef.current = props.mode; }, [props.mode]);
  useEffect(() => { onSelectRef.current = props.onSelect; }, [props.onSelect]);
  useEffect(() => { onCancelRef.current = props.onCancel; }, [props.onCancel]);
  // + 3 useEffects com lógica real de sincronização
  // Total: 9 useEffects para o que poderia ser muito menos
}
```

## Solução 1: useLatest helper (mais elegante)

```typescript
// hooks/useLatest.ts
export function useLatest<T>(value: T): RefObject<T> {
  const ref = useRef(value);
  // Atualiza de forma síncrona, sem useEffect
  ref.current = value;
  return ref;
}

// Uso: 6 linhas no lugar de 12
function useContraventamentoRefs(props: Props) {
  const pilotiRef = useLatest(props.piloti);
  const canvasRef = useLatest(props.canvas);
  const viewRef = useLatest(props.view);
  const modeRef = useLatest(props.mode);
  const onSelectRef = useLatest(props.onSelect);
  const onCancelRef = useLatest(props.onCancel);

  // Apenas os 3 useEffects com lógica real permanecem
}
```

## Solução 2: objeto de refs consolidado

```typescript
// Quando as refs são sempre usadas juntas
function useContraventamentoRefs(props: Props) {
  const refs = useRef(props);

  // Um único useEffect no lugar de 6
  useEffect(() => {
    refs.current = props;
  }); // sem deps = sempre atualiza após render (seguro para funções/handlers)

  return refs;
}
```

## Quando usar cada solução

| Cenário                              | Solução preferida                      |
|--------------------------------------|----------------------------------------|
| Refs usadas independentemente        | `useLatest` por prop                   |
| Refs sempre acessadas juntas         | Objeto consolidado                     |
| Callbacks (evitar stale closure)     | `useLatest` ou `useCallbackRef`        |
| Estado que precisa de lógica na sync | `useEffect` explícito (não substituir) |

## Regra de ouro

> Se o `useEffect` não tem lógica além de `ref.current = value`,
> substitua por atribuição direta ou `useLatest`.

## Aplicabilidade

- React e React Native: idêntico
- Qualquer padrão que sincroniza "valor mais recente" com closure/callback estável

## Referências

- Origem: analysis-report 2026-02-27 (9 useEffects em useContraventamentoRefs)
