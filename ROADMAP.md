# ROADMAP.md — Roadmap de Desenvolvimento

**Projeto:** SaaS Chatbot / Chatbot_Mauricio  
**Atualizado em:** 2026-07-02  
**Metodologia:** Specification-Driven Development (SDD) + Test-Driven Development (TDD)

---

## 1. Visão Geral do Roadmap

Este roadmap organiza a evolução do projeto em fases, épicos, histórias e tarefas. Todo trabalho segue SDD (especificar antes de codificar) e TDD (teste antes de implementar). A documentação estrutural é a base para qualquer mudança futura.

O projeto está em fase de consolidação do MVP após migração do backend para Node.js. O frontend PHP foi mantido como interface principal. Os legados .NET e Python são preservados, mas não ativos.

---

## 2. Convenções de Status

- `PENDENTE` — ainda não iniciado.
- `EM ESPECIFICAÇÃO` — especificação em andamento.
- `ESPECIFICADO` — especificação pronta, aguardando desenvolvimento.
- `EM TESTE` — testes sendo escritos/ajustados (RED).
- `EM DESENVOLVIMENTO` — implementação em andamento (GREEN).
- `EM REVISÃO` — código pronto, em revisão/documentação.
- `CONCLUÍDO` — tarefa finalizada e validada.
- `BLOQUEADO` — dependência externa impedindo continuidade.
- `CANCELADO` — descontinuado por decisão.

---

## 3. Fases do Projeto

### Fase 0 — Diagnóstico e Governança

**Objetivo:** entender o projeto, documentar arquitetura, banco, escopo e definir regras de execução para agentes.

**Épicos:**
- EP-0001 — Criar governança do repositório (`AGENTS.md`).
- EP-0002 — Mapear arquitetura atual (`ARQUITETURA.md`).
- EP-0003 — Mapear banco de dados (`BANCO_DADOS.md`).
- EP-0004 — Definir escopo inicial (`ESCOPO.md`).
- EP-0005 — Definir estratégia de testes e roadmap (`ROADMAP.md`).
- EP-0006 — Criar contexto vivo e relatório diário (`CONTEXTO.md`, `RELATORIO.md`).

**Critérios de aceite:**
- [x] `AGENTS.md` criado.
- [x] `ARQUITETURA.md` criado.
- [x] `BANCO_DADOS.md` criado.
- [x] `ESCOPO.md` criado.
- [x] `ROADMAP.md` criado.
- [x] `CONTEXTO.md` criado.
- [x] `RELATORIO.md` criado.
- [ ] Stack validada com build/teste local.
- [ ] Pendências críticas registradas.

---

### Fase 1 — Base Técnica e Ambiente

**Objetivo:** garantir que o projeto rode localmente, tenha testes básicos e documentação mínima de execução.

#### Tarefas

- [ ] Validar instalação do backend Node (`npm install` em `node-version`).
- [ ] Validar instalação do frontend PHP (`composer install` em `chatbot`).
- [ ] Validar variáveis de ambiente e arquivos `.env`.
- [ ] Validar build do TypeScript na raiz (`npm run build`).
- [ ] Levantar Docker Compose completo (`docker compose up --build`).
- [ ] Validar conexão PostgreSQL, MongoDB, MySQL, Redis, RabbitMQ, Ollama.
- [ ] Documentar comandos de execução em `ARQUITETURA.md`.
- [ ] Identificar e registrar riscos técnicos.

#### História: Ambiente de desenvolvimento funcional
- **Como:** desenvolvedor.
- **Quero:** subir o ambiente completo com Docker Compose.
- **Para:** garantir que backend, frontend e serviços dependências funcionem juntos.
- **Status:** PENDENTE

##### SDD
- **Especificação:** documentar passos de instalação e configuração de ambiente.
- **Critérios de aceite:** `docker compose up --build` inicia todos os serviços; API responde `/health`; frontend responde na porta 8081.
- **Impacto técnico:** documentação, validação de `.env.example`, ajuste de portas se necessário.

##### TDD
- **Teste RED:** criar teste de integração básico que verifica `/health` e conexão de bancos.
- **Implementação GREEN:** ajustar configurações e Docker Compose conforme necessário.
- **Refatoração:** padronizar variáveis de ambiente entre backend e frontend.
- **Teste de regressão:** garantir que o ambiente continue subindo após mudanças.

##### Tarefas
- [ ] Criar `.env.example` na raiz se não existir.
- [ ] Validar conflitos de porta (8000, 8080, 8081, 3309, 5432, 5672, 15672, 6379, 27017, 11434).
- [ ] Documentar passos no `ARQUITETURA.md`.

---

### Fase 2 — Funcionalidades Core

**Objetivo:** implementar ou estabilizar funcionalidades principais do MVP.

#### História: Autenticação e JWT
- **Como:** usuário.
- **Quero:** me cadastrar e fazer login.
- **Para:** acessar o painel de forma segura.
- **Status:** CONCLUÍDO (código existente; testes pendentes)

##### SDD
- **Especificação:** confirmar fluxo de login, registro, roles, JWT e tenants.
- **Critérios de aceite:** login retorna JWT válido; token protege rotas; tenant isolado; superadmin protegido.
- **Impacto técnico:** `authController.js`, `authMiddleware.js`, `models/sql/models.js`.

##### TDD
- **Teste RED:** testes de login com credenciais válidas e inválidas; teste de acesso a rota protegida.
- **Implementação GREEN:** revisar código existente e corrigir se necessário.
- **Refatoração:** remover fallbacks hardcoded de secrets.
- **Teste de regressão:** login e proteção de rotas continuam funcionando.

##### Tarefas
- [ ] Adicionar teste de auth no Node.js.
- [ ] Revisar secrets hardcoded em `authMiddleware.js`.
- [ ] Documentar fluxo em `ARQUITETURA.md`.

---

#### História: WhatsApp Gateway
- **Como:** cliente final.
- **Quero:** conectar meu número de WhatsApp e enviar/receber mensagens.
- **Para:** atender meus clientes pelo chatbot.
- **Status:** PARCIAL

##### SDD
- **Especificação:** confirmar ciclo de vida da sessão Baileys, QR code, envio/recebimento, deduplicação.
- **Critérios de aceite:** `/api/v1/bot/start` gera QR; scan conecta; mensagem recebida aparece no chat; envio manual funciona; ack atualizado.
- **Impacto técnico:** `whatsappCore.js`, `botController.js`, `Message.js`, `connectionManager.js`, `ackWorker.js`.

##### TDD
- **Teste RED:** mock do Baileys para simular mensagem recebida e enviada.
- **Implementação GREEN:** ajustar handlers se necessário.
- **Refatoração:** separar responsabilidades de `whatsappCore.js` se muito acoplado.
- **Teste de regressão:** simular mensagem e verificar persistência e WebSocket.

##### Tarefas
- [ ] Testar conexão real com QR code.
- [ ] Validar envio e recebimento de texto.
- [ ] Validar mídia.
- [ ] Documentar gatilhos e handlers.

---

#### História: Chat / Inbox em tempo real
- **Como:** atendente.
- **Quero:** ver conversas e responder via WebSocket.
- **Para:** atender clientes em tempo real.
- **Status:** PARCIAL

##### SDD
- **Especificação:** confirmar endpoints de chat, histórico, transferência e WebSocket.
- **Critérios de aceite:** lista de conversas; histórico por contato; envio manual; atualização em tempo real; transferência.
- **Impacto técnico:** `chatController.js`, `connectionManager.js`, `Message.js`.

##### TDD
- **Teste RED:** teste de API para envio e listagem; teste de WebSocket com autenticação.
- **Implementação GREEN:** ajustar se necessário.
- **Refatoração:** padronizar payload do WebSocket.
- **Teste de regressão:** garantir que mensagens não dupliquem no frontend.

##### Tarefas
- [ ] Validar endpoints REST de chat.
- [ ] Validar WebSocket.
- [ ] Implementar/adicionar notas internas se fizer parte do escopo.

---

#### História: Flow Engine
- **Como:** cliente final.
- **Quero:** criar fluxos automáticos de atendimento.
- **Para:** responder perguntas frequentes sem atendente humano.
- **Status:** PARCIAL

##### SDD
- **Especificação:** confirmar formato de nodes/edges, gatilhos, execução, transbordo.
- **Critérios de aceite:** criar fluxo; mensagem com palavra-chave dispara fluxo; resposta automática enviada; transbordo funciona.
- **Impacto técnico:** `Flow.js`, `flowWorker.js`, `services/flow/`.

##### TDD
- **Teste RED:** teste de criação de fluxo e execução simulada.
- **Implementação GREEN:** ajustar engine se necessário.
- **Refatoração:** documentar formato de nodes/edges.
- **Teste de regressão:** fluxos existentes não quebram.

##### Tarefas
- [ ] Validar criação de fluxo via API.
- [ ] Validar execução de fluxo.
- [ ] Validar transbordo humano.

---

#### História: Contatos e Tags
- **Como:** cliente final.
- **Quero:** gerenciar contatos e segmentar por tags.
- **Para:** organizar minha base de atendimento.
- **Status:** CONCLUÍDO (código existente; testes pendentes)

##### Tarefas
- [ ] Testar CRUD de contatos.
- [ ] Testar importação em lote.
- [ ] Testar sincronização do WhatsApp.
- [ ] Validar tags.

---

### Fase 3 — Integrações

**Objetivo:** implementar ou estabilizar integrações externas.

#### História: IA com Ollama e provedores alternativos
- **Como:** cliente final.
- **Quero:** configurar IA para responder automaticamente.
- **Para:** automatizar respostas inteligentes.
- **Status:** PARCIAL

##### Tarefas
- [ ] Validar Ollama llama3.2.
- [ ] A CONFIRMAR: integração real com OpenAI/Gemini/Anthropic.
- [ ] Validar ingestão RAG.
- [ ] Documentar configuração de IA.

---

#### História: Gateway de pagamento
- **Como:** admin master.
- **Quero:** cobrar assinaturas automaticamente.
- **Para:** faturar clientes e revendas.
- **Status:** A CONFIRMAR

##### Tarefas
- [ ] Identificar gateway de pagamento configurado.
- [ ] Validar webhook de pagamento.
- [ ] Testar fatura e transação.

---

### Fase 4 — Segurança, Performance e Qualidade

**Objetivo:** endurecer o sistema para uso real.

#### Épicos
- EP-0401 — Adotar migrations formais (Sequelize CLI).
- EP-0402 — Configurar suite de testes Node.js e PHP.
- EP-0403 — Revisar segurança: secrets, credenciais de IA, validação de entrada.
- EP-0404 — Implementar política de retenção de mensagens (TTL MongoDB).
- EP-0405 — Adicionar índices de performance.
- EP-0406 — LGPD: mapeamento de dados pessoais e direito ao esquecimento.

#### Tarefas
- [ ] Configurar Jest/Mocha para Node.js.
- [ ] Configurar PHPUnit para PHP.
- [ ] Adicionar testes de integração para rotas críticas.
- [ ] Remover fallbacks hardcoded de secrets.
- [ ] Criptografar `api_key` de `ai_configs`.
- [ ] Definir TTL para `chat_history`.
- [ ] Adicionar índices em `users`, `invoices`, `transactions`, `campaigns`.
- [ ] Documentar decisões em `CONTEXTO.md`.

---

### Fase 5 — Deploy, Observabilidade e Operação

**Objetivo:** preparar o sistema para ambiente de produção.

#### Épicos
- EP-0501 — Configurar CI/CD (GitHub Actions, GitLab CI ou similar).
- EP-0502 — Configurar backup automatizado de bancos.
- EP-0503 — Melhorar observabilidade (métricas, health checks, alertas).
- EP-0504 — Documentar deploy em produção (Docker, Nginx, PM2, certificados SSL).
- EP-0505 — Testar deploy em staging.

#### Tarefas
- [ ] Verificar scripts `deploy-pm2.sh` e `ecosystem.config.js`.
- [ ] Validar configuração Nginx (`nginx/`).
- [ ] Criar pipeline de build e teste.
- [ ] Configurar backup de volumes Docker.
- [ ] Adicionar health checks completos.

---

### Fase 6 — Evolução Contínua

**Objetivo:** planejar melhorias futuras.

#### Épicos
- EP-0601 — Integração oficial com WhatsApp Business API (Meta).
- EP-0602 — Aplicativo mobile para atendentes.
- EP-0603 — Dashboards avançados de analytics.
- EP-0604 — Multi-idioma automático.
- EP-0605 — Canais adicionais (Instagram, Telegram, Messenger).
- EP-0606 — Embeddings avançados e fine-tuning de IA.

---

## 4. Backlog Geral

| ID | Tipo | Descrição | Prioridade | Status |
|---|---|---|---|---|
| BK-001 | Documentação | Criar governança do repositório (`AGENTS.md`) | Alta | CONCLUÍDO |
| BK-002 | Documentação | Mapear arquitetura (`ARQUITETURA.md`) | Alta | CONCLUÍDO |
| BK-003 | Documentação | Mapear banco de dados (`BANCO_DADOS.md`) | Alta | CONCLUÍDO |
| BK-004 | Documentação | Definir escopo (`ESCOPO.md`) | Alta | CONCLUÍDO |
| BK-005 | Documentação | Criar roadmap (`ROADMAP.md`) | Alta | CONCLUÍDO |
| BK-006 | Documentação | Criar contexto vivo (`CONTEXTO.md`) | Alta | EM DESENVOLVIMENTO |
| BK-007 | Documentação | Criar relatório diário (`RELATORIO.md`) | Alta | EM DESENVOLVIMENTO |
| BK-008 | Teste | Configurar suite de testes Node.js | Alta | PENDENTE |
| BK-009 | Teste | Configurar suite de testes PHP | Alta | PENDENTE |
| BK-010 | Teste | Validar ambiente Docker Compose | Alta | PENDENTE |
| BK-011 | Teste | Testar autenticação JWT | Média | PENDENTE |
| BK-012 | Teste | Testar WhatsApp Gateway | Média | PENDENTE |
| BK-013 | Teste | Testar chat/inbox em tempo real | Média | PENDENTE |
| BK-014 | Teste | Testar Flow Engine | Média | PENDENTE |
| BK-015 | Teste | Testar IA/Ollama | Média | PENDENTE |
| BK-016 | Débito técnico | Adotar migrations formais | Alta | PENDENTE |
| BK-017 | Débito técnico | Remover secrets hardcoded | Alta | PENDENTE |
| BK-018 | Débito técnico | Criptografar `api_key` de IA | Média | PENDENTE |
| BK-019 | Débito técnico | Definir TTL para `chat_history` | Média | PENDENTE |
| BK-020 | Débito técnico | Adicionar índices de performance | Média | PENDENTE |
| BK-021 | Infra | Configurar CI/CD | Média | PENDENTE |
| BK-022 | Infra | Configurar backups | Média | PENDENTE |
| BK-023 | Infra | Melhorar observabilidade | Média | PENDENTE |
| BK-024 | Funcionalidade | Validar gateway de pagamento | Média | A CONFIRMAR |
| BK-025 | Funcionalidade | Integrar OpenAI/Gemini | Baixa | A CONFIRMAR |
| BK-026 | Funcionalidade | Aplicativo mobile | Baixa | PENDENTE |
| BK-027 | Funcionalidade | Dashboards avançados | Baixa | PENDENTE |

---

## 5. Matriz SDD/TDD por Tarefa

| ID | Tarefa | Spec criada | Teste criado | Implementado | Documentado |
|---|---|---|---|---|---|
| T-001 | Criar governança do repositório | Sim | N/A | Sim | Sim |
| T-002 | Mapear arquitetura atual | Sim | N/A | N/A | Sim |
| T-003 | Mapear banco de dados | Sim | N/A | N/A | Sim |
| T-004 | Definir escopo | Sim | N/A | N/A | Sim |
| T-005 | Configurar testes Node.js | Sim | Não | Não | Não |
| T-006 | Configurar testes PHP | Sim | Não | Não | Não |
| T-007 | Validar autenticação | Sim | Não | Parcial | Parcial |
| T-008 | Validar WhatsApp Gateway | Sim | Não | Parcial | Parcial |
| T-009 | Validar chat em tempo real | Sim | Não | Parcial | Parcial |
| T-010 | Validar Flow Engine | Sim | Não | Parcial | Parcial |
| T-011 | Adotar migrations formais | Sim | Não | Não | Não |
| T-012 | Criptografar api_key de IA | Sim | Não | Não | Não |
| T-013 | Configurar CI/CD | Sim | Não | Não | Não |

---

## 6. Definição de Pronto

Uma tarefa só é considerada pronta quando:

- [ ] Requisito documentado.
- [ ] Critérios de aceite definidos.
- [ ] Teste criado ou justificativa registrada.
- [ ] Implementação concluída.
- [ ] Testes passando.
- [ ] Documentação atualizada (`ARQUITETURA.md`, `BANCO_DADOS.md`, `ESCOPO.md`, `ROADMAP.md` conforme impacto).
- [ ] `CONTEXTO.md` atualizado.
- [ ] `RELATORIO.md` atualizado.
- [ ] Não reduziu cobertura de testes sem justificativa.
- [ ] Não expôs secrets nem credenciais.
