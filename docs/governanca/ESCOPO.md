# ESCOPO.md — Escopo do Projeto

**Projeto:** SaaS Chatbot / Chatbot_Mauricio  
**Atualizado em:** 2026-07-02  
**Fase:** MVP / Refatoração (backend migrado para Node.js, frontend PHP, legados preservados)

---

## 1. Objetivo do Projeto

Construir uma plataforma SaaS omnichannel de chatbots e atendimento humano, com foco em WhatsApp, que permita a pequenas e médias empresas — e suas revendas — automatizar atendimentos, gerenciar conversas em tempo real, segmentar contatos, faturar por planos e operar com marca própria (white-label).

---

## 2. Problema que o Sistema Resolve

Empresas e revendas precisam centralizar o atendimento ao cliente via WhatsApp em uma única plataforma, sem depender de múltiplos celulares ou contas pessoais. O sistema resolve:

- Dificuldade de escalar atendimento humano.
- Falta de histórico centralizado de conversas.
- Ausência de automação para perguntas frequentes.
- Necessidade de controle de planos e limites de uso.
- Demanda de revendas por white-label para seus próprios clientes.

---

## 3. Público-Alvo / Usuários

| Perfil | Descrição | Permissões |
|---|---|---|
| Admin Master | Dono da plataforma SaaS | Gerenciar revendas, tenants, planos, faturas, auditoria, configurações globais |
| Revenda | Parceiro white-label | Criar e gerenciar sub-tenants (clientes finais), definir planos, marca própria |
| Cliente Final | Empresa que usa o chatbot | Configurar bot, contatos, fluxos, campanhas, atendentes |
| Atendente/Agente | Usuário operacional do cliente final | Responder conversas, transferir chats, adicionar notas |
| Usuário Final | Contato do cliente (pessoa que envia mensagem) | Interagir com bot ou atendente via WhatsApp |

---

## 4. Escopo Funcional

### 4.1 Módulo Autenticação e Identidade

- [x] Cadastro de usuário com criação automática de tenant.
- [x] Login com JWT.
- [x] Perfis de usuário (admin, agente, superadmin).
- [x] Hierarquia de revendas e sub-tenants.
- [x] Controle de acesso baseado em roles (RBAC).
- [ ] MFA (autenticação de dois fatores) — A CONFIRMAR.
- [ ] Recuperação de senha por email — A CONFIRMAR.

**Detalhamento: Cadastro de usuário**
- **Descrição:** permite criar conta com email, senha e nome, gerando um tenant_id exclusivo.
- **Usuário beneficiado:** Cliente Final.
- **Entrada:** email, senha, nome completo.
- **Processamento:** validação de email, hash de senha, criação de usuário e tenant, emissão de JWT.
- **Saída:** token JWT e dados do usuário.
- **Critérios de aceite:** email único, senha mínima, tenant_id gerado, JWT válido.
- **Status:** confirmado.

### 4.2 Módulo WhatsApp Gateway

- [x] Conectar instância WhatsApp via QR code.
- [x] Inicializar, parar e reiniciar sessão.
- [x] Receber mensagens de texto e mídia.
- [x] Enviar mensagens de texto e mídia.
- [x] Deduplicação de mensagens.
- [x] Status de mensagens (pending, sent, delivered, read).
- [ ] Integração oficial com WhatsApp Business API (Meta) — FORA DO ESCOPO ATUAL.
- [ ] Chamadas de voz completas — PARCIAL (CallLog existe, funcionalidade a confirmar).

**Detalhamento: Conexão WhatsApp via QR**
- **Descrição:** permite ao cliente conectar um número de WhatsApp escaneando QR code.
- **Usuário beneficiado:** Cliente Final.
- **Entrada:** solicitação de início de sessão (autenticado).
- **Processamento:** inicializa Baileys, gera QR, persiste estado em `whatsapp_instances`.
- **Saída:** QR code base64 e status da conexão.
- **Critérios de aceite:** QR gerado, sessão persistida, status atualizado para CONNECTED após scan.
- **Status:** confirmado.

### 4.3 Módulo Chat / Inbox

- [x] Caixa de entrada com conversas ativas.
- [x] Histórico de mensagens por conversa.
- [x] Envio manual de mensagens por atendentes.
- [x] Atualização em tempo real via WebSocket.
- [x] Transferência de conversa entre atendentes.
- [ ] Notas internas em conversas — A CONFIRMAR.
- [ ] Filtros avançados (por status, atendente, tag) — A CONFIRMAR.

### 4.4 Módulo Contatos

- [x] CRUD de contatos.
- [x] Importação de contatos em lote (CSV/JSON).
- [x] Tags para segmentação.
- [x] Sincronização de contatos do WhatsApp (scan Baileys).
- [x] Fotos de perfil de contatos.
- [ ] Blacklist com regras automáticas — A CONFIRMAR.

### 4.5 Módulo Flow Engine (Automação)

- [x] Criação de fluxos com nodes e edges.
- [x] Palavras-chave de gatilho.
- [x] Execução de fluxos por mensagem recebida.
- [x] Sessões de fluxo por contato.
- [x] Transbordo para atendimento humano.
- [ ] Editor visual de fluxos no frontend — A CONFIRMAR.
- [ ] Variáveis e condições avançadas — A CONFIRMAR.

### 4.6 Módulo Inteligência Artificial

- [x] Configuração de modelo de IA por tenant.
- [x] Integração com Ollama (llama3.2) local.
- [x] Ingestão de conhecimento para RAG.
- [x] Respostas automáticas com IA.
- [ ] Integração com OpenAI, Gemini, Anthropic — A CONFIRMAR (campos existem, código não verificado).
- [ ] Fine-tuning ou embeddings avançados — FORA DO ESCOPO ATUAL.

### 4.7 Módulo Billing

- [x] Planos com limites de bots, agentes e mensagens.
- [x] Assinaturas por tenant.
- [x] Faturas e transações.
- [x] Heartbeat de billing e notificações.
- [ ] Gateway de pagamento real — A CONFIRMAR.
- [ ] Cobrança recorrente automática — A CONFIRMAR.

### 4.8 Módulo Campanhas

- [x] Criação de campanhas com template de mensagem.
- [x] Importação de contatos para campanha.
- [x] Agendamento de disparo.
- [x] Delays aleatórios anti-ban.
- [x] Métricas de envio, entrega, resposta e erro.
- [ ] Segmentação avançada de contatos — A CONFIRMAR.
- [ ] A/B testing — A CONFIRMAR.

### 4.9 Módulo Admin e Revendas

- [x] Painel administrativo com roles.
- [x] Auditoria de ações administrativas.
- [x] CRUD de revendas.
- [x] Gestão de sub-tenants por revenda.
- [x] Bloqueio emergencial de tenants.
- [ ] Dashboards financeiros detalhados — A CONFIRMAR.

### 4.10 Módulo Frontend PHP

- [x] Home com status do bot e lista de contatos.
- [x] Autenticação via API Node.
- [x] Proxy JSON para consumo da API.
- [x] Consola do programador.
- [ ] Interface de chat completa — A CONFIRMAR (controllers existem, mas UI não verificada).
- [ ] Dashboards e relatórios — A CONFIRMAR.

---

## 5. Escopo Não Funcional

| Requisito | Descrição | Status |
|---|---|---|
| Segurança | Autenticação JWT, isolamento por tenant, validação de entrada, rate-limit | Parcial |
| Performance | WebSocket em tempo real, cache Redis, pool de conexões PostgreSQL | Parcial |
| Disponibilidade | Docker Compose, graceful shutdown, workers RabbitMQ | Parcial |
| Escalabilidade | Arquitetura monolith modular; escalabilidade horizontal do backend a confirmar | A CONFIRMAR |
| Usabilidade | Frontend PHP server-side, acessível via navegador | Parcial |
| Acessibilidade | NÃO IDENTIFICADO | A CONFIRMAR |
| Observabilidade | Logs com Pino, audit logs, logs de erro | Parcial |
| Manutenibilidade | Estrutura modular em `src/`, documentação em evolução | Parcial |
| Compatibilidade | WhatsApp Web via Baileys; compatibilidade com Meta API a confirmar | A CONFIRMAR |
| LGPD | NÃO IDENTIFICADO política formal | PENDENTE |

---

## 6. Fora do Escopo

- Aplicativo mobile nativo (iOS/Android) — mencionado como evolução futura.
- Integração oficial com WhatsApp Business API (Meta) — a solução atual usa Baileys.
- E-commerce, catálogo de produtos e pagamentos dentro do chat.
- Integração com canais além do WhatsApp (Instagram, Telegram, Messenger, email) — FORA DO ESCOPO ATUAL.
- Fine-tuning de modelos de IA.
- Business Intelligence avançado e machine learning preditivo.
- Multi-idioma automático.

---

## 7. Regras de Negócio

| Código | Regra | Módulo | Status |
|---|---|---|---|
| RN-001 | O usuário precisa estar autenticado com JWT para acessar a API | Auth | Confirmado |
| RN-002 | Cada usuário pertence a um `tenant_id` e só acessa dados do próprio tenant | Tenancy | Confirmado |
| RN-003 | Revendas podem criar e gerenciar sub-tenants dentro do limite contratado | Revenda | Confirmado |
| RN-004 | Apenas superadmin pode acessar rotas `/api/v1/sadmin/*` | Admin | Confirmado |
| RN-005 | O envio de mensagens depende de uma instância WhatsApp conectada | Bot | Confirmado |
| RN-006 | Mensagens recebidas são salvas no MongoDB e retransmitidas via WebSocket | Chat | Confirmado |
| RN-007 | Fluxos ativos com palavras-chave de gatilho respondem automaticamente | Flow | Confirmado |
| RN-008 | Quando o fluxo não resolve, a conversa pode ser transferida para atendimento humano | Flow | Confirmado |
| RN-009 | Campanhas respeitam janela de sono (`sleep_start` e `sleep_end`) | Campaign | Confirmado |
| RN-010 | Campanhas usam delays aleatórios entre mensagens para evitar bloqueio | Campaign | Confirmado |
| RN-011 | Planos definem limites de bots, agentes e mensagens mensais | Billing | Confirmado |
| RN-012 | Assinatura ativa é necessária para uso do bot (HTTP 402 documentado) | Billing | Parcial |
| RN-013 | `api_key` de provedores de IA é armazenada por tenant | AI | Confirmado (texto plano) |
| RN-014 | Ações administrativas são registradas em `audit_logs` | Admin | Confirmado |
| RN-015 | Contatos duplicados devem ser evitados por tenant | Contatos | A CONFIRMAR (constraint não verificada) |
| RN-016 | Apenas números autorizados podem iniciar sessão WhatsApp | Bot | A CONFIRMAR |

---

## 8. Critérios Gerais de Aceite

- [ ] Sistema executa localmente com instruções documentadas.
- [ ] Testes principais passam.
- [ ] Funcionalidades principais documentadas.
- [ ] Banco de dados documentado.
- [ ] Fluxos principais validados.
- [ ] Sem credenciais expostas.
- [ ] Relatório diário atualizado.

---

## 9. Entregáveis

- Código-fonte do backend Node.js (`/node-version`).
- Código-fonte do frontend PHP (`/chatbot`).
- Legado .NET (`SaaS-Chatbot.sln`) e Python (`requirements.txt`, `Dockerfile.python`) preservados.
- Documentação estrutural (`AGENTS.md`, `ARQUITETURA.md`, `BANCO_DADOS.md`, `ESCOPO.md`, `ROADMAP.md`, `CONTEXTO.md`, `RELATORIO.md`).
- Scripts de teste manuais (`/node-version/test_*.js`, `/chatbot/test_*.js`).
- Configuração Docker (`docker-compose.yml`, Dockerfiles, `nginx/`).
- Scripts de deploy (`deploy-pm2.sh`, `ecosystem.config.js`).
- OpenAPI (`openapi.json`) e documentação específica (`projeto-documentacao.md`, `MVP_FEATURES.md`, etc.).
- Manual de execução a ser complementado.

---

## 10. Premissas

- O backend Node.js é a stack ativa para API e lógica de negócio.
- O frontend PHP consome a API Node.js e não possui lógica de negócio independente.
- O ambiente de desenvolvimento usa Docker Compose.
- O WhatsApp é o canal principal de comunicação.
- O Ollama local é o provedor de IA padrão.
- Cada tenant é isolado por `tenant_id` no banco de dados.
- Revendas operam como white-label para seus clientes finais.

---

## 11. Restrições

- Legado .NET e Python não devem ser modificados sem autorização.
- Não há migrations formais — alterações de schema dependem de `sequelize.sync`.
- Baileys pode sofrer limitações ou bloqueios por parte do WhatsApp.
- IA local (Ollama) requer recursos computacionais adequados.
- O sistema não possui testes automatizados formais para Node.js e PHP.
- Não há CI/CD identificado.

---

## 12. Riscos

| Risco | Impacto | Probabilidade | Mitigação |
|---|---|---|---|
| Baileys sofre bloqueio ou instabilidade | Alto | Média | Monitorar, considerar fallback oficial Meta API |
| `sequelize.sync({ alter: true })` corromper schema em produção | Alto | Média | Adotar migrations formais e backup |
| Vazamento de secrets (fallbacks hardcoded) | Alto | Média | Remover fallbacks, usar .env exclusivo |
| Crescimento ilimitado de `chat_history` | Médio | Alta | Definir TTL e arquivamento |
| Falta de testes gera regressões | Alto | Média | Configurar suite de testes |
| Dependência de Ollama local limita deploy | Médio | Média | Documentar requisitos de hardware, oferecer alternativas |
| Integração de pagamento não confirmada | Médio | Média | Validar gateway e testar faturamento real |
| Concorrência e escalabilidade não testadas | Médio | Média | Testes de carga e revisão de arquitetura |
