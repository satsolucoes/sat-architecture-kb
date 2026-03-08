# Architecture Knowledge Base

> **Dono:** Felipe Desiderati
> **Última atualização:** 2026-03-08
> **Entradas:** 11 | Princípios: 3 | Padrões: 2 | Decisões: 3 | Refactorings: 3

Base de conhecimento arquitetural viva. Alimentada por decisões reais tomadas em projetos reais.
O objetivo é que Claude (e qualquer colaborador) nunca comece do zero.

---

## Estrutura

```
arch-kb/
├── principles/          # Princípios universais (stack-agnostic)
├── patterns/            # Padrões de solução abstratos validados
├── decisions/           # ADRs — Architecture Decision Records
├── refactorings/        # Lições de refactorings abstraídas (sem contexto de projeto)
├── implementations/     # Como os princípios/padrões se aplicam por stack
│   ├── react/
│   ├── typescript/
│   ├── kotlin/          # (futuro)
│   └── java/            # (futuro)
└── index.json           # Índice queryável (gerado/atualizado manualmente por ora)
```

---

## Como usar com Claude Code

Injete o contexto relevante no prompt:

```bash
# Exemplo: buscando padrões de state management em React
cat index.json | node scripts/kb-context.js --tags react,state-management
```

Ou referencie diretamente:

```
"Antes de implementar, consulte:
- principles/pragmatic-abstraction.md
- implementations/react/derived-state-no-useeffect.md"
```

---

## Como adicionar uma entrada

1. Crie o arquivo no diretório correto
2. Use o frontmatter YAML com os campos obrigatórios:
   ```yaml
   ---
   type: principle | pattern | decision | refactoring
   title: Título descritivo
   status: active | deprecated | experimental
   impact: high | medium | low
   date: YYYY-MM-DD
   tags: [tag1, tag2]
   applies_to:
     stacks: [universal | react | typescript | kotlin | java | angular]
     contexts: [architecture | testing | performance | ...]
   outcome: validated | partial | failed
   ---
   ```
3. Atualize `index.json`

---

## Princípios fundamentais deste KB

1. **Stack-agnostic primeiro**: princípios e padrões vivem separados das implementações
2. **Sem contexto de projeto**: nenhuma entrada menciona algum projeto específico ou entidades de negócio específicas
3. **Lição > Implementação**: o foco é o aprendizado, não o código exato
4. **Outcome explícito**: toda entrada documenta se funcionou ou não

---

## Entradas por tag

| Tag                | Entradas                                                                 |
|--------------------|--------------------------------------------------------------------------|
| clean-architecture | pragmatic-abstraction, port-adapter-persistence                          |
| SOLID              | pragmatic-abstraction, explicit-domain-rules-naming, hooks-separation    |
| react              | derived-state-no-useeffect, hooks-separation                             |
| typescript         | custom-props-typing                                                      |
| universal          | pragmatic-abstraction, one-source-of-truth, documentation-as-first-class |
| testing            | smoke-test-naming-convention                                             |
| automation         | precise-automation-triggers                                              |
