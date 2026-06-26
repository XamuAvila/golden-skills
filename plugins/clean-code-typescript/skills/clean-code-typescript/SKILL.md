---
name: clean-code-typescript
description: >
  Use when writing, reviewing, or refactoring TypeScript or JavaScript code and the user
  asks about clean code practices, naming conventions, function design, SOLID principles,
  error handling, immutability, or code readability. Also use when the user mentions
  clean code, código limpo, Robert C. Martin, or asks for best practices in TS/JS code
  quality. Triggers on: variable naming, function parameters, side effects, DRY, single
  responsibility, open/closed principle, Liskov substitution, interface segregation,
  dependency inversion, async/await patterns, error handling, test organization, code
  comments. Covers TypeScript-specific patterns: strict types, readonly, discriminated
  unions, branded types, exhaustive checks, Result pattern.
---

# Clean Code TypeScript — Guia de Referência

Princípios de Clean Code adaptados para TypeScript, baseados no livro "Código Limpo"
de Robert C. Martin e no repositório clean-code-javascript de Ryan McDermott.

## Quando esta skill é acionada

- Usuário pergunta sobre boas práticas de código TypeScript/JavaScript
- Usuário pede revisão de qualidade de código TS/JS
- Usuário quer melhorar legibilidade, manutenibilidade ou testabilidade
- Usuário menciona clean code, código limpo, ou Robert C. Martin no contexto TS/JS
- Usuário tem code smells: funções longas, nomes ruins, efeitos colaterais, deep nesting
- Usuário quer entender SOLID aplicado a TypeScript
- Usuário pergunta sobre tratamento de erros, imutabilidade, ou async patterns em TS

## Como usar os arquivos de referência

Os exemplos e descrições ficam em arquivos separados por seção.
**Leia APENAS o arquivo da seção relevante**, não carregue tudo de uma vez.

### Estrutura dos arquivos

```
sections/
├── variables.md              # Nomes, constantes, contexto
├── functions.md               # Parâmetros, SRP, side effects, composição
├── objects-and-data-structures.md  # Getters/setters, encapsulamento, privacidade
├── classes.md                 # ES6+, encadeamento, composição vs herança
├── solid.md                   # SRP, OCP, LSP, ISP, DIP com exemplos TS
├── testing.md                 # Um conceito por teste, organização
├── concurrency.md             # Promises, async/await
├── error-handling.md          # Try/catch, Result pattern, never swallow errors
├── formatting.md              # Capitalização, proximidade de funções
├── comments.md                # Quando comentar, código morto, marcadores
└── typescript-specific.md     # Strict types, branded types, exhaustive checks
```

### Índice rápido por seção

| Seção | Arquivo | Princípios-chave |
|-------|---------|-----------------|
| Variáveis | `sections/variables.md` | Nomes significativos, pesquisáveis, sem mapeamento mental |
| Funções | `sections/functions.md` | ≤2 params, SRP, sem flags, sem side effects |
| Objetos e Dados | `sections/objects-and-data-structures.md` | Getters/setters, membros privados, encapsulamento |
| Classes | `sections/classes.md` | ES6+, fluent API, composição > herança |
| SOLID | `sections/solid.md` | SRP, OCP, LSP, ISP, DIP com TypeScript |
| Testes | `sections/testing.md` | Um conceito por teste, AAA pattern |
| Concorrência | `sections/concurrency.md` | Async/await > Promises > callbacks |
| Erros | `sections/error-handling.md` | Never swallow, Result pattern, typed errors |
| Formatação | `sections/formatting.md` | Consistência, proximidade vertical |
| Comentários | `sections/comments.md` | Só lógica de negócio, sem código morto |
| TypeScript | `sections/typescript-specific.md` | Strict mode, branded types, exhaustive checks |

## Fluxo de resposta

### 1. Identificar a seção

Se o usuário descreve um problema de qualidade de código, use o índice acima para
encontrar a seção relevante e leia o arquivo correspondente.

### 2. Apresentar a resposta

A resposta deve incluir:

1. **Princípio** — Qual regra de clean code se aplica
2. **Ruim** — O anti-pattern com explicação do problema
3. **Bom** — A versão limpa em TypeScript com explicação
4. **Por quê** — Benefício concreto (testabilidade, legibilidade, manutenção)

### 3. Adaptar ao contexto TypeScript

Ao apresentar soluções, use recursos modernos de TypeScript:
- **Tipos estritos** em vez de `any` — use generics, union types, type guards
- **Readonly** para imutabilidade — `readonly`, `Readonly<T>`, `ReadonlyArray<T>`
- **Discriminated unions** em vez de switch/if em strings
- **Branded types** para type-safety em primitivos (IDs, emails, etc.)
- **Exhaustive checks** com `never` para garantir cobertura de cases
- **Result pattern** em vez de try/catch quando o erro é esperado

## Guia de decisão: qual seção consultar?

| Sintoma no código | Seção |
|-------------------|-------|
| Nomes de variáveis confusos ou abreviados | Variáveis |
| Função com muitos parâmetros | Funções |
| Função que faz várias coisas | Funções, SOLID (SRP) |
| Mutação de estado compartilhado | Funções (side effects) |
| Classe com muitas responsabilidades | SOLID (SRP) |
| Switch/if longo para decidir comportamento | SOLID (OCP), Classes |
| Herança mal modelada | SOLID (LSP), Classes |
| Interface/config com propriedades opcionais demais | SOLID (ISP) |
| Dependência concreta em vez de abstração | SOLID (DIP) |
| Callbacks aninhados, promise chains | Concorrência |
| `catch` vazio ou `console.log(error)` | Erros |
| Comentários descrevendo o que o código faz | Comentários |
| Código comentado no codebase | Comentários |
| `any` usado como "escape hatch" | TypeScript-específico |
| Type assertions desnecessárias (`as`) | TypeScript-específico |
| Testes com múltiplas asserções não relacionadas | Testes |

## Princípios fundamentais (resumo)

1. **Clareza > Brevidade** — Código é lido 10x mais do que escrito
2. **Uma função, uma coisa** — SRP vale para funções, classes e módulos
3. **Imutabilidade por padrão** — `const`, `readonly`, spread operators
4. **Tipos como documentação** — O sistema de tipos substitui comentários
5. **Erros são cidadãos de primeira classe** — Nunca ignore, nunca engula
6. **Testes documentam comportamento** — O nome do teste é a especificação
7. **Composição > Herança** — "has-a" antes de "is-a"

## Fonte

Baseado em:
- [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript) — Ryan McDermott
- [Código Limpo](https://www.amazon.com.br/C%C3%B3digo-Limpo-Habilidades-Pr%C3%A1ticas-Software/dp/8576082675) — Robert C. Martin
