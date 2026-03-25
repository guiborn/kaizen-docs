---
layout: page
title: "Integrações com Sistemas Externos"
permalink: /tecnico/integracoes/
category: "Integrações"
order: 5
toc: true
---

# Kaizen — Integrações com Sistemas Externos

## Sumário
1. [Salesforce CRM / Field Service (FSL)](#1-salesforce-crm--field-service-fsl)
2. [Prevision — Planejamento de Obras](#2-prevision-planejamento-de-obras)
3. [Azure Synapse Analytics](#3-azure-synapse-analytics)
4. [Mega ERP (Materiais)](#4-mega-erp-materiais)
5. [Firebase Cloud Messaging (Notificações)](#5-firebase-cloud-messaging-notificações)
   - [5.4 Central de Notificações (Inbox)](#54-central-de-notificações-inbox)
6. [Resumo das Integrações](#6-resumo-das-integrações)
7. [Considerações de Segurança](#7-considerações-de-segurança)

---

## 1. Salesforce CRM / Field Service (FSL)

### 1.1 Visão Geral

O Salesforce é a principal fonte de dados do Módulo de Assistência Técnica. As entidades Work Order, Service Appointment, Case e Asset são sincronizadas do Salesforce para o Kaizen, onde são enriquecidas com dados operacionais (agendamentos, colaboradores, materiais).

O Kaizen **não substitui** o Salesforce como CRM — ele consome os dados e garante que as equipes de campo tenham uma interface otimizada para o trabalho operacional.

### 1.2 Objetos Sincronizados

| Objeto Salesforce | Coleção Firestore | Direção | Frequência |
|------------------|------------------|---------|------------|
| `WorkOrder` | `workOrders` | SF → Kaizen | Incremental, ~15 min |
| `ServiceAppointment` | `serviceAppointments` | SF → Kaizen | Incremental, ~15 min |
| `Case` | `cases` | SF → Kaizen | Incremental, ~15 min |
| `Asset` | `units` (campo vinculado) | SF → Kaizen | Por demanda |
| Status de Work Order | SF via Cloud Function | Kaizen → SF | Por ação do usuário |

### 1.3 Mecanismo de Sincronização

**Sincronização Incremental (padrão):**
```
Cloud Scheduler → trigger a cada 15 min
      ↓
Cloud Function: syncSalesforceData()
      ↓
SOQL query: SELECT ... FROM WorkOrder 
            WHERE LastModifiedDate > :lastSyncTime
      ↓
Para cada registro retornado:
  mapSalesforceWorkOrder(sfRecord) → formato Firestore
  firestore.batch().set(docRef, data, { merge: true })
      ↓
batch.commit() (até 500 documentos por batch)
      ↓
Atualiza sync_metadata com novo lastSyncTime
```

**Idempotência:** a operação `set(..., { merge: true })` garante que executar a sincronização múltiplas vezes com os mesmos dados não gera duplicatas nem sobrescreve campos locais extras.

**Tratamento de falhas:**
- Falha total da API Salesforce → log de erro, aguarda próximo ciclo (sem retry imediato)
- Falha parcial (registro específico) → log do erro + ID do registro, continua os demais
- Rate limit Salesforce → consultas em bulk (queries de até 2.000 registros por chamada)

### 1.4 Autenticação

O Kaizen usa **OAuth 2.0 com JWT Bearer Grant** para autenticação servidor-a-servidor com o Salesforce:

```
Cloud Function
      ↓
Gera JWT assinado com chave privada (armazenada em env var Firebase)
      ↓
POST para https://login.salesforce.com/services/oauth2/token
      ↓
Salesforce retorna Access Token
      ↓
Todas as chamadas subsequentes incluem Authorization: Bearer {token}
```

- **Credenciais:** armazenadas em variáveis de ambiente do Firebase (nunca no código-fonte ou no app)
- **Usuário de integração:** perfil API-only no Salesforce com permissões mínimas necessárias
- **Escopos OAuth:** `api`, `refresh_token` (para fluxo de renovação)
- **Sandbox toggle:** `SandboxToggleService` permite alternar entre org de produção e sandbox para testes

### 1.5 Campos Sincronizados (Work Order)

| Campo Salesforce | Campo Firestore | Tipo |
|-----------------|----------------|------|
| `Id` | `sfId` (Doc ID) | String (18 chars) |
| `WorkOrderNumber` | `workOrderNumber` | String |
| `Status` | `status` | String |
| `Priority` | `priority` | String |
| `Description` | `description` | String |
| `WorkType.Name` | `workType` | String |
| `Account.Name` | `accountName` | String |
| `AccountId` | `accountId` | String (SFID) |
| `CreatedDate` | `createdDate` | Timestamp |
| `LastModifiedDate` | `lastModifiedDate` | Timestamp |
| `ScheduledStartTime` | `scheduledStart` | Timestamp |
| `ScheduledEndTime` | `scheduledEnd` | Timestamp |
| `Torre__c` | `tower` | String |
| `Unidade__c` | `unit` | String |
| `Branch__c` | `branch` | String |
| `Area__c` | `area` | String |
| `Project__c` | `project` | String |
| `DefectType__c` | `defectType` | String |
| `RootCause__c` | `rootCause` | String |
| `Resolution__c` | `resolution` | String |
| `PlannedHours` | `plannedHours` | Number |
| `ActualHours` | `actualHours` | Number |

Campos com sufixo `__c` são campos customizados do Salesforce. O modelo `WorkOrderItem` no Flutter usa `_getField()` para aceitar tanto `Field__c` quanto `Field` no nome, garantindo compatibilidade entre org com distintas configurações.

### 1.6 Cloud Functions Expostas (Endpoints de Integração)

| Cloud Function | Método | Descrição |
|----------------|--------|-----------|
| `getWorkOrders` | GET | Lista Work Orders com filtros e paginação |
| `getWorkOrdersByIds` | POST | Retorna WOs por lista de IDs específicos |
| `getCases` | GET | Lista Cases com filtros e paginação |
| `getUnitsAssets` | GET | Retorna Assets vinculados a unidades |
| `togglePedidoTopicSubscription` | POST | Inscreve/desinsce device em tópico de pedidos FCM |
| `updateWorkOrderStatus` | POST | Atualiza status de WO no Salesforce |
| `syncSalesforceData` | Agendado | Trigger de sincronização incremental (interno) |

### 1.7 Fallback para Azure Synapse

Quando a API Salesforce está indisponível (manutenção, rate limit, falha de rede), o sistema ativa automaticamente o fallback para dados do Azure Synapse:

```
Chamada ao Salesforce falha
      ↓
SynapseController.fetchWorkOrdersFallback()
      ↓
Query no Azure Synapse Analytics (via REST API)
      ↓
Retorna dados históricos/batch equivalentes
      ↓
Exibe para o usuário com indicador "Dados do Data Lake"
```

---

## 2. Prevision — Planejamento de Obras

### 2.1 Visão Geral

O **Prevision** é a plataforma de planejamento físico-financeiro integrada ao Kaizen. Fornece o cronograma mestre da obra, orçamento WBS, Curva S, Cronograma Físico-Financeiro (CFF) e dados de medição — todos consumidos via API GraphQL pelo `PrevisionController` e exibidos nos módulos de [Planejamento Físico-Financeiro](02_modulos_gestao_obra.md#2-planejamento-físico-financeiro) e [Controle de Produção](02_modulos_gestao_obra.md#3-controle-de-produção).

O Kaizen **não replica** o Prevision como ferramenta de planejamento — ele consome os dados já validados lá e os contextualiza para consumo por engenheiros, estagiários e gestores em campo ou no escritório.

### 2.2 Dados Importados do Prevision

| Objeto Prevision | Usado Em | Query GraphQL |
|-----------------|---------|----------------|
| `activitiesPage` | Tabela de Atividades, Gantt, Painel de Produção | `readPrevisionActivities` |
| `activitiesWithinPeriodReport` | Curva S (cálculo por período) | `readPrevisionActivitiesV2` |
| `criticalPath` | Destaque do caminho crítico | `readPrevisionCriticalPath` |
| `floorsPage` | Lotes / pavimentos | `readPrevisionLots` |
| `servicesPage` | Serviços da EAP | `readPrevisionServices` |
| `sCurve` | Curva S direto da API | `readPrevisionSCurveV2` |
| `budgetReportsPage` | CFF — lista de relatórios de orçamento | `readPrevisionCFF` |
| `budgetReport(id).cffTable` | CFF — distribuição temporal de custo por item WBS | `readPrevisionCffTable` |
| `budgetItemsPage` com `budgetWeights` | Vinculação orçamento × atividades | `readPrevisionBudgetWeights` |
| `measurementsTasksPage` | Lista datas de medição disponíveis | `readPrevisionMeasurementDates` |
| `measurementTask(id)` | Dados de medição por data selecionada | `readPrevisionMeasurementTaskData` |
| `calendar` | Calendário de dias úteis do projeto | `readPrevisionCalendar` |
| `projectsPage` | Lista de projetos disponíveis para vinculação | `readPrevisionProjects` |

### 2.3 Autenticação e Conexão

- **Protocolo:** GraphQL sobre HTTPS (`HttpLink` do pacote `graphql_flutter`)
- **Autenticação:** JWT Bearer Token — o token é obtido com as credenciais do usuário Prevision configuradas por tenant, armazenadas de forma segura nas variáveis de ambiente das Cloud Functions
- **Endpoint:** `https://api-docs.prevision.com.br` (configurável por tenant)
- **Vinculação:** cada obra no Kaizen possui um campo `previsionProjectId` que mapeia ao ID do projeto na plataforma Prevision; sem esse vínculo, os módulos de planejamento ficam inacessíveis

### 2.4 Mecanismo de Sincronização

Diferente do Salesforce (push incremental), a integração com o Prevision é **pull sob demanda**:

```
Usuário abre módulo de Planejamento
      ↓
PrevisionController verifica lastPrevisionSync
      ↓
Se desatualizado (ou forçado via botão "Sincronizar"):
  Executa sequência de queries GraphQL paginadas
  Atividades → Lotes → Serviços → Curva S → CFF → Budget Weights
      ↓
Armazena resultados em Hive (cache local) e no cronograma no Firestore
      ↓
Widgets reativas reconstruídas via Obx/GetX
```

**Paginação:** Todas as queries usam `first` + `after` (cursor-based) para varrer conjuntos grandes. O controller detecta `hasNextPage` e executa queries adicionais automaticamente até obter todo o conjunto.

**Cache e versionamento:** O documento do cronograma no Firestore armazena `cffCalculatedAt` e `cffVersion`, evitando reprocessamento quando os dados não mudaram.

### 2.5 Funcionalidades Dependentes do Prevision

| Funcionalidade Kaizen | Dependência Prevision |
|----------------------|----------------------|
| Tabela de Atividades | `activitiesPage` completa |
| Gantt Reprogramável | `activitiesPage` + dependências |
| Caminho Crítico | `criticalPath` (array de IDs) |
| Curva S (séries S/B/D) | `sCurve` ou cálculo via `activitiesWithinPeriodReport` |
| CFF — Cronograma Físico-Financeiro | `budgetReportsPage` + `cffTable` |
| Vinculação Orçamento × Atividades | `budgetItemsPage` com `budgetWeights` |
| Medições Físicas (importação) | `measurementsTasksPage` + `measurementTask` |
| Histograma de Recursos | Calculado sobre `activitiesPage` |

### 2.6 Fallback e Resiliência

- Se a API Prevision estiver indisponível, o Kaizen exibe os dados cacheados no Hive (última sincronização bem-sucedida) com indicador de data do cache
- Imports manuais via Excel ou JSON servem como fallback para obras sem acesso ao Prevision
- Erros de autenticação (token expirado) são tratados com reexibição do fluxo de login; o erro é logado com `tenantId` e `projectId` para diagnóstico

---

## 3. Azure Synapse Analytics

### 3.1 Papel no Kaizen

O Azure Synapse atua como:
1. **Fonte de dados de custo realizado** (orçamento × realizado da obra)
2. **Fallback/Data Lake** para Work Orders quando Salesforce está indisponível
3. **Repositório de dados históricos** para análises de longo prazo

### 3.2 Consultas de Custo Realizado

O módulo de **Controle de Custo** da obra consulta o Synapse para obter o custo realizado integrado ao ERP:
- Valores lançados no sistema de contabilidade/ERP organizados por rubrica da WBS
- Comparação com o orçado e comprometido (cotações aprovadas)
- Atualização periódica (pode ser diária ou por demanda, configurável)

### 3.3 ETL e Batch Import

Para carga inicial de dados históricos (ex.: ao configurar um novo tenant):
1. Extração de dados do Salesforce para Azure Data Lake (Azure Data Factory ou exportação direta)
2. Transformação dos dados para o formato Firestore (schema definido)
3. Importação em bulk para o Firestore via script de migração (Cloud Function batch)

Esse processo também serve como **reconciliação diária**: um job noturno compara Firestore vs. Synapse e corrige discrepâncias.

### 3.4 Autenticação Synapse

- Autenticação via **Service Principal (Azure AD)** com `client_id` + `client_secret`
- Token OAuth2 obtido do endpoint `https://login.microsoftonline.com/{tenantId}/oauth2/token`
- Credenciais armazenadas em variáveis de ambiente das Cloud Functions
- Chamadas ao Synapse REST API com `Authorization: Bearer {token}`

---

## 4. Mega ERP (Materiais)

### 4.1 Papel no Kaizen

O Mega ERP é o sistema de gestão de estoque e compras utilizado pelas construtoras. O Kaizen integra com ele para:
- Consultar saldo de materiais em tempo real
- Exibir catálogo de produtos atualizado
- Encaminhar requisições de compra geradas no Kaizen para processamento no ERP

### 4.2 Dados Sincronizados

| Campo ERP | Campo Kaizen | Descrição |
|-----------|-------------|-----------|
| `CodigoProduto` | `productCode` | Código interno do produto no ERP |
| `NomeProduto` | `productName` | Descrição do produto |
| `NomeGrupo` | `groupName` | Categoria/grupo de material |
| `UnidadeMedida` | `unit` | Unidade de medida (m², kg, un, m³, etc.) |
| `SaldoEstoque` | `stockBalance` | Quantidade disponível em estoque |
| `LocalEstoque` | `stockLocation` | Almoxarifado/filial onde o estoque está |

### 4.3 Mecanismo de Integração

**Catálogo de produtos:**
- Sincronização periódica do catálogo completo (por demanda ou diária)
- Armazenado localmente para disponibilidade offline

**Saldo em tempo real:**
- Consulta REST ao Mega ERP na hora da requisição (não armazenado em cache longo)
- Fallback: se ERP indisponível, exibe último saldo conhecido com timestamp

**Pedidos de compra:**
- Pedido criado no Kaizen → Cloud Function formata e encaminha ao endpoint de pedidos do Mega ERP
- Retorno async: ERP processa e notifica via webhook (ou polling periódico)
- Status do pedido atualizado no Kaizen ao receber confirmação do ERP

### 4.4 Autenticação Mega ERP

- **REST API com autenticação Basic Auth ou API Key** (conforme config da instância do cliente)
- Credenciais armazenadas em variáveis de ambiente das Cloud Functions
- Chamadas sempre via proxy das Cloud Functions (nunca diretamente do app)

---

## 5. Firebase Cloud Messaging (Notificações)

O FCM gerencia as notificações push do Kaizen para dispositivos móveis e navegadores (PWA).

### 5.1 Tipos de Notificações

| Evento | Canal | Destinatários |
|--------|-------|---------------|
| Novo pedido de compra | Tópico `pedidos_{clientId}_{branchId}` | Aprovadores da filial |
| Plano de ação criado/vencido | Device token | Responsável pelo plano |
| Restrição crítica de prazo | Device token | Responsável pela restrição |
| Work Order atribuída | Device token | Técnico designado |
| Agendamento criado/alterado | Device token | Técnico responsável |
| Comentário no mapa de cotação | Device token | Membros do mapa |

### 5.2 Modelo de Tópicos

Para notificações de grupo (ex.: toda a equipe de compras de uma filial):
```
Subscrição: /api/togglePedidoTopicSubscription
Body: { topicName: "pedidos_{clientId}_{branchId}", subscribe: true }
```
- Dispositivo se inscreve no tópico ao fazer login
- Mensagem enviada ao tópico chega a todos os dispositivos inscritos
- Desinscritura automática ao fazer logout ou trocar de filial

### 5.3 Notificações no Navegador (PWA)

- Service Worker intercepta mensagens FCM mesmo com o app fechado
- Banner de notificação nativo do sistema operacional exibido
- Clique na notificação abre diretamente a tela relevante (deep link via `route` no payload)

### 5.4 Central de Notificações (Inbox)

Além das notificações push nativas do sistema operacional, o Kaizen mantém um **inbox persistente** no Firestore com uma central de notificações acessível de qualquer tela via ícone de sino.

#### Arquitetura

```
FCM push recebido
     │
     ├─► Handler foreground (FirebaseMessagingService)
     │        └─► _persistToInbox() → Firestore users/{uid}/notifications/{docId}
     │
     └─► Cloud Function (backend) → grava o mesmo documento Firestore
              (mesmo docId determinístico → sem duplicatas)
                        │
              NotificationInboxService (stream em tempo real)
                        │
              NotificationPanel (overlay lateral na UI)
```

#### `NotificationInboxService`

Serviço GetX registrado globalmente após login. Mantém stream Firestore em tempo real:

- **Path Firestore:** `users/{uid}/notifications/{notifId}` (escopo por usuário, não por tenant — garante portabilidade entre obras)
- **Limite:** 100 documentos consultados; exibe as 50 mais recentes ordenadas por `createdAt DESC` (ordenação feita no Dart para evitar documentos invisíveis com `serverTimestamp()` pendente)
- **Observáveis:** `RxList<NotificationModel> notifications`, `RxInt unreadCount`
- **Auto-reconexão:** em caso de erro no stream, agenda nova tentativa em 5 segundos (`_listeningUid + _listeningClientId` garantem que só reconecta para o mesmo contexto)
- **Ciclo de vida:** `startListening()` chamado ao fazer login; `stopListening()` ao fazer logout (limpa lista e zera contador)
- **`ensureListening()`:** chamado ao abrir o painel — reinicia o stream se mudou de usuário/cliente

#### `_persistToInbox` — Idempotência entre app e Cloud Function

Para evitar duplicatas quando tanto o app quanto a Cloud Function gravam a mesma notificação:

```
docId = "${type}_${entityId}" (sanitizado)
→ ref.set({...}, SetOptions(merge: true))
```

O mesmo `docId` determinístico garante que o segundo write (app ou CF) apenas sobrescreva o primeiro, sem criar duplicatas. `createdAt` usa `Timestamp.fromDate(DateTime.now())` em vez de `serverTimestamp()` para que o documento seja visível imediatamente no cache local.

#### `NotificationPanel` — Widget do painel lateral

Overlay animado disponível em qualquer tela via `Stack` global:

| Propriedade | Valor |
|-------------|-------|
| Largura | 380px (desktop) / 100% da tela (mobile < 600px) |
| Animação | `AnimatedSwitcher` 280ms |
| Fundo dim | `Colors.black.withOpacity(0.45)` — clique fecha o painel |
| Acionamento | Ícone de sino (`NotificationBadge`) no top bar |

**Cabeçalho do painel:** ícone de notificação + título "Central de Notificações" + botão *Marcar todas como lidas* (visível somente quando `unreadCount > 0`) + botão fechar.

**Lista:** `ListView.separated` com separador sutil. Cada item marcado como lido automaticamente ao ser tocado, chamando `FirebaseMessagingService.navigateFromData()` para navegar até a entidade relacionada.

**Estados:** lista vazia → "Tudo em dia ✓" com ícone grande; serviço não disponível (não logado) → mensagem de login.

#### `NotificationBadge`

Widget wrapper que envolve qualquer widget filho (o ícone de sino) com um badge vermelho:

```dart
NotificationBadge(child: Icon(Icons.notifications))
```

Usa `Obx` para observar `NotificationInboxService.unreadCount` e renderizar o badge de forma reativa. Se o serviço não estiver registrado (usuário não logado), retorna o filho sem badge.

#### `NotificationModel` — Tipos suportados

| Tipo | Ícone | Cor | Label | Deep-link |
|------|-------|-----|-------|-----------|
| `mention_comment` | `alternate_email` | Azul | Menção | Tela de Planos de Ação |
| `action_plan_assigned` | `assignment_ind` | Roxo | Plano de Ação | Tela de Planos de Ação |
| `restriction_due` | `warning_amber_rounded` | Laranja | Restrição | Tela de Restrições (Mid-term) |
| `restriction_assigned` | `assignment_late` | Laranja escuro | Restrição | Tela de Restrições |
| `workorder_chatter` | `chat_bubble_outline` | Teal | Chat OT | Módulo AT |
| `appointment_created` | `calendar_month` | Verde | Agendamento | Work Order específica |
| `appointment_reminder` | `alarm` | Âmbar | Lembrete | Work Order específica |
| `appointment_status_change` | `update` | Índigo | Status | Work Order específica |
| `requisition_status_changed` | `inventory_2_outlined` | Roxo | Requisição | Tela de Requisições |
| `new_material_order` | `shopping_cart_outlined` | Roxo escuro | Pedido | Work Order específica |
| `inspection_assigned` | `assignment_ind_outlined` | Teal | Inspeção | Menu da obra |
| `inspection_closed` | `verified_outlined` | Verde | Inspeção | Menu da obra |
| `inspection_reinspection_requested` | `refresh` | Laranja | Reinspeção | Menu da obra |
| `unknown` | `notifications_outlined` | Cinza | Notificação | — |

#### Deep-link por tipo (`_navigateFromNotification`)

A lógica de navegação em `FirebaseMessagingService` usa o campo `type` do payload para decidir para onde navegar:

- **appointments / material_order / workorder_chatter:** extrai `workOrderId` do payload e abre `WorkOrderDetailsPage` (se já no módulo AT) ou navega para `Routes.AT_MAIN`; armazena `pendingOpenWorkOrderId` para abrir automaticamente quando o controller carregar
- **action_plan / mention_comment:** armazena `pendingOpenActionPlanId`; se `ActionPlanController` já está registrado, chama `checkAndOpenPendingPlan()` diretamente; senão, navega para `Routes.ACTION_PLAN`
- **restriction_due / restriction_assigned:** armazena `pendingOpenRestrictionId`; navega para `Routes.MID_TERM` ou chama `checkAndOpenPendingRestriction()`
- **requisition_status_changed:** armazena `pendingOpenRequisitionId`; navega para `Routes.SITE_MENU` com `openWarehouse: true`
- **inspection_*:** navega para `Routes.SITE_MENU` passando `siteId`

---

## 6. Resumo das Integrações

| Sistema | Tipo de Integração | Direção | Autenticação | Frequência |
|---------|--------------------|---------|--------------|------------|
| **Salesforce CRM** | REST API (via Cloud Functions) | Bidirecional | OAuth2 / JWT Bearer | 15 min (incremental) |
| **Prevision** | GraphQL (via app Flutter) | Leitura | JWT Bearer | Pull sob demanda |
| **Azure Synapse** | REST API (via Cloud Functions) | Leitura | Service Principal (Azure AD) | Diária / por demanda |
| **Mega ERP** | REST API (via Cloud Functions) | Bidirecional | API Key / Basic Auth | Por demanda + diário (catálogo) |
| **Firebase FCM** | SDK nativo Firebase | Saída | Firebase Admin SDK | Por evento |
| **Firebase Auth** | SDK Firebase | Interno | Firebase JWT | Por sessão |
| **Firebase Storage** | SDK Firebase | Bidirecional | Firebase Rules | Por upload/download |

---

## 7. Considerações de Segurança

### 7.1 Proxy Obrigatório via Cloud Functions
Nenhuma credencial de sistema externo (Salesforce, Synapse, Mega ERP) é exposta no app Flutter. Todas as chamadas externas passam pelo proxy das Cloud Functions, que:
- Valida a autenticação do usuário Kaizen (Firebase JWT)
- Verifica suas permissões e filial
- Realiza a chamada ao sistema externo com as credenciais do servidor
- Retorna apenas os dados relevantes para aquele usuário

### 7.2 Armazenamento de Credenciais
- Todas as credenciais externas (client IDs, secrets, certificados, API keys) são armazenadas em **variáveis de ambiente** do Firebase Functions (`functions.config()` ou Secret Manager do GCP)
- **Nunca** embutidas no código-fonte ou no bundle do app Flutter
- Rotacionadas periodicamente conforme política de segurança

### 7.3 Validação de Tenant nas Integrações
- Todas as Cloud Functions verificam o `clientId` do usuário autenticado antes de qualquer consulta
- Consultas ao Salesforce são filtradas pela org do cliente (cada tenant pode ter sua própria Salesforce Org)
- Dados de um tenant **nunca** são expostos a outro tenant

### 7.4 Auditoria de Chamadas Externas
- Todas as chamadas às APIs externas são logadas no Cloud Logging do GCP
- Logs incluem: timestamp, Cloud Function, tenantId, tipo de objeto, número de registros retornados
- Alertas configuráveis para falhas consecutivas ou volumes anômalos

---

*Documento: Integrações com Sistemas Externos | Versão 1.0 | Kaizen Gerenciamento de Obras*
