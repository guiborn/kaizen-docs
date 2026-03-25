---
layout: page
title: "Arquitetura Geral"
permalink: /tecnico/arquitetura/
category: "Arquitetura"
order: 1
toc: true
---

# Kaizen — Arquitetura Geral e Infraestrutura Técnica

## Sumário
1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Stack Tecnológica](#2-stack-tecnológica)
3. [Modelo Multi-tenant](#3-modelo-multi-tenant)
4. [Camadas da Aplicação](#4-camadas-da-aplicação)
5. [Armazenamento e Banco de Dados](#5-armazenamento-e-banco-de-dados)
6. [Arquitetura Offline e Sincronização](#6-arquitetura-offline-e-sincronização)
7. [Segurança e Autenticação](#7-segurança-e-autenticação)
8. [Escalabilidade e Performance](#8-escalabilidade-e-performance)
9. [Infraestrutura Recomendada para o Cliente](#9-infraestrutura-recomendada-para-o-cliente)

---

## 1. Visão Geral da Arquitetura

O Kaizen é uma plataforma SaaS (Software as a Service) entregue via web (PWA) e aplicativo nativo para Android/iOS. Toda a lógica de negócio e persistência de dados é hospedada em infraestrutura gerenciada pela Google (Firebase/GCP), sem necessidade de servidor dedicado no cliente.

```
┌──────────────────────────────────────────────────────────┐
│                   CLIENT DEVICES                         │
│  Browser (PWA) │ Android App │ iOS App                   │
│       Flutter 3.x + GetX State Management                │
└─────────────────────┬────────────────────────────────────┘
                      │  HTTPS / WebSocket (Firestore streams)
┌─────────────────────▼────────────────────────────────────┐
│               FIREBASE PLATFORM (Google Cloud)           │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ Firebase    │  │  Firestore   │  │  Firebase      │  │
│  │   Auth      │  │  (NoSQL DB)  │  │  Storage       │  │
│  └─────────────┘  └──────────────┘  └────────────────┘  │
│         ┌──────────────────────────────┐                 │
│         │   Cloud Functions (Node.js)  │                 │
│         │   Express REST API           │                 │
│         └──────────────┬───────────────┘                 │
└──────────────────────── │ ────────────────────────────────┘
                          │  REST / OAuth2
        ┌─────────────────┼──────────────────┐
        │                 │                  │
  ┌─────▼─────┐   ┌───────▼──────┐   ┌──────▼──────┐
  │ Salesforce│   │ Azure Synapse│   │  Mega ERP   │
  │    CRM    │   │  (Data Lake) │   │ (Materiais) │
  └───────────┘   └──────────────┘   └─────────────┘
```

---

## 2. Stack Tecnológica

| Camada | Tecnologia | Versão | Função |
|--------|-----------|--------|--------|
| Frontend | Flutter | 3.x | UI multiplataforma (Web/Android/iOS) |
| State Management | GetX | 4.x | Reatividade, injeção de dependências, rotas |
| Backend (BaaS) | Firebase | SDK 2.x+ | Auth, Firestore, Storage, Hosting, Messaging |
| API Backend | Cloud Functions | Node.js 20 | Lógica de negócio serverless, integrações externas |
| Banco de Dados | Cloud Firestore | — | NoSQL orientado a documentos, multi-tenant |
| Armazenamento de Arquivos | Firebase Storage | — | Fotos, PDFs, documentos, vídeos |
| Cache local | Hive | 2.x | Persistência offline estruturada no dispositivo |
| Notificações Push | Firebase Cloud Messaging | — | Push notifications por tópico ou device token |
| Hospedagem Web | Firebase Hosting | — | CDN global, HTTPS automático |
| BI / Dados Históricos | Azure Synapse Analytics | — | Data lake, consultas analíticas em larga escala |
| ERP Materiais | Mega ERP | REST API | Estoque, movimentações, pedidos de compra |
| CRM de Serviço | Salesforce Field Service | REST + OAuth2 | Work Orders, Cases, Service Appointments, Assets |

---

## 3. Modelo Multi-tenant

O Kaizen suporta múltiplos clientes (tenants/empreendimentos) de forma isolada dentro de um único ambiente Firebase. O isolamento é garantido em três níveis:

### 3.1 Isolamento de Dados no Firestore

Todos os dados de negócio são armazenados sob caminhos prefix por `clientId`:

```
/clients/{clientId}/obras/{obraId}/...
/clients/{clientId}/workOrders/{woId}
/clients/{clientId}/serviceAppointments/{saId}
/clients/{clientId}/units/{unitId}
/clients/{clientId}/quotationMaps/{mapId}
```

Cada documento também armazena o campo `tenantId` explicitamente, para filtros por query e suporte à migração futura para SQL.

### 3.2 Isolamento por Filial (Branch)

Dentro de um mesmo tenant, os dados podem ser isolados por filial:
- Usuários comuns (`type: 'normal'`) enxergam apenas os dados da própria filial
- Usuários administradores (`type: 'admin'`, `'super'`, `'dev'`) podem selecionar qualquer filial no dropdown ou visualizar dados globais
- Todas as queries Firestore críticas incluem o filtro `.where('branch', isEqualTo: activeBranch)` aplicado server-side

### 3.3 Isolamento de Tokens e Sessões

Cada usuário autentica via Firebase Auth e carrega custom claims que indicam `clientId` e `userType`. As Cloud Functions validam esses claims antes de executar qualquer acesso a dados.

---

## 4. Camadas da Aplicação

### 4.1 Frontend (Flutter)

```
lib/app/
├── modules/          # Módulos de negócio (obra, at, cotações, etc.)
│   └── {modulo}/
│       ├── controllers/    # Lógica de estado (GetxController)
│       ├── views/          # Telas e widgets
│       ├── models/         # Entidades de dados
│       └── bindings/       # Injeção de dependências (GetX Bindings)
├── services/         # Serviços transversais (auth, branch, sync)
├── widgets/          # Componentes compartilhados
└── constants/        # Constantes globais, temas, rotas
```

**Padrão de reatividade:** Variáveis `Rx` (ex.: `RxList<WorkOrderItem> workOrders = <WorkOrderItem>[].obs`) notificam automaticamente a UI ao serem alteradas, sem chamadas manuais a `setState()`.

### 4.2 Backend (Cloud Functions)

As Cloud Functions atuam como camada intermediária entre o app e as integrações externas:

- **Chamadas ao Salesforce:** o app nunca chama a API Salesforce diretamente; usa a Cloud Function autenticada
- **Lógica sensível:** cálculos de custo, validações de negócio e geração de tokens
- **Webhooks:** receptores para notificações do Salesforce e sistemas externos

### 4.3 Serviços Transversais

| Serviço | Arquivo | Responsabilidade |
|---------|---------|-----------------|
| `BranchAccessService` | `branch_access_service.dart` | Filtro de filial em todas as queries |
| `SyncQueueService` | `sync_queue_service.dart` | Fila de escritas pendentes (offline) |
| `CriticalPathService` | `critical_path_service.dart` | Cálculo de caminho crítico Gantt |
| `SandboxToggleService` | — | Toggle ambiente Salesforce (prod/sandbox) |
| `NotificationService` | — | Subscrição a tópicos FCM por filial/tipo |

---

## 5. Armazenamento e Banco de Dados

### 5.1 Firestore (Banco Primário)

- **Tipo:** NoSQL orientado a documentos, com coleções e subcoleções
- **Consistência eventual** com suporte a transações ACID (até 500 documentos por batch)
- **Streams em tempo real:** o app se inscreve em queries e recebe updates automaticamente via WebSocket
- **Índices compostos:** configurados em `firestore.indexes.json` para queries combinadas (ex.: filial + status + data)
- **Regras de Segurança:** definidas em `firestore.rules` — validação server-side de autenticação e estrutura de dados

#### 5.1.1 Point-in-Time Recovery (PITR)

O Firestore oferece recuperação a qualquer ponto no tempo dentro de uma janela de **7 dias**, habilitável por banco de dados. Com PITR ativo:

- **Leitura histórica:** é possível consultar o estado exato de qualquer documento ou coleção em qualquer timestamp dos últimos 7 dias, sem necessidade de ter feito backup manual
- **Recuperação sob demanda:** em caso de exclusão acidental, corrupção de dados ou operação em lote errada, a equipe técnica pode restaurar o estado anterior sem interromper o acesso ao banco em produção
- **Granularidade de segundo:** o ponto de recuperação pode ser especificado com precisão de microssegundo dentro da janela disponível
- **Restauração parcial:** é possível recuperar apenas uma coleção, um conjunto de documentos ou até um único documento — sem necessidade de restaurar todo o banco
- **Habilitação:** configurada via Google Cloud Console ou `gcloud firestore databases update --enable-point-in-time-recovery`; aplica um custo adicional de armazenamento proporcional ao volume de dados modificados no período

**Casos de uso relevantes para o Kaizen:**
| Cenário | Ação via PITR |
|---------|--------------|
| Exclusão acidental de Work Orders em lote | Restaurar coleção `workOrders` para timestamp anterior à operação |
| Dados de orçamento corrompidos por bug de sincronização | Recuperar versão anterior dos documentos da coleção `budgets` |
| Sobrescrita indevida de cronograma (schedule) | Restaurar `schedules/{id}` ao estado de horas atrás |
| Auditoria: comparar estado atual com estado de 3 dias atrás | Leitura somente-leitura no timestamp desejado sem impacto em produção |

### 5.2 Firebase Storage

- Armazenamento de arquivos binários: fotos de inspeção, documentos PDF, vídeos comprimidos
- Organização por `clientId/obraId/tipo/`
- Regras em `storage.rules` com controle de acesso por usuário autenticado
- Compressão de vídeo implementada no app antes do upload

### 5.3 Hive (Cache Local)

- Banco chave-valor local no dispositivo, TypeSafe via code generation
- Utilizado para: unidades (personalizacao), pré-carregamento de dados frequentes
- **Estratégia de 3 camadas:**
  1. Hive (instantâneo, offline)
  2. Firestore (atualização em background)  
  3. API externa/Salesforce (fallback)

---

## 6. Arquitetura Offline e Sincronização

O Kaizen suporta trabalho em campo com conectividade intermitente:

### 6.1 SyncQueueService

- Operações de escrita são enfileiradas localmente (Hive) quando offline
- Ao restaurar conexão, a fila é processada em ordem cronológica
- Cada item da fila contém: operação, coleção, documento ID, payload, timestamp
- Conflitos são resolvidos por "last-write-wins" com timestamp do servidor

### 6.2 Dados pré-carregados localmente

- Unidades de empreendimento, checklist de obras, dados de referência
- Atualização em background via listeners Firestore (quando online)
- Indicadores visuais no app quando operando em modo offline

### 6.3 PWA (Progressive Web App)

- Service Worker para cache de assets estáticos
- Funciona parcialmente offline em navegadores modernos
- Manifest configurado para instalação no dispositivo (ícone no menu principal)

---

## 7. Segurança e Autenticação

### 7.1 Autenticação

- **Firebase Authentication** com suporte a email/senha e SSO corporativo (configurável)
- JWT tokens com validade de 1 hora, renovados automaticamente pelo SDK
- Custom claims para `clientId`, `userType`, `branchId`

### 7.2 Autorização

- **Nível de plataforma:** Firebase Security Rules validam autenticação em todas as leituras/escritas Firestore e Storage
- **Nível de aplicação:** `BranchAccessService` filtra dados por filial do usuário
- **Nível de UI:** `hasPrivileges(key)` controla visibilidade de módulos no menu lateral
- **Nível de API:** Cloud Functions validam custom claims no `context.auth` antes de executar

### 7.3 Integrações Externas (segurança)

- Credenciais Salesforce/Synapse/Mega armazenadas como **variáveis de ambiente** nas Cloud Functions (nunca embutidas no app)
- Autenticação Salesforce via OAuth2/JWT Bearer (sem password em texto plano)
- O app nunca acessa APIs externas diretamente — sempre via proxy nas Cloud Functions autenticadas

---

## 8. Escalabilidade e Performance

| Característica | Implementação |
|---------------|--------------|
| Paginação server-side | Cursores Firestore (`startAfter`, `limit`) — até 300 registros por página no módulo AT |
| Lazy loading | Módulos Flutter carregados sob demanda (lazy routes GetX) |
| Streams filtrados | listeners Firestore sempre com cláusulas `where` para reduzir tráfego |
| Imagens comprimidas | `flutter_image_compress` antes de upload (qualidade configurável) |
| Vídeos comprimidos | Compressão aplicada antes do upload com feedback de progresso |
| Índices Firestore | Índices compostos pré-definidos para queries críticas |
| CDN global | Firebase Hosting com CDN multi-região via Google Cloud |
| Serverless auto-scaling | Cloud Functions escalam automaticamente com demanda |

### 8.1 Limites Operacionais Relevantes

| Recurso | Limite / Referência |
|---------|---------------------|
| Work Orders por carregamento | 300 (paginação) |
| Documentos por lote Firestore | 500 (batch commit) |
| Tamanho máximo por documento Firestore | 1 MB |
| Armazenamento Firebase Storage | Escalável (GCP) |
| Usuários simultâneos | Sem limite fixo (escala horizontal) |

---

## 9. Infraestrutura Recomendada para o Cliente

Como plataforma SaaS em nuvem, **não é necessária nenhuma infraestrutura on-premise** para uso padrão do Kaizen. A infraestrutura abaixo refere-se aos requisitos nos dispositivos dos usuários:

### 9.1 Dispositivos Suportados

| Tipo | Requisito Mínimo | Recomendado |
|------|-----------------|-------------|
| **Navegador Web** | Chrome 90+, Edge 90+, Firefox 88+, Safari 14+ | Chrome 120+ |
| **Android** | Android 9.0 (API 28), 2 GB RAM | Android 11+, 4 GB RAM |
| **iOS** | iOS 14.0 | iOS 16+ |
| **Conexão** | 3G (funcionalidade básica) | 4G/Wi-Fi para upload de fotos/vídeos |

### 9.2 Rede

- Todo o tráfego ocorre via HTTPS (porta 443)
- Não há requisitos de VPN ou abertura de portas adicionais no firewall corporativo
- Domínio de acesso: `*.kaizenconstruct...` (Firebase Hosting)
- Firestore requer acesso a `*.googleapis.com` e `*.firebaseio.com`

### 9.3 E-mail Corporativo

- Integração com serviço de envio de e-mail transacional via **Resend** (suporte a anexos PDF)
- Notificações de Plano de Ação, alertas de restrições e relatórios automáticos

---

*Documento: Arquitetura Geral | Versão 1.0 | Kaizen Gerenciamento de Obras*
