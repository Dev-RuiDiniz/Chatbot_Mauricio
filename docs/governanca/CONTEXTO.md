# CONTEXTO.md — Contexto Vivo do Projeto

**Projeto:** SaaS Chatbot / Chatbot_Mauricio  
**Atualizado em:** 2026-07-02  
**Responsável pela atualização:** Agente IA / Desenvolvedor

---

## 1. Resumo Executivo

O **SaaS Chatbot** é uma plataforma omnichannel de chatbots e atendimento humano, com foco em WhatsApp. O repositório contém múltiplas stacks: um legado .NET, um legado Python/FastAPI, o backend ativo em **Node.js/Express** (`/node-version`) e o frontend ativo em **PHP 8 MVC** (`/chatbot`).

**Estado atual:** o backend Node.js é a stack funcional ativa. O frontend PHP consome a API Node. O projeto está em fase de consolidação do MVP, com documentação estrutural sendo criada e várias pendências técnicas identificadas (migrations, testes, segurança de secrets, gateway de pagamento, integrações de IA alternativas).

**Principal objetivo técnico atual:** organizar a governança, documentação e rastreabilidade do repositório para permitir evolução segura e colaborativa.

**Principais riscos:**
- Ausência de migrations formais e uso de `sequelize.sync({ alter: true })`.
- Secrets com fallback hardcoded no código.
- Ausência de testes automatizados para Node.js e PHP.
- `api_key` de IA armazenada em texto plano.
- Crescimento ilimitado do histórico de mensagens no MongoDB.

---

## 2. Estado Atual do Projeto

| Área | Status | Observações |
|---|---|---|
| Backend Node.js | Funcional / Parcial | API REST, WebSocket, workers, autenticação, WhatsApp, billing confirmados. Testes pendentes. |
| Frontend PHP | Funcional / Parcial | MVC próprio, consome API Node. Schema MySQL mínimo. UI completa a confirmar. |
| Banco de dados | Funcional / Parcial | PostgreSQL + MongoDB + MySQL operacionais. Sem migrations formais. |
| Testes | PENDENTE DE VALIDAÇÃO | Não há suite formal para Node.js e PHP. Python tem pytest configurado. |
| Infraestrutura | Funcional / Parcial | Docker Compose completo. Nginx, PM2, deploy scripts. CI/CD não identificado. |
| Documentação | Em desenvolvimento | Arquivos de governança criados nesta sessão. Outros docs técnicos existem. |
| Legado .NET | Preservado | Não modificar sem autorização. |
| Legado Python | Preservado | Não modificar sem autorização. |

---

## 3. Histórico de Desenvolvimento

### 2026-07-02 — Governança e documentação estrutural

- **O que foi analisado:**
  - Estrutura completa do repositório (`list_dir`, `find_by_name`).
  - Stack ativa: backend Node.js (`/node-version`), frontend PHP (`/chatbot`), legados .NET e Python.
  - Bancos: PostgreSQL (Sequelize), MongoDB (Mongoose), MySQL (PDO), Redis, RabbitMQ.
  - Infraestrutura: Docker Compose, Nginx, PM2, Ollama.
  - Documentação existente: `MVP_FEATURES.md`, `MVP_ROADMAP.md`, `projeto-documentacao.md`, `openapi.json`, `README.md`, `docs/`, `sprints/`.

- **O que foi decidido:**
  - Criar os 7 arquivos de governança e documentação solicitados na raiz.
  - Preservar legados .NET e Python sem modificações.
  - Documentar tudo o que foi confirmado e marcar o que não foi como `A CONFIRMAR` / `PENDENTE DE VALIDAÇÃO` / `NÃO IDENTIFICADO`.
  - Executar commit e push dos arquivos de documentação após autorização explícita do usuário.

- **O que foi criado:**
  - `AGENTS.md` — regras de execução dos agentes.
  - `ARQUITETURA.md` — arquitetura e funcionalidades do sistema.
  - `BANCO_DADOS.md` — modelos, relacionamentos e migrações.
  - `ESCOPO.md` — escopo funcional e não funcional.
  - `ROADMAP.md` — fases de desenvolvimento com SDD/TDD.
  - `CONTEXTO.md` — este arquivo.
  - `RELATORIO.md` — registro diário.

- **O que foi alterado:**
  - Nenhum código funcional alterado.
  - Nenhuma configuração alterada.
  - `RELATORIO.md` e `CONTEXTO.md` atualizados para refletir o commit e push realizados.

- **O que ficou pendente:**
  - Validar build e testes locais.
  - Commitar a atualização de `RELATORIO.md` e `CONTEXTO.md` (se desejado pelo usuário).

- **Evidências no repositório:**
  - `node-version/package.json`
  - `node-version/src/models/sql/models.js`
  - `node-version/src/models/nosql/*.js`
  - `node-version/server.js`
  - `chatbot/composer.json`, `chatbot/database/schema.sql`, `chatbot/src/`
  - `docker-compose.yml`
  - `MVP_FEATURES.md`, `MVP_ROADMAP.md`, `projeto-documentacao.md`

### 2026-07-02 — Reorganização da documentação

- **O que foi decidido:**
  - Manter na raiz apenas `README.md` e `AGENTS.md`.
  - Organizar toda a documentação em `docs/` com subpastas temáticas.

- **O que foi alterado:**
  - `AGENTS.md` permaneceu na raiz.
  - `README.md` permaneceu na raiz.
  - `ARQUITETURA.md` movido para `docs/governanca/ARQUITETURA.md`.
  - `BANCO_DADOS.md` movido para `docs/governanca/BANCO_DADOS.md`.
  - `ESCOPO.md` movido para `docs/governanca/ESCOPO.md`.
  - `ROADMAP.md` movido para `docs/governanca/ROADMAP.md`.
  - `CONTEXTO.md` movido para `docs/governanca/CONTEXTO.md`.
  - `RELATORIO.md` movido para `docs/governanca/RELATORIO.md`.
  - `API_INTEGRATION_GUIDE.md` movido para `docs/technical/API_INTEGRATION_GUIDE.md`.
  - `BACKEND_API_GUIDE.md` movido para `docs/technical/BACKEND_API_GUIDE.md`.
  - `agent_flow_integration_guide.md` movido para `docs/technical/agent_flow_integration_guide.md`.
  - `api_integration_documentation.md` movido para `docs/technical/api_integration_documentation.md`.
  - `integration_guide.md` movido para `docs/technical/integration_guide.md`.
  - `MVP_FEATURES.md` movido para `docs/technical/MVP_FEATURES.md`.
  - `MVP_ROADMAP.md` movido para `docs/technical/MVP_ROADMAP.md`.
  - `projeto-documentacao.md` movido para `docs/technical/projeto-documentacao.md`.
  - `walkthrough.md` movido para `docs/technical/walkthrough.md`.
  - `walkthrough_fotos_perfil.md` movido para `docs/technical/walkthrough_fotos_perfil.md`.
  - `backend_audit_report.md` movido para `docs/auditoria/backend_audit_report.md`.
  - `backend_minucious_audit_report.md` movido para `docs/auditoria/backend_minucious_audit_report.md`.
  - `backend_audit_report.pdf` movido para `docs/auditoria/backend_audit_report.pdf`.
  - `build_output.txt` movido para `docs/build/build_output.txt`.

- **O que ficou pendente:**
  - Verificar se há links Markdown quebrados após a movimentação.
  - Commitar e publicar a reorganização.

- **Resultado final:**
  - Commit realizado: `f56666d` — `docs(organizacao): reorganiza documentacao em docs/ mantendo apenas README.md e AGENTS.md na raiz`.
  - Push realizado: `01b3543..f56666d` para `main`.

---

## 4. Decisões Técnicas e Arquiteturais

| Data | Decisão | Motivo | Impacto | Status |
|---|---|---|---|---|
| 2026-07-02 | Manter backend ativo em Node.js | Legado Python foi migrado para Node.js; este é o código funcional | Stack principal consolidada | Confirmado |
| 2026-07-02 | Manter frontend PHP como consumidor da API Node | Frontend MVC próprio já integrado com API | Separação clara de backend e frontend | Confirmado |
| 2026-07-02 | Usar persistência híbrida PostgreSQL + MongoDB | Dados relacionais em SQL, mensagens/fluxos em NoSQL | Arquitetura de dados flexível | Confirmado |
| 2026-07-02 | WebSocket nativo (`ws`) em vez de Socket.io | Compatibilidade e simplicidade | Requer gerenciamento manual de reconexão | Confirmado |
| 2026-07-02 | Baileys para WhatsApp | Comunicação nativa sem API paga | Risco de bloqueio/instabilidade | Confirmado |
| 2026-07-02 | Ollama local como IA padrão | Custo zero de API externa | Requer recursos locais | Confirmado |
| 2026-07-02 | `sequelize.sync({ alter: true })` | Facilitar evolução sem migrations | Risco em produção; exige controle | PENDENTE DE VALIDAÇÃO |
| 2026-07-02 | Preservar legados .NET e Python | Histórico e possível referência | Não alterar sem autorização | Decidido |

---

## 5. Decisões de Produto e Escopo

| Data | Decisão | Motivo | Impacto |
|---|---|---|---|
| 2026-07-02 | Escopo inicial focado em WhatsApp | Canal principal de uso no Brasil | Outros canais fora do escopo atual |
| 2026-07-02 | Multi-tenancy com revendas | Modelo SaaS com white-label | Hierarquia de `Reseller` e `ResellerSubTenant` |
| 2026-07-02 | Planos com limites de uso | Faturamento simplificado | Modelos `Plan`, `Subscription`, `Invoice`, `Transaction` |
| 2026-07-02 | IA local como padrão | Reduzir custo e dependência de terceiros | Ollama integrado; provedores externos a confirmar |
| 2026-07-02 | Documentação estrutural obrigatória | Permitir colaboração segura | 7 arquivos de governança criados |

---

## 6. Pendências Atuais

| Pendência | Área | Prioridade | Próxima ação |
|---|---|---|---|
| Configurar suite de testes Node.js | Testes | Alta | Adicionar Jest/Mocha, testar rotas críticas |
| Configurar suite de testes PHP | Testes | Alta | Adicionar PHPUnit, testar controllers |
| Adotar migrations formais | Banco | Alta | Implementar Sequelize CLI ou similar |
| Remover secrets hardcoded | Segurança | Alta | Revisar `authMiddleware.js` e usar .env exclusivo |
| Criptografar `api_key` de IA | Segurança | Média | Criptografar em repouso |
| Definir TTL para `chat_history` | Performance | Média | Configurar índice TTL no MongoDB |
| Validar gateway de pagamento | Billing | Média | Identificar gateway e testar webhook |
| Validar integração OpenAI/Gemini | AI | Média | Verificar `services/ai/` e testar endpoints |
| Adicionar CI/CD | Infra | Média | Criar pipeline de build/teste/deploy |
| Configurar backups | Infra | Média | Backup de volumes PostgreSQL, MongoDB, MySQL |
| Melhorar observabilidade | Infra | Média | Métricas, health checks, alertas |
| Documentar manual de execução | Docs | Média | Criar guia passo a passo de instalação |
| Validar UI completa do frontend | Frontend | Média | Testar todas as views e fluxos de tela |

---

## 7. Bloqueios

| Bloqueio | Severidade | Descrição | Dependência |
|---|---|---|---|
| Ausência de testes automatizados | Alta | Impede validação segura de mudanças | Configuração de Jest/PHPUnit |
| `sequelize.sync({ alter: true })` | Média | Risco de alterar schema em produção sem controle | Adoção de migrations formais |
| Secrets hardcoded | Média | Risco de segurança se `.env` não for usado | Revisão de código e variáveis de ambiente |
| CI/CD ausente | Média | Deploy manual e sem validação automática | Definição de pipeline |

---

## 8. Riscos Técnicos

| Risco | Impacto | Mitigação |
|---|---|---|
| Baileys sofre bloqueio ou instabilidade | Alto | Monitorar, considerar integração oficial Meta API como alternativa futura |
| Crescimento ilimitado de mensagens no MongoDB | Médio | Definir TTL e rotina de arquivamento |
| Falta de testes gera regressões | Alto | Configurar suite de testes e executar antes de commits |
| Vazamento de secrets | Alto | Remover fallbacks, usar gestão de secrets, criptografar credenciais |
| Dependência de Ollama local dificulta deploy | Médio | Documentar requisitos, oferecer provedores alternativos |
| Schema PostgreSQL não versionado | Médio | Adotar migrations e backup antes de alterações |

---

## 9. Próximos Passos

1. **Validar ambiente local** com Docker Compose e registrar resultado em `RELATORIO.md`.
2. **Configurar suite de testes** para Node.js (Jest/Mocha) e PHP (PHPUnit).
3. **Adotar migrations formais** para PostgreSQL (Sequelize CLI).
4. **Revisar segurança** de secrets e credenciais de IA.
5. **Testar fluxos principais** (auth, WhatsApp, chat, flow, billing) e documentar.
6. **Atualizar `RELATORIO.md`** ao final de cada sessão de trabalho.

---

## 10. Notas Importantes para Próximos Agentes

- Sempre ler este arquivo antes de trabalhar.
- Sempre atualizar este arquivo ao final da sessão.
- Não remover histórico antigo.
- Registrar decisões, bloqueios e mudanças de direção.
- Nunca modificar legados .NET (`SaaS-Chatbot.sln`) e Python (`requirements.txt`, `Dockerfile.python`) sem autorização explícita.
- Sempre verificar impacto em `ARQUITETURA.md`, `BANCO_DADOS.md`, `ESCOPO.md`, `ROADMAP.md` e `RELATORIO.md`.
- Se não conseguir confirmar uma informação, escrever `A CONFIRMAR`, `PENDENTE DE VALIDAÇÃO` ou `NÃO IDENTIFICADO NO REPOSITÓRIO`.
- Não executar commit nem push sem autorização do usuário.
