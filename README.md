# 🤖 Agent Task Manager - Trello AI Agent

<div align="center">

![Python](https://img.shields.io/badge/Python-3.7%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Google ADK](https://img.shields.io/badge/Google%20ADK-Gemini%202.5%20Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Trello](https://img.shields.io/badge/Trello-API%20v1-0052CC?style=for-the-badge&logo=trello&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**Agente conversacional de IA que gerencia seu quadro Trello por linguagem natural — sem cliques, sem fricção.**

</div>

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Demonstração](#-demonstração)
- [Arquitetura](#-arquitetura)
- [Funcionalidades](#-funcionalidades)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração das Credenciais Trello](#-configuração-das-credenciais-trello)
- [Uso](#-uso)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Stack Tecnológica](#-stack-tecnológica)
- [Decisões de Design](#-decisões-de-design)
- [Próximos Passos](#-próximos-passos)

---

## 💡 Sobre o Projeto

O **Agent Task Manager** é um agente de IA conversacional construído sobre o **Google Agent Development Kit (ADK)** com o modelo **Gemini 2.5 Flash**. Ele atua como um assistente pessoal de produtividade que interage com o **Trello** por meio de linguagem natural, eliminando a necessidade de navegar manualmente pela interface do Kanban.

Em vez de abrir o browser, arrastar cards e preencher formulários, o usuário simplesmente conversa:

> *"Adiciona uma tarefa para revisar o relatório até sexta"*  
> *"Quais tarefas estão em andamento hoje?"*  
> *"Marca o deploy de produção como concluído"*

O agente interpreta a intenção, chama a ferramenta correta e confirma a ação — tudo em linguagem natural.

---

## 🎬 Demonstração

```
Agente: Olá! Hoje é 2026/06/03 09:15:22. Quais são as suas tarefas para hoje?

Usuário: Preciso revisar o PR do módulo de pagamentos e preparar a apresentação para o cliente

Agente: Perfeito! Criei dois cards na lista "To Do":
  ✅ "Revisar PR - módulo de pagamentos"
  ✅ "Preparar apresentação para o cliente"
  Tem mais alguma tarefa para adicionar?

Usuário: Não. Pode listar tudo que está em andamento?

Agente: Tarefas em andamento:
  📌 Refatoração do serviço de autenticação — vence em 05/06/2026
  📌 Setup do ambiente de staging

Usuário: Move a refatoração para concluído

Agente: ✅ 'Refatoração do serviço de autenticação': Em Andamento → Concluído
```

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────┐
│                    Usuário (Chat)                        │
└────────────────────────┬────────────────────────────────┘
                         │ linguagem natural
                         ▼
┌─────────────────────────────────────────────────────────┐
│              Google ADK — root_agent                     │
│                  Gemini 2.5 Flash                        │
│                                                          │
│  Raciocínio → Seleção de Tool → Chamada → Resposta       │
└──────┬────────────────┬──────────────────────────────────┘
       │                │
       ▼                ▼
┌─────────────┐  ┌───────────────────────────────────────┐
│  Temporal   │  │         Trello Tools                  │
│  Context    │  │                                       │
│             │  │  adicionar_tarefa()                   │
│  get_       │  │  listar_tarefas(status)               │
│  temporal_  │  │  mudar_status_tarefa(nome, status)    │
│  context()  │  │                                       │
└─────────────┘  └──────────────────┬────────────────────┘
                                    │
                                    ▼
                        ┌───────────────────────┐
                        │     py-trello SDK     │
                        │    Trello REST API    │
                        │    Board "DIO"        │
                        │  To Do / Em Andamento │
                        │      / Concluído      │
                        └───────────────────────┘
```

O agente segue um ciclo **Perceber → Raciocinar → Agir**:

1. **Perceber** — recebe a mensagem do usuário em linguagem natural
2. **Raciocinar** — o Gemini interpreta a intenção e decide qual tool invocar
3. **Agir** — executa a função Python correspondente via Trello API
4. **Responder** — sintetiza o resultado em linguagem natural

---

## ✨ Funcionalidades

| Funcionalidade | Descrição | Tool |
|---|---|---|
| ➕ Criar tarefa | Adiciona um card com nome, descrição e data de vencimento na lista "To Do" | `adicionar_tarefa()` |
| 📋 Listar tarefas | Lista todos os cards do board, com filtro por status | `listar_tarefas()` |
| 🔄 Mudar status | Move um card entre as listas (A Fazer → Em Andamento → Concluído) | `mudar_status_tarefa()` |
| 🕐 Contexto temporal | Fornece data e hora atual para o agente organizar as tarefas do dia | `get_temporal_context()` |

### Filtros de Status Suportados

| Parâmetro | Listas Mapeadas no Trello |
|---|---|
| `todas` | Todas as listas do board |
| `a fazer` | `A FAZER`, `TO DO`, `TODO` |
| `em andamento` | `EM ANDAMENTO`, `DOING` |
| `concluido` | `CONCLUÍDO`, `CONCLUIDO`, `DONE` |

---

## 🔧 Pré-requisitos

- Python **3.7+**
- Conta no **Trello** com um board chamado **`DIO`** contendo as listas:
  - `To Do` (ou `A Fazer`)
  - `Em Andamento` (ou `Doing`)
  - `Concluído` (ou `Done`)
- **Google API Key** com acesso ao Gemini (via [Google AI Studio](https://aistudio.google.com/))
- **Credenciais Trello** (API Key, Secret e Token — veja a seção de configuração)

---

## 🚀 Instalação

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/agent-task-manager.git
cd agent-task-manager
```

### 2. Crie e ative um ambiente virtual

```bash
python -m venv .venv

# Linux / macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

### 3. Instale as dependências

```bash
pip install -r requirements.txt
```

**`requirements.txt`:**
```
google-adk
py-trello
python-dotenv
```

> ⚠️ **Atenção:** O pacote `datetime` é parte da biblioteca padrão do Python e não precisa ser listado.

### 4. Configure as variáveis de ambiente

```bash
cp .env.exemplo .env
```

Edite o arquivo `.env` com suas credenciais:

```env
GOOGLE_GENAI_USE_VERTEXAI=0
GOOGLE_API_KEY=sua-chave-google-aqui
TRELLO_API_KEY=sua-api-key-trello
TRELLO_API_SECRET=seu-secret-trello
TRELLO_TOKEN=seu-token-trello
```

---

## 🔑 Configuração das Credenciais Trello

### Passo 1 — Criar um Power-Up

1. Acesse [https://trello.com/power-ups/admin/](https://trello.com/power-ups/admin/)
2. Clique em **"New"** e preencha os dados do aplicativo
3. Na página do Power-Up criado, copie a **API Key** e o **Secret**

### Passo 2 — Gerar o Token de Acesso

Monte a URL abaixo substituindo `SUA_API_KEY`:

```
https://trello.com/1/authorize?expiration=never&name=AgentTaskManager&scope=read,write&response_type=token&key=SUA_API_KEY
```

Acesse a URL no navegador, autorize o aplicativo e copie o **token** exibido na página de confirmação.

### Passo 3 — Preparar o Board

Crie um board no Trello chamado **`DIO`** com as seguintes listas (exatamente nesses nomes):

```
To Do  |  Em Andamento  |  Concluído
```

---

## 🖥️ Uso

### Iniciando o agente com Google ADK

```bash
adk run agents/agent04/agenttaskmanager
```

Ou usando a interface web do ADK:

```bash
adk web
```

### Exemplos de comandos

```
# Criar tarefas
"Cria uma tarefa chamada 'Deploy em produção' para amanhã"
"Adiciona: reunião com o cliente, descrição: alinhamento de requisitos, vence sexta"

# Listar tarefas
"Quais são minhas tarefas de hoje?"
"Lista tudo que está em andamento"
"Mostra as tarefas concluídas"

# Mudar status
"Muda 'Deploy em produção' para em andamento"
"Marca 'Reunião com o cliente' como concluída"
```

---

## 📁 Estrutura do Projeto

```
agents/agent04/
│
├── agenttaskmanager/
│   ├── __init__.py          # Exporta o módulo agent
│   └── agent.py             # Definição do agente e tools Trello
│
├── .env.exemplo             # Template de variáveis de ambiente
├── requirements.txt         # Dependências do projeto
└── README.md                # Este arquivo
```

### Descrição dos arquivos principais

| Arquivo | Responsabilidade |
|---|---|
| `agent.py` | Núcleo do projeto: define as 4 tools, instancia o `root_agent` com o Gemini e configura as instruções do agente |
| `__init__.py` | Expõe o módulo para o Google ADK descobrir e executar o agente |
| `.env.exemplo` | Template com as variáveis de ambiente necessárias (nunca commitar o `.env` real) |

---

## 🛠️ Stack Tecnológica

| Tecnologia | Versão | Papel |
|---|---|---|
| [Python](https://www.python.org/) | 3.7+ | Linguagem principal |
| [Google ADK](https://google.github.io/adk-docs/) | latest | Framework para construção de agentes de IA |
| [Gemini 2.5 Flash](https://deepmind.google/technologies/gemini/) | gemini-2.5-flash | Modelo de linguagem (reasoning + tool calling) |
| [py-trello](https://github.com/sarumont/py-trello) | latest | SDK Python para a API REST do Trello |
| [python-dotenv](https://github.com/theskumar/python-dotenv) | latest | Gerenciamento de variáveis de ambiente |

---

## 🎨 Decisões de Design

### Por que Google ADK + Gemini?

O Google ADK abstrai o ciclo de raciocínio do agente (ReAct pattern), gerenciando automaticamente o loop de *tool selection → execution → observation → response*. Isso permite focar na lógica de negócio (as tools do Trello) em vez de implementar o protocolo de orquestração manualmente.

### Por que tool calling em vez de prompts diretos?

As ferramentas expostas ao agente (`adicionar_tarefa`, `listar_tarefas`, `mudar_status_tarefa`, `get_temporal_context`) têm assinaturas tipadas em Python. O Gemini usa essa interface para decidir qual função invocar e com quais argumentos — tornando o sistema **determinístico nas ações** e **flexível na linguagem**.

### Mapeamento de status case-insensitive

A função `mudar_status_tarefa` normaliza o nome da lista com `.upper()` antes de comparar, e `listar_tarefas` usa um dicionário de mapeamento com variantes comuns (`DONE`, `DOING`, `TODO`). Isso garante resiliência a diferentes convenções de nomeação no Trello.

### Contexto temporal explícito

A tool `get_temporal_context()` injeta data e hora no raciocínio do agente. Sem isso, o LLM não teria referência temporal para perguntas como "quais tarefas vencem hoje?" ou para formatar datas de vencimento corretamente.

---

## 🚧 Próximos Passos

- [ ] **Suporte a múltiplos boards** — parametrizar o nome do board em vez de fixar `"DIO"`
- [ ] **Remoção de tarefas** — implementar `remover_tarefa()` com soft delete (arquivamento)
- [ ] **Edição de tarefas** — permitir atualizar nome, descrição e data de vencimento
- [ ] **Notificações de vencimento** — alertar sobre cards próximos do prazo
- [ ] **Suporte a checklists** — criar e marcar itens dentro de um card
- [ ] **Testes automatizados** — unit tests com mock da API Trello
- [ ] **Deploy como serviço** — containerizar com Docker para execução contínua

---

## 🔒 Segurança

> ⚠️ **Nunca commite o arquivo `.env`** com suas credenciais reais.  
> O arquivo `.env.exemplo` serve apenas como template e **não contém dados sensíveis**.

Adicione ao `.gitignore`:

```
.env
*.env
```

---

## 📄 Licença

Distribuído sob a licença MIT. Veja `LICENSE` para mais informações.

---

## 🙋 Autor: Ricardo Medeiros

Desenvolvido como desafio prático de construção de agentes de IA com integração a ferramentas externas.

> *"A melhor interface é aquela que desaparece — o usuário só pensa na tarefa, não na ferramenta."*

---

<div align="center">
  <sub>Feito com ☕ e Google ADK · <a href="https://github.com/seu-usuario/agent-task-manager">GitHub</a></sub>
</div>
