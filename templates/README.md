# Templates Obsidian — Project Mnemonic

Templates para criação de novas entradas no KB via plugin **Templater**.

## Instalação

### 1. Copiar os templates para o vault

Coloque esta pasta `_templates/` dentro do vault:

```
sat-architecture-kb/
├── _templates/          ← esta pasta
│   ├── new-principle.md
│   ├── new-pattern.md
│   ├── new-decision.md
│   ├── new-refactoring.md
│   └── new-implementation.md
├── decisions/
├── implementations/
...
```

### 2. Instalar o plugin Templater

1. Obsidian → Settings → Community plugins → Browse
2. Buscar **Templater** → Install → Enable

### 3. Configurar o Templater

1. Settings → Templater
2. **Template folder location** → `_templates`
3. Habilitar **Trigger Templater on new file creation** (opcional)

## Como usar

### Criar nova entrada

1. `Ctrl+P` → digitar **Templater: Open Insert Template Modal**
2. Escolher o template pelo tipo (`new-principle`, `new-pattern`, etc.)
3. Preencher os prompts interativos (título, tags, stacks, impact...)
4. O arquivo é criado com frontmatter YAML preenchido

### Salvar no lugar certo

Após criar, mover o arquivo para a pasta correta:

| Tipo                      | Pasta                                          |
|---------------------------|------------------------------------------------|
| principle                 | `principles/`                                  |
| pattern                   | `patterns/`                                    |
| decision                  | `decisions/` com prefixo de data `YYYY-MM-DD_` |
| refactoring               | `refactorings/`                                |
| implementation React      | `implementations/react/`                       |
| implementation TypeScript | `implementations/typescript/`                  |

### Atualizar o index.json

Após criar uma nova entrada, adicionar ao `index.json` manualmente ou via script:

```bash
# (futuro) script de sync automático
node ~/.claude/skills/arch-kb-skill/scripts/sync-index.js
```

## Plugins recomendados

| Plugin         | Uso                                                      |
|----------------|----------------------------------------------------------|
| **Templater**  | Templates com prompts interativos                        |
| **Dataview**   | Queries no KB — ex: listar todas entradas `impact: high` |
| **Graph View** | Visualizar conexões entre entradas                       |
| **Git**        | Commit e push direto do Obsidian                         |

## Query Dataview úteis

```dataview
TABLE impact, status, date
FROM "principles"
WHERE status = "active"
SORT impact DESC
```

```dataview
TABLE impact, stacks, date
FROM "patterns" OR "decisions"
WHERE contains(stacks, "react")
SORT date DESC
```

```dataview
TABLE outcome, date
FROM "refactorings"
WHERE outcome = "validated"
```
