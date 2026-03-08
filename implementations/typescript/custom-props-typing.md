---
type: refactoring
title: Tipagem forte para propriedades custom em objetos de terceiros
status: active
impact: medium
date: 2026-02-27
tags: [typescript, typing, type-safety, fabric, third-party, custom-props]
applies_to:
  stacks: [typescript, javascript]
  contexts: [typing, third-party-integration, serialization]
outcome: validated
---

## Problema

Ao estender objetos de bibliotecas de terceiros com propriedades customizadas, é comum:

- Casts diretos (`as any`, `as CustomType`) espalhados pelo código
- Lista de serialização desalinhada com o tipo real
- Propriedades herdadas da lib misturadas com propriedades do domínio

## Padrão: Exclude para isolar propriedades custom

```typescript
// Define tipo que exclui propriedades da lib base
type CanvasObjectProps = Exclude<keyof CanvasObject, keyof FabricObject>;

// Lista de runtime tipada — não pode conter propriedades do Fabric
const CANVAS_OBJECT_PROPS: CanvasObjectProps[] = [
  'myType',
  'houseViewType',
  'pilotiId',
  'isContraventamento',
  // ...somente props do domínio
];
```

## Helpers para casts seguros (centralizados)

```typescript
// Em vez de cast direto espalhado pelo código:
// ❌ (group as any).pilotiId
// ❌ (object as CanvasGroup).myType

// ✅ Helpers centralizados com tipagem correta
function toCanvasGroup(group: Group): CanvasGroup {
  return group as unknown as CanvasGroup;
}

function toCanvasObject(
  object: FabricObject | null | undefined
): CanvasObject | null {
  if (!object) return null;
  return object as unknown as CanvasObject;
}
```

## Regras

1. Lista de serialização contém **apenas** propriedades do domínio (não da lib)
2. A ordem do array deve seguir a ordem de declaração no tipo
3. Casts devem ser **centralizados em helpers**, nunca espalhados inline
4. `CanvasObjectProps` é compile-time — o array runtime ainda é necessário

## Aplicabilidade

- Qualquer integração com libs que você estende com dados de domínio
- **Kotlin/Java**: equivalente com data classes que wrappam tipos externos
- Padrão genérico para adapters de terceiros

## Lições aprendidas

- Incluir propriedades do Fabric na lista de serialização causava comportamento inesperado na
  deserialização (Fabric sobrescrevia seus próprios valores)
- Ordenar o array igual à declaração do tipo ajudou em code review e auditorias

## Referências

- Origem: changelog-20260227 (CanvasObject / customProps)
