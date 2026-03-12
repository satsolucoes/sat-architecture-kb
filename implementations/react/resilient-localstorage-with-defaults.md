# Storage resiliente — leitura com defaults e silent fail na escrita

## Contexto

Acesso a `localStorage` falha silenciosamente em modo privado, quando a quota é excedida ou em ambientes com política de storage restrita. Se o app não tratar isso, gera exceções não capturadas que quebram fluxos críticos por uma razão que não tem nada a ver com lógica de negócio.

## Padrão

### Leitura: sempre retorna defaults, nunca lança

```typescript
// infra/storage/settings.storage.ts
export function readStorage<T extends object>(key: string, defaults: T): T {
  try {
    const raw = localStorage.getItem(key);
    if (!raw) return { ...defaults };
    const parsed: unknown = JSON.parse(raw);
    if (!isRecord(parsed)) return { ...defaults };  // guarda contra JSON corrompido
    return { ...defaults, ...parsed };              // merge: defaults completam campos ausentes
  } catch {
    return { ...defaults };
  }
}

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}
```

### Escrita: silent fail — não quebra a UI

```typescript
// infra/settings.ts
export function updateSetting<K extends keyof AppSettings>(
  key: K,
  value: AppSettings[K],
): void {
  const next: AppSettings = { ...getSettings(), [key]: value };
  try {
    writeStorage(STORAGE_KEYS.settings, next);
  } catch {
    // UI permanece funcional; estado em memória é suficiente para a sessão
  }
}
```

### Flags booleanas: helper genérico

```typescript
function getFlag(key: string): boolean {
  try { return localStorage.getItem(key) === 'true'; }
  catch { return false; }
}

function setFlag(key: string, value: boolean): void {
  try {
    value ? localStorage.setItem(key, 'true') : localStorage.removeItem(key);
  } catch { /* noop */ }
}
```

## Merge defensivo com defaults

```typescript
// Storage tem dado parcial (campo novo adicionado depois)
// defaults = { a: 1, b: 2, c: 3 }
// storage  = { a: 99 }
// resultado = { a: 99, b: 2, c: 3 }  ← nunca perde campos

return { ...defaults, ...parsed };
```

## Separação de camadas

```
shared/config.ts              ← STORAGE_KEYS, APP_SETTINGS_DEFAULTS
infra/storage/*.storage.ts    ← leitura/escrita bruta no localStorage
infra/settings.ts             ← API pública: getSettings(), updateSetting()
```

O domínio não importa nada de `infra/`. Quem precisa de settings importa de `infra/settings.ts`.

## Testando o comportamento resiliente

```typescript
it('does not throw when storage write fails', () => {
  vi.spyOn(Storage.prototype, 'setItem').mockImplementation(() => {
    throw new Error('quota exceeded');
  });

  expect(() => updateSetting('someFlag', true)).not.toThrow();
  expect(getSettings().someFlag).toBe(false); // defaults mantidos
});
```

## Quando aplicar

- Qualquer acesso a `localStorage`/`sessionStorage` em app que roda em browsers variados
- Configurações de usuário que não são críticas para o fluxo principal
- Flags de onboarding/tutorial que podem ser perdidas sem impacto grave

## O que não fazer

```typescript
// ❌ Acesso direto sem proteção
const settings = JSON.parse(localStorage.getItem('key')!);

// ❌ Tratar falha de storage como erro crítico
} catch (e) {
  throw new Error('Cannot save settings'); // quebra a UI por nada
}
```

## Exemplo de origem

Padrão aplicado na migração v2→v3 do `rac-designer-teto` para leitura de configurações do editor e flags de progresso do tutorial.

## Referências

- [[one-source-of-truth]]
- [[clean-architecture-layers]]

### Links KB
- [[port-adapter-persistence]] — mesmo princípio de isolamento da infra
- [[incremental-data-compatibility]] — merge com defaults protege contra formato antigo no storage
