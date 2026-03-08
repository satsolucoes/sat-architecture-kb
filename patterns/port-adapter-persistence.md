---
type: pattern
title: Port/Adapter para persistência (Persistence Port Pattern)
status: active
impact: high
date: 2026-02-27
tags: [clean-architecture, port-adapter, persistence, hexagonal, dependency-inversion]
applies_to:
  stacks: [universal]
  contexts: [architecture, persistence, domain]
outcome: validated
---

## Problema

Acoplamento direto do domínio ao mecanismo de persistência (memória, localStorage, API, banco)
torna migrações futuras custosas e testa domínio junto com infraestrutura.

## Padrão

Definir uma **interface de porta** no domínio e implementar **adaptadores** na infraestrutura.
O domínio nunca conhece o mecanismo concreto.

## Estrutura

```
src/
├── domain/
│   └── house-persistence.port.ts     # Interface (contrato)
└── infra/
    └── persistence/
        ├── in-memory-house-persistence.adapter.ts   # Implementação atual
        └── in-memory-house-persistence.adapter.smoke.test.ts
```

## Implementação

```typescript
// domain/house-persistence.port.ts
export interface HousePersistencePort {
  save(house: House): void;
  load(id: string): House | null;
  listAll(): House[];
}

// infra/persistence/in-memory-house-persistence.adapter.ts
export class InMemoryHousePersistenceAdapter implements HousePersistencePort {
  private store = new Map<string, House>();

  save(house: House): void {
    this.store.set(house.id, house);
  }

  load(id: string): House | null {
    return this.store.get(id) ?? null;
  }

  listAll(): House[] {
    return [...this.store.values()];
  }
}
```

## Quando aplicar

- Quando a persistência tem chance real de mudar (memória → API, localStorage → banco)
- Quando você quer testar o domínio sem dependência de infraestrutura
- **Não aplicar** quando a persistência é definitiva e simples — overkill sem ganho

## Evolução natural

```
InMemoryAdapter → LocalStorageAdapter → APIAdapter
```

Troca de adaptador sem tocar no domínio.

## Aplicabilidade em outras stacks

- **Kotlin/Java**: Repository pattern com interface no domínio
- **Angular**: Service abstrato com implementação injetável
- Padrão universal — funciona em qualquer linguagem OO

## Lições aprendidas

- `HouseAggregate` foi removido, mas `HousePersistencePort` foi mantido: um tinha ganho real, o outro não
- O adaptador em memória funcionou como stepping stone antes de persistência real

## Referências

- Origem: changelog-20260227 (Discussão arquitetural e decisões finais)
