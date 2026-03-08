---
type: decision
title: Documentação arquitetural como artefato de primeira classe
status: active
impact: high
date: 2026-03-04
tags: [documentation, guidelines, architecture, team, knowledge-management]
applies_to:
  stacks: [universal]
  contexts: [team, process, architecture, governance]
outcome: validated
---

## Contexto

Regras arquiteturais, padrões e decisões viviam implicitamente no código ou na cabeça do time.
Novos contextos (ou agentes de IA) começavam do zero sem consciência do que já foi decidido.

## Decisão

Criar e manter artefatos de documentação explícitos:

- `.guidelines/` — padrões arquiteturais, fluxos obrigatórios, anti-padrões
- `.rules/` — regras funcionais por módulo (em linguagem não-técnica)
- `.changelogs/` — histórico de decisões e mudanças

## Estrutura adotada

```
.guidelines/
├── architecture-patterns.md    # reuse-first, one source of truth, matriz de decisão
├── architectural-standards.md  # preferências de engenharia sênior
├── governance.md               # protocolo de alinhamento antes de implementar
└── testing-validation-workflow.md  # nomenclatura de testes, checklist de validação

.rules/
├── README.md       # visão geral + tolerância numérica
├── canvas.md       # regras do canvas
├── toolbar.md      # regras da toolbar
└── [por módulo]    # cada módulo tem seu arquivo de regras
```

## Regras do processo

1. **Leitura obrigatória** de `.guidelines/` e `.rules/` no início de cada sessão de trabalho
2. **Atualização obrigatória** ao implementar mudança arquitetural relevante
3. Linguagem das `.rules/` voltada para **não-técnicos** (comportamento esperado, não implementação)
4. Linguagem das `.guidelines/` voltada para **engenheiros** (padrões, decisões, anti-padrões)

## Fluxo obrigatório antes de codar

```
1. Inventário: o que já existe que resolve isso?
2. Decisão: reutilizar vs extrair comum vs criar novo
3. Implementação: com rastreabilidade no changelog
```

## Lições aprendidas

- Guidelines em linguagem de usuário (não-técnica) nas `.rules/` reduziu divergência de interpretação
- Protocolo de alinhamento prévio (operacional vs estratégico) evitou retrabalho em refactorings
- `AGENTS.md` com instrução de leitura obrigatória garantiu que agentes de IA não partissem do zero

## Aplicabilidade

- Qualquer projeto com mais de um colaborador (humano ou IA)
- Especialmente valioso em projetos de longa duração com decisões acumuladas

## Referências

- Origem: changelog-20260227 (.guidelines / padrões arquiteturais)
- Origem: changelog-20260304 (documentação, guideline de testes)

### Links KB

- Princípio: [[mandatory-decision-flow]]
- Relacionado: [[2026-02-27_code-templates-by-artifact]]
