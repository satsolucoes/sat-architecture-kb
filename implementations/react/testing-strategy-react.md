---
type: implementation
title: Estratégia de testes — React/TypeScript com Vitest, RTL e Playwright
status: active
impact: high
date: 2026-02-27
tags: [testing, react, vitest, testing-library, playwright, typescript]
applies_to:
  stacks: [react, typescript]
  contexts: [testing, quality, CI-CD]
parent_pattern: decisions/2026-02-27_testing-strategy-adapted-pyramid.md
outcome: validated
---

## Stack de testes

| Camada     | Ferramenta                     | Uso                             |
|------------|--------------------------------|---------------------------------|
| Unit/Smoke | Vitest                         | Lógica pura de domínio, utils   |
| Integração | React Testing Library + Vitest | Componentes com interação       |
| E2E        | Playwright                     | Fluxos críticos em browser real |

## Seletores — ordem de prioridade (RTL)

```typescript
// 1. getByRole — simula navegação por tecnologia assistiva ← preferido
screen.getByRole('button', { name: 'Salvar' })
screen.getByRole('heading', { level: 1 })

// 2. getByLabelText — campos de formulário
screen.getByLabelText('Email')

// 3. getByPlaceholderText — inputs sem label visível
screen.getByPlaceholderText('Digite seu email')

// 4. getByText — elementos não-interativos
screen.getByText('Bem-vindo')

// 5. getByDisplayValue — input pelo valor atual
screen.getByDisplayValue('São Paulo')

// ❌ NUNCA como primeira opção
screen.getByTestId('submit-button') // escape hatch apenas
```

## Templates

### Smoke test co-localizado

```typescript
// src/domain/house/house-aggregate.smoke.test.ts
import { describe, it, expect } from 'vitest';
import { createHouse } from './house-aggregate';

describe('house-aggregate (smoke)', () => {
  it('creates house with default values', () => {
    const house = createHouse();
    expect(house.id).toBeDefined();
  });

  it('merges initial values', () => {
    const house = createHouse({ id: 'test-1' });
    expect(house.id).toBe('test-1');
  });
});
```

### Teste de componente React (integração)

```tsx
// src/components/house/ui/HouseCard.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { HouseCard } from './HouseCard';

describe('HouseCard', () => {
  it('renders house name', () => {
    render(<HouseCard name="Casa 1" onSelect={vi.fn()} />);
    expect(screen.getByRole('heading', { name: 'Casa 1' })).toBeInTheDocument();
  });

  it('calls onSelect when clicked', async () => {
    const onSelect = vi.fn();
    render(<HouseCard name="Casa 1" onSelect={onSelect} />);
    await userEvent.click(screen.getByRole('button', { name: 'Selecionar' }));
    expect(onSelect).toHaveBeenCalledOnce();
  });
});
```

### E2E com Playwright — Page Object

```typescript
// e2e/helpers/editor-page.ts
export async function createHouseOfType(page: Page, houseType: string) {
  await page.getByRole('button', { name: 'Nova casa' }).click();
  await page.getByRole('option', { name: houseType }).click();
  await page.getByRole('button', { name: 'Confirmar' }).click();
}

// e2e/house-creation.spec.ts
test('should create house tipo6', async ({ page }) => {
  await page.goto('/editor');
  await createHouseOfType(page, 'tipo6');
  await expect(page.getByRole('heading', { name: 'tipo6' })).toBeVisible();
});
```

## Comandos

```bash
# Unit + Integração
npx vitest run

# E2E paralelo (desenvolvimento)
npx playwright test

# E2E serial (validação de regressão — mais confiável)
npx playwright test --workers=1
```

## Checklist de pipeline

```
□ smoke tests passando
□ vitest run sem falhas
□ playwright --workers=1 nos fluxos críticos
□ tsc sem erros
□ eslint sem erros
```

## Anti-padrões específicos desta stack

```typescript
// ❌ Testa estado interno do componente
expect(component.state.isLoading).toBe(true);

// ❌ Acessa DOM por id/CSS direto
document.getElementById('my-button');

// ❌ getByTestId como primeira opção
screen.getByTestId('save-btn');

// ❌ Assertion fake
it('should work', () => { expect(true).toBe(true); });
```

## Referências

- Decisão base: decisions/2026-02-27_testing-strategy-adapted-pyramid.md
- Origem: 08_testing.md (2026-02-27)
- Relacionado: decisions/2026-03-04_smoke-test-naming-convention.md
