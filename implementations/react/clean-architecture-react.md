---
type: implementation
title: Clean Architecture — implementação React/TypeScript/Vite
status: active
impact: high
date: 2026-02-27
tags: [clean-architecture, react, typescript, vite, hooks, context]
applies_to:
  stacks: [react, typescript]
  contexts: [architecture, project-structure, scaffolding]
parent_pattern: patterns/clean-architecture-layers.md
outcome: validated
---

## Estrutura de pastas (React/Vite)

```
src/
├── domain/
│   └── {model}/
│       ├── {model}-aggregate.ts
│       ├── {model}-persistence-port.ts
│       └── use-cases/
│           └── {action}.use-case.ts
├── infra/
│   ├── persistence/
│   │   └── in-memory-{model}-persistence-adapter.ts
│   └── canvas/          ← libs de canvas SOMENTE aqui
│       └── fabric-canvas-adapter.ts
├── components/
│   └── {feature}/
│       ├── ui/          ← componentes presentacionais
│       ├── hooks/       ← bindings, state, commands
│       └── store/       ← estado da feature
├── pages/               ← route-level components
├── shared/              ← tipos, constantes, utils puros
└── bootstrap/           ← wiring e composition root
```

## Convenções de nomenclatura

| Artefato     | Convenção                               | Extensão |
|--------------|-----------------------------------------|----------|
| Agregado     | `{model}-aggregate`                     | `.ts`    |
| Port         | `{model}-{concern}-port`                | `.ts`    |
| Adapter      | `in-memory-{model}-persistence-adapter` | `.ts`    |
| Use Case     | `{action}.use-case`                     | `.ts`    |
| Strategy     | `{element}.strategy`                    | `.ts`    |
| Componente   | PascalCase                              | `.tsx`   |
| Hook         | `use` + PascalCase                      | `.ts`    |
| Lógica/utils | kebab-case                              | `.ts`    |

## Templates

### Agregado

```typescript
// src/domain/{model}/{model}-aggregate.ts
export type {Model}State = {
  id: string;
  // campos do modelo
};

export function create{Model}(initial?: Partial<{Model}State>): {Model}State {
  return {
    id: crypto.randomUUID(),
    ...initial,
  };
}
```

### Port de persistência

```typescript
// src/domain/{model}/{model}-persistence-port.ts
import type { {Model}State } from './{model}-aggregate';

export interface {Model}PersistencePort {
  save(state: {Model}State): Promise<void>;
  load(): Promise<{Model}State | null>;
}
```

### Adapter in-memory

```typescript
// src/infra/persistence/in-memory-{model}-persistence-adapter.ts
import type { {Model}State } from '@/domain/{model}/{model}-aggregate';
import type { {Model}PersistencePort } from '@/domain/{model}/{model}-persistence-port';

export function createInMemory{Model}PersistenceAdapter(): {Model}PersistencePort {
  let stored: {Model}State | null = null;
  return {
    async save(state) { stored = structuredClone(state); },
    async load() { return stored ? structuredClone(stored) : null; },
  };
}
```

### Use Case

```typescript
// src/domain/{model}/use-cases/{action}.use-case.ts
import type { {Model}State } from '../{model}-aggregate';
import type { {Model}PersistencePort } from '../{model}-persistence-port';

export function {action}UseCase(persistence: {Model}PersistencePort) {
  return {
    async execute(): Promise<{Model}State | null> {
      return persistence.load();
    },
  };
}
```

### Bootstrap (wiring)

```typescript
// src/bootstrap/editor-bootstrap.ts
const persistence = createInMemory{Model}PersistenceAdapter();
const load{Model} = {action}UseCase(persistence);

// ✅ injetar via Context — nunca exportar como singleton
export function createEditorDependencies() {
  return { persistence, load{Model} };
}
```

### Store da feature (não-singleton)

```typescript
// ✅ Criado no bootstrap, injetado via Context
const store = createHouseStateStore(persistence);

// ❌ Singleton global — PROIBIDO
export const houseStore = new HouseStateStore();
```

## Restrições específicas desta stack

```
❌ Não importar 'fabric' fora de src/infra/canvas/
❌ Não criar src/services/ genérico na raiz
❌ Não usar src/shared/ para regras de negócio
❌ Não tratar canvas como fonte de verdade do estado
❌ strict: false no tsconfig é estado transitório — escrever código defensivo mesmo assim
```

## Checklist antes de criar qualquer artefato novo

```
□ Auditei a base de código com rg/grep?
□ O artefato pertence à camada correta?
□ O nome segue a convenção da tabela acima?
□ Não estou importando para a direção errada?
□ Criei smoke test co-localizado?
```

## Referências

- Padrão base: patterns/clean-architecture-layers.md
- Origem: 03_project_structure.md, 04_naming_conventions.md (2026-02-27)
- Templates: aggregate_ts.hbs, persistence-port_ts.hbs, persistence-adapter_ts.hbs, use-case_ts.hbs
