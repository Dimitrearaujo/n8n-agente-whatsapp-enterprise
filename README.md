# 🚀 Agente WhatsApp Enterprise — Arquitetura de Produção

> O template mais completo da coleção: agente de IA para WhatsApp com **dual API** (Meta Oficial + Evolution API), gestão de leads, transcrição de áudio, RAG, memória persistente em PostgreSQL, buffer em Redis, escalonamento humano e integração com **MCP Tools** — pronto para produção em escala.

[![n8n](https://img.shields.io/badge/n8n-workflow-orange?logo=n8n)](https://n8n.io)
[![OpenRouter](https://img.shields.io/badge/OpenRouter-multi--LLM-6366f1)](https://openrouter.ai)
[![Supabase](https://img.shields.io/badge/Supabase-banco_e_vetores-3ECF8E?logo=supabase)](https://supabase.com)
[![Redis](https://img.shields.io/badge/Redis-buffer-DC382D?logo=redis)](https://redis.io)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-memória-336791?logo=postgresql)](https://www.postgresql.org)
[![Evolution API](https://img.shields.io/badge/Evolution_API-WhatsApp-25D366?logo=whatsapp)](https://evolution-api.com)

---

## 📌 O que este sistema faz

Um agente de atendimento para WhatsApp construído com arquitetura de produção real, cobrindo todos os cenários de uma operação de alto volume:

- Recebe mensagens via **duas APIs** WhatsApp simultaneamente (oficial Meta + Evolution API)
- **Gerencia leads** automaticamente: cria, busca, atualiza e deleta no Supabase
- **Transcreve áudios** em texto antes de processar
- **Agrupa mensagens** em janela de tempo com Redis (evita responder mensagem por mensagem)
- Busca contexto na **base de conhecimento** via RAG (Supabase + Gemini Embeddings)
- Mantém **memória persistente** de cada conversa no PostgreSQL
- Detecta quando escalar para **atendimento humano** e notifica automaticamente
- Mostra **"digitando..."** enquanto processa
- **Divide respostas longas** em múltiplas mensagens
- Integra ferramentas externas via **MCP (Model Context Protocol)**

---

## 🗂️ Workflows incluídos

| Arquivo | Função | Nós |
|---|---|---|
| `01-agente-ia.json` | **Core** — lógica completa do agente | 92 |
| `02-atendimento-humano.json` | Escalonamento — notifica humano e adiciona tags | 13 |
| `03-mcp-tools.json` | Servidor MCP com ferramentas customizadas | 11 |
| `04-rag-pipeline.json` | Pipeline de ingestão de documentos para RAG | 26 |

**Total: 142 nós**

---

## 🏗️ Arquitetura completa

```
WhatsApp (Meta Oficial)   WhatsApp (Evolution API)
         │                        │
         └──────────┬─────────────┘
                    ▼
          ┌─────────────────┐
          │   Normalização  │  ← unifica formato das duas APIs
          │   de Dados      │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  Gestão de Lead │  ← busca/cria/atualiza no Supabase
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  Tipo de msg?   │  ← texto / áudio / arquivo
          │  (texto/áudio)  │
          └────────┬────────┘
                   │ Áudio → Transcrição (Gemini)
                   │
          ┌────────▼────────┐
          │  Redis Buffer   │  ← agrupa msgs em janela de tempo
          │  (anti-spam)    │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  Escalar        │  ← atendimento humano ativo?
          │  Humano?        │
          └────┬────────┬───┘
               │ Não    │ Sim
               ▼        ▼
        ┌──────────┐ ┌──────────────────┐
        │ AI Agent │ │02-atend-humano   │
        │          │ │(notifica equipe) │
        │ OpenRouter│ └──────────────────┘
        │ (multi-LLM)
        │          │
        │ Ferramentas:
        │ ├── RAG (Supabase pgvector)
        │ ├── MCP Tools (03-mcp-tools)
        │ └── Calculadora
        └────┬─────┘
             │
    ┌────────▼────────┐
    │ Quebrar msgs    │  ← divide resposta longa
    │ + "Digitando..."│  ← indicador de digitação
    └────────┬────────┘
             │
    ┌────────▼────────┐
    │  Enviar resposta│  ← pelo canal correto (Oficial ou Evolution)
    └─────────────────┘

─────── Pipeline paralelo ───────

Google Drive (arquivo novo/atualizado)
         │
    04-rag-pipeline
         ├── Baixar arquivo
         ├── Extrair texto
         ├── Chunking (text splitter)
         ├── Embeddings (Gemini)
         └── Upsert → Supabase Vector Store
```

---

## 🔧 Integrações

| Serviço | Uso |
|---|---|
| **WhatsApp Business API (Meta)** | Canal oficial de entrada/saída |
| **Evolution API** | Canal alternativo (WhatsApp não-oficial) |
| **OpenRouter** | LLM — acessa qualquer modelo (GPT-4o, Claude, Gemini, etc.) |
| **Google Gemini Embeddings** | Vetorização de documentos para RAG |
| **Supabase** | Leads (banco relacional) + Vector Store (RAG) |
| **PostgreSQL** | Memória persistente das conversas |
| **Redis** | Buffer de mensagens (janela de tempo) |
| **Google Drive** | Fonte de documentos para o RAG |
| **MCP Protocol** | Ferramentas externas customizáveis |

---

## ✅ Pré-requisitos

- [ ] n8n v1.0+ com nodes Evolution API instalados
- [ ] Conta OpenRouter com API Key
- [ ] Google AI Studio API Key (Gemini Embeddings)
- [ ] Projeto Supabase com `pgvector` habilitado
- [ ] Banco PostgreSQL acessível pelo n8n
- [ ] Redis (local ou cloud — ex: Upstash)
- [ ] WhatsApp Business API (Meta) **e/ou** Evolution API instalado

---

## ⚙️ Configuração

### 1. Credenciais necessárias

| Nó | Credencial |
|---|---|
| `OpenRouter Chat Model` | OpenRouter API Key |
| `Embeddings Google Gemini` | Google AI Studio API Key |
| `MemoriaPostgres` | PostgreSQL connection string |
| `BuscarBuffer`, `DeletarBuffer` | Redis URL |
| `BuscarLead`, `CriarLead`, etc. | Supabase URL + Service Key |
| `EnviarResposta` (oficial) | WhatsApp Business Token |
| `EnviarResposta1` (Evolution) | Evolution API URL + Key |
| `ArquivoCriado` (RAG) | Google Drive OAuth2 |
| `Supabase Vector Store` (RAG) | Supabase URL + Service Key |

### 2. Configurar Supabase

```sql
-- Habilitar pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- Tabela de leads
CREATE TABLE leads (
  id BIGSERIAL PRIMARY KEY,
  telefone TEXT UNIQUE,
  nome TEXT,
  status TEXT DEFAULT 'ativo',
  atendimento_humano BOOLEAN DEFAULT false,
  criado_em TIMESTAMPTZ DEFAULT NOW()
);

-- Tabela para RAG
CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(768)  -- Gemini usa 768 dimensões
);

-- Index para busca vetorial
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops);
```

### 3. Configurar Redis

No n8n, adicione a credencial Redis apontando para seu servidor. Recomendado: [Upstash](https://upstash.com) para hosting gratuito.

O buffer usa como chave o número de telefone do cliente e acumula mensagens por X segundos antes de processar — evita que o agente responda cada mensagem separadamente.

### 4. Configurar o RAG (04-rag-pipeline)

1. Ative o workflow `04-rag-pipeline`
2. Configure a pasta do Google Drive que ele deve monitorar
3. Adicione documentos à pasta (PDF, DOCX, TXT)
4. O pipeline ingere automaticamente quando um arquivo é criado ou atualizado

### 5. Configurar MCP Tools (03-mcp-tools)

O workflow `03-mcp-tools` expõe ferramentas customizadas para o agente via MCP Protocol. Edite os nós `Tool 1` a `Tool 4` com as ferramentas do seu negócio (ex: consultar estoque, verificar pedido, buscar cliente).

### 6. Configurar o Agente Principal (01-agente-ia)

No nó **AI Agent**, edite o `System Message` com:
- Identidade e tom da empresa
- Produtos/serviços disponíveis
- Regras de escalonamento humano
- Horário de atendimento

### 7. Ordem de importação

```
04 → 03 → 02 → 01
```

---

## 📊 Métricas do sistema

| Métrica | Valor |
|---|---|
| Total de workflows | 4 |
| Total de nós | 142 |
| Nós no agente principal | 92 |
| APIs WhatsApp suportadas | 2 (Oficial + Evolution) |
| LLM | OpenRouter (qualquer modelo) |
| Memória | PostgreSQL persistente |
| Buffer anti-spam | Redis |
| Busca semântica | Supabase pgvector + RAG |
| Integração MCP | Sim (4 tools customizáveis) |

---

## 💡 Dicas de personalização

- **Trocar o modelo**: no OpenRouter, basta mudar o `model` no nó — acessa GPT-4o, Claude 3.5, Gemini, Llama e outros com a mesma credencial
- **Janela do buffer**: ajuste o tempo de espera no Redis para controlar quanto tempo o agente aguarda novas mensagens antes de responder
- **MCP Tools**: adicione quantas ferramentas precisar — o agente decide automaticamente quando e como usá-las
- **Escalonamento**: o agente detecta frases de gatilho (configuráveis no prompt) e ativa o workflow de atendimento humano
- **Multicanal**: o mesmo agente atende pela API oficial e Evolution simultaneamente, com lógica de resposta pelo canal correto

---

[← Voltar ao índice](../README.md)

---

<details>
<summary>🇺🇸 English</summary>

# 🚀 Enterprise WhatsApp Agent — Production Architecture

> The most complete template in the collection: an AI agent for WhatsApp with **dual API** (Official Meta + Evolution API), lead management, audio transcription, RAG, persistent memory in PostgreSQL, Redis buffer, human escalation and **MCP Tools** integration — production-ready at scale.

## 📌 What this system does

A WhatsApp customer service agent built with real production architecture, covering every scenario of a high-volume operation:

- Receives messages via **two WhatsApp APIs** simultaneously (official Meta + Evolution API)
- **Manages leads** automatically: create, search, update and delete in Supabase
- **Transcribes audio** to text before processing
- **Groups messages** in a time window with Redis (avoids responding message by message)
- Fetches context from the **knowledge base** via RAG (Supabase + Gemini Embeddings)
- Maintains **persistent memory** of each conversation in PostgreSQL
- Detects when to escalate to **human support** and notifies automatically
- Shows **"typing..."** while processing
- **Splits long responses** into multiple messages
- Integrates external tools via **MCP (Model Context Protocol)**

## 🗂️ Included workflows

| File | Function | Nodes |
|---|---|---|
| `01-agente-ia.json` | **Core** — complete agent logic | 92 |
| `02-atendimento-humano.json` | Escalation — notifies human agent and adds tags | 13 |
| `03-mcp-tools.json` | MCP server with custom tools | 11 |
| `04-rag-pipeline.json` | Document ingestion pipeline for RAG | 26 |

**Total: 142 nodes**

## 🔧 Integrations

| Service | Use |
|---|---|
| **WhatsApp Business API (Meta)** | Official input/output channel |
| **Evolution API** | Alternative channel (unofficial WhatsApp) |
| **OpenRouter** | LLM — accesses any model (GPT-4o, Claude, Gemini, etc.) |
| **Google Gemini Embeddings** | Document vectorization for RAG |
| **Supabase** | Leads (relational DB) + Vector Store (RAG) |
| **PostgreSQL** | Persistent conversation memory |
| **Redis** | Message buffer (time window) |
| **Google Drive** | Source of documents for RAG |
| **MCP Protocol** | Customizable external tools |

</details>