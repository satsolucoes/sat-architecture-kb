---
type: refactoring
title: Automações com gatilhos precisos — evitar disparo global genérico
status: active
impact: high
date: 2026-03-05
tags: [automation, state-management, side-effects, precision, reliability]
applies_to:
  stacks: [universal]
  contexts: [automation, state-management, events, triggers]
outcome: validated
---

## Problema

Automações disparadas via `notify` global ou em ações genéricas executam em contextos errados,
causando comportamento inesperado, testes frágeis e difícil rastreabilidade.

## Regra

> Toda automação deve ter um **gatilho preciso e explícito**, disparando apenas quando a
> condição real de negócio foi satisfeita.

## Exemplo de aplicação

### ❌ Antes: disparo global genérico

```typescript
// HouseManager — qualquer notify disparava auto-contraventamento
notify(event: HouseEvent) {
  autoContraventamento.run(this.house); // dispara em QUALQUER mudança
}
```

### ✅ Depois: gatilho preciso por condição real

```typescript
updatePiloti(pilotiId: string, patch: PilotiPatch): void {
  // Só dispara auto-contraventamento se nivel realmente mudou
  const nivelChanged = patch.nivel !== undefined
    && patch.nivel !== currentPiloti.nivel;

  this.applyPilotiPatch(pilotiId, patch);

  if (nivelChanged && this.hasTopView()) {
    autoContraventamento.run(this.house);
  }
}

registerView(view: HouseView): void {
  this.views.push(view);

  // Dispara ao inserir vista superior (cenário de inserção inicial)
  if (view.type === 'top') {
    autoContraventamento.run(this.house);
  }
}
```

## Categorias de automação e seus gatilhos

| Automação                          | Gatilho correto                                       |
|------------------------------------|-------------------------------------------------------|
| Auto-contraventamento              | Mudança real de `nivel` OU inserção de vista superior |
| Recálculo de níveis intermediários | Mudança de nível em piloti de canto                   |
| Rerender de elevações              | Após criação/remoção de contraventamento              |
| Aplicação de settings              | Confirmação explícita do modal de settings            |

## Reconciliação vs disparo único

```typescript
// ✅ Reconciliar por estado atual (idempotente e seguro)
function reconcileAutoContraventamento(house: House): void {
  for (const column of house.columns) {
    const needsContraventamento = isPilotiOutOfProportion(column);
    const hasContraventamento = column.contraventamentos.some(c => c.isAuto);

    if (needsContraventamento && !hasContraventamento) {
      addAutoContraventamento(column);
    } else if (!needsContraventamento && hasContraventamento) {
      removeAutoContraventamento(column); // remove apenas o automático
    }
    // Contraventamento manual: preservado em qualquer caso
  }
}
```

## Lições aprendidas

- `notify` global causava auto-contraventamento em ações que não mudavam nível
- Flag `__autoContraventamentoInitialized` foi removida — reconciliação idempotente elimina
  a necessidade de controle de execução única
- Preservar contraventamento manual durante reconciliação automática é regra de negócio crítica

## Referências

- Origem: changelog-20260305 (Auto Contraventamento no commit de nível)
