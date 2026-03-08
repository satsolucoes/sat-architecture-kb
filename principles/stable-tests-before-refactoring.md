---
type: principle
title: Suíte de testes estável é pré-condição para qualquer refactoring
status: active
impact: high
date: 2026-02-24
tags: [testing, refactoring, safety, regression, CI-CD, quality]
applies_to:
  stacks: [universal]
  contexts: [testing, refactoring, process, CI-CD]
outcome: validated
---

## Princípio

> Nunca inicie uma refatoração estrutural com testes falhando.
> Testes vermelhos são ruído — você não sabe se quebrou algo novo ou se já estava quebrado.

## Por quê isso importa

- Refactoring sem testes = você voa às cegas
- Testes falhando antes = baseline corrompido
- Você termina a refatoração sem saber se regrediu ou não

## Ordem obrigatória

```
1. Testes todos verdes (baseline limpo)
2. Refactoring incremental
3. Testes verdes a cada incremento
4. Commit
```

Se chegou num ponto com testes falhando sem ter feito a refactoring ainda:

```
1. Corrigir os testes falhando PRIMEIRO (sem refactoring junto)
2. Commit de correção isolado
3. Depois iniciar refactoring
```

## Causas comuns de testes quebrando em refactorings

### 1. Funções movidas sem re-exportar

```typescript
// ❌ Moveu a função mas esqueceu o export
// house-views-layout.use-case.ts
function resolveHouseViewInsertion(...) { ... } // sem export!

// Teste tenta importar → quebra silencioso
import { resolveHouseViewInsertion } from './house-views-layout.use-case';
```

**Fix**: sempre auditar exports ao mover funções entre módulos.

### 2. Contexto de `this` perdido em funções standalone

```typescript
// ❌ Função standalone usando this — contexto undefined em runtime
export function readDataFromCanvas() {
  return this.house?.data; // this é undefined!
}

// ✅ Passar dependência como parâmetro explícito
export function readDataFromCanvas(house: House) {
  return house?.data;
}
```

### 3. Dependência de teste não declarada

```bash
# Erro: Cannot find module '@testing-library/dom'
# Causa: peer dependency não declarada no package.json
npm install --save-dev @testing-library/dom
```

### 4. Mock incompleto de API de browser

```typescript
// ❌ JSDOM não implementa canvas API do browser
// Testes que usam libs de canvas falham com TypeError

// ✅ Mock robusto no setup de testes
// src/test/setup.ts
HTMLCanvasElement.prototype.getContext = vi.fn(() => ({
  fillRect: vi.fn(),
  clearRect: vi.fn(),
  getImageData: vi.fn(),
  // ... demais métodos necessários
}));
```

## Checklist mínimo antes de iniciar refactoring

```
□ npm run test -- --run → 100% passando
□ npm run build → sem erros
□ npx tsc --noEmit → sem erros de tipo
□ Lint dos arquivos que serão tocados → limpo
```

## Checklist mínimo a cada commit de refactoring

```
□ Mesmo checklist acima
□ Nenhum novo warning introduzido
□ Comportamento visível idêntico ao anterior
```

## Quando testes quebram durante refactoring

```
1. PARE imediatamente — não continue refatorando
2. Entenda a causa raiz (não apenas o sintoma)
3. Corrija no commit atual ou reverta
4. Só avance quando verde novamente
```

## Cobertura mínima por camada (referência)

| Camada                    | Cobertura mínima recomendada |
|---------------------------|------------------------------|
| Domínio/regras de negócio | 80%+                         |
| Infraestrutura            | 50%+                         |
| Componentes de UI         | 20%+ (smoke tests)           |
| Utils/helpers             | 60%+                         |

## Lições aprendidas

- Mega refactoring com 149 arquivos alterados gerou 12 testes falhando — baseline corrompido
- Correções de teste foram commitadas separadas da refactoring (correto)
- Guard automático de imports legados (`legacy-imports.smoke.test.ts`) criado para evitar regressão silenciosa de paths
- E2E em paralelo apresentou flakiness — suíte serial (`--workers=1`) mais confiável para validação de regressão

## Referências

- Origem: regression-checklist.md (2026-02-24)
- Origem: regression-run.md (2026-02-25)
- Origem: analysis-report.md (12 testes falhando, 2026-02-27)
