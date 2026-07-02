# Especificação: Contatos e Tags

**Código:** SPEC-CONTACTS  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que clientes finais gerenciem sua base de contatos, segmentem por tags, importem listas em lote e sincronizem contatos do WhatsApp.

---

## 2. Contexto

A base de contatos é essencial para automação, campanhas e atendimento. Cada contato pertence a um tenant e pode ter múltiplas tags. Contatos podem ser criados manualmente, importados ou sincronizados a partir do WhatsApp conectado.

### Usuários envolvidos

- Cliente final (admin do tenant).
- Atendente (visualiza contatos).
- Revenda (não gerencia contatos diretamente, apenas sub-tenants).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-CONTACT-001 | Contato é identificado por `phone_number` dentro do tenant. | `contacts` |
| RN-CONTACT-002 | Email de contato, se informado, deve ser válido. | Validação |
| RN-CONTACT-003 | Cada tenant tem seu próprio conjunto de contatos e tags. | Multi-tenancy |
| RN-CONTACT-004 | Uma tag pode estar associada a múltiplos contatos. | `tags` / `contact_tags_assoc` |
| RN-CONTACT-005 | Contatos podem ser importados em lote via CSV ou JSON. | `contactsController.js` |
| RN-CONTACT-006 | Sincronização de contatos do WhatsApp não sobrescreve dados existentes sem confirmação. | `contactsController.js` |
| RN-CONTACT-007 | Contatos em blacklist não devem receber campanhas. | `contacts` (`is_blacklisted`) |
| RN-CONTACT-008 | Foto de perfil do WhatsApp é temporária e pode ser atualizada. | `contacts` (`profile_pic_url`) |

---

## 4. Fluxo Esperado

### 4.1 Cadastrar contato

1. Usuário envia nome, telefone, email e tags opcionais.
2. Sistema valida telefone único por tenant.
3. Sistema cria contato e associa tags.
4. Sistema retorna contato criado.

### 4.2 Importar contatos

1. Usuário faz upload de arquivo CSV/JSON.
2. Sistema valida formato e campos obrigatórios.
3. Sistema processa registros, criando contatos e tags.
4. Sistema retorna resumo: criados, atualizados, erros.

### 4.3 Sincronizar contatos do WhatsApp

1. Usuário solicita scan de contatos do WhatsApp conectado.
2. Sistema consulta Baileys e retorna lista.
3. Usuário seleciona contatos para importar.
4. Sistema cria/atualiza contatos.

### 4.4 Segmentar por tags

1. Usuário cria tag com nome e cor.
2. Usuário associa tag a contatos.
3. Sistema permite filtrar campanhas e conversas por tag.

---

## 5. Critérios de Aceite

- [ ] CRUD de contatos funciona por tenant.
- [ ] Telefone duplicado no mesmo tenant é rejeitado.
- [ ] Importação em lote processa corretamente.
- [ ] Sincronização do WhatsApp traz contatos e fotos de perfil.
- [ ] Tags são criadas e associadas a contatos.
- [ ] Contatos em blacklist são excluídos de campanhas.
- [ ] Listagem de contatos respeita paginação e filtros.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/contactsController.js`
- `node-version/src/models/sql/models.js` (Contact, Tag)
- `node-version/src/services/whatsappCore.js` (para sincronização)

### Frontend
- `chatbot/src/Controller/HomeController.php`
- Views de contatos.

### Banco
- PostgreSQL: `contacts`, `tags`, `contact_tags_assoc`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Unitário | Validação de telefone | Média |
| Integração | Criar contato | Alta |
| Integração | Rejeitar telefone duplicado no mesmo tenant | Alta |
| Integração | Importar CSV com sucesso | Média |
| Integração | Importar CSV com erros parciais | Média |
| Integração | Sincronizar contatos do WhatsApp | Média |
| Integração | Associar tags a contatos | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Importação de arquivo mal formatado | Médio | Validar cabeçalho e tipos de dados |
| Duplicidade de contatos entre tenants | Baixo | Isolamento por `tenant_id` |
| Blacklist não respeitada em campanhas | Médio | Filtro obrigatório nas queries de campanha |

---

## 9. Dependências

- PostgreSQL operacional.
- WhatsApp conectado para sincronização.

---

## 10. A CONFIRMAR / PENDENTE

- Regras automáticas de blacklist.
- Campos customizados de contato.
- Segmentação avançada com múltiplos critérios.
- Exportação de contatos.
