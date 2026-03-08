---
type: principle
title: God Object — detecção, riscos e decomposição gradual
status: active
impact: high
date: 2026-02-27
tags: [SOLID, single-responsibility, god-object, refactoring, architecture, complexity]
applies_to:
  stacks: [universal]
  contexts: [architecture, refactoring, component-design, service-design]
outcome: validated
---

## O que é um God Object

Módulo (classe, componente, hook, service) que concentra responsabilidades demais:

- Conhece e controla muitos outros módulos
- É dependência de quase tudo no sistema
- Cresce continuamente sem fronteira clara

## Sinais de alerta (detecção)

### Em componentes React

| Sinal                        | Threshold de alerta |
|------------------------------|---------------------|
| Número de hooks instanciados | > 15                |
| Linhas do componente         | > 400               |
| Estados de modal/flag        | > 8                 |
| Responsabilidades distintas  | > 3                 |

### Em classes/services (qualquer stack)

| Sinal                     | Threshold de alerta |
|---------------------------|---------------------|
| Linhas do arquivo         | > 600               |
| Métodos públicos          | > 20                |
| Dependências injetadas    | > 5                 |
| Áreas de negócio cobertas | > 2                 |

## Riscos concretos

1. **Teste difícil**: testar qualquer coisa exige mockar o God Object inteiro
2. **Efeitos colaterais ocultos**: mudança em uma área quebra comportamento em outra
3. **Paralelização impossível**: todo mundo toca o mesmo arquivo — conflito de merge constante
4. **Onboarding lento**: novo dev precisa entender tudo antes de mudar qualquer coisa

## Estratégia de decomposição (gradual, sem big bang)

### Passo 1: Mapear responsabilidades

```
// Exemplo de mapeamento real
GodComponent.tsx (569 linhas, 94 hooks) contém:
- Gerenciamento de estado de 12 modais
- Fluxo de seleção de tipo de entidade
- Ações de modo de edição
- Ações de toolbar
- Tutorial
- Debug bridge
- Hotkeys
- Canvas tools
```

### Passo 2: Identificar fronteiras naturais

Agrupe por coesão — o que muda junto, fica junto:

```
Grupo A: estado de modais → useEditorModalState
Grupo B: ações de toolbar → useToolbarActions
Grupo C: interação com canvas → useCanvasInteractions
Grupo D: hotkeys → useHotkeys (já existe?)
```

### Passo 3: Extrair um grupo por vez (never big bang)

```typescript
// ❌ Big bang: refatorar tudo de uma vez
// Resultado: PR gigante, review impossível, regressões difíceis de rastrear

// ✅ Incremental: um grupo por commit
// Commit 1: extrair useEditorModalState (12 estados de modal)
// Commit 2: extrair useToolbarActions
// Commit 3: extrair useCanvasInteractions
// Cada commit é validado individualmente
```

### Passo 4: Validar comportamento a cada extração

- Smoke tests antes e depois
- Nenhuma mudança de comportamento visível
- Métricas do componente original decrescem gradualmente

## Critério de sucesso

O God Object foi decomposto quando:

- Nenhum módulo individual tem mais de uma responsabilidade clara
- Cada módulo pode ser testado isoladamente
- Um novo dev consegue entender um módulo sem ler os outros

## Nota importante: nem todo objeto grande é um God Object

```
// Arquivo grande ≠ God Object
// God Object = múltiplas RESPONSABILIDADES, não múltiplas LINHAS

// Arquivo com 600 linhas de lógica de domínio coesa → OK
// Arquivo com 300 linhas gerenciando 5 áreas diferentes → God Object
```

## Referências

- Origem: analysis-report.md (God Component RacEditor.tsx, 2026-02-27)
- Origem: refactoring-plan.md (Fase 4 — Desacoplamento, 2026-02-26)

### Links KB

- Princípio: [[pragmatic-abstraction]]
- Princípio: [[stable-tests-before-refactoring]]
- Decisão: [[2026-02-24_incremental-refactoring-never-big-bang]]
- Decisão: [[2026-02-23_incremental-refactoring-phases]]
