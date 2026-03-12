---
type: implementation
title: Reentry Guard — implementação TypeScript com generics
status: active
impact: medium
date: 2026-02-23
tags: [typescript, recursion-guard, generics, DRY, event-handlers]
applies_to:
  stacks: [typescript]
  contexts: [transformation, event-handlers, observers]
parent_pattern: patterns/reentry-guard.md
outcome: validated
---

## Implementação genérica

```typescript
// src/shared/with-reentry-guard.ts

type GuardableObject = Record<string, unknown>;

export function withReentryGuard<T extends GuardableObject>(
  target: T,
  guardKey: string,
  fn: (target: T) => void
): void {
  if (target[guardKey]) return;

  (target as Record<string, unknown>)[guardKey] = true;
  try {
    fn(target);
  } finally {
    (target as Record<string, unknown>)[guardKey] = false;
  }
}
```

## Exemplo de uso

```typescript
import { withReentryGuard } from '@/shared/with-reentry-guard';

// ❌ Antes: guard duplicado em cada função
function normalizeScalingA(obj: SomeObject) {
  if ((obj as any).__normalizing) return;
  (obj as any).__normalizing = true;
  try {
    applyNormalization(obj, 'typeA');
  } finally {
    (obj as any).__normalizing = false;
  }
}

// ✅ Depois: lógica específica, guard centralizado
function normalizeScalingA(obj: SomeObject) {
  withReentryGuard(obj, '__normalizingScale', (o) => {
    applyNormalization(o, 'typeA');
  });
}

function normalizeScalingB(obj: SomeObject) {
  withReentryGuard(obj, '__normalizingScale', (o) => {
    applyNormalization(o, 'typeB');
  });
}
```

## Convenção de nomes de flag

```typescript
// ✅ Descreve o que está sendo guardado
'__normalizingScale'
'__updatingLayout'
'__processingTransform'

// ❌ Genérico demais
'__lock'
'__busy'
'__running'
```

## Smoke test

```typescript
// src/shared/with-reentry-guard.smoke.test.ts
import { describe, it, expect, vi } from 'vitest';
import { withReentryGuard } from './with-reentry-guard';

describe('withReentryGuard (smoke)', () => {
  it('executes fn on first call', () => {
    const obj = {};
    const fn = vi.fn();
    withReentryGuard(obj, '__guard', fn);
    expect(fn).toHaveBeenCalledOnce();
  });

  it('blocks reentrant calls', () => {
    const obj: any = {};
    const fn = vi.fn((target) => {
      withReentryGuard(target, '__guard', vi.fn());
    });
    withReentryGuard(obj, '__guard', fn);
    expect(fn).toHaveBeenCalledOnce();
  });

  it('clears guard even when fn throws', () => {
    const obj: any = {};
    const throwing = () => { throw new Error('boom'); };
    expect(() => withReentryGuard(obj, '__guard', throwing)).toThrow();
    // guard foi limpo pelo finally
    expect(obj.__guard).toBe(false);
  });
});
```

### Links KB

- Padrão base: [[reentry-guard]]
- Relacionado: [[normalization-binding-extraction]]
