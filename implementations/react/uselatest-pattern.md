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
// ❌ N useEffects fazendo apenas ref.current = prop
function useComponentRefs(props: Props) {
  const valueARef = useRef(props.valueA);
  const valueBRef = useRef(props.valueB);
  const onSelectRef = useRef(props.onSelect);
  const onCancelRef = useRef(props.onCancel);

  useEffect(() => { valueARef.current = props.valueA; }, [props.valueA]);
  useEffect(() => { valueBRef.current = props.valueB; }, [props.valueB]);
  useEffect(() => { onSelectRef.current = props.onSelect; }, [props.onSelect]);
  useEffect(() => { onCancelRef.current = props.onCancel; }, [props.onCancel]);
  // + useEffects com lógica real de sincronização
  // Resultado: muitos effects para o que poderia ser bem menos
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

// Uso: metade das linhas, sem effects triviais
function useComponentRefs(props: Props) {
  const valueARef = useLatest(props.valueA);
  const valueBRef = useLatest(props.valueB);
  const onSelectRef = useLatest(props.onSelect);
  const onCancelRef = useLatest(props.onCancel);

  // Apenas os useEffects com lógica real permanecem
}
```

## Solução 2: objeto de refs consolidado

```typescript
// Quando as refs são sempre usadas juntas
function useComponentRefs(props: Props) {
  const refs = useRef(props);

  // Um único useEffect no lugar de N
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

### Links KB

- Relacionado: [[hooks-srp-usereducer-uselatest]]
- Relacionado: [[hooks-patterns-and-antipatterns]]
