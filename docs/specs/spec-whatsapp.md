# Especificação: WhatsApp Gateway

**Código:** SPEC-WHATSAPP  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que clientes finais conectem números de WhatsApp à plataforma, enviem e recebam mensagens de texto e mídia, e acompanhem o status de entrega e leitura das mensagens.

---

## 2. Contexto

O WhatsApp é o canal principal de comunicação da plataforma. A conexão é feita via Baileys (WhatsApp Web), gerando QR code para pareamento. Cada tenant pode ter uma ou mais instâncias ativas.

### Usuários envolvidos

- Cliente final (configura a conexão).
- Atendente (envia e recebe mensagens).
- Usuário final (contato do cliente, envia mensagem pelo WhatsApp).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-WA-001 | Apenas usuários autenticados do tenant podem iniciar/parar uma instância. | Segurança |
| RN-WA-002 | Cada instância gera um QR code único para pareamento. | Baileys |
| RN-WA-003 | Mensagens recebidas devem ser deduplicadas por `message_id`. | `whatsappCore.js` |
| RN-WA-004 | Status de mensagens: 0=Pending, 1=Sent, 2=Delivered, 3=Read. | `Message.js` |
| RN-WA-005 | Envio de mensagem manual requer instância conectada. | `chatController.js` |
| RN-WA-006 | Mídia enviada deve ser armazenada ou referenciada por URL. | `storageService.js` |
| RN-WA-007 | Fotos de perfil de contatos podem ser sincronizadas do WhatsApp. | `contactsController.js` |
| RN-WA-008 | Conexões inativas devem ser detectadas por health check. | `whatsapp_instances` |

---

## 4. Fluxo Esperado

### 4.1 Conectar WhatsApp

1. Cliente autenticado solicita início de sessão (`POST /api/v1/bot/start`).
2. Sistema inicializa Baileys para o tenant.
3. Sistema gera QR code e retorna base64.
4. Cliente escaneia QR code com o WhatsApp do celular.
5. Sistema atualiza status para `CONNECTED` e armazena número conectado.

### 4.2 Receber mensagem

1. Evento `messages.upsert` do Baileys dispara.
2. Sistema deduplica por `message_id`.
3. Sistema identifica ou cria contato.
4. Sistema salva mensagem no MongoDB (`chat_history`).
5. Sistema publica mensagem via WebSocket para atendentes.
6. Se houver gatilho de fluxo ou IA, executa automação.

### 4.3 Enviar mensagem manual

1. Atendente envia mensagem via chat (`POST /api/v1/chat/send`).
2. Sistema valida instância conectada e limite do plano.
3. Sistema envia mensagem via Baileys.
4. Sistema salva mensagem no MongoDB com status `Pending`.
5. Sistema atualiza status conforme ack recebido.

---

## 5. Critérios de Aceite

- [ ] QR code é gerado e exibido para o cliente.
- [ ] Após scan, status muda para `CONNECTED`.
- [ ] Mensagem recebida aparece na caixa de entrada em tempo real.
- [ ] Mensagem enviada manualmente é entregue ao destinatário.
- [ ] Status de mensagens atualiza (sent, delivered, read).
- [ ] Mensagens duplicadas não são salvas duas vezes.
- [ ] Mídia (imagem, documento) pode ser enviada e recebida.
- [ ] Instância desconectada não permite envio de mensagens.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/services/whatsappCore.js`
- `node-version/src/controllers/botController.js`
- `node-version/src/controllers/chatController.js`
- `node-version/src/models/nosql/Message.js`
- `node-version/src/models/sql/models.js` (WhatsAppInstance, Contact)
- `node-version/src/websockets/connectionManager.js`
- `node-version/src/workers/ackWorker.js`
- `node-version/src/workers/outgoingWorker.js`

### Infra
- Redis para deduplicação e controle de LIDs.
- RabbitMQ para fila de mensagens enviadas.
- MongoDB para histórico.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Iniciar sessão e gerar QR code | Média |
| Integração | Receber mensagem e salvar no MongoDB | Alta |
| Integração | Deduplicação de mensagem duplicada | Alta |
| Integração | Enviar mensagem manual com instância conectada | Alta |
| Integração | Rejeitar envio com instância desconectada | Média |
| Integração | Atualizar status de ack | Média |
| Integração | Sincronizar contatos do WhatsApp | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Baileys bloqueado ou instável | Alto | Monitorar, considerar Meta API oficial como alternativa futura |
| QR code expira antes do scan | Médio | Permitir regeneração |
| Múltiplas instâncias por tenant consumirem memória | Médio | Limitar instâncias por plano |
| Números banidos | Médio | Orientar cliente sobre uso responsável |

---

## 9. Dependências

- Baileys operacional.
- Redis disponível.
- RabbitMQ disponível.
- MongoDB disponível.
- Instância do WhatsApp do cliente disponível para scan.

---

## 10. A CONFIRMAR / PENDENTE

- Integração oficial com WhatsApp Business API (Meta).
- Chamadas de voz completas.
- Suporte a grupos como remetentes de mensagens.
- Política de retenção de mídia.
