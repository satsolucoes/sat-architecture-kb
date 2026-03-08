---
type: decision
title: Refactoring incremental — nunca big bang
status: active
impact: high
date: 2026-02-24
tags: [refactoring, process, risk-management, incremental, architecture]
applies_to:
  stacks: [universal]
  contexts: [refactoring, process, architecture, team]
outcome: validated
---

## Contexto

Necessidade de refatorar módulos grandes e complexos sem introduzir regressões ou bloquear
o desenvolvimento de novas features.

## Decisão

Toda refatoração estrutural deve ser incremental — um grupo de responsabilidades por vez,
com validação completa entre cada passo.

## Por que não big bang

- PR gigante impossível de revisar com confiança
- Regressões difíceis de isolar (qual commit quebrou?)
- Bloqueia outros desenvolvedores durante a refatoração
- Risco alto de rollback parcial (impossível sem bagunça)

## Estratégia incremental

### 1. Mapeamento antes de qualquer código

```
Antes de tocar qualquer arquivo:
- Liste todas as responsabilidades do módulo
- Agrupe por coesão (o que muda junto)
- Defina a ordem de extração (menor risco primeiro)
- Estime impacto de cada grupo
```

### 2. Um grupo por commit

```
Commit 1: extrair grupo A (testes verdes)
Commit 2: extrair grupo B (testes verdes)
Commit 3: extrair grupo C (testes verdes)
...
```

### 3. Commits atômicos e descritivos

```bash
# ✅ Bom
git commit -m "refactor(hooks): extrair useEditorModalState de RacEditor"
git commit -m "refactor(hooks): extrair useToolbarActions de RacEditor"

# ❌ Ruim
git commit -m "refactor: mega refactoring geral"
```

### 4. Validação entre cada commit

```bash
npm run test -- --run   # 100% passando
npm run build           # sem erros
npx tsc --noEmit        # sem erros de tipo
```

## Quando é aceitável um commit maior

- Renomeação de arquivos + atualização de imports (necessariamente junto)
- Movimentação de módulo inteiro para novo namespace
- Mesmo assim: isolar num único commit sem misturar com mudanças lógicas

## Tamanho de PR saudável para refactoring

| Tipo                       | Arquivos | Linhas              |
|----------------------------|----------|---------------------|
| Extração de hook/service   | < 10     | < 200               |
| Movimentação de módulo     | < 30     | < 500               |
| Reorganização de namespace | < 50     | qualquer (só moves) |

PRs maiores que isso: considerar dividir em múltiplos PRs sequenciais.

## Lições aprendidas

- Commit de 149 arquivos alterados numa única rodada gerou 12 testes falhando e risco alto
- Correções de regressão tomaram mais tempo que a própria refactoring
- Fases pequenas e validadas individualmente permitem reverter com precisão cirúrgica
- Separar "correção de bug" de "refactoring" em commits diferentes é fundamental para
  rastreabilidade de causa raiz

## Referências

- Origem: regression-checklist.md (mega refactoring 2026-02-24)
- Origem: regression-run.md (correções pós-refactoring 2026-02-25)
- Origem: refactoring-plan.md (plano em fases 2026-02-27)

### Links KB

- Princípio: [[stable-tests-before-refactoring]]
- Relacionado: [[2026-02-23_incremental-refactoring-phases]]
- Relacionado: [[2026-02-24_regression-checklist-as-living-artifact]]
