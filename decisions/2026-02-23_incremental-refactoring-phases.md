---
type: decision
title: Refactoring incremental em fases com critérios de aceite explícitos
status: active
impact: high
date: 2026-02-23
tags: [refactoring, process, incremental, safety, planning]
applies_to:
  stacks: [universal]
  contexts: [process, refactoring, planning, governance]
outcome: validated
---

## Decisão

Toda refatoração significativa deve ser dividida em fases com:

- Escopo bem delimitado por fase
- Critério de aceite explícito (o que precisa estar verde antes de avançar)
- Commit semântico por fase (rastreabilidade)
- Validação antes de iniciar a próxima fase

## Estrutura de fase

```markdown
### Fase N: [Nome descritivo]
**Objetivo:** [Uma frase clara]
**Tarefas:**
  - [ ] Tarefa específica e atômica
**Commit sugerido:** `tipo(escopo): descrição`
**Critério de aceite:** [o que precisa estar verde]
**Entregável:** [benefício concreto e mensurável]
```

## Exemplo de plano bem estruturado

```markdown
### Fase 1: Estabilização (Urgente)
**Objetivo:** Suíte de testes 100% verde antes de qualquer refatoração
**Tarefas:**
  - [ ] Adicionar dependência faltante
  - [ ] Criar mock de ambiente de teste
  - [ ] Corrigir funções não exportadas após move
**Commit:** `fix(test): stabilize test suite`
**Critério de aceite:** npm test → 100% passando
**Entregável:** base segura para próximas fases

### Fase 2: Quick Wins (Alta prioridade, baixo esforço)
**Objetivo:** Remover ruído do código sem risco
**Tarefas:**
  - [ ] Corrigir typos em nomes de arquivo
  - [ ] Remover console.logs de produção
  - [ ] Centralizar magic numbers em constants
**Commit:** `chore: apply quick wins`
**Critério de aceite:** lint limpo, testes verdes
**Entregável:** código mais limpo, menos ruído

### Fase 3: Padrões e Abstrações (Médio prazo)
**Objetivo:** Eliminar duplicação estrutural
**Tarefas:**
  - [ ] Extrair scaling guard genérico
  - [ ] Aplicar nas 4 strategies afetadas
**Commit:** `refactor(factory): abstract scaling guard`
**Critério de aceite:** -30 linhas, testes verdes
**Entregável:** um ponto de manutenção para o padrão

### Fase 4: Arquitetura (Longo prazo)
**Objetivo:** Atacar complexity estrutural
**Tarefas:**
  - [ ] Decompor God Component gradualmente
  - [ ] Migrar viewport para useReducer
**Commit:** `refactor(editor): extract editor state hooks`
**Critério de aceite:** componente < 400 linhas, testes verdes
**Entregável:** componente orquestrador, não acumulador
```

## Princípios do processo

1. **Fase 1 é sempre estabilização** — nunca começa refatoração com testes vermelhos
2. **Quick wins antes de arquitetura** — ganhos visíveis rápidos mantêm momentum
3. **Um commit por fase** — rastreabilidade clara de "o que mudou e por quê"
4. **Critério de aceite binário** — ou passa ou não passa, sem "quase verde"
5. **Fase longa → divida em sub-fases** — qualquer fase com mais de 5 dias precisa ser quebrada

## Critério de aceite padrão (baseline)

```
- tsc --noEmit → PASS (sem erros novos introduzidos)
- npm test → 100% passando
- npm run build → PASS
- E2E serial → 100% passando
- lint nos arquivos alterados → sem erros
```

## Sinal de que a fase está mal definida

- Você não consegue escrever o critério de aceite em uma frase
- A fase tem mais de 10 tarefas
- O commit sugerido não consegue ser descrito em uma linha semântica

## Referências

- Origem: refactoring-plan 2026-02-23 (plano em 5 fases)
- Origem: refactoring-plan 2026-02-27 (plano dinâmico pós-análise)
