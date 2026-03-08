---
type: implementation
title: Reentry Guard — implementação TypeScript com generics
status: active
impact: medium
date: 2026-02-23
tags: [typescript, recursion-guard, generics, DRY, canvas, fabric]
applies_to:
  stacks: [typescript]
  contexts: [canvas, transformation, event-handlers]
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

## Uso com Fabric.js (canvas)

```typescript
// src/infra/canvas/strategies/line.strategy.ts
import { withReentryGuard } from '@/shared/with-reentry-guard';
import type { FabricObject } from 'fabric';

// ❌ Antes: guard duplicado em cada função
function normalizeLineScaling(obj: FabricObject) {
  if ((obj as any).__normalizing) return;
  (obj as any).__normalizing = true;
  try {
    obj.set({ scaleX: 1, scaleY: 1, width: obj.width! * obj.scaleX! });
  } finally {
    (obj as any).__normalizing = false;
  }
}

// ✅ Depois: lógica específica, guard centralizado
function normalizeLineScaling(obj: FabricObject) {
  withReentryGuard(obj, '__normalizingScale', (o) => {
    o.set({ scaleX: 1, scaleY: 1, width: o.width! * o.scaleX! });
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
      // simula reentrada durante execução
      withReentryGuard(target, '__guard', vi.fn());
    });
    withReentryGuard(obj, '__guard', fn);
    // fn externa executou 1x, reentrada bloqueada
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

## Referências

- Padrão base: patterns/reentry-guard.md
- Origem: refactoring-plan.md (2026-02-23)
- Origem: analysis-report.md (duplicação detectada em 4 strategies)
