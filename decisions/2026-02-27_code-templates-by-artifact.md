---
type: decision
title: Templates de código — referência rápida por tipo de artefato
status: active
impact: medium
date: 2026-02-27
tags: [templates, scaffolding, clean-architecture, react, typescript, conventions]
applies_to:
  stacks: [react, typescript]
  contexts: [scaffolding, architecture, conventions, onboarding]
outcome: validated
---

## Propósito

Templates padronizados eliminam fricção na criação de novos artefatos e
garantem consistência estrutural desde o primeiro commit.

---

## Agregado de domínio

```typescript
// src/domain/{model}/{model}-aggregate.ts
export type {Model}State = {
  id: string;
  // ...campos do modelo
};

export function create{Model}(initial?: Partial<{Model}State>): {Model}State {
  return {
    id: crypto.randomUUID(),
    ...initial,
  };
}
```

---

## Port de persistência

```typescript
// src/domain/{model}/{model}-persistence-port.ts
import type { {Model}State } from './{model}-aggregate';

export interface {Model}PersistencePort {
  save(state: {Model}State): Promise<void>;
  load(): Promise<{Model}State | null>;
}
```

---

## Adapter de persistência (in-memory)

```typescript
// src/infra/persistence/in-memory-{model}-persistence-adapter.ts
import type { {Model}State } from '@/domain/{model}/{model}-aggregate';
import type { {Model}PersistencePort } from '@/domain/{model}/{model}-persistence-port';

export function createInMemory{Model}PersistenceAdapter(): {Model}PersistencePort {
  let stored: {Model}State | null = null;

  return {
    async save(state) {
      stored = structuredClone(state);
    },
    async load() {
      return stored ? structuredClone(stored) : null;
    },
  };
}
```

---

## Use Case

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

---

## Hook customizado

```typescript
// src/components/{feature}/hooks/use{Name}.ts
import { useState } from 'react';

export function use{Name}() {
  const [state, setState] = useState(null);

  // lógica do hook

  return { state }; // sempre objeto, nunca array
}
```

---

## Componente React

```tsx
// src/components/{feature}/ui/{Name}.tsx
type {Name}Props = {
  // props explícitas com tipos
};

export function {Name}({}: {Name}Props) {
  return (
    <div>
      {/* conteúdo */}
    </div>
  );
}
```

---

## Página (route-level)

```tsx
// src/pages/{name}.tsx
export default function {Name}Page() {
  return (
    <main>
      <h1>{Name}</h1>
      {/* composição de views e containers */}
    </main>
  );
}
```

---

## Smoke Test co-localizado

```typescript
// {source-file}.smoke.test.ts
import { describe, it, expect } from 'vitest';

describe('{SourceFile} (smoke)', () => {
  it('should exist and behave correctly for base case', () => {
    // substituir por assertion real e significativa
    expect(true).toBe(true);
  });
});
```

---

## Teste de componente React

```tsx
// {Component}.test.tsx
import { render, screen } from '@testing-library/react';
import { {Component} } from './{Component}';

describe('{Component}', () => {
  it('should render correctly', () => {
    render(<{Component} />);
    expect(screen.getByRole('...')).toBeInTheDocument();
  });
});
```

---

## Convenções de nomenclatura resumidas

| Artefato            | Formato              | Extensão |
|---------------------|----------------------|----------|
| Componente React    | PascalCase           | `.tsx`   |
| Hook customizado    | `use` + PascalCase   | `.ts`    |
| Lógica/utils/tipos  | kebab-case           | `.ts`    |
| Constante global    | UPPER_SNAKE_CASE     | —        |
| Tipo/Interface      | PascalCase           | —        |
| Props de componente | PascalCase + `Props` | —        |

## Referências

- Origem: templates HBS (aggregate, component, hook, page, persistence, use-case, tests)
- Origem: 04_naming_conventions.md (2026-02-27)
