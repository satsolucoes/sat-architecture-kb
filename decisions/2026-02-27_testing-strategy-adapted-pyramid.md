---
type: decision
title: Estratégia de testes — pirâmide adaptada e filosofia de comportamento
status: active
impact: high
date: 2026-02-27
tags: [testing, strategy, behavior, pyramid, integration-tests, e2e, smoke-tests]
applies_to:
  stacks: [universal]
  contexts: [testing, quality, CI-CD]
outcome: validated
---

## Filosofia central

> Teste o comportamento, não a implementação.

Testes que verificam detalhes internos (estado, estrutura de classes, chamadas específicas) quebram a cada refactoring —
mesmo sem mudar o comportamento observável. Testes que verificam o que o usuário vê e faz sobrevivem a refactorings.

## Pirâmide adaptada

```
          /\ 
         /  \
        /E2E \         ← fluxos críticos em browser ou sistema real
       /------\
      / Integr.\       ← foco principal — comportamento de componentes/módulos
     /  (larga) \
    /------------\
   / Unit + Smoke \    ← lógica pura de domínio, funções isoladas
  /________________\
```

### Quando usar cada camada

| Camada     | Quando usar                               | Onde fica                 |
|------------|-------------------------------------------|---------------------------|
| Unit/Smoke | Lógica pura complexa (domain, lib, utils) | co-localizado com source  |
| Integração | Módulos com interação entre partes        | próximo ao módulo         |
| E2E        | Happy paths de fluxos críticos            | diretório `e2e/` separado |

## Seletores — princípio de hierarquia (universal)

Selecionar por **semântica** antes de **detalhe de implementação**:

```
Preferência crescente de fragilidade:

1. Papel semântico / intenção do elemento   ← mais resistente
2. Label / nome visível ao usuário
3. Texto visível
4. Valor de entrada
5. Atributo de teste explícito (data-testid) ← escape hatch, último recurso
6. Seletor CSS / XPath / ID                 ← evitar
```

## O que um teste de integração deve responder

```
1. Renderiza/inicia corretamente com os dados fornecidos?
2. Exibe diferentes estados (carregando, erro, vazio, sucesso)?
3. Ação do usuário/sistema produz o efeito esperado?
4. Mensagens de feedback (erro, confirmação) aparecem quando deveriam?
```

## Smoke test — propósito e regras

- **Propósito**: verificar que o artefato existe, carrega e executa o caso base sem explodir
- **Localização**: co-localizado com o arquivo source (`{arquivo}.smoke.test.{ext}`)
- **Tamanho**: pequeno — smoke é cobertura mínima, não cobertura completa
- **Regra crítica**: nunca `expect(true).toBe(true)` — toda assertion deve ser real

```
// ✅ smoke mínimo e honesto
it('creates entity with default values', () => {
  const entity = createEntity();
  expect(entity.id).toBeDefined();
  expect(entity.status).toBe('draft');
});

// ❌ smoke falso
it('should work', () => {
  expect(true).toBe(true);
});
```

## E2E — boas práticas universais

- Cobrir happy paths de fluxos críticos, não exaustividade
- Page Object / helpers para fluxos complexos — evita repetição nos specs
- Rodar serial (workers=1) para validação de regressão — mais lento, confiável

## Anti-padrões universais

```
❌ Testar estado interno — quebra a cada refactoring interno
❌ Testar implementação em vez de comportamento
❌ Assertions sem valor real (expect(true).toBe(true))
❌ Acesso direto ao DOM por seletor CSS como primeira opção
❌ E2E cobrindo casos que integração cobre melhor
```

## Acessibilidade como bônus

Usar seletores semânticos (role, label) como padrão tem duplo benefício:

1. Testes mais resilientes a refactoring
2. Força a criação de interfaces acessíveis — elemento sem semântica = teste difícil

## Implementações por stack

- `implementations/react/testing-strategy-react.md` — Vitest, React Testing Library, Playwright
- `implementations/kotlin/` ← JUnit, MockK, Kotest (quando chegar)
- `implementations/java/` ← JUnit, Mockito (quando chegar)

## Referências

- Origem: 08_testing.md (2026-02-27)
- Relacionado: decisions/2026-03-04_smoke-test-naming-convention.md

### Links KB

- Princípio: [[stable-tests-before-refactoring]]
- Relacionado: [[2026-03-04_smoke-test-naming-convention]]
- Relacionado: [[2026-02-24_regression-checklist-as-living-artifact]]
- Implementação React: [[testing-strategy-react]]
