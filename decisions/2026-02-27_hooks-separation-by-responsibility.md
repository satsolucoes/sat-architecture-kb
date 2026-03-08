---
type: decision
title: Separação de hooks por responsabilidade — sem hooks combinados genéricos
status: active
impact: medium
date: 2026-02-27
tags: [react, hooks, single-responsibility, SOLID, coupling]
applies_to:
  stacks: [react, react-native]
  contexts: [hooks, component-design, architecture]
outcome: validated
---

## Contexto

Hook combinado `useLineArrowEditorActions` era responsável por aplicar mudanças de Linha e Seta.

## Decisão

Separar em hooks dedicados por tipo:

- `useLineEditorActions` — responsabilidade única: Linha
- `useArrowEditorActions` — responsabilidade única: Seta

## Motivação

- Menor acoplamento: mudança na lógica de Seta não afeta Linha
- Melhor manutenção: cada hook tem seu contexto claro
- Testabilidade: testar comportamentos isolados

## Padrão resultante

```typescript
// ❌ Antes: hook combinado com apply compartilhado
function useLineArrowEditorActions() {
  function applyChanges(type: 'line' | 'arrow', ...) {
    if (type === 'line') { ... }
    else { ... }
  }
}

// ✅ Depois: hooks dedicados
function useLineEditorActions() {
  function applyChanges(...) { /* só Linha */ }
}

function useArrowEditorActions() {
  function applyChanges(...) { /* só Seta */ }
}

// Orquestração no componente pai (RacEditor)
const lineActions = useLineEditorActions();
const arrowActions = useArrowEditorActions();
```

## Quando separar hooks

- Comportamentos distintos que crescem independentemente → Separe
- Mesmo contexto de uso mas lógicas divergentes → Separe
- Dois tipos que eventualmente precisarão de regras diferentes → Separe preventivamente

## Quando manter combinado

- Comportamento genuinamente compartilhado sem divergência futura provável
- Menos de 20 linhas no total — separar seria overhead sem ganho

## Lições aprendidas

- Centralizar lógica de escala (scaling) em helpers do factory e reutilizar nos hooks eliminou
  duplicação entre criação e edição sem precisar de hook combinado
- O RacEditor orquestra os dois hooks sem mudança no contrato de tela

## Referências

- Origem: changelog-20260227 (refatoração incremental + correções Linha/Seta)
