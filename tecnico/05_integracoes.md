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
2. [Azure Synapse Analytics](#2-azure-synapse-analytics)
3. [Mega ERP (Materiais)](#3-mega-erp-materiais)
4. [Firebase Cloud Messaging (Notificações)](#4-firebase-cloud-messaging-notificações)
5. [Resumo das Integrações](#5-resumo-das-integrações)
6. [Considerações de Segurança](#6-considerações-de-segurança)

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

## 2. Azure Synapse Analytics

### 2.1 Papel no Kaizen

O Azure Synapse atua como:
1. **Fonte de dados de custo realizado** (orçamento × realizado da obra)
2. **Fallback/Data Lake** para Work Orders quando Salesforce está indisponível
3. **Repositório de dados históricos** para análises de longo prazo

### 2.2 Consultas de Custo Realizado

O módulo de **Controle de Custo** da obra consulta o Synapse para obter o custo realizado integrado ao ERP:
- Valores lançados no sistema de contabilidade/ERP organizados por rubrica da WBS
- Comparação com o orçado e comprometido (cotações aprovadas)
- Atualização periódica (pode ser diária ou por demanda, configurável)

### 2.3 ETL e Batch Import

Para carga inicial de dados históricos (ex.: ao configurar um novo tenant):
1. Extração de dados do Salesforce para Azure Data Lake (Azure Data Factory ou exportação direta)
2. Transformação dos dados para o formato Firestore (schema definido)
3. Importação em bulk para o Firestore via script de migração (Cloud Function batch)

Esse processo também serve como **reconciliação diária**: um job noturno compara Firestore vs. Synapse e corrige discrepâncias.

### 2.4 Autenticação Synapse

- Autenticação via **Service Principal (Azure AD)** com `client_id` + `client_secret`
- Token OAuth2 obtido do endpoint `https://login.microsoftonline.com/{tenantId}/oauth2/token`
- Credenciais armazenadas em variáveis de ambiente das Cloud Functions
- Chamadas ao Synapse REST API com `Authorization: Bearer {token}`

---

## 3. Mega ERP (Materiais)

### 3.1 Papel no Kaizen

O Mega ERP é o sistema de gestão de estoque e compras utilizado pelas construtoras. O Kaizen integra com ele para:
- Consultar saldo de materiais em tempo real
- Exibir catálogo de produtos atualizado
- Encaminhar requisições de compra geradas no Kaizen para processamento no ERP

### 3.2 Dados Sincronizados

| Campo ERP | Campo Kaizen | Descrição |
|-----------|-------------|-----------|
| `CodigoProduto` | `productCode` | Código interno do produto no ERP |
| `NomeProduto` | `productName` | Descrição do produto |
| `NomeGrupo` | `groupName` | Categoria/grupo de material |
| `UnidadeMedida` | `unit` | Unidade de medida (m², kg, un, m³, etc.) |
| `SaldoEstoque` | `stockBalance` | Quantidade disponível em estoque |
| `LocalEstoque` | `stockLocation` | Almoxarifado/filial onde o estoque está |

### 3.3 Mecanismo de Integração

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

### 3.4 Autenticação Mega ERP

- **REST API com autenticação Basic Auth ou API Key** (conforme config da instância do cliente)
- Credenciais armazenadas em variáveis de ambiente das Cloud Functions
- Chamadas sempre via proxy das Cloud Functions (nunca diretamente do app)

---

## 4. Firebase Cloud Messaging (Notificações)

O FCM gerencia as notificações push do Kaizen para dispositivos móveis e navegadores (PWA).

### 4.1 Tipos de Notificações

| Evento | Canal | Destinatários |
|--------|-------|---------------|
| Novo pedido de compra | Tópico `pedidos_{clientId}_{branchId}` | Aprovadores da filial |
| Plano de ação criado/vencido | Device token | Responsável pelo plano |
| Restrição crítica de prazo | Device token | Responsável pela restrição |
| Work Order atribuída | Device token | Técnico designado |
| Agendamento criado/alterado | Device token | Técnico responsável |
| Comentário no mapa de cotação | Device token | Membros do mapa |

### 4.2 Modelo de Tópicos

Para notificações de grupo (ex.: toda a equipe de compras de uma filial):
```
Subscrição: /api/togglePedidoTopicSubscription
Body: { topicName: "pedidos_{clientId}_{branchId}", subscribe: true }
```
- Dispositivo se inscreve no tópico ao fazer login
- Mensagem enviada ao tópico chega a todos os dispositivos inscritos
- Desinscritura automática ao fazer logout ou trocar de filial

### 4.3 Notificações no Navegador (PWA)

- Service Worker intercepta mensagens FCM mesmo com o app fechado
- Banner de notificação nativo do sistema operacional exibido
- Clique na notificação abre diretamente a tela relevante (deep link via `route` no payload)

---

## 5. Resumo das Integrações

| Sistema | Tipo de Integração | Direção | Autenticação | Frequência |
|---------|--------------------|---------|--------------|------------|
| **Salesforce CRM** | REST API (via Cloud Functions) | Bidirecional | OAuth2 / JWT Bearer | 15 min (incremental) |
| **Azure Synapse** | REST API (via Cloud Functions) | Leitura | Service Principal (Azure AD) | Diária / por demanda |
| **Mega ERP** | REST API (via Cloud Functions) | Bidirecional | API Key / Basic Auth | Por demanda + diário (catálogo) |
| **Firebase FCM** | SDK nativo Firebase | Saída | Firebase Admin SDK | Por evento |
| **Firebase Auth** | SDK Firebase | Interno | Firebase JWT | Por sessão |
| **Firebase Storage** | SDK Firebase | Bidirecional | Firebase Rules | Por upload/download |

---

## 6. Considerações de Segurança

### 6.1 Proxy Obrigatório via Cloud Functions
Nenhuma credencial de sistema externo (Salesforce, Synapse, Mega ERP) é exposta no app Flutter. Todas as chamadas externas passam pelo proxy das Cloud Functions, que:
- Valida a autenticação do usuário Kaizen (Firebase JWT)
- Verifica suas permissões e filial
- Realiza a chamada ao sistema externo com as credenciais do servidor
- Retorna apenas os dados relevantes para aquele usuário

### 6.2 Armazenamento de Credenciais
- Todas as credenciais externas (client IDs, secrets, certificados, API keys) são armazenadas em **variáveis de ambiente** do Firebase Functions (`functions.config()` ou Secret Manager do GCP)
- **Nunca** embutidas no código-fonte ou no bundle do app Flutter
- Rotacionadas periodicamente conforme política de segurança

### 6.3 Validação de Tenant nas Integrações
- Todas as Cloud Functions verificam o `clientId` do usuário autenticado antes de qualquer consulta
- Consultas ao Salesforce são filtradas pela org do cliente (cada tenant pode ter sua própria Salesforce Org)
- Dados de um tenant **nunca** são expostos a outro tenant

### 6.4 Auditoria de Chamadas Externas
- Todas as chamadas às APIs externas são logadas no Cloud Logging do GCP
- Logs incluem: timestamp, Cloud Function, tenantId, tipo de objeto, número de registros retornados
- Alertas configuráveis para falhas consecutivas ou volumes anômalos

---

*Documento: Integrações com Sistemas Externos | Versão 1.0 | Kaizen Gerenciamento de Obras*
