# AGENTS.md — Regras de Execução dos Agentes

**Projeto:** SaaS Chatbot / Chatbot_Mauricio  
**Atualizado em:** 2026-07-02  
**Versão:** 1.0  

---

## 1. Objetivo deste arquivo

Este documento define as regras oficiais para agentes de IA, automações e colaboradores técnicos que atuarem no repositório.

Ele estabelece:

- Como agentes devem operar no projeto.
- Como investigar o contexto antes de modificar qualquer arquivo.
- Como documentar mudanças de código.
- Como aplicar Specification-Driven Development (SDD) e Test-Driven Development (TDD).
- Como padronizar commits em português do Brasil usando Conventional Commits.
- Como manter `ARQUITETURA.md`, `BANCO_DADOS.md`, `ESCOPO.md`, `ROADMAP.md`, `CONTEXTO.md` e `RELATORIO.md` atualizados.

---

## 2. Princípios obrigatórios

- Sempre buscar contexto antes de alterar código, banco, configuração ou documentação.
- Sempre trabalhar com mudanças pequenas, focadas e rastreáveis.
- Sempre atualizar documentação junto com o código.
- Nunca inventar comportamento, endpoints, schemas, tabelas ou regras de negócio.
- Nunca mascarar erro com mock, stub ou fallback falso sem documentar explicitamente.
- Nunca expor secrets, tokens, senhas, chaves privadas ou credenciais.
- Nunca fazer commit ou push sem autorização explícita do usuário.
- Nunca deixar o projeto em estado quebrado ou não executável.
- Sempre registrar decisões relevantes em `CONTEXTO.md`.
- Sempre respeitar o idioma técnico, claro, objetivo e profissional em português do Brasil.

---

## 3. Investigação obrigatória antes de modificar

Antes de criar ou editar qualquer arquivo, o agente deve:

1. Inspecionar a estrutura do repositório (`list_dir`, `find_by_name`).
2. Identificar a stack real do projeto lendo:
   - `README.md`
   - `package.json` / `package-lock.json`
   - `requirements.txt` / `pyproject.toml`
   - `composer.json`
   - `*.csproj` / `*.sln`
   - `docker-compose.yml` / `Dockerfile` / `Dockerfile.*`
   - `tsconfig.json`
   - arquivos de migrations / schemas SQL
   - diretórios `src/`, `app/`, `backend/`, `frontend/`, `api/`, `docs/`, `tests/`
3. Confirmar requisitos e comportamento existentes antes de propor mudanças.
4. Quando uma informação não puder ser confirmada, escrever:
   - `A CONFIRMAR`
   - `PENDENTE DE VALIDAÇÃO`
   - `NÃO IDENTIFICADO NO REPOSITÓRIO`
5. Não alterar código funcional do sistema sem necessidade comprovada.
6. Não alterar arquivos de configuração sem autorização explícita.
7. Não apagar documentação existente.
8. Se um arquivo já existir, atualizar preservando histórico, decisões e informações úteis.

---

## 4. Regras de commit em pt-BR usando Conventional Commits

Todo commit deve ser escrito em **português do Brasil** e seguir o padrão **Conventional Commits** adaptado.

### Tipos permitidos

- `feat:` nova funcionalidade
- `fix:` correção de bug
- `docs:` documentação
- `test:` testes
- `refactor:` refatoração sem mudança funcional
- `style:` formatação sem mudança lógica
- `chore:` tarefas auxiliares, scripts, configurações simples
- `ci:` pipeline, deploy e integração contínua
- `perf:` melhoria de performance
- `build:` mudanças de build, dependências ou empacotamento
- `revert:` reversão de alteração anterior

### Formato obrigatório

```text
<tipo>(<escopo>): <descrição curta em pt-BR>

<corpo opcional explicando o motivo, impacto e arquivos principais>
```

### Exemplos

```text
feat(auth): adiciona autenticação com JWT
fix(api): corrige validação de payload no cadastro de cliente
docs(arquitetura): atualiza fluxo de autenticação
test(banco): adiciona testes para migrations de usuários
refactor(frontend): reorganiza componentes do dashboard
```

### Regras adicionais

- Um commit por tarefa concluída.
- Não misturar mudanças não relacionadas.
- Descrição curta no imperativo ou presente.
- Escopo deve refletir a área alterada: `api`, `auth`, `frontend`, `backend`, `banco`, `docs`, `infra`, `tests`, `dashboard`, `admin`, `mobile`, `whatsapp`, `billing`, etc.
- Antes de commit, validar testes, build e documentação.
- Commit e push só podem ser feitos quando o usuário autorizar explicitamente.

---

## 5. SDD — Specification-Driven Development

Toda funcionalidade deve começar por especificação. Antes de codificar, o agente deve:

1. Ler a documentação existente.
2. Confirmar o requisito com evidência no código ou com o usuário.
3. Criar ou atualizar a especificação na documentação.
4. Definir critérios de aceite claros.
5. Mapear impacto em:
   - arquitetura
   - banco de dados
   - API
   - frontend
   - testes
   - infraestrutura
   - segurança
6. Registrar decisões relevantes em `CONTEXTO.md`.
7. Atualizar `ROADMAP.md` se a tarefa fizer parte de fase, épico ou história.

### Toda especificação deve conter

- Objetivo
- Contexto
- Regras de negócio
- Fluxo esperado
- Critérios de aceite
- Impacto técnico
- Testes necessários
- Riscos
- Dependências

---

## 6. TDD — Test-Driven Development

O fluxo obrigatório é:

1. **RED** — escrever ou ajustar o teste que falha.
2. **GREEN** — implementar o mínimo necessário para o teste passar.
3. **REFACTOR** — melhorar o código mantendo os testes verdes.

### Regras

- Toda feature nova precisa de teste.
- Todo bug corrigido precisa de teste de regressão.
- Toda regra de negócio crítica precisa de teste.
- Toda migration relevante precisa ser validada.
- Não reduzir cobertura de testes sem justificativa em `RELATORIO.md`.
- Se a stack não tiver testes configurados, registrar isso e propor configuração inicial.

### Comandos de teste/build detectados no repositório

```bash
# Node.js (backend monolith /node-version)
npm install
npm run start
npm run dev

# TypeScript (raiz)
npm run build

# Python (legado / Dockerfile.python)
pip install -r requirements.txt
pytest
python -m pytest
uvicorn src.main:app --host 0.0.0.0 --port 8000

# PHP (frontend /chatbot)
composer install
php -S localhost:8000 -t public public/router.php

# Docker
podman compose up --build
docker compose up --build
```

> Atenção: a suite de testes formais não está completamente configurada para Node.js e PHP. Scripts de teste manuais existem em `/node-version/test_*.js` e `/chatbot/test_*.js`. Registrar como `PENDENTE DE VALIDAÇÃO` quando não houver teste automatizado.

---

## 7. Atualização obrigatória de documentação

Toda tarefa técnica deve atualizar documentação conforme impacto:

- `ARQUITETURA.md` — quando houver mudança estrutural, módulo, fluxo ou integração.
- `BANCO_DADOS.md` — quando houver mudança em tabelas, models, migrations, índices, constraints ou seeds.
- `ESCOPO.md` — quando houver mudança de requisito, regra de negócio ou limite do projeto.
- `ROADMAP.md` — quando houver mudança de fase, épico, prioridade ou tarefa.
- `CONTEXTO.md` — quando houver decisão técnica, bloqueio, mudança importante ou estado novo.
- `RELATORIO.md` — ao final de cada sessão/dia de trabalho.

### Checklist obrigatório por tarefa

```markdown
- [ ] Requisito compreendido
- [ ] Especificação criada/atualizada
- [ ] Teste criado/atualizado
- [ ] Implementação validada
- [ ] Documentação atualizada
- [ ] Arquitetura atualizada, se aplicável
- [ ] Banco de dados atualizado, se aplicável
- [ ] Roadmap atualizado, se aplicável
- [ ] Contexto atualizado
- [ ] Relatório do dia atualizado
```

---

## 8. Segurança e integridade

- Nunca expor `.env`, tokens, senhas, chaves privadas ou credenciais.
- Nunca copiar secrets para documentação.
- Antes de registrar exemplos, usar valores fictícios seguros.
- Não criar backdoors, bypass de autenticação ou desativar validações sem autorização.
- Validar entradas de usuário em todos os endpoints.
- Documentar riscos de segurança encontrados.
- Registrar pendências críticas em `CONTEXTO.md`.
- Sempre que encontrar segredos hardcoded, sinalizar como débito técnico de segurança e não propagar.

---

## 9. Conduta para agentes

- Ser conservador em mudanças.
- Priorizar consistência com padrões existentes.
- Não reescrever o projeto sem necessidade.
- Não substituir bibliotecas principais sem justificativa documentada.
- Não remover testes existentes.
- Não apagar histórico de documentação.
- Em caso de dúvida, investigar antes de perguntar.
- Se ainda houver dúvida, registrar como `A CONFIRMAR`.
- Não executar commit nem push sem autorização explícita do usuário.

---

## 10. Regras específicas deste repositório

- O backend ativo está em `/node-version` (Node.js/Express/Sequelize/Mongoose).
- O frontend ativo está em `/chatbot` (PHP 8 MVC com Apache).
- O legado .NET está na solution `SaaS-Chatbot.sln` e não deve ser modificado sem autorização.
- O legado Python está em `requirements.txt` / `Dockerfile.python` e não deve ser alterado sem justificativa.
- O banco de dados do backend Node usa PostgreSQL via `sequelize.sync({ alter: true })` — não há migrations formais identificadas.
- O banco de dados do frontend PHP usa MySQL via `chatbot/database/schema.sql`.
- MongoDB é usado para mensagens e fluxos (`chat_history`, `flows`, `flow_sessions`).
- RabbitMQ, Redis e Ollama são dependências de infraestrutura declaradas no `docker-compose.yml`.
- Sempre validar alterações contra `docker-compose.yml` antes de propor mudanças de infraestrutura.

---

## 11. Checklist de entrada em nova sessão

Antes de começar a trabalhar, o agente deve:

1. Ler `CONTEXTO.md` e `RELATORIO.md`.
2. Verificar `ROADMAP.md` para entender a fase atual.
3. Confirmar a stack ativa em `ARQUITETURA.md`.
4. Revisar `BANCO_DADOS.md` antes de tocar em models ou queries.
5. Verificar pendências e bloqueios em `CONTEXTO.md`.
