---
type: principle
title: Separação explícita de regras de domínio com nomes distintos
status: active
impact: high
date: 2026-03-03
tags: [SOLID, clean-code, domain, naming, single-responsibility]
applies_to:
  stacks: [universal]
  contexts: [domain, business-rules, naming]
outcome: validated
---

## Problema

Usar uma mesma função para múltiplas regras de negócio similares (mas distintas) causa regressões
silenciosas: ao ajustar a regra A, você quebra a regra B sem perceber.

## Princípio

Quando duas regras de domínio têm propósitos distintos — mesmo que a fórmula seja parecida —
separe em funções com **nomes que expressam o propósito**, não a implementação.

## Exemplo de aplicação

### ❌ Problema: função única para duas regras distintas

```typescript
// Uma função usada tanto para recomendação quanto para elegibilidade
function getMinimumPilotiHeight(nivel: number) {
  return nivel * 2; // mudança aqui quebrou o auto-contraventamento
}
```

### ✅ Solução: funções separadas por propósito

```typescript
// Regra 1: altura mínima para recomendação e consistência de nível
function getMinimumPilotiHeightForNivel(nivel: number): number {
  return nivel * 3;
}

// Regra 2: elegibilidade para contraventamento automático
function isPilotiOutOfProportion(height: number, nivel: number): boolean {
  return height < nivel * 3;
}
```

## Critérios para separar

- Os contextos de uso são diferentes? → Separe
- Uma mudança numa regra não deveria afetar a outra? → Separe
- Os nomes revelam o propósito de cada uma? → Valide antes de commitar

## Lição aprendida

Unificar `getMinimumPilotiHeightForNivel` para `*2` quebrou silenciosamente o auto-contraventamento
que esperava `*3`. A separação explícita eliminou a regressão e tornou cada regra auditável
independentemente.

## Aplicabilidade em outras stacks

- **Kotlin/Java**: mesmo princípio — funções/métodos com nomes que revelam o contexto de uso
- **Angular**: services com métodos distintos por regra de negócio
- Padrão universal de Clean Code: *"Don't use a comment when you can use a function or variable"*

## Referências

- Origem: changelog-20260303 (correção de regressão no auto contraventamento)

### Links KB

- Relacionado: [[pragmatic-abstraction]]
- Relacionado: [[one-source-of-truth]]
- Padrão: [[clean-architecture-layers]]
