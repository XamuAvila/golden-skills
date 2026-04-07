# Golden Skills

Marketplace de skills para [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — referências, patterns, arquitetura e boas práticas de engenharia de software.

---

## Instalação

```bash
# 1. Adicione o marketplace
/plugin marketplace add XamuAvila/golden-skills

# 2. Liste os plugins disponíveis
/plugin search @golden-skills

# 3. Instale o plugin desejado
/plugin install <nome-do-plugin>@golden-skills
```

### Clonando o repositório

Este repo usa git submodules para incluir skills oficiais da LangChain. Para clonar com todo o conteúdo:

```bash
git clone --recurse-submodules https://github.com/XamuAvila/golden-skills.git

# Se já clonou sem submodules:
git submodule update --init --recursive
```

---

## Plugins Disponíveis

### All-in-One

| Plugin | Skills | Descrição |
|--------|--------|-----------|
| `golden-suite` | `full-review` | Orquestrador que combina TODAS as capabilities num único comando. Carrega referências sob demanda. |

```bash
/plugin install golden-suite@golden-skills
/golden-suite:full-review
```

> **Recomendado para a maioria dos usuários.** Instale apenas este plugin para ter acesso a Clean Code, Clean Architecture, System Architecture, Design Patterns (C#/TS/Go), Impact Analysis e Prompt Architecture Review — tudo numa só skill.

---

### C#/.NET

| Plugin | Skills | Descrição |
|--------|--------|-----------|
| `csharp-quality-suite` | `clean-code-csharp`, `clean-architecture-csharp`, `system-architecture` | Suite completa de qualidade de software C#/.NET combinando Clean Code, Clean Architecture e System Architecture |
| `design-patterns-csharp` | `design-patterns-csharp` | 22 Design Patterns (GoF) com exemplos em C#/.NET e diagramas Mermaid |

### TypeScript

| Plugin | Skills | Descrição |
|--------|--------|-----------|
| `design-patterns-typescript` | `design-patterns-typescript` | 22 Design Patterns (GoF) com exemplos em TypeScript e diagramas Mermaid |

### Go

| Plugin | Skills | Descrição |
|--------|--------|-----------|
| `design-patterns-go` | `design-patterns-go` | 22 Design Patterns (GoF) com exemplos idiomáticos em Go e framework de decisão arquitetural |

### AI/LLM

| Plugin | Skills | Descrição | Origem |
|--------|--------|-----------|--------|
| `langchain-skills` | 11 skills (LangGraph, LangChain, Deep Agents, RAG) | Skills oficiais para LangChain, LangGraph e Deep Agents | [langchain-ai/langchain-skills](https://github.com/langchain-ai/langchain-skills) (submodule) |
| `langsmith-skills` | `langsmith-trace`, `langsmith-dataset`, `langsmith-evaluator` | Skills oficiais para observabilidade e avaliação com LangSmith | [langchain-ai/langsmith-skills](https://github.com/langchain-ai/langsmith-skills) (submodule) |
| `prompt-architect` | `prompt-architecture-reviewer` | Reviewer de arquitetura de prompts para produção OpenAI API (GPT-4o, o1/o3, GPT-4.1, mini, fine-tuned) | Próprio |

### Engenharia de Software

| Plugin | Skills | Descrição |
|--------|--------|-----------|
| `impact-analysis` | `impact-analysis` | Análise de impacto pré-implementação com busca semântica, diagramas Mermaid e checklist de riscos |

---

## Detalhamento dos Plugins

### `csharp-quality-suite`

Suite unificada com 3 skills complementares para qualidade de software em C#/.NET:

**`/csharp-quality-suite:clean-code-csharp`**
Advisor de qualidade de código baseado nos princípios de *Clean Code* de Robert C. Martin. Identifica code smells, sugere refatorações e ensina práticas de código limpo com exemplos concretos em C#.
- Catálogo completo de code smells e heurísticas
- Guia de princípios: naming, funções, classes, comentários, error handling, formatação e testes
- Classificação por severidade: Critical, Major, Minor, Suggestion
- Adaptações específicas para C# (LINQ, async/await, pattern matching, records, nullable)

**`/csharp-quality-suite:clean-architecture-csharp`**
Advisor de arquitetura baseado nos princípios de *Clean Architecture* de Robert C. Martin. Orienta sobre estrutura de projetos, direção de dependências, boundaries e design de componentes.
- SOLID no nível arquitetural (SRP como Common Closure Principle, DIP entre camadas)
- Dependency Rule e camadas (Entities, Use Cases, Adapters, Frameworks & Drivers)
- Implementação prática em .NET: project structure, interfaces, dependency injection
- Orientação pragmática — escala a arquitetura para o tamanho do problema

**`/csharp-quality-suite:system-architecture`**
Guia completo de arquitetura de sistemas com 11 capítulos:
1. System Architecture — fundamentos e princípios
2. API Design Patterns — padrões de design de APIs
3. System Architecture Diagram — diagramas de arquitetura
4. Application Dependency Mapping — mapeamento de dependências
5. Multi-Cloud Architecture — arquitetura multi-cloud
6. Technical Debt Examples — exemplos de dívida técnica
7. Platform Engineering — engenharia de plataforma
8. Application Architecture Diagram — diagramas de aplicação
9. Software Design Document Template — templates de documentação
10. Web Application Architecture — arquitetura web
11. Legacy Application Modernization — modernização de legado

**Combinações recomendadas:**
- Review de código: `clean-code-csharp` + `design-patterns-csharp`
- Novo projeto: `clean-architecture-csharp` + `system-architecture`
- Refatoração: `clean-code-csharp` + `clean-architecture-csharp` + `impact-analysis`
- Documentação de arquitetura: `system-architecture` + `impact-analysis`

```bash
/plugin install csharp-quality-suite@golden-skills
```

---

### `design-patterns-csharp`

Referência completa dos 22 Design Patterns do Gang of Four com implementações em C#/.NET:

- **Creational** (5): Singleton, Factory Method, Abstract Factory, Builder, Prototype
- **Structural** (7): Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy
- **Behavioral** (10): Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor

Cada pattern inclui: diagrama Mermaid, código C# compilável, quando usar, vantagens/desvantagens e relações com outros patterns.

```bash
/plugin install design-patterns-csharp@golden-skills
/design-patterns-csharp:design-patterns-csharp
```

---

### `design-patterns-typescript`

Mesma cobertura dos 22 GoF patterns, com implementações em TypeScript para Node.js, frontend e frameworks como NestJS e Next.js.

```bash
/plugin install design-patterns-typescript@golden-skills
/design-patterns-typescript:design-patterns-typescript
```

---

### `design-patterns-go`

Catálogo dos 22 GoF patterns com implementações idiomáticas em Go. Inclui framework de decisão arquitetural que avalia o problema antes de recomendar patterns, respeitando os idiomas de Go (composição sobre herança, interfaces implícitas, goroutines/channels, `sync.Once`, erros explícitos).

```bash
/plugin install design-patterns-go@golden-skills
/design-patterns-go:design-patterns-go
```

---

### `impact-analysis`

Análise de impacto pré-implementação que mapeia dependências, fluxos afetados e riscos antes de escrever código:

- Busca semântica com LSP/Serena para encontrar símbolos afetados
- Perguntas assertivas para alinhar escopo
- Diagramas Mermaid dos fluxos impactados
- Checklist de riscos, breaking changes e pontos de atenção

Ideal para usar **antes** de implementar features, refatorações ou bugfixes.

```bash
/plugin install impact-analysis@golden-skills
/impact-analysis:impact-analysis
```

---

### `langchain-skills` (submodule oficial)

11 skills oficiais mantidas pela equipe da LangChain, organizadas em 4 categorias:

| Categoria | Skills |
|-----------|--------|
| **Getting Started** | `framework-selection`, `langchain-dependencies` |
| **LangChain** | `langchain-fundamentals`, `langchain-middleware`, `langchain-rag` |
| **LangGraph** | `langgraph-fundamentals`, `langgraph-persistence`, `langgraph-human-in-the-loop` |
| **Deep Agents** | `deep-agents-core`, `deep-agents-memory`, `deep-agents-orchestration` |

Fonte: [langchain-ai/langchain-skills](https://github.com/langchain-ai/langchain-skills) — incluído como git submodule.

```bash
/plugin install langchain-skills@golden-skills
```

---

### `langsmith-skills` (submodule oficial)

3 skills oficiais para observabilidade e avaliação de agentes LLM:

| Skill | Uso |
|-------|-----|
| `/langsmith-skills:langsmith-trace` | Rastreamento de execução e queries de traces |
| `/langsmith-skills:langsmith-dataset` | Criação e gerenciamento de datasets de avaliação |
| `/langsmith-skills:langsmith-evaluator` | Pipelines de avaliação (LLM-as-Judge, custom code, auto-run) |

Fonte: [langchain-ai/langsmith-skills](https://github.com/langchain-ai/langsmith-skills) — incluído como git submodule.

```bash
/plugin install langsmith-skills@golden-skills
```

---

### `prompt-architect`

Senior prompt architecture reviewer para sistemas em produção com OpenAI API:

- Analisa prompts contra 10 critérios de qualidade com scoring 0-30
- Reescreve prompts otimizados para produção (copy-paste ready)
- Cobre todo o stack OpenAI: GPT-4o, o1/o3, GPT-4.1, mini, fine-tuned
- Patterns especializados: tool calling, structured outputs, multi-agent, RAG
- Gera variantes opcionais: JSON output, tool calling, reasoning, supervisor multi-agent

```bash
/plugin install prompt-architect@golden-skills
/prompt-architect:prompt-architecture-reviewer
```

---

## Estrutura do Repositório

```
golden-skills/
├── .claude-plugin/
│   └── marketplace.json              # Registro de todos os plugins
├── plugins/
│   ├── csharp-quality-suite/         # Suite C#: Clean Code + Clean Arch + System Arch
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── clean-code-csharp/
│   │       │   ├── SKILL.md
│   │       │   └── references/
│   │       ├── clean-architecture-csharp/
│   │       │   ├── SKILL.md
│   │       │   └── references/
│   │       └── system-architecture/
│   │           ├── SKILL.md
│   │           └── chapters/         # 11 capítulos
│   ├── design-patterns-csharp/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       └── design-patterns-csharp/
│   │           ├── SKILL.md
│   │           └── patterns/         # behavioral/ creational/ structural/
│   ├── design-patterns-typescript/
│   ├── design-patterns-go/
│   ├── impact-analysis/
│   ├── langchain-skills/        # git submodule → langchain-ai/langchain-skills
│   ├── langsmith-skills/        # git submodule → langchain-ai/langsmith-skills
│   └── prompt-architect/
└── README.md
```

---

## Contribuindo

Para adicionar um novo plugin:

1. Crie a pasta `plugins/<nome-do-plugin>/`
2. Adicione `.claude-plugin/plugin.json`:
   ```json
   {
     "name": "meu-plugin",
     "version": "1.0.0",
     "description": "Descrição do plugin",
     "skills": "./skills/"
   }
   ```
3. Crie `skills/<nome-da-skill>/SKILL.md` com frontmatter:
   ```yaml
   ---
   name: minha-skill
   description: "Quando e como a skill deve ser ativada"
   ---
   ```
4. Registre o plugin no `marketplace.json`
5. Abra um PR

### Multi-skill plugins

Para plugins com múltiplas skills, crie subdiretórios separados:

```
plugins/meu-plugin/
├── .claude-plugin/plugin.json
└── skills/
    ├── skill-a/
    │   └── SKILL.md
    └── skill-b/
        └── SKILL.md
```

As skills ficam acessíveis como `/meu-plugin:skill-a` e `/meu-plugin:skill-b`.
