# Guide complet : Creer des agents avec Claude

## Table des matieres

1. [Introduction](#introduction)
2. [Les deux approches](#les-deux-approches)
3. [Approche 1 : Claude Agent SDK](#approche-1--claude-agent-sdk)
4. [Approche 2 : API Claude avec boucle agentique](#approche-2--api-claude-avec-boucle-agentique)
5. [Definir des outils (Tools)](#definir-des-outils-tools)
6. [Patterns multi-agents](#patterns-multi-agents)
7. [Hooks et personnalisation](#hooks-et-personnalisation)
8. [Gestion des sessions](#gestion-des-sessions)
9. [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction

Claude propose deux surfaces principales pour creer des agents IA :

| Surface | Cas d'usage | Complexite |
|---------|------------|------------|
| **Claude Agent SDK** | Agent avec outils integres (fichiers, web, terminal, MCP) | Simple |
| **Claude API + Tool Use** | Agent custom avec vos propres outils, controle total | Flexible |

### Arbre de decision

```
Votre agent a-t-il besoin de lire/ecrire des fichiers,
naviguer sur le web ou executer des commandes shell ?
  |
  +-- Oui --> Agent SDK (outils integres, pas besoin de les reimplementer)
  |
  +-- Non, j'ai mes propres outils --> API Claude + Tool Use
      |
      +-- Je veux un loop automatique --> Tool Runner (beta)
      +-- Je veux un controle total   --> Boucle agentique manuelle
```

---

## Les deux approches

### Agent SDK

- Outils integres : lecture/ecriture de fichiers, recherche, terminal, web
- Systeme de permissions
- Support MCP (Model Context Protocol)
- Sous-agents natifs
- Disponible en **Python** et **TypeScript**

### API Claude + Tool Use

- Vous definissez vos propres outils
- Controle total sur la boucle d'execution
- Tool Runner (beta) pour automatiser la boucle
- Disponible dans **tous les SDK** (Python, TypeScript, Java, Go, Ruby, C#, PHP)

---

## Approche 1 : Claude Agent SDK

### Installation

**Python :**

```bash
pip install claude-agent-sdk
```

**TypeScript :**

```bash
npm install @anthropic-ai/claude-agent-sdk
```

### Usage rapide (Python)

```python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explique ce projet",
        options=ClaudeAgentOptions(
            cwd="/chemin/vers/projet",
            allowed_tools=["Read", "Glob", "Grep"]
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
```

### Usage rapide (TypeScript)

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Explique ce projet",
  options: {
    cwd: "/chemin/vers/projet",
    allowedTools: ["Read", "Glob", "Grep"],
  },
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
```

### Outils integres disponibles

| Outil | Description |
|-------|------------|
| `Read` | Lire des fichiers |
| `Write` | Creer des fichiers |
| `Edit` | Modifier des fichiers existants |
| `Bash` | Executer des commandes shell |
| `Glob` | Chercher des fichiers par pattern |
| `Grep` | Chercher du contenu dans les fichiers |
| `WebSearch` | Recherche web |
| `WebFetch` | Recuperer des pages web |
| `Agent` | Lancer des sous-agents |
| `AskUserQuestion` | Poser des questions a l'utilisateur |

### Modes de permission

```python
# Par defaut : demande confirmation pour les operations dangereuses
options = ClaudeAgentOptions(permission_mode="default")

# Accepter automatiquement les modifications de fichiers
options = ClaudeAgentOptions(permission_mode="acceptEdits")

# Mode planification uniquement (pas d'execution)
options = ClaudeAgentOptions(permission_mode="plan")

# Bypass total (a utiliser avec precaution)
options = ClaudeAgentOptions(permission_mode="bypassPermissions")
```

### Interface ClaudeSDKClient (controle complet)

Pour un controle avance (outils custom, interruption, streaming) :

```python
import anyio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock

async def main():
    options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    async with ClaudeSDKClient(options=options) as client:
        await client.query("Analyse ce code")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
```

### Outils custom via MCP

Les outils custom necessitent un serveur MCP. Utilisez `ClaudeSDKClient` :

**Python :**

```python
import anyio
from claude_agent_sdk import (
    tool,
    create_sdk_mcp_server,
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)

# 1. Definir un outil custom
@tool("get_weather", "Obtenir la meteo d'un lieu", {"location": str})
async def get_weather(args):
    location = args["location"]
    return {
        "content": [
            {"type": "text", "text": f"Il fait 22°C et ensoleille a {location}."}
        ]
    }

# 2. Creer un serveur MCP avec les outils
server = create_sdk_mcp_server("weather-tools", tools=[get_weather])

# 3. Lancer l'agent avec le serveur MCP
async def main():
    options = ClaudeAgentOptions(mcp_servers={"weather": server})
    async with ClaudeSDKClient(options=options) as client:
        await client.query("Quel temps fait-il a Paris ?")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
```

**TypeScript :**

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const getWeather = tool(
  "get_weather",
  "Obtenir la meteo",
  { location: z.string() },
  async (args) => {
    return {
      content: [{ type: "text", text: `22°C et ensoleille a ${args.location}` }],
    };
  }
);

const server = createSdkMcpServer({ name: "weather", tools: [getWeather] });

for await (const message of query({
  prompt: "Quel temps fait-il a Paris ?",
  options: { mcpServers: { weather: server } },
})) {
  if ("result" in message) console.log(message.result);
}
```

### Integration MCP avec des serveurs externes

```python
# Automatisation navigateur avec Playwright
async for message in query(
    prompt="Va sur example.com et decris ce que tu vois",
    options=ClaudeAgentOptions(
        mcp_servers={
            "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

```python
# Acces base de donnees PostgreSQL
async for message in query(
    prompt="Montre les 10 meilleurs utilisateurs",
    options=ClaudeAgentOptions(
        mcp_servers={
            "postgres": {
                "command": "npx",
                "args": ["-y", "@modelcontextprotocol/server-postgres"],
                "env": {"DATABASE_URL": os.environ["DATABASE_URL"]}
            }
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

---

## Approche 2 : API Claude avec boucle agentique

### Installation

**Python :**

```bash
pip install anthropic
```

**TypeScript :**

```bash
npm install @anthropic-ai/sdk
```

### Methode 1 : Tool Runner (recommande, beta)

Le Tool Runner gere automatiquement la boucle agentique.

**Python :**

```python
import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """Obtenir la meteo pour un lieu.

    Args:
        location: Ville et pays, ex: Paris, France.
        unit: Unite de temperature, "celsius" ou "fahrenheit".
    """
    return f"22°C et ensoleille a {location}"

@beta_tool
def search_database(query: str, limit: int = 10) -> str:
    """Rechercher dans la base de donnees.

    Args:
        query: La requete de recherche.
        limit: Nombre max de resultats.
    """
    return f"3 resultats trouves pour '{query}'"

# Le tool runner gere tout automatiquement
runner = client.beta.messages.tool_runner(
    model="claude-opus-4-6",
    max_tokens=16000,
    tools=[get_weather, search_database],
    messages=[{"role": "user", "content": "Quel temps fait-il a Paris ?"}],
)

for message in runner:
    print(message)
```

**TypeScript :**

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = betaZodTool({
  name: "get_weather",
  description: "Obtenir la meteo pour un lieu",
  inputSchema: z.object({
    location: z.string().describe("Ville et pays"),
    unit: z.enum(["celsius", "fahrenheit"]).optional(),
  }),
  run: async (input) => `22°C et ensoleille a ${input.location}`,
});

const finalMessage = await client.beta.messages.toolRunner({
  model: "claude-opus-4-6",
  max_tokens: 16000,
  tools: [getWeather],
  messages: [{ role: "user", content: "Quel temps fait-il a Paris ?" }],
});

console.log(finalMessage.content);
```

### Methode 2 : Boucle agentique manuelle

Pour un controle total (logs, approbation humaine, logique conditionnelle) :

**Python :**

```python
import anthropic

client = anthropic.Anthropic()

# 1. Definir les outils avec un schema JSON
tools = [
    {
        "name": "get_weather",
        "description": "Obtenir la meteo pour un lieu",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "Ville, ex: Paris"
                }
            },
            "required": ["location"]
        }
    },
    {
        "name": "search_database",
        "description": "Rechercher dans la base de donnees",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "limit": {"type": "integer", "default": 10}
            },
            "required": ["query"]
        }
    }
]

# 2. Fonction d'execution des outils
def execute_tool(name: str, input_data: dict) -> str:
    if name == "get_weather":
        return f"22°C et ensoleille a {input_data['location']}"
    elif name == "search_database":
        return f"3 resultats pour '{input_data['query']}'"
    return "Outil inconnu"

# 3. Boucle agentique
messages = [{"role": "user", "content": "Quel temps fait-il a Paris ?"}]

while True:
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=16000,
        tools=tools,
        messages=messages
    )

    # Si Claude a fini, on sort de la boucle
    if response.stop_reason == "end_turn":
        break

    # Si un outil serveur a atteint sa limite d'iterations
    if response.stop_reason == "pause_turn":
        messages = [
            {"role": "user", "content": messages[0]["content"]},
            {"role": "assistant", "content": response.content},
        ]
        continue

    # Ajouter la reponse de l'assistant a l'historique
    messages.append({"role": "assistant", "content": response.content})

    # Executer chaque outil demande
    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            result = execute_tool(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result
            })

    # Renvoyer les resultats
    messages.append({"role": "user", "content": tool_results})

# Afficher la reponse finale
for block in response.content:
    if block.type == "text":
        print(block.text)
```

**TypeScript :**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "get_weather",
    description: "Obtenir la meteo pour un lieu",
    input_schema: {
      type: "object",
      properties: {
        location: { type: "string", description: "Ville" },
      },
      required: ["location"],
    },
  },
];

async function executeTool(name: string, input: any): Promise<string> {
  if (name === "get_weather") {
    return `22°C et ensoleille a ${input.location}`;
  }
  return "Outil inconnu";
}

const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "Quel temps fait-il a Paris ?" },
];

while (true) {
  const response = await client.messages.create({
    model: "claude-opus-4-6",
    max_tokens: 16000,
    tools,
    messages,
  });

  if (response.stop_reason === "end_turn") {
    for (const block of response.content) {
      if (block.type === "text") console.log(block.text);
    }
    break;
  }

  if (response.stop_reason === "pause_turn") {
    messages.push({ role: "assistant", content: response.content });
    continue;
  }

  const toolUseBlocks = response.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use"
  );

  messages.push({ role: "assistant", content: response.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

### Boucle agentique avec streaming

Pour afficher les tokens en temps reel pendant la boucle :

**TypeScript :**

```typescript
while (true) {
  const stream = client.messages.stream({
    model: "claude-opus-4-6",
    max_tokens: 64000,
    tools,
    messages,
  });

  stream.on("text", (delta) => process.stdout.write(delta));

  const message = await stream.finalMessage();

  if (message.stop_reason === "end_turn") break;

  const toolUseBlocks = message.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use"
  );

  messages.push({ role: "assistant", content: message.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

**Python :**

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=64000,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # Continuer la boucle avec response.stop_reason
```

---

## Definir des outils (Tools)

### Structure d'un outil (JSON Schema)

```json
{
  "name": "nom_de_loutil",
  "description": "Description detaillee de ce que fait l'outil",
  "input_schema": {
    "type": "object",
    "properties": {
      "param1": {
        "type": "string",
        "description": "Description du parametre"
      },
      "param2": {
        "type": "integer",
        "enum": [1, 5, 10],
        "description": "Nombre de resultats"
      }
    },
    "required": ["param1"]
  }
}
```

### Outils cote serveur (integres a l'API)

Claude fournit des outils qui s'executent sur l'infrastructure d'Anthropic :

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16000,
    tools=[
        {"type": "web_search_20260209", "name": "web_search"},
        {"type": "web_fetch_20260209", "name": "web_fetch"},
        {"type": "code_execution_20260120", "name": "code_execution"},
        {"type": "bash_20250124", "name": "bash"},
        {"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"},
    ],
    messages=[{"role": "user", "content": "Recherche les dernieres news IA"}]
)
```

### Controle du choix d'outil (Tool Choice)

```python
# Claude decide seul (defaut)
tool_choice = {"type": "auto"}

# Claude DOIT utiliser au moins un outil
tool_choice = {"type": "any"}

# Claude DOIT utiliser cet outil specifique
tool_choice = {"type": "tool", "name": "get_weather"}

# Claude ne peut PAS utiliser d'outils
tool_choice = {"type": "none"}
```

### Gestion des erreurs d'outils

```python
tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "Erreur : ville 'xyz' introuvable. Fournissez un nom valide.",
    "is_error": True
}
```

---

## Patterns multi-agents

### Sous-agents avec l'Agent SDK

**Python :**

```python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async for message in query(
    prompt="Utilise le reviewer pour analyser ce code puis le fixer pour corriger les bugs",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Edit", "Agent"],
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert en revue de code securite et qualite.",
                prompt="Analyse la qualite du code et suggere des ameliorations.",
                tools=["Read", "Glob", "Grep"]
            ),
            "bug-fixer": AgentDefinition(
                description="Expert en correction de bugs.",
                prompt="Trouve et corrige les bugs dans le code.",
                tools=["Read", "Edit", "Grep"]
            )
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

**TypeScript :**

```typescript
for await (const message of query({
  prompt: "Utilise le reviewer pour analyser ce code",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert en revue de code.",
        prompt: "Analyse la qualite du code.",
        tools: ["Read", "Glob", "Grep"],
      },
      "security-auditor": {
        description: "Expert en securite.",
        prompt: "Cherche les vulnerabilites.",
        tools: ["Read", "Grep"],
      },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

### System prompt personnalise

```python
async for message in query(
    prompt="Analyse ce code",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep"],
        system_prompt="""Tu es un reviewer senior specialise en :
1. Vulnerabilites de securite
2. Problemes de performance
3. Maintenabilite du code

Fournis toujours des numeros de ligne et des suggestions concretes."""
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

---

## Hooks et personnalisation

Les hooks permettent d'intercepter et de reagir aux actions de l'agent.

### Hook apres utilisation d'outil

**Python :**

```python
from datetime import datetime
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    with open('./audit.log', 'a') as f:
        f.write(f"{datetime.now()}: modifie {file_path}\n")
    return {}

async for message in query(
    prompt="Refactorise utils.py",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write"],
        permission_mode="acceptEdits",
        hooks={
            "PostToolUse": [
                HookMatcher(matcher="Edit|Write", hooks=[log_file_change])
            ]
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

### Evenements de hooks disponibles

| Evenement | Declencheur |
|-----------|------------|
| `PreToolUse` | Avant l'execution d'un outil |
| `PostToolUse` | Apres l'execution d'un outil |
| `PostToolUseFailure` | Apres un echec d'outil |
| `UserPromptSubmit` | Quand l'utilisateur envoie un message |
| `Stop` | Quand l'agent s'arrete |
| `SubagentStop` | Quand un sous-agent s'arrete |
| `SubagentStart` | Quand un sous-agent demarre |
| `PreCompact` | Avant la compaction du contexte |
| `Notification` | Notifications de l'agent |
| `PermissionRequest` | Demande de permission |

---

## Gestion des sessions

### Reprendre une session (Python)

```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

session_id = None

# Premier echange : capturer le session_id
async for message in query(
    prompt="Lis le module d'authentification",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"])
):
    if isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.data.get("session_id")

# Reprendre avec tout le contexte du premier echange
async for message in query(
    prompt="Maintenant trouve tous les endroits qui l'appellent",
    options=ClaudeAgentOptions(resume=session_id)
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

### Historique des sessions

```python
from claude_agent_sdk import list_sessions, get_session_messages

# Lister les sessions passees
sessions = list_sessions()
for session in sessions:
    print(f"{session.session_id}: {session.cwd}")

# Recuperer les messages d'une session
messages = get_session_messages(session_id="...")
for msg in messages:
    print(msg)
```

### Mutations de session

```python
from claude_agent_sdk import rename_session, tag_session

rename_session(session_id="...", title="Refactoring auth")
tag_session(session_id="...", tag="experiment-v2")
tag_session(session_id="...", tag=None)  # supprimer le tag
```

---

## Bonnes pratiques

### 1. Toujours specifier les outils autorises

```python
# Bien : liste explicite
options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])

# A eviter : tout autoriser sans reflexion
```

### 2. Limiter le nombre de tours

```python
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Bash"],
    max_turns=10  # Empeche les agents qui tournent en boucle
)
```

### 3. Definir un budget maximum

```python
options = ClaudeAgentOptions(
    max_budget_usd=5.0  # Limite de cout
)
```

### 4. Utiliser le bon mode de permission

Commencer par `"default"` et n'escalader que si necessaire.

### 5. Descriptions d'outils detaillees

Claude s'appuie enormement sur les descriptions pour decider quand et comment utiliser un outil. Soyez precis.

### 6. Gerer les erreurs

**Agent SDK :**

```python
from claude_agent_sdk import (
    query, ClaudeAgentOptions,
    CLINotFoundError, CLIConnectionError, ProcessError, ResultMessage
)

try:
    async for message in query(
        prompt="...",
        options=ClaudeAgentOptions(allowed_tools=["Read"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)
except CLINotFoundError:
    print("CLI introuvable. Installez avec : pip install claude-agent-sdk")
except CLIConnectionError as e:
    print(f"Erreur de connexion : {e}")
except ProcessError as e:
    print(f"Erreur de processus : {e}")
```

**API Claude :**

```python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.RateLimitError as e:
    print("Limite de debit atteinte, reessayez plus tard")
except anthropic.BadRequestError as e:
    print(f"Requete invalide : {e.message}")
except anthropic.APIStatusError as e:
    if e.status_code >= 500:
        print("Erreur serveur, reessayez")
```

### 7. Thinking adaptatif pour le raisonnement complexe

```python
# Pour les taches de raisonnement complexe
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    tools=tools,
    messages=messages
)
```

### 8. Streaming pour les reponses longues

Toujours utiliser le streaming pour eviter les timeouts sur les reponses longues :

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=64000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    final = stream.get_final_message()
```

---

## Resume des modeles recommandes

| Modele | ID | Usage |
|--------|-----|-------|
| Claude Opus 4.6 | `claude-opus-4-6` | Agents complexes, raisonnement avance |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Bon equilibre vitesse/intelligence |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Sous-agents rapides, taches simples |

---

## Liens utiles

- SDK Python : `pip install anthropic` / `pip install claude-agent-sdk`
- SDK TypeScript : `npm install @anthropic-ai/sdk` / `npm install @anthropic-ai/claude-agent-sdk`
- Documentation API : https://platform.claude.com/docs
- Serveurs MCP : https://github.com/modelcontextprotocol
