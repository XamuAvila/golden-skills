# Golden Skills

Marketplace de skills para Claude Code.

## Como usar

```bash
# 1. Adicione o marketplace
/plugin marketplace add XamuAvila/golden-skills

# 2. Instale um plugin
/plugin install design-patterns-csharp@golden-skills

# 3. Use a skill
/design-patterns-csharp:design-patterns-csharp
```

## Skills disponíveis

| Plugin | Descrição |
|--------|-----------|
| `design-patterns-csharp` | Referência completa dos 22 Design Patterns (GoF) com exemplos em C#/.NET, diagramas Mermaid e orientação prática. |
| `design-patterns-typescript` | Referência completa dos 22 Design Patterns (GoF) com exemplos em TypeScript, diagramas Mermaid e orientação prática. |
| `system-architecture` | Guia completo de System Architecture com 11 chapters: arquitetura de sistemas, API design, multi-cloud, technical debt, platform engineering e mais. |
| `impact-analysis` | Análise de impacto pré-implementação: busca semântica LSP/Serena, perguntas assertivas, diagramas Mermaid de fluxos afetados e checklist de riscos. |

## Estrutura

```
golden-skills/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── design-patterns-csharp/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           └── design-patterns-csharp/
│               ├── SKILL.md
│               └── patterns/
│                   ├── behavioral/
│                   ├── creational/
│                   └── structural/
└── README.md
```

## Contribuindo

Para adicionar uma nova skill:

1. Crie uma pasta em `plugins/<nome-do-plugin>/`
2. Adicione `.claude-plugin/plugin.json` com metadados
3. Crie `skills/<nome-da-skill>/SKILL.md`
4. Registre o plugin no `marketplace.json`
