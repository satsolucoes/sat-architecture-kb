---
type: principle
title: Mandatory Decision Flow — fluxo obrigatório antes de escrever qualquer código
status: active
impact: high
date: 2026-02-27
tags: [process, architecture, reuse, decision-making, clean-code, SOLID]
applies_to:
  stacks: [universal]
  contexts: [architecture, process, team, refactoring, feature-development]
outcome: validated
---

## O problema que este fluxo resolve

Código criado por reflexo, sem auditoria prévia, gera duplicação, abstração prematura
e inconsistências arquiteturais que custam caro para desfazer.

## Fluxo obrigatório (5 passos — nesta ordem)

### 1. Define Problem

Descreva o problema em uma única frase:

```
[sintoma observado] + [impacto no negócio/usuário]

Ex: "O componente X não atualiza após mudança de estado
     porque lê valor stale, causando inconsistência visual para o usuário."
```

### 2. Inventory Existing Code

Busca global antes de criar qualquer coisa:

```bash
rg -n "nomeDoComportamento" src/
rg -n "palavraChave" src/ --type ts
```

**A evidência da busca deve ser registrada.** Se não buscou, não passou no fluxo.

### 3. Classify Logic

Separe o que foi encontrado em duas categorias:

- **Comum**: lógica de UI, fluxo, estado — pode ser compartilhada
- **Específica**: regra de negócio de um domínio particular — fica no consumidor

### 4. Apply Decision Matrix

Com base na classificação:

| Situação                                                     | Ação                                          |
|--------------------------------------------------------------|-----------------------------------------------|
| Comportamento já existe, diferença é só em dados/props/texto | **REUSE** — estenda API existente             |
| Duplicação em 2+ pontos, chance concreta de terceiro uso     | **EXTRACT** — crie módulo compartilhado       |
| Auditoria prova que não existe equivalente                   | **CREATE** — documente por que não reutilizou |

### 5. Implement

Só após os passos 1–4 concluídos.

## Matriz de decisão detalhada

### REUSE (Reutilizar)

```
Quando:
- Comportamento desejado já existe
- Diferença é em dados, texto, estilo ou props simples
- Contrato de interação é idêntico (abrir/editar/confirmar/cancelar)

Ação: estenda API existente com menor superfície de mudança possível
NÃO crie módulo paralelo
```

### EXTRACT (Extrair comum)

```
Quando:
- Duplicação de lógica/estrutura em 2+ pontos
- Há chance concreta e imediata de um terceiro uso
- Fluxo base é o mesmo, muda apenas regra de domínio no final

Ação: extraia para módulo compartilhado
Regra de negócio específica fica no consumidor (injetada como prop/callback)
```

### CREATE (Criar novo)

```
Quando:
- Auditoria comprova que não existe implementação equivalente
- Natureza da interação/domínio é fundamentalmente diferente
- Reutilização forçaria acoplamento prejudicial ou abstração excessiva

Ação: crie novo módulo
OBRIGATÓRIO: documente por que reuse/extract não foram possíveis
```

## Anti-padrões que este fluxo previne

```
❌ Criar componente/hook sem auditoria prévia documentada
❌ Copiar/colar lógica de UI "para acelerar"
❌ Abstração complexa por necessidade hipotética futura
❌ Resolver problema visual sem validar contrato funcional
```

## Critério de abstração (quando justificar uma nova)

Uma abstração só é justificada se atender **pelo menos 2** dos seguintes **imediatamente**:

1. Elimina duplicação de lógica crítica já existente
2. Reduz acoplamento problemático já existente
3. Permite troca REAL de implementação necessária no curto/médio prazo
4. Melhora significativamente testabilidade de comportamento central

## Referências

- Origem: 01_core_principles.md (Principles 1-5, 2026-02-27)

### Links KB

- Relacionado: [[pragmatic-abstraction]]
- Relacionado: [[explicit-domain-rules-naming]]
- Relacionado: [[documentation-as-first-class-artifact]]
