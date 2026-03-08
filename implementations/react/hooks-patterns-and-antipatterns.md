---
type: pattern
title: Hooks — responsabilidades, hierarquia de estado e anti-padrões
status: active
impact: high
date: 2026-02-27
tags: [react, hooks, state-management, architecture, SRP, context]
applies_to:
  stacks: [react, react-native]
  contexts: [hooks, state-management, architecture, component-design]
outcome: validated
---

## Quando criar um hook customizado

- Comportamento reutilizável entre componentes
- Ligação com estado compartilhado
- Coordenação entre UI e dependências externas (canvas, events, timers)

## Retorno de hooks: objeto, não array

```typescript
// ✅ Objeto — contrato explícito, sem erro de ordem
function useEditor() {
  return { state, dispatch, isLoading };
}
const { state, isLoading } = useEditor();

// ❌ Array — propenso a erro de ordem
function useEditor() {
  return [state, dispatch, isLoading];
}
const [state, , isLoading] = useEditor(); // bug silencioso
```

## Hierarquia de estado (usar na ordem)

```
1. useState / useReducer    → UI simples e isolada
2. Lifting state up         → irmãos compartilham mesmo dado
3. Context (useContext)     → múltiplos níveis precisam da mesma instância
4. Store de domínio da feature → estado complexo com fonte de verdade explícita
```

Não pular níveis sem justificativa. Context antes de store global, store global
antes de solução externa (Zustand/Redux).

## Separação de responsabilidades em hooks de feature

```
hooks/state/    → leitura e seleção de estado
hooks/commands/ → dispatch de comandos a partir da UI
hooks/bindings/ → integração com canvas, listeners e eventos externos
```

### O que hooks PODEM fazer

```typescript
✅ Ler estado do store
✅ Despachar comandos no store
✅ Registrar listeners e bindings (canvas, keyboard, resize)
✅ Coordenar consumo de estado e eventos
```

### O que hooks NÃO PODEM fazer

```typescript
❌ Importar fabric/canvas diretamente (vai para infra/)
❌ Conter regra de negócio de domínio
❌ Alterar estado complexo por fora do fluxo de dispatch
❌ Virar "god-hook" misturando state + command + binding sem necessidade
```

## Store de feature não é singleton global

```typescript
// ✅ Criado no bootstrap, injetado via Context
// bootstrap/editor-bootstrap.ts
const store = createHouseStateStore(persistence);

// ✅ Consumido via Context
const store = useContext(HouseStateStoreContext);

// ❌ Singleton exportado diretamente
export const houseStateStore = new HouseStateStore(); // PROIBIDO
```

## Thresholds de alerta para hooks

| Métrica             | Threshold de alerta |
|---------------------|---------------------|
| `useState` count    | > 5 no mesmo hook   |
| `useEffect` count   | > 3 no mesmo hook   |
| `useCallback` count | > 8 no mesmo hook   |
| Linhas do hook      | > 150               |

Acima: avaliar extração ou consolidação com `useReducer`.

## Hierarquia de componentes: Smart vs Dumb

```typescript
// Dumb (presentational) — só props, sem estado próprio
function UserCard({ user, onSave }: UserCardProps) {
  return <form>...</form>;
}

// Smart (container) — gerencia estado, faz fetch, passa dados pro dumb
function UserCardContainer({ userId }: { userId: string }) {
  const { data: user } = useUserQuery(userId);
  const { mutate: save } = useSaveUserMutation();
  if (!user) return <Spinner />;
  return <UserCard user={user} onSave={save} />;
}
```

## Convenções de nomenclatura

| Tipo              | Convenção                                  |
|-------------------|--------------------------------------------|
| Hook              | `use` + PascalCase (ex: `useEditorState`)  |
| Handler de evento | `handle` + evento (ex: `handleClick`)      |
| Prop de callback  | `on` + evento (ex: `onSave`)               |
| Booleano          | prefixo `is/has/should/can` (ex: `isOpen`) |

## Referências

- Origem: 05_component_patterns.md (2026-02-27)
- Origem: 06_hooks_and_state.md (2026-02-27)
- Origem: implementations/react/hooks-srp-usereducer-uselatest.md
