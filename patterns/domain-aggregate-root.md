# Domain Aggregate — estado de domínio centralizado em classe raiz

## Contexto

Lógica de domínio espalhada entre componentes, hooks e funções utilitárias torna difícil saber onde está a source of truth e quais invariantes valem. O comportamento fica implícito no fluxo da UI, não no domínio — e invariantes de negócio acabam vivendo em handlers ou `useEffect`.

## Decisão

Extrair toda a lógica de domínio para uma classe `Aggregate` com construção controlada via factory method e acesso explícito ao estado via `toState()`. Use-cases são funções puras chamadas pelo aggregate — ele orquestra, elas calculam.

## Estrutura

```
src/domain/<entidade>/
├── <entidade>.aggregate.ts              ← raiz: orquestra tudo
├── <entidade>-persistence.port.ts       ← contrato de persistência
└── use-cases/
    ├── <entidade>-state.use-case.ts     ← criação de estado inicial
    ├── <entidade>-views.use-case.ts     ← operações sobre coleções
    ├── <entidade>-layout.use-case.ts    ← regras de layout/relacionamento
    └── <entidade>-rebuild.use-case.ts   ← reconstrução a partir de fontes externas
```

## Padrão

```typescript
export class EntityAggregate<TRenderable> {
  // Construção sempre via factory — nunca new direto
  private constructor(private readonly state: EntityState<TRenderable>) {}

  static fromState<T>(state: EntityState<T>): EntityAggregate<T> {
    return new EntityAggregate(state);
  }

  static createInitial<T>(params: { id: string; defaults: EntityDefaults }): EntityState<T> {
    return { id: params.id, items: createDefaultItems(params.defaults), ... };
  }

  // Leitura do estado
  toState(): EntityState<TRenderable> { return this.state; }

  // Operação com invariante de domínio
  applyPatch(itemId: string, patch: Partial<Item>): { sideEffects: string[] } {
    // Invariante: apenas um item pode ser "master" por vez
    if (patch.isMaster === true) {
      // limpa masters anteriores, registra quais foram limpos
    }
    return { sideEffects: clearedIds };
  }
}
```

## Use-cases como funções puras

```typescript
// <entidade>-state.use-case.ts — sem dependência de React, canvas ou UI
export function createDefaultItems(params: {
  itemIds: string[];
  defaultItem: Item;
}): Record<string, Item> {
  const items: Record<string, Item> = {};
  params.itemIds.forEach((id) => {
    items[id] = { ...params.defaultItem };  // clone, não referência
  });
  return items;
}
```

## Por que funciona

- **Invariantes no aggregate, não no hook**: a regra "só um master" está em `applyPatch()`, garantida em qualquer caller.
- **Use-cases testáveis isoladamente**: Vitest puro, sem `renderHook`, sem mocks pesados.
- **Generic `TRenderable`**: o domínio não sabe o que é um `FabricGroup` ou um nó Three.js — aceita `{ id: string }` nos testes e o tipo real em produção.
- **Persistência via Port**: `EntityPersistencePort<T>` desacopla totalmente a implementação de storage.

## Quando aplicar

- Mais de 3 hooks precisam do mesmo "estado de domínio"
- Existem invariantes de negócio que hoje vivem em hooks ou handlers
- O estado precisa ser testado sem renderizar React ou instanciar libs de UI

## Quando não aplicar

- Estado puramente de UI (modal aberto, aba selecionada) — fica nos hooks
- Entidades simples sem invariantes — um tipo TypeScript resolve

## Cuidados

- O aggregate **não importa nada de React, Fabric, Three.js ou qualquer lib de UI**
- `toState()` expõe referência interna — callers não devem mutá-la diretamente
- Operações que retornam resultados ricos (`{ sideEffects }`) são intencionais — o caller precisa saber o que aconteceu

## Exemplo de origem

Padrão extraído da migração v2→v3 do `rac-designer-teto`, onde `HouseAggregate<FabricGroup>` centralizou lógica de pilotis e vistas que estava espalhada em `RACEditor.tsx` (2125 linhas) e `Canvas.tsx` (1634 linhas).

## Referências

- [[clean-architecture-layers]]
- [[port-adapter-persistence]]
- [[stable-tests-before-refactoring]]

### Links KB
- [[explicit-domain-rules-naming]] — nomenclatura dos use-cases
- [[god-object-detection-decomposition]] — o que esse padrão resolve
- [[pragmatic-abstraction]] — critério para criar ou não o aggregate
