# Chatbot de WhatsApp com RAG

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white) ![LangChain](https://img.shields.io/badge/LangChain-1C3C3C) ![Groq](https://img.shields.io/badge/Groq-Llama%203.1-F55036) ![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

Chatbot que responde mensagens de WhatsApp usando IA. As respostas são baseadas no conteúdo de um PDF
(técnica **RAG — Retrieval-Augmented Generation**): o bot busca os trechos mais relevantes do documento
e usa esse contexto para responder, em vez de inventar informações.

## Como funciona

```
WhatsApp ──► WAHA (API de WhatsApp) ──webhook──► Flask (/chatbot/webhook/)
                                                    │
                                                    ├─ busca histórico da conversa (WAHA)
                                                    ├─ busca trechos relevantes no ChromaDB (RAG)
                                                    ├─ gera a resposta com Llama 3.1 (Groq)
                                                    └─ envia a resposta de volta (WAHA)
```

- Mensagens de **grupos** e de **status** são ignoradas.
- O bot mostra "digitando…" enquanto gera a resposta.
- O histórico recente da conversa entra no prompt, para manter o contexto.

## Tecnologias

| Ferramenta | Papel |
|---|---|
| Flask | API que recebe o webhook do WhatsApp |
| [WAHA](https://waha.devlike.pro/) | ponte com o WhatsApp (envio, recebimento, histórico) |
| LangChain | orquestração do prompt e da cadeia de documentos |
| Groq (`llama-3.1-8b-instant`) | modelo de linguagem |
| ChromaDB + embeddings Hugging Face | banco vetorial para a busca semântica |
| Docker Compose | sobe a API e o WAHA juntos |

## Estrutura

```
app.py            # webhook Flask: recebe a mensagem e orquestra a resposta
bot/ai_bot.py     # retriever (ChromaDB) + prompt + chamada ao modelo
rag/rag.py        # indexa o PDF: divide em trechos e grava os embeddings
rag/data/         # PDF usado como base de conhecimento
services/waha.py  # cliente HTTP da API do WAHA
```

## Como rodar

**Pré-requisitos:** Docker e Docker Compose, uma chave da [Groq](https://console.groq.com/) e um token do Hugging Face.

```bash
# 1. Configure as variáveis de ambiente
cp .env.example .env   # preencha GROQ_API_KEY e HUGGINGFACEHUB_API_TOKEN

# 2. Suba os containers
docker compose up -d --build

# 3. Indexe o PDF no banco vetorial (uma vez)
docker compose exec api python rag/rag.py
```

Depois, abra o painel do WAHA em `http://localhost:3000`, conecte o WhatsApp pelo QR Code e configure o
webhook para `http://api:5000/chatbot/webhook/`.

Para usar outro documento, troque o PDF em `rag/data/` e rode a indexação de novo.

## Próximos passos

- Testes automatizados do webhook com um fake do WAHA e do modelo.
- Tratamento de erros (modelo indisponível, mensagens de mídia).
- Remover arquivos `__pycache__` do repositório.
