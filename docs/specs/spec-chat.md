# Especificação: Chat / Inbox

**Código:** SPEC-CHAT  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Fornecer uma interface unificada para atendentes visualizarem conversas ativas, acessarem histórico, responderem mensagens em tempo real e transferirem atendimentos entre agentes.

---

## 2. Contexto

O Inbox é o principal ponto de trabalho dos atendentes. Ele consome mensagens do MongoDB e se comunica com o backend via WebSocket para atualizações em tempo real. Cada conversa é identificada por tenant, sessão e telefone do contato.

### Usuários envolvidos

- Atendente/agente.
- Cliente final (admin do tenant, visualiza métricas).
- Usuário final (interage via WhatsApp, não acessa o Inbox).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-CHAT-001 | Atendente só visualiza conversas do próprio tenant. | Multi-tenancy |
| RN-CHAT-002 | Conversa ativa é identificada por `tenant_id` + `session_name` + `contact_phone`. | `Message.js` |
| RN-CHAT-003 | Atendente pode enviar mensagem manualmente se houver instância conectada. | `chatController.js` |
| RN-CHAT-004 | Atendente pode transferir conversa para outro atendente. | `chatController.js` |
| RN-CHAT-005 | Mensagens enviadas por atendente têm `source` diferente de mensagens do bot. | `Message.js` |
| RN-CHAT-006 | Atualizações de novas mensagens devem chegar em tempo real via WebSocket. | `connectionManager.js` |
| RN-CHAT-007 | Histórico de mensagens deve ser carregado por demanda (paginação). | Performance |
| RN-CHAT-008 | Atendente não pode exceder `max_concurrent_chats` simultâneos. | `users` |

---

## 4. Fluxo Esperado

### 4.1 Listar conversas

1. Atendente acessa a tela de Inbox.
2. Sistema busca sessões ativas do tenant no MongoDB.
3. Sistema retorna lista com última mensagem, contato e status.

### 4.2 Abrir conversa

1. Atendente seleciona conversa.
2. Sistema carrega histórico de mensagens (`tenant_id` + `session_name` + `contact_phone`).
3. Sistema marca mensagens como visualizadas.

### 4.3 Enviar mensagem manual

1. Atendente digita mensagem e envia.
2. Sistema valida instância conectada e permissão.
3. Sistema envia via WhatsApp Gateway.
4. Sistema salva mensagem no histórico.
5. Sistema atualiza UI em tempo real.

### 4.4 Transferir conversa

1. Atendente seleciona outro agente para transferir.
2. Sistema atualiza atribuição da conversa.
3. Sistema notifica novo atendente via WebSocket.

---

## 5. Critérios de Aceite

- [ ] Inbox lista apenas conversas do tenant do atendente.
- [ ] Conversa aberta exibe histórico completo de mensagens.
- [ ] Nova mensagem recebida aparece em tempo real no Inbox.
- [ ] Atendente envia mensagem manual com sucesso.
- [ ] Transferência de conversa funciona e notifica novo atendente.
- [ ] Atendente respeita limite de chats simultâneos.
- [ ] Mensagens do bot e do atendente são visualmente diferenciadas.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/chatController.js`
- `node-version/src/models/nosql/Message.js`
- `node-version/src/websockets/connectionManager.js`
- `node-version/src/middlewares/authMiddleware.js`
- `node-version/src/middlewares/tenancyMiddleware.js`

### Frontend
- `chatbot/src/Controller/HomeController.php`
- `chatbot/src/Controller/ApiOmniController.php`
- `chatbot/views/` (templates de chat).

### Banco
- MongoDB: `chat_history`.
- PostgreSQL: `users`, `whatsapp_instances`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Listar conversas do tenant | Alta |
| Integração | Carregar histórico de conversa | Alta |
| Integração | WebSocket recebe nova mensagem | Alta |
| Integração | Enviar mensagem manual | Alta |
| Integração | Transferir conversa | Média |
| Integração | Bloquear atendente acima do limite de chats | Média |
| Integração | Impedir acesso a conversa de outro tenant | Alta |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| WebSocket desconectar sem reconexão | Médio | Implementar heartbeat e reconexão no cliente |
| Histórico muito grande carregado de uma vez | Médio | Paginar carregamento |
| Concorrência na atribuição de conversa | Médio | Usar lock ou transação |

---

## 9. Dependências

- WhatsApp Gateway operacional.
- WebSocket server ativo.
- MongoDB com índice composto em `chat_history`.

---

## 10. A CONFIRMAR / PENDENTE

- Notas internas em conversas.
- Filtros avançados (status, atendente, tag).
- Mensagens rápidas/canned responses.
- Indicador de digitação.
