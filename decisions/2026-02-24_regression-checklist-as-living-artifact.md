---
type: decision
title: Checklist de regressão adaptativo como artefato vivo de qualidade
status: active
impact: medium
date: 2026-02-24
tags: [testing, regression, checklist, process, quality, CI-CD]
applies_to:
  stacks: [universal]
  contexts: [testing, process, quality, CI-CD, team]
outcome: validated
---

## Contexto

Em mudanças estruturais de alto impacto, testes automatizados não cobrem 100% dos cenários.
É necessário um protocolo de validação manual e automatizada combinados.

## Decisão

Criar e manter um checklist de regressão adaptativo por rodada de mudança significativa.

## Estrutura do checklist

### Camadas de validação

```
1. Automática obrigatória (sempre rodar antes de merge):
   □ tsc --noEmit (tipagem)
   □ lint (qualidade de código)
   □ test --run (unit + smoke)
   □ build (build de produção)
   □ test:e2e --workers=1 (E2E serial — mais estável que paralelo)

2. Manual por área de risco (baseado na mudança atual):
   □ Lote 1: fluxo crítico de abertura (editor abre?)
   □ Lote 2: regras de negócio principais
   □ Lote 3: ferramentas e elementos
   □ Lote N: área específica da mudança

3. Arquitetura (validação de contratos):
   □ Sem imports para módulos removidos
   □ Contratos de tipo consistentes entre camadas
   □ Wiring de componentes/hooks correto
```

### Critério de aceite

Rodada **estável** apenas quando:

- 100% dos testes automatizados verdes
- Itens manuais críticos (não apenas "nice to have") validados
- Zero erros de console nos fluxos críticos
- Zero dependência residual de arquivos removidos

## Guard automático para regressões silenciosas

```typescript
// smoke test que bloqueia imports de módulos legados
// src/infra/legacy-imports.smoke.test.ts
import { glob } from 'glob';

test('nenhum arquivo importa de módulos legados', async () => {
  const files = await glob('src/**/*.{ts,tsx}');
  const legacyPattern = /@\/lib\/|src\/lib\//;

  for (const file of files) {
    const content = await fs.readFile(file, 'utf-8');
    expect(content).not.toMatch(legacyPattern);
  }
});
```

**Padrão poderoso**: ao invés de lembrar de verificar manualmente, automatize o guard.

## E2E: serial vs paralelo

```bash
# ❌ Paralelo — flakiness em máquinas com menos recursos
npm run test:e2e

# ✅ Serial — mais lento, mas confiável para validação de regressão
npm run test:e2e -- --workers=1
```

Usar serial como critério de aceite. Paralelo apenas para feedback rápido em desenvolvimento.

## Evidências de execução

Toda rodada de regressão deve registrar evidências:

```markdown
### Evidências — 2026-02-25
- tsc strict → PASS
- test → PASS (117/117)
- build → PASS
- e2e workers=1 → PASS (17/17)
- lint → FAIL (dívida legada — fora do escopo desta rodada, documentado)
```

Documentar falhas conhecidas e fora do escopo é tão importante quanto registrar os passes.

## Lições aprendidas

- Checklist adaptativo por rodada é mais útil que checklist genérico fixo
- Guard de imports legados automatizado evitou regressão silenciosa em refactoring de namespace
- Evidências de execução no changelog servem como rastreabilidade para futuras investigações
- Lint com dívida legada ampla: separar lint dos arquivos alterados do lint geral

## Referências

- Origem: regression-checklist.md (2026-02-24)
- Origem: regression-run.md (2026-02-25)
