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
