# Especificação: Inteligência Artificial

**Código:** SPEC-AI  
**Versão:** 1.0  
**Status:** MVP / Parcial  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que clientes finais configurem modelos de IA para responder automaticamente a perguntas dos usuários finais, com base em conhecimento específico do negócio (RAG) e modelos locais ou de terceiros.

---

## 2. Contexto

O módulo de IA permite configurar provider, modelo, prompt e parâmetros por tenant. A integração padrão é com Ollama local (llama3.2), mas campos existem para provedores externos. A base de conhecimento (RAG) é armazenada no MongoDB.

### Usuários envolvidos

- Cliente final (configura IA e conhecimento).
- Usuário final (recebe respostas automáticas).
- Atendente (recebe conversa quando IA não resolve).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-AI-001 | Configuração de IA é por tenant. | `ai_configs` (`tenant_id`) |
| RN-AI-002 | Provider padrão é Ollama (`llama3.2`). | `AiConfig` |
| RN-AI-003 | `api_key` de provedores externos deve ser armazenada de forma segura. | Segurança — PENDENTE |
| RN-AI-004 | Respostas da IA só são enviadas se IA estiver ativa para o tenant. | `is_active` |
| RN-AI-005 | Quando RAG está habilitado, a IA deve usar base de conhecimento do tenant. | `is_rag_enabled` |
| RN-AI-006 | IA pode ser desabilitada por tenant a qualquer momento. | `is_active` |
| RN-AI-007 | Limite de tokens e temperatura são configuráveis. | `AiConfig` |
| RN-AI-008 | Documentos de conhecimento são isolados por tenant. | `Knowledge` (`tenant_id`) |

---

## 4. Fluxo Esperado

### 4.1 Configurar IA

1. Cliente final acessa configurações de IA.
2. Seleciona provider (Ollama/OpenAI/Gemini/Anthropic), modelo, temperatura, max tokens.
3. Define system prompt.
4. Habilita/desabilita RAG.
5. Sistema salva configuração em `ai_configs`.

### 4.2 Ingerir conhecimento

1. Cliente faz upload de documento ou texto.
2. Sistema processa e armazena na coleção `knowledge`.
3. Sistema associa ao tenant.

### 4.3 Responder com IA

1. Mensagem recebida de usuário final.
2. Sistema verifica se IA está ativa e se há fluxo ativo para o gatilho.
3. Se IA deve responder, sistema monta prompt com contexto (RAG se habilitado).
4. Sistema chama Ollama (ou provider configurado).
5. Sistema envia resposta ao usuário via WhatsApp.
6. Sistema salva mensagem no histórico.

---

## 5. Critérios de Aceite

- [ ] Configurar IA por tenant com sucesso.
- [ ] Ingerir documento de conhecimento.
- [ ] IA responde mensagem com base no conhecimento ingerido.
- [ ] IA desabilitada não responde automaticamente.
- [ ] Resposta da IA respeita limite de tokens configurado.
- [ ] Ollama llama3.2 está disponível e funcional no ambiente Docker.
- [ ] Provedores alternativos (OpenAI/Gemini/Anthropic) funcionam quando configurados — A CONFIRMAR.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/aiController.js`
- `node-version/src/services/ai/llamaService.js`
- `node-version/src/services/ai/ragService.js`
- `node-version/src/models/sql/models.js` (`AiConfig`)
- `node-version/src/models/nosql/Knowledge.js`
- `node-version/src/services/whatsappCore.js`

### Infra
- Ollama container (`saas_ollama` no Docker Compose).

### Banco
- PostgreSQL: `ai_configs`.
- MongoDB: `knowledge`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Configurar IA por tenant | Alta |
| Integração | Ingerir conhecimento | Alta |
| Integração | Obter resposta da IA com RAG | Média |
| Integração | Obter resposta da IA sem RAG | Média |
| Integração | IA desabilitada não responde | Média |
| Integração | Ollama disponível no ambiente | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| `api_key` em texto plano | Alto | Criptografar em repouso |
| Ollama indisponível | Médio | Verificar container e fallback para provider externo |
| Respostas incorretas da IA | Médio | Permitir transbordo humano fácil |
| Custo de API externa | Médio | Controlar limites de tokens e uso |

---

## 9. Dependências

- Ollama operacional.
- WhatsApp Gateway para envio de respostas.
- MongoDB para `knowledge`.
- PostgreSQL para `ai_configs`.

---

## 10. A CONFIRMAR / PENDENTE

- Integração real com OpenAI, Gemini, Anthropic.
- Criptografia de `api_key`.
- Embeddings avançados e chunking de documentos.
- Fine-tuning de modelos.
