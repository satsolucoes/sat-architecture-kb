---
type: pattern
title: Clean Architecture — camadas, fluxo de dependência e responsabilidades
status: active
impact: high
date: 2026-02-27
tags: [clean-architecture, layers, dependency-flow, domain, infra, hexagonal]
applies_to:
  stacks: [universal]
  contexts: [architecture, project-structure, domain, infra]
outcome: validated
---

## Conceito central

O domínio não conhece ninguém. Todo o resto conhece o domínio.

```
Bootstrap (composition root)
    ↓ pode depender de tudo
Components / UI
    ↓ depende de domain + shared
Infra (implementações concretas)
    ↓ depende apenas de contratos do domain
Domain (núcleo)
    ↓ não depende de nada externo
Shared (tipos e utilitários puros)
    ↓ não depende de ninguém
```

## Camadas e responsabilidades

### Domain (núcleo)

```
✅ Regras de negócio
✅ Invariantes do modelo
✅ Agregados — estado + comportamento do domínio
✅ Ports — interfaces que definem O QUE precisa, não COMO
✅ Use cases — orquestra regras + ports

❌ Frameworks de UI (React, Angular, etc.)
❌ Bibliotecas de terceiros (canvas, HTTP, banco, ORM)
❌ Detalhes concretos de persistência
```

### Infra

```
✅ Implementações concretas dos ports (adapters)
✅ Integração com libs técnicas (banco, HTTP, storage, canvas)
✅ Adapters de persistência

❌ Regras de negócio
❌ Componentes visuais
```

### Bootstrap (composition root)

```
✅ Criação de stores e adapters
✅ Composição e wiring de dependências
✅ Injeção via contexto/DI

❌ Regras de negócio
❌ Lógica de renderização visual
```

### Shared

```
✅ Tipos e interfaces compartilhados
✅ Constantes globais
✅ Utilitários puros (sem side effects)

❌ Regras de negócio de domínio
❌ Importações de domain, infra ou UI
```

## Artefatos e nomenclatura (stack-agnostic)

| Artefato      | Papel                                      | Onde vive           |
|---------------|--------------------------------------------|---------------------|
| **Aggregate** | Estado + invariantes de um modelo          | `domain/`           |
| **Port**      | Interface (contrato) que o domínio precisa | `domain/`           |
| **Adapter**   | Implementação concreta de um port          | `infra/`            |
| **Use Case**  | Orquestra domain + ports para uma ação     | `domain/use-cases/` |
| **Bootstrap** | Composição e wiring de tudo                | `bootstrap/`        |

## Violações mais comuns

```
❌ Domain importando framework de UI
❌ Domain importando biblioteca concreta (HTTP, ORM, canvas)
❌ Infra contendo regra de negócio
❌ Shared acumulando regras de domínio ("lixeira arquitetural")
❌ Store/singleton exportado diretamente de módulo (deve passar pelo bootstrap)
❌ UI ou canvas como fonte de verdade do estado
```

## A violação mais silenciosa

Domain conhecer detalhe concreto de infra sem perceber:

```
// ❌ ERRADO — domain acoplado a detalhe concreto
import { postgresDb } from '../infra/database';

// ✅ CERTO — domain define contrato, infra implementa
// domain define:
interface HousePersistencePort {
  save(house: House): Promise<void>;
  load(id: string): Promise<House | null>;
}
// infra implementa — domain nunca sabe que Postgres existe
```

## Implementações por stack

- `implementations/react/clean-architecture-react.md` — estrutura de pastas, templates, convenções React/TypeScript
- `implementations/kotlin/` ← (quando chegar Kotlin)
- `implementations/java/` ← (quando chegar Java/Spring)

## Referências

- Origem: 03_project_structure.md (2026-02-27)
- Origem: 06_hooks_and_state.md (2026-02-27)

### Links KB

- Princípio: [[pragmatic-abstraction]]
- Princípio: [[explicit-domain-rules-naming]]
- Relacionado: [[port-adapter-persistence]]
- Relacionado: [[strategy-registry-by-type]]
- Implementação React: [[clean-architecture-react]]
