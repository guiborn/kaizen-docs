---
layout: page
title: "Cloud Functions e API REST"
permalink: /kaizen/tecnico/api/
category: "Backend"
order: 8
toc: true
---

# Kaizen — Cloud Functions e Kaizen Firestore API

## Sumário
1. [Visão Geral das Camadas de Backend](#1-visão-geral-das-camadas-de-backend)
2. [Cloud Functions (kaizen-functions)](#2-cloud-functions-kaizen-functions)
3. [Kaizen Firestore API (kaizen-firestore-api)](#3-kaizen-firestore-api-kaizen-firestore-api)
4. [Padrões de Segurança e Autenticação](#4-padrões-de-segurança-e-autenticação)
5. [Monitoramento e Observabilidade](#5-monitoramento-e-observabilidade)

---

## 1. Visão Geral das Camadas de Backend

O Kaizen possui **duas camadas de backend** complementares, com responsabilidades distintas:

```
┌──────────────────────────────────────────────────────────────────┐
│                       CLIENTES / SISTEMAS                        │
│  App Flutter (usuários)  │  Data Lake / BI  │  Sistemas Externos │
└──────────┬───────────────┴────────┬──────────┴───────────────────┘
           │ Firebase JWT           │ API Key (x-api-key)
           │                        │
┌──────────▼────────────┐  ┌────────▼──────────────────────────────┐
│  Cloud Functions      │  │  Kaizen Firestore API                  │
│  (kaizen-functions)   │  │  (kaizen-firestore-api)                │
│                       │  │                                         │
│  • Notificações FCM   │  │  • REST endpoints estruturados          │
│  • Integrações ext.   │  │  • Para Data Lake e BI externos         │
│  • Agentes KAI        │  │  • Filtros, paginação, aggregações      │
│  • Triggers Firestore │  │  • OpenAPI 3.0 documentado              │
│  • Funções agendadas  │  │                                         │
└──────────┬────────────┘  └────────┬──────────────────────────────┘
           │                        │
           └──────────┬─────────────┘
                      │
           ┌──────────▼──────────┐
           │     Firestore       │
           │  (fonte de verdade) │
           └─────────────────────┘
```

**Cloud Functions** são acionadas principalmente pelo app Flutter (usuários autenticados) e por eventos internos (triggers Firestore, agendadores).

**Kaizen Firestore API** é um servidor Express.js que expõe endpoints REST com autenticação por API Key, projetado para integração com ferramentas de BI (Power BI, Tableau, Azure Synapse, etc.) e sistemas externos de dados.

---

## 2. Cloud Functions (kaizen-functions)

As Cloud Functions são funções serverless Node.js hospedadas no **Firebase Functions v2** (`us-central1`), auto-escaláveis e pagas por execução.

### 2.1 Categorias de Funções

#### Notificações e Triggers Firestore

Funções reativas que disparam automaticamente quando documentos Firestore são criados ou modificados:

| Função | Trigger | O que faz |
|--------|---------|-----------|
| `onRestrictionWrite` | `onDocumentWritten` em restrições | Detecta mudança de responsável; envia FCM ao novo responsável; grava no inbox de notificações (`users/{uid}/notifications`) |
| `onCommentWrite` | `onDocumentWritten` em comentários | Notifica membros mencionados em comentários de qualquer entidade |
| `onInspectionFinalWritten` | `onDocumentWritten` em inspeções | Notifica equipe ao registrar uma inspeção final de unidade |
| `onPedidoCreated` | `onDocumentCreated` em pedidos | Notifica aprovadores via FCM (tópico `pedidos_{clientId}_{branchId}`) ao criar novo pedido de compra |
| `onRequisitionWrite` | `onDocumentWritten` em requisições | Notifica responsável ao criar ou atualizar uma requisição de material |
| `onWorkOrderChatter` | `onDocumentWritten` em chatter WO | Notifica técnico ao receber mensagem no chatter de uma Work Order |

#### Funções Agendadas (Scheduled Functions)

Executadas automaticamente em intervalos configurados:

| Função | Periodicidade | O que faz |
|--------|--------------|-----------|
| `sendUpcomingAppointmentReminders` | A cada 5 minutos | Verifica agendamentos AT que começam em ~30 minutos; envia lembrete via FCM ao técnico responsável; marca `reminderSent = true` para não reenviar |
| `checkDueRestrictions` | Diário | Verifica restrições com prazo vencendo em breve; envia alertas preventivos via FCM e e-mail |
| `supplyAlerts` | Configurável | Verifica itens do Farol de Contratações com SLA em risco; dispara alertas para equipe de suprimentos |

#### Funções HTTP — Integrações Externas

Endpoints REST chamados pelo app Flutter com autenticação Firebase JWT:

**E-mail (Resend):**

| Endpoint | Método | Descrição |
|---------|--------|-----------|
| `sendEmail` | POST | Envio de e-mails transacionais via Resend: notificações de restrições, relatórios de planos de ação, alertas de suprimentos. Suporta anexos em base64 (PDFs, planilhas) |

**Integração Salesforce (Módulo AT):**

| Função | Descrição |
|--------|-----------|
| `salesforceWorkOrders` | Lista e detalha Work Orders do Salesforce com filtros (status, filial, tipo, período, responsável) |
| `salesforceWorkOrderLineItems` | Itens de linha de uma Work Order (peças e serviços) |
| `salesforceWorkPlans` | Planos de trabalho vinculados às Work Orders |
| `salesforceWorkSteps` | Etapas dos planos de trabalho (checklists de execução) |
| `salesforceAssets` | Assets do Salesforce vinculados a unidades do empreendimento |
| `salesforceAuth` | Gerenciamento de tokens OAuth2 Salesforce (JWT Bearer) |
| `salesforceTaxonomy` | Taxonomia de tipos de serviço e categorias do Salesforce |
| `salesforceWorkOrderSuggestTaxonomy` | Sugere categoria/tipo de serviço para uma Work Order via KAI |
| `togglePedidoTopicSubscription` | Inscreve/desincreve device em tópico FCM de pedidos de compra |

**Integração Azure Synapse:**

| Função | Descrição |
|--------|-----------|
| `azureSynapseQuery` | Executa queries no Azure Synapse Analytics para consulta de custo realizado; retorna dados para o módulo de controle de orçamento |

**Integração Mega ERP:**

| Função | Descrição |
|--------|-----------|
| `megaProxy` | Proxy autenticado para chamadas ao Mega ERP; consulta saldo de materiais e catálogo de produtos sem expor credenciais ao app |

**AI / KAI:**

| Função | Modelo | Descrição |
|--------|--------|-----------|
| `generateRestrictionReport` | `kai-restriction-analyst` | Gera relatório gerencial de restrições em markdown |
| `generateActionPlanReport` | `kai-actionplan-analyst` | Gera relatório de execução de planos de ação |
| `generatePlanReport` | `kai-plan-analyst` | Gera análise do planejamento para reuniões de produção |
| `suggestRestrictions` | `kai-suggestion` | Sugere restrições a catalogar com base no contexto da obra |
| `identifyProblems` | `kai-problems` | Identifica problemas e anomalias na execução |
| `kbUpsertText` | Gemini `text-embedding-004` | Indexa documento na base de conhecimento da obra (RAG) |
| `kbQuery` | Gemini `text-embedding-004` | Busca semântica na base de conhecimento da obra |

**Utilitários:**

| Função | Descrição |
|--------|-----------|
| `excelToJson` | Converte arquivos Excel (.xlsx) enviados via upload para JSON estruturado; usado na importação de colaboradores, itens de cotação e materiais |
| `previsionProxy` | Proxy para consulta de dados de cronograma na plataforma Prevision.com.br (planejamento integrado) |

#### Sistema de Notificações no Inbox

Além do FCM (push notification), as Cloud Functions mantêm um **inbox de notificações** por usuário no Firestore:

```
users/{uid}/notifications/{notifId}
  ├── title        — Título da notificação
  ├── body         — Corpo da mensagem
  ├── type         — Tipo (restriction_assigned, pedido_created, etc.)
  ├── entityId     — ID da entidade relacionada
  ├── entityType   — Tipo da entidade (restriction, workOrder, etc.)
  ├── siteId       — ID da obra
  ├── clientId     — ID do tenant
  ├── read         — false (marcado true ao visualizar)
  └── createdAt    — Timestamp de criação
```

O app Flutter lê esse inbox em tempo real (Firestore stream) e exibe o Centro de Notificações no menu superior. Notificações push (FCM) e inbox são sempre entregues juntos, garantindo que mesmo com o dispositivo offline, o usuário veja a notificação ao abrir o app.

### 2.2 Configuração e Deploy

- Código-fonte: `kaizen-functions/`
- Runtime: Node.js 20
- Deploy: `firebase deploy --only functions`
- Secrets: gerenciados via Google Cloud Secret Manager (RESEND_API_KEY, OPENWEBUI_API_KEY, GEMINI_API_KEY, credenciais Salesforce, Synapse, Mega)
- Variáveis de ambiente: `kaizen-functions/.env.{projectId}` (não versionadas)

---

## 3. Kaizen Firestore API (kaizen-firestore-api)

A **Kaizen Firestore API** é uma camada REST dedicada para integração do Kaizen com **sistemas de dados externos** — Data Lake, Azure Synapse, Power BI, dashboards de BI corporativos e qualquer sistema que precise consumir dados operacionais do Kaizen de forma estruturada.

### 3.1 Propósito

Enquanto as Cloud Functions atendem o app Flutter (usuários humanos autenticados via Firebase), a Kaizen Firestore API serve **sistemas máquina-a-máquina**:

- Pipelines de dados do Azure Data Factory
- Queries do Azure Synapse Analytics
- Dashboards Power BI via conector REST
- Scripts de ETL e relatórios automatizados
- Sistemas de BI da construtora

### 3.2 Autenticação

A API usa **API Key por tenant** via header HTTP:

```
GET /api/clients/{clientId}/default_sites/{siteId}/restrictions
x-api-key: {chave_do_tenant}
```

Cada API Key é mapeada a um `clientId` específico. Tentativas de acessar dados de um `clientId` diferente da chave são bloqueadas com `403 Forbidden`.

A documentação interativa da API (Swagger UI) é acessível publicamente em `/docs` sem autenticação.

### 3.3 Endpoints Disponíveis

#### Restrições

| Endpoint | Método | Descrição |
|---------|--------|-----------|
| `GET /api/clients/{clientId}/default_sites` | GET | Lista todas as obras do cliente |
| `GET /api/clients/{clientId}/default_sites/{siteId}/restrictions` | GET | Lista restrições de uma obra com filtros |
| `GET /api/clients/{clientId}/restrictions` | GET | Lista restrições de todas as obras do cliente |
| `GET /api/clients/{clientId}/default_sites/{siteId}/restrictions/{id}` | GET | Detalhe de uma restrição |
| `POST /api/clients/.../restrictions` | POST | Cria restrição via API externa |
| `PUT /api/clients/.../restrictions/{id}` | PUT | Atualiza restrição |
| `DELETE /api/clients/.../restrictions/{id}` | DELETE | Remove restrição |

**Filtros disponíveis para listagem:** `updatedAt_gte`, `updatedAt_lte`, `status`, `responsible`, `type`

#### Desvios Orçamentários

| Endpoint | Descrição |
|---------|-----------|
| `GET /api/clients/{clientId}/default_sites/{siteId}/deviations` | Lista desvios com filtros de data e valor |
| `GET /api/clients/{clientId}/deviations` | Desvios consolidados de todas as obras |

#### Farol de Contratações (Supply Forecast)

| Endpoint | Descrição |
|---------|-----------|
| `GET /api/clients/{clientId}/default_sites/{siteId}/lighthouse/latest` | Snapshot mais recente do farol de suprimentos |
| `GET /api/clients/{clientId}/default_sites/{siteId}/lighthouse/versions` | Versões históricas do farol |

Retorna itens "achatados" (`LighthouseItem`) com: `itemName`, `itemType`, `itemGroup`, `activitiesStartAt/EndAt`, `forecastStartAt/EndAt`, `leadTimeDays`, `slaCount`, etc.

#### Processos de Suprimento

| Endpoint | Descrição |
|---------|-----------|
| `GET /api/clients/{clientId}/default_sites/{siteId}/supply-processes` | Lista processos de contratação com status e movimentos recentes |

Campos: `id`, `siteId`, `description`, `status`, `currentSector`, `updatedAt`, `startedAt`, `finishedAt`.

#### Planos de Ação

| Endpoint | Descrição |
|---------|-----------|
| `GET /api/clients/{clientId}/default_sites/{siteId}/action-plans` | Lista planos de ação da obra com filtros |
| `GET /api/clients/{clientId}/action-plans` | Planos consolidados por cliente |

Filtros: `status`, `responsible`, `updatedAt_gte`, `updatedAt_lte`, `dueDate_gte`, `dueDate_lte`.

#### Medições Físicas

| Endpoint | Descrição |
|---------|-----------|
| `GET /api/clients/{clientId}/default_sites/{siteId}/physical-measurements` | Avanço físico registrado por obra, por período e por atividade |

Dados exportados para o Data Lake para construção de curvas S e indicadores de performance no BI.

### 3.4 Documentação Interativa (Swagger UI)

A API possui documentação completa em **OpenAPI 3.0** acessível via:

```
https://{base-url}/docs
```

A interface Swagger UI permite explorar todos os endpoints, visualizar schemas de request/response e testar chamadas diretamente pelo navegador (com inserção da API Key).

O arquivo de especificação também está disponível em:
```
GET /docs.json
```

Isso permite que ferramentas de BI (como Power BI com conector OpenAPI) descubram automaticamente os endpoints disponíveis.

### 3.5 Formatos de Resposta

Todas as respostas seguem o padrão:

```json
{
  "data": [...],          // array de resultados
  "total": 142,           // total de registros (para paginação)
  "page": 1,
  "pageSize": 50,
  "hasMore": true
}
```

Erros retornam:
```json
{
  "error": "Mensagem descritiva",
  "code": "ERROR_CODE"
}
```

### 3.6 Casos de Uso com Data Lake (Azure Synapse)

A Kaizen Firestore API viabiliza os seguintes cenários de integração com o Azure Synapse:

| Cenário | Como funciona |
|---------|--------------|
| **Carga diária de restrições** | Pipeline ADF chama `GET /restrictions?updatedAt_gte={yesterday}` e carrega incrementos na tabela `facts_restricoes` do Synapse |
| **Dashboard de desvios no Power BI** | Power BI usa conector REST com `GET /deviations` para construir análises de variação orçamentária |
| **Rastreabilidade de suprimentos** | Script ETL consome `GET /supply-processes` e `GET /lighthouse` para alimentar relatório de lead time e SLA |
| **Curva S no BI** | `GET /physical-measurements` alimenta a série histórica de avanço físico para construção da curva S no Synapse/Power BI |
| **Reconciliação de planos de ação** | Pipeline semanal compara planos abertos no Kaizen vs. itens auditados no ERP corporativo |

### 3.7 Deploy e Configuração

- Código-fonte: `kaizen-firestore-api/`
- Runtime: Node.js (Express.js)
- Deploy: como Cloud Function (`onRequest`) via `firebase deploy`
- Autenticação: `.env` com mapeamento `CLIENT_A_API_KEY` → `CLIENT_A_ID` por tenant
- API Keys: geradas pelo time Kaizen e entregues securely ao cliente para configuração em seus pipelines

---

## 4. Padrões de Segurança e Autenticação

### 4.1 Cloud Functions — Firebase JWT

```
App Flutter → Authorization: Bearer {idToken}
Cloud Function → admin.auth().verifyIdToken(token)
  → Extrai: uid, clientId, userType, branchId (custom claims)
  → Se inválido: retorna HTTP 401
  → Se válido: executa a operação com escopo do tenant
```

- Tokens expiram em 1 hora (renovados automaticamente pelo SDK Firebase no app)
- Custom claims definem o tenant e tipo de usuário no token, sem consulta extra ao banco
- Chamadas internas entre Cloud Functions usam uma chave interna (`x-internal-key`) em vez de JWT, para evitar overhead de renovação de token em chamadas machine-to-machine

### 4.2 Kaizen Firestore API — API Key

```
Sistema externo → x-api-key: {apiKey}
clientAuth middleware → apiKeyToClientId[apiKey]
  → Se não encontrado: retorna HTTP 401
  → Se clientId do path ≠ clientId da chave: retorna HTTP 403
  → Se válido: req.clientId = resolvedClientId, passa para o controller
```

- Uma API Key por tenant/cliente
- API Keys não expiram automaticamente (revogadas manualmente se necessário)
- Endpoint `/docs` é público (sem autenticação) para facilitar integração por equipes de BI

### 4.3 CORS

Cloud Functions restringem CORS a origens explicitamente permitidas:
- `https://kaizen-v3.web.app`
- `https://kaizen-v3.firebaseapp.com`
- `http://localhost:5173` (desenvolvimento)

Requisições de origens não listadas são bloqueadas com erro `403 CORS`.

A Kaizen Firestore API aceita qualquer origem (`cors()` aberto), pois a autenticação é por API Key no header — adequado para integração com sistemas de BI que não têm origem fixa.

---

## 5. Monitoramento e Observabilidade

### 5.1 Cloud Logging (GCP)

Todas as Cloud Functions emitem logs estruturados para o Google Cloud Logging:

- **Logs de invocação:** função chamada, duração, status HTTP
- **Logs de negócio:** ações relevantes com contexto (`[tenant ${clientId}] Synced ${count} work orders`)
- **Logs de erro:** stack trace completo + identificadores (tenantId, entityId, functionName)

Filtros disponíveis por `tenantId`, `functionName`, nível de severidade e intervalo de tempo.

### 5.2 Alertas de Função

Configurados no Firebase Console / GCP Monitoring:
- Alertas de taxa de erros acima do threshold
- Alertas de timeout de função (cold start excessivo)
- Alertas de fallback Salesforce (quando a API Salesforce está indisponível)

### 5.3 Métricas Coletadas

| Métrica | Onde visualizar |
|---------|----------------|
| Invocações por função / dia | Firebase Console > Functions |
| Taxa de erros | GCP Cloud Monitoring |
| Latência (p50/p95/p99) | GCP Cloud Monitoring |
| Custo por função | Firebase Console > Usage |
| Notificações FCM entregues | Firebase Console > Cloud Messaging |

---

*Documento: Cloud Functions e Kaizen Firestore API | Versão 1.0 | Kaizen Construction Management*
