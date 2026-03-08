---
type: principle
title: Compatibilidade incremental em mudanças de formato de dados
status: active
impact: high
date: 2026-02-27
tags: [migration, data, compatibility, serialization, backward-compatibility, refactoring]
applies_to:
  stacks: [universal]
  contexts: [data-migration, serialization, refactoring, persistence]
outcome: validated
---

## Problema

Breaking changes silenciosos em formato de dados destroem dados existentes de usuários
e são difíceis de detectar porque não geram erro de compilação.

## Princípio

> Ao refatorar estruturas de dados: leitura suporta formato antigo E novo.
> Escrita usa sempre o novo formato. Nunca ao contrário.

## Estratégia de migração incremental

### Fase 1: Suporte dual na leitura

```typescript
// ❌ Breaking change silencioso
function loadHouse(data: NewFormat): House {
  return { id: data.houseId }; // campo renomeado — dados antigos quebram
}

// ✅ Compatibilidade incremental
function loadHouse(data: NewFormat | LegacyFormat): House {
  return {
    // suporta ambos os formatos durante transição
    id: 'houseId' in data ? data.houseId : data.id,
  };
}
```

### Fase 2: Escrita sempre no novo formato

```typescript
function saveHouse(house: House): NewFormat {
  return {
    houseId: house.id, // sempre salva no novo formato
  };
}
```

### Fase 3: Remoção do suporte legado (após migração completa)

```typescript
// Só remova quando tiver certeza que não existem mais dados no formato antigo
function loadHouse(data: NewFormat): House {
  return { id: data.houseId };
}
```

## Quando aplicar

- Renomeação de campos em tipos/interfaces usados em persistência
- Mudança de estrutura de JSON salvo (localStorage, arquivo, banco)
- Mudança de contrato entre camadas durante refactoring
- Serialização/deserialização de qualquer formato

## Quando NÃO é necessário

- Tipos que existem apenas em memória (não são persistidos)
- Campos novos opcionais adicionados sem remover antigos

## Lições aprendidas

- Renomeações de campo em tipos compartilhados entre editor, manager e 3D
  quebraram silenciosamente dados existentes durante mega refactoring
- A detecção veio tarde — durante testes E2E de import/export
- Smoke test de compatibilidade de formato é investimento que se paga na primeira vez

## Smoke test de compatibilidade (template)

```typescript
describe('House format compatibility', () => {
  it('loads legacy format without error', () => {
    const legacyData = { id: 'house-1' }; // formato antigo
    const house = loadHouse(legacyData);
    expect(house.id).toBe('house-1');
  });

  it('loads new format correctly', () => {
    const newData = { houseId: 'house-1' }; // formato novo
    const house = loadHouse(newData);
    expect(house.id).toBe('house-1');
  });

  it('saves always in new format', () => {
    const house = { id: 'house-1' };
    const saved = saveHouse(house);
    expect(saved).toHaveProperty('houseId'); // novo formato
    expect(saved).not.toHaveProperty('id'); // não mais legado
  });
});
```

## Referências

- Origem: 01_core_principles.md (Principle 7: Incremental Compatibility)

### Links KB

- Relacionado: [[precise-automation-triggers]]
- Relacionado: [[incremental-refactoring-never-big-bang]]
