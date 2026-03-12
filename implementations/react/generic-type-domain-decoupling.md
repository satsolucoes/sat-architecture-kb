# Generic Type Parameter — desacoplando domínio de infraestrutura de renderização

## Contexto

Um domínio que precisa rastrear objetos visuais (nós de canvas, grupos de cena 3D, elementos de SVG) acaba importando diretamente o tipo da lib de renderização. Isso contamina o domínio com infra: testes precisam instanciar a lib, e trocar a lib de renderização quebraria o domínio.

## Solução

Parametrizar o tipo do objeto visual com um generic `TRenderable`. O domínio armazena, compara e passa `TRenderable` para callbacks — mas nunca inspeciona suas propriedades. A infra instancia com o tipo real. Os testes instanciam com `{ id: string }`.

## Padrão

```typescript
// shared/types/<entidade>.ts — tipo parametrizado
export interface ItemInstance<TRenderable> {
  instanceId: string;
  renderObject: TRenderable;
  metadata?: Record<string, unknown>;
}

export interface EntityState<TRenderable> {
  id: string;
  items: Record<string, ItemInstance<TRenderable>[]>;
}

// domain/<entidade>.aggregate.ts — domínio puro
export class EntityAggregate<TRenderable> {
  registerItem(params: { renderObject: TRenderable; ... }): void {
    // armazena TRenderable sem saber o que é
  }

  cleanupStale(isAlive: (obj: TRenderable) => boolean): number {
    // callback injeta o conhecimento de "vivo" — o domínio não precisa saber
  }
}
```

## Uso em produção

```typescript
// infra — sabe que TRenderable = FabricGroup (ou Three.Mesh, ou SVGElement)
const aggregate = EntityAggregate.fromState<FabricGroup>(loadedState);

aggregate.cleanupStale((obj) => canvas.getObjects().includes(obj));
```

## Uso em testes

```typescript
// test — TRenderable = objeto simples, sem lib
const aggregate = EntityAggregate.fromState<{ id: string }>(createTestState());

aggregate.registerItem({ renderObject: { id: 'obj-1' }, instanceId: 'i1' });
expect(aggregate.toState().items).toHaveLength(1);
// Sem canvas, sem FabricCanvas, sem Three.js, sem mocks pesados
```

## Regra de isolamento

O domínio pode **armazenar**, **comparar por referência** e **passar para callbacks** o `TRenderable` — nunca pode **inspecionar suas propriedades**. Se precisar inspecionar, extrai uma interface mínima:

```typescript
// ✅ Interface mínima no domínio — aceitável
interface HasId { id: string; }
export class EntityAggregate<TRenderable extends HasId> { ... }

// ❌ Violação — domínio acessa propriedade específica da lib
cleanupStale((obj) => obj.canvas !== null)  // obj.canvas é Fabric
```

## Quando aplicar

- Domínio precisa rastrear ou referenciar objetos de uma lib de renderização/UI
- O aggregate precisa ser testável sem instanciar a lib visual
- Há possibilidade de trocar ou abstrair a lib de renderização

## Exemplo de origem

Padrão aplicado na migração v2→v3 do `rac-designer-teto`: `HouseAggregate<TGroup>` permite testar toda a lógica de domínio sem instanciar Fabric.js. Em produção: `HouseAggregate<FabricGroup>`. Nos testes: `HouseAggregate<{ id: string }>`.

## Referências

- [[domain-aggregate-root]]
- [[clean-architecture-layers]]
- [[stable-tests-before-refactoring]]

### Links KB
- [[port-adapter-persistence]] — mesmo princípio aplicado à persistência
- [[custom-props-typing]] — tipagem de propriedades em objetos de terceiros
