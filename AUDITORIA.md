# Auditoria Técnica e de Segurança — SaaS Chatbot

**Projeto:** Chatbot_Mauricio / SaaS Chatbot  
**Data da auditoria:** 2026-07-02  
**Auditor:** IA Agent (Cascata)  
**Escopo:** Backend Node.js (`node-version/`), Frontend PHP (`chatbot/`), Infraestrutura Docker, Banco de dados, Segurança e Governança.  
**Metodologia:** Revisão estática de código, análise de configurações, inspeção de secrets e fluxos críticos.

---

## 1. Resumo Executivo

A plataforma possui um MVP funcional com backend Node.js, frontend PHP e infraestrutura containerizada. Vários módulos principais estão implementados, mas há **problemas críticos de segurança, configuração e qualidade de código** que devem ser resolvidos antes de qualquer uso em produção ou demonstração comercial.

**Principais riscos encontrados:**

1. Secrets e credenciais com fallbacks hardcoded ou expostos no repositório.
2. Arquivo `.env` do backend versionado no Git.
3. Backdoor de usuário "programador" com email default hardcoded.
4. Uploads de arquivos servidos publicamente sem autenticação.
5. Bancos de dados e serviços expostos em portas do host no `docker-compose.yml`.
6. Tratamento de erros silencioso (`catch` vazio) e exposição de mensagens internas em respostas 500.
7. Multi-tenancy com controle fraco em alguns pontos.
8. Falta de testes automatizados formais.
9. Diretórios temporários e de backup versionados no Git.
10. Código de teste/debug exposto em rotas públicas (`/v1/force-error`).

---

## 2. Status por Módulo (Pronto / Parcial / Pendente)

| Módulo | Status | Evidências | Observações |
|---|---|---|---|
| **Autenticação e Identidade** | Parcial | `authController.js`, `authMiddleware.js`, `security.js`, `models.js` | JWT funcional, roles de admin, single-session, refresh token gerado. Faltam MFA, recuperação de senha, refresh endpoint, política de senha forte no backend, e criptografia de secrets. |
| **Multi-tenancy** | Parcial | `tenancyMiddleware.js`, `models/sql/index.js` | AsyncLocalStorage injeta tenant_id nas queries. Risco de bypass em `ignoreTenant`, `beforeFind` sobrescreve where, não há RLS no PostgreSQL. |
| **WhatsApp Gateway** | Parcial | `whatsappCore.js`, `botController.js`, `chatController.js` | Baileys conecta, envia/recebe, sincroniza contatos e LIDs. Problemas de catch vazio, persistência de tokens não criptografada, deduplicação depende de Redis, mapeamento de status pode estar deslocado. |
| **Chat / Inbox** | Parcial | `chatController.js`, `connectionManager.js` | Histórico, paginação, fallback para Baileys, WebSocket. Faltam notas internas, filtros avançados, limite de chats sendo validado em todas as rotas. |
| **Contatos e Tags** | Parcial | `contactsController.js`, `models.js` | CRUD, importação, tags. Não há unique constraint por tenant, risco de duplicados. |
| **Flow Engine** | Parcial | `flowWorker.js`, `FlowExecutor.js`, `Flow.js` | Estrutura de nodes/edges e execução. Não implementa gatilhos por palavra-chave, só um fluxo ativo por tenant, sem editor visual. |
| **Inteligência Artificial** | Parcial | `llamaService.js`, `aiController.js`, `AiConfig` | Ollama integrado, RAG estruturado. `api_key` em texto plano, provedores externos não confirmados, temperatura fixa, dotenv com path errado. |
| **Billing e Planos** | Parcial | `billingNotificationService.js`, `invoicingService.js`, `models.js` | Planos, assinaturas, faturas, heartbeat. Sem gateway de pagamento real, sem envio de notificações, gera fatura pós suspensão. |
| **Campanhas** | Parcial | `campaignWorker.js`, `campaignsController.js` | Disparo com delays e janela de sono. Horário UTC, sem verificação de limite de plano, sem verificação de blacklist, sem DLQ. |
| **Admin e Revendas** | Parcial | `adminController.js`, `resellerController.js`, `resellerMiddleware.js` | CRUD de admins, revendas, sub-tenants, auditoria. Primeiro admin pode ser criado sem autenticação, exposição de `e.message` em erros. |
| **Frontend PHP** | Parcial | `Router.php`, `HomeController.php`, `ApiOmniController.php`, `OmniChannelApiClient.php` | Login, proxy para API, home. Backdoor programador, router sem rotas dinâmicas, inputs não sanitizados, token JWT em URL de WebSocket. |
| **Infraestrutura Docker** | Parcial | `docker-compose.yml`, Dockerfiles, `nginx/` | Orquestração completa. Senhas hardcoded, serviços expostos em portas host, certificado autoassinado, Dockerfiles copiam `.env`/tokens. |
| **Testes Automatizados** | Pendente | `test_*.js` manuais, sem Jest/PHPUnit | Apenas scripts manuais. Não há suite de testes formais. |
| **Documentação** | Pronto | `docs/governanca/`, `docs/specs/`, `README.md`, `AGENTS.md` | Documentação de governança, especificações SDD e README comercial criados. |

---

## 3. Falhas Críticas

| ID | Problema | Arquivo(s) | Impacto | Correção necessária |
|---|---|---|---|---|
| C01 | **Fallback hardcoded de SECRET_KEY e ADMIN_SECRET_KEY** | `authMiddleware.js:6`, `authMiddleware.js:94`, `adminController.js:22`, `security.js:4` | Se a variável de ambiente não for definida, o sistema usa chave pública conhecida, permitindo forja de JWT. | Remover fallbacks. Fazer a aplicação falhar no startup se `SECRET_KEY` ou `ADMIN_SECRET_KEY` estiverem ausentes. |
| C02 | **Arquivo `.env` versionado no Git** | `node-version/.env` | Exposição de configurações e risco de vazamento de secrets. | Remover do histórico do Git, adicionar `.env` e `node-version/.env` ao `.gitignore`, criar `.env.example` na raiz. |
| C03 | **Backdoor de usuário programador com email default** | `chatbot/src/Support/ProgramadorAccess.php:17` | Qualquer conta com email `programador@teste.com` obtém acesso especial. | Remover email default. Usar apenas lista configurada via ambiente, com validação de domínio. |
| C04 | **API_BASE_URL hardcoded para IP externo** | `chatbot/src/Service/OmniChannelApiClient.php:25` | Frontend pode apontar para servidor de produção/desconhecido. | Remover fallback, exigir `API_BASE_URL` no `.env`. |
| C05 | **Credenciais de banco e serviços hardcoded no `docker-compose.yml`** | `docker-compose.yml:6-8, 28-29, 39-40, 62-66, 89-91, 111` | Senhas padrão expostas e reutilizadas. | Usar variáveis de ambiente externas ou `.env` com `env_file`. Nunca commitar secrets. |
| C06 | **Uploads servidos publicamente sem autenticação** | `server.js:24`, `storageController.js`, `storageService.js`, `nginx/default.conf:56-59` | Qualquer pessoa pode acessar arquivos de qualquer tenant. | Proteger `/uploads/*` por middleware de autenticação/tenant ou gerar URLs assinadas. |
| C07 | **Tokens do WhatsApp armazenados em disco sem criptografia** | `whatsappCore.js:48`, `docker-compose.yml:73` | Credenciais de sessão do WhatsApp em arquivos locais. | Criptografar tokens em repouso, restringir permissões, não versionar `tokens/`. |
| C08 | **Rota de teste de erro exposta publicamente** | `routes.js:30-34` | Permite gerar falhas 500 e poluir logs/relatórios. | Remover ou proteger por `NODE_ENV=development`. |
| C09 | **Serviços internos expostos em portas do host** | `docker-compose.yml:11-12, 20-21, 30-32, 41-42, 91-92, 79-80` | PostgreSQL, Redis, RabbitMQ, MongoDB, MySQL e Ollama acessíveis externamente. | Remover ports ou restringir a `127.0.0.1`. Usar rede Docker interna. |
| C10 | **Exposição de `e.message` e stack traces em respostas 500** | `adminController.js`, `resellerController.js`, `contactsController.js`, `resellerMiddleware.js` | Vazamento de detalhes internos e potenciais vulnerabilidades. | Retornar mensagens genéricas ao cliente, logar detalhes internamente. |

---

## 4. Falhas de Segurança Médias

| ID | Problema | Arquivo(s) | Impacto | Correção necessária |
|---|---|---|---|---|
| M01 | **`api_key` de IA armazenada em texto plano** | `models.js` (`AiConfig`) | Vazamento de chaves de API externas. | Criptografar `api_key` com AES-256 ou similar. |
| M02 | **ErrorMiddleware armazena `req.body` completo** | `errorMiddleware.js:15-23` | Pode persistir senhas, tokens e dados sensíveis. | Sanitizar campos sensíveis antes de salvar. |
| M03 | **AuditMiddleware captura payload de resposta** | `auditMiddleware.js:48-58` | Pode logar tokens e dados sensíveis. | Filtrar campos sensíveis e rotas de login. |
| M04 | **Trust proxy não configurado** | `server.js` | `req.ip` pode ser o IP do proxy, não do cliente. | Adicionar `app.set('trust proxy', true)` com validação. |
| M05 | **Primeiro admin pode ser criado sem autenticação** | `adminController.js:54-77` | Qualquer pessoa pode criar superadmin. | Exigir setup token ou desativar endpoint pós-primeiro admin. |
| M06 | **JWT do usuário exposto em URL de WebSocket** | `ApiOmniController.php:534` | Token pode ser logado por proxies/servidores. | Usar header ou cookie seguro para WS. |
| M07 | **Senhas padrão e fracas** | `docker-compose.yml`, `security.js` | Senhas como `password123` e `3702959` usadas. | Gerar senhas fortes e únicas por ambiente. |
| M08 | **Router PHP não sanitiza `$_GET['url']`** | `Router.php:46-51` | Potencial path traversal ou open redirect. | Validar e restringir caracteres permitidos. |
| M09 | **Multi-tenancy: `beforeFind` sobrescreve `options.where`** | `models/sql/index.js:14-20` | Pode quebrar queries com operadores complexos (`$or`, `$and`). | Fazer merge seguro do `tenant_id`. |
| M10 | **Redis sem senha no fallback** | `redis.js:11`, `docker-compose.yml:17-22` | Redis acessível sem autenticação. | Configurar `requirepass` e `REDIS_URL` com credenciais. |

---

## 5. Erros de Lógica e Qualidade de Código

| ID | Problema | Arquivo(s) | Impacto | Correção necessária |
|---|---|---|---|---|
| L01 | **Catch vazio em `whatsappCore.js`** | `whatsappCore.js` (9 ocorrências) | Erros de sincronização/DB são silenciados. | Logar erros e, quando apropriado, retornar falha. |
| L02 | **FlowWorker não usa palavras-chave de gatilho** | `flowWorker.js:35` | Só executa um fluxo ativo, ignorando `triggers`. | Implementar matching de palavras-chave e gatilhos. |
| L03 | **CampaignWorker usa horário UTC para janela de sono** | `campaignWorker.js:73` | Horário de sono pode estar errado para o fuso do cliente. | Usar timezone do tenant ou UTC configurável. |
| L04 | **CampaignWorker não verifica limite de plano** | `campaignWorker.js` | Pode extrapolar `max_messages_month`. | Consultar subscription/plan antes de enviar. |
| L05 | **Inconsistência de status de mensagens** | `chatController.js:31` | Mapeamento de status pode estar deslocado. | Alinhar com valores reais do Baileys. |
| L06 | **Access token expire: 15 min vs 1440 min** | `security.js:6,21` | Configuração `ACCESS_TOKEN_EXPIRE_MINUTES` é ignorada. | Usar a variável de ambiente ou remover. |
| L07 | **Refresh token usa mesmo secret do access token** | `security.js:19-41` | Quebra isolamento de tokens. | Usar `REFRESH_SECRET_KEY` separado. |
| L08 | **`sequelize.sync({ alter: true })` em produção** | `server.js:91` | Pode corromper dados ou alterar schema inesperadamente. | Usar migrations formais (Sequelize CLI). |
| L09 | **Dotenv com path relativo inconsistente** | `database.js:4`, `redis.js:3`, `rabbitmq.js:3`, `llamaService.js:3` | Arquivos recarregam `.env` de caminhos errados. | Centralizar carregamento de dotenv no `server.js` e remover `.config()` duplicados. |
| L10 | **RabbitMQ descarta mensagens em erro** | `rabbitmq.js:66` | `nack(msg, false, false)` perde mensagens. | Configurar Dead Letter Exchange (DLX) ou requeue limitado. |
| L11 | **Health check não verifica dependências** | `server.js:70-72` | Retorna OK mesmo se bancos/filas estiverem indisponíveis. | Verificar PostgreSQL, MongoDB, Redis, RabbitMQ. |
| L12 | **Grateful shutdown incompleto** | `server.js:142-147` | Não fecha WhatsApp, PostgreSQL, MongoDB, workers. | Fechar conexões e processos antes de sair. |
| L13 | **Modelos sem constraints de unique por tenant** | `models.js` | `Contact`, `WhatsAppInstance`, `Tag` podem duplicar. | Adicionar unique composite (`tenant_id` + campo). |
| L14 | **Frontend PHP: inputs não sanitizados** | `AuthController.php`, `ApiOmniController.php` | Risco de XSS/Injection. | Aplicar `htmlspecialchars` e validação. |
| L15 | **Frontend PHP: Router sem rotas dinâmicas** | `Router.php` | Não suporta parâmetros de URL. | Implementar pattern matching ou usar framework. |
| L16 | **Single active session impede uso multi-dispositivo** | `authController.js:64-72` | UX ruim. | Tornar opcional por tenant ou usar refresh. |
| L17 | **BillingNotificationService apenas loga alertas** | `billingNotificationService.js:27-29` | Não envia notificações reais. | Implementar envio de email/push. |
| L18 | **Dockerfile PHP não executa `composer install`** | `chatbot/Dockerfile` | Dependências não são instaladas no build. | Adicionar `composer install` ou copiar `vendor/`. |
| L19 | **Diretórios temporários versionados** | `temp_extract/`, `unzipped_chatbot/`, `chatbot/arquivos/`, `node-version/scratch/` | Poluição do repositório. | Remover e adicionar ao `.gitignore`. |
| L20 | **`.gitignore` não ignora `.env` e `tokens/`** | `.gitignore` | Risco de commitar secrets. | Adicionar `.env`, `node-version/.env`, `node-version/tokens/`, `chatbot/.env`. |

---

## 6. O que Está Pronto (Funcional e Funcionando)

| Área | Evidência | Observação |
|---|---|---|
| Backend Node.js estruturado | `server.js`, `src/` | MVC/services/controllers organizados. |
| Autenticação JWT | `authController.js`, `authMiddleware.js` | Login, registro, roles, single-session. |
| Multi-tenancy com AsyncLocalStorage | `tenancyMiddleware.js`, `models/sql/index.js` | Isolamento básico por tenant. |
| WhatsApp via Baileys | `whatsappCore.js` | Conexão, QR, envio/recebimento, sincronização. |
| Chat com histórico e WebSocket | `chatController.js`, `connectionManager.js` | Funcionalidade principal operacional. |
| Contatos e tags | `contactsController.js`, `models.js` | CRUD e importação. |
| Flow engine estrutural | `flowWorker.js`, `FlowExecutor.js` | Execução de nodes/edges. |
| IA com Ollama | `llamaService.js` | Respostas automáticas com llama3.2. |
| Billing estrutural | `billingNotificationService.js`, `invoicingService.js` | Planos, assinaturas, faturas. |
| Admin/revendas | `adminController.js`, `resellerController.js` | Painel admin e hierarquia. |
| Frontend PHP básico | `chatbot/src/` | Login, home, proxy de API. |
| Docker Compose completo | `docker-compose.yml` | Todos os serviços mapeados. |
| Swagger/OpenAPI | `server.js:33-67` | Documentação automática da API. |
| Documentação de governança | `docs/governanca/`, `docs/specs/`, `README.md` | Documentação estrutural e comercial. |

---

## 7. Checklist de Correções para Conclusão do Projeto

### Segurança (obrigatório antes de produção)

- [ ] C01: Remover fallbacks hardcoded de `SECRET_KEY`, `ADMIN_SECRET_KEY` e `PROVISION_API_KEY`.
- [ ] C02: Remover `node-version/.env` do Git e adicionar `.env` ao `.gitignore`.
- [ ] C02: Criar `.env.example` na raiz com valores fictícios.
- [ ] C03: Remover backdoor `programador@teste.com` do `ProgramadorAccess`.
- [ ] C04: Remover IP hardcoded de `API_BASE_URL` no frontend.
- [ ] C05: Externalizar senhas do `docker-compose.yml` para `.env`.
- [ ] C06: Proteger rota `/uploads/*` com autenticação ou URLs assinadas.
- [ ] C07: Criptografar tokens do WhatsApp em repouso e restringir acesso.
- [ ] C08: Remover rota `/v1/force-error` ou proteger por ambiente de dev.
- [ ] C09: Restringir portas de serviços internos ao Docker (remover ports ou bind 127.0.0.1).
- [ ] C10: Não expor `e.message` ou stack traces em respostas 500.
- [ ] M01: Criptografar `api_key` de IA em `AiConfig`.
- [ ] M02: Sanitizar `req.body` no `errorMiddleware` antes de persistir.
- [ ] M03: Filtrar dados sensíveis no `auditMiddleware`.
- [ ] M04: Configurar `trust proxy` no Express.
- [ ] M05: Proteger criação do primeiro admin.
- [ ] M06: Não colocar JWT em URL de WebSocket.
- [ ] M07: Gerar senhas fortes e aleatórias para serviços.
- [ ] M08: Validar `$_GET['url']` no `Router.php`.
- [ ] M09: Corrigir merge de `tenant_id` nos hooks do Sequelize.
- [ ] M10: Configurar autenticação no Redis.

### Qualidade de Código e Lógica

- [ ] L01: Substituir catchs vazios por logging adequado em `whatsappCore.js` e `connectionManager.js`.
- [ ] L02: Implementar gatilhos de palavras-chave no `FlowWorker`.
- [ ] L03: Usar timezone configurável no `CampaignWorker`.
- [ ] L04: Verificar limite de mensagens do plano antes de disparar campanha.
- [ ] L05: Revisar mapeamento de status de mensagens do Baileys.
- [ ] L06: Tornar expiração do access token configurável.
- [ ] L07: Usar secret separado para refresh token.
- [ ] L08: Migrar de `sequelize.sync({ alter: true })` para migrations formais.
- [ ] L09: Centralizar carregamento do dotenv e remover `.config()` duplicados.
- [ ] L10: Configurar Dead Letter Exchange no RabbitMQ.
- [ ] L11: Implementar health check real de dependências.
- [ ] L12: Implementar graceful shutdown completo.
- [ ] L13: Adicionar constraints unique por tenant nos modelos.
- [ ] L14: Sanitizar inputs no frontend PHP.
- [ ] L15: Suportar rotas dinâmicas no `Router.php`.
- [ ] L16: Revisar política de single active session.
- [ ] L17: Implementar envio real de notificações de billing.
- [ ] L18: Adicionar `composer install` no Dockerfile PHP.
- [ ] L19: Remover diretórios temporários do repositório.
- [ ] L20: Atualizar `.gitignore` para `.env`, `tokens/`, `scratch/`, `arquivos/`, etc.

### Testes e CI/CD

- [ ] Configurar Jest ou Mocha no backend Node.js.
- [ ] Configurar PHPUnit no frontend PHP.
- [ ] Criar testes unitários para autenticação, multi-tenancy, WhatsApp mock, billing.
- [ ] Criar testes de integração para os principais fluxos.
- [ ] Adicionar pipeline de CI/CD com build, lint e testes.
- [ ] Adicionar husky/lint-staged para qualidade de código.

### Documentação

- [ ] Criar `SECURITY.md` com políticas de segurança.
- [ ] Criar `CONTRIBUTING.md` com padrões de código.
- [ ] Atualizar `docs/governanca/BANCO_DADOS.md` com constraints e RLS.
- [ ] Atualizar `docs/governanca/ARQUITETURA.md` com decisões de segurança.
- [ ] Criar runbook de deploy e operação.

---

## 8. Riscos e Dependências Pendentes

| Risco | Impacto | Mitigação |
|---|---|---|
| Baileys instável ou bloqueado | Alto | Monitorar, planejar integração com Meta API oficial. |
| Ollama sem recursos suficientes | Médio | Usar modelos menores ou provider externo. |
| Falta de testes | Alto | Priorizar suite de testes críticos. |
| Secrets expostos | Crítico | Remover imediatamente, rotacionar credenciais. |
| Banco exposto em portas host | Alto | Fechar portas externas. |
| Não há gateway de pagamento | Médio | Definir e integrar provider (Stripe, MercadoPago, etc.). |

---

## 9. Recomendações Finais

1. **Não utilize este código em produção** até resolver as falhas críticas de segurança.
2. Realize um **security review** manual após as correções iniciais.
3. Adote **migrations formais** e remova `sequelize.sync({ alter: true })`.
4. Implemente **testes automatizados** antes de novas funcionalidades (TDD).
5. Crie um **processo de secrets management** (Docker secrets, Vault, GitHub Secrets).
6. Documente cada correção em `docs/governanca/CONTEXTO.md` e `docs/governanca/RELATORIO.md`.

---

*Esta auditoria foi gerada com base em revisão estática do código. Recomenda-se validação dinâmica (testes, pentest, análise de runtime) antes de liberação comercial.*
