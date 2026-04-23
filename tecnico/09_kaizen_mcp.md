---
layout: page
title: "Kaizen MCP Server"
permalink: /tecnico/mcp/
category: "IA"
order: 9
toc: true
---

# Kaizen MCP Server (`kaizen-mcp`)

## Sumário
1. [O que é o Kaizen MCP](#1-o-que-é-o-kaizen-mcp)
2. [Arquitetura e Tecnologia](#2-arquitetura-e-tecnologia)
3. [Configuração e Instalação](#3-configuração-e-instalação)
4. [Ferramentas Disponíveis](#4-ferramentas-disponíveis)
5. [Integração com Claude Desktop](#5-integração-com-claude-desktop)
6. [Referência de Ferramentas](#6-referência-de-ferramentas)
7. [Segurança](#7-segurança)

---

## 1. O que é o Kaizen MCP

O **Kaizen MCP** é um servidor [Model Context Protocol (MCP)](https://modelcontextprotocol.io) que expõe os dados do Kaizen para assistentes de IA como o **Claude (Anthropic)**, **GitHub Copilot** e outros clientes compatíveis com MCP.

Enquanto o [KAI](./07_ia_kai.md) é o assistente embutido no app com análises pré-formatadas, o Kaizen MCP serve um caso de uso diferente: permite que qualquer AI assistant externo consulte **dados reais da obra em tempo real** como parte de uma conversa — sem necessidade de copiar/colar dados manualmente.

### Diferença entre KAI e kaizen-mcp

| | KAI (assistente nativo) | kaizen-mcp (MCP server) |
|---|---|---|
| **Onde roda** | Cloud Functions Firebase | Processo local (Node.js) |
| **Interface** | Telas dedicadas no app | Qualquer cliente MCP (Claude, Copilot etc.) |
| **Transporte** | HTTPS | stdio |
| **Autenticação** | Firebase Auth | API Key (`x-api-key`) |
| **Caso de uso** | Relatórios e chat embutidos no app | Consultas ad-hoc em ferramentas de IA externas |

---

## 2. Arquitetura e Tecnologia

```
Cliente MCP (ex: Claude Desktop)
        │  stdio (JSON-RPC)
        ▼
  kaizen-mcp (Node.js)
        │  HTTPS + x-api-key
        ▼
  Kaizen Firestore API
  (us-central1-kaizen-1ee16.cloudfunctions.net/api)
        │
        ▼
  Firestore (multi-tenant)
```

### Stack

| Camada | Tecnologia |
|--------|-----------|
| Linguagem | TypeScript (ES2022, NodeNext modules) |
| SDK MCP | `@modelcontextprotocol/sdk` ^1.0.0 |
| Validação de schema | `zod` ^3.22.0 |
| Variáveis de ambiente | `dotenv` ^16.0.0 |
| Transporte | `StdioServerTransport` |

### Estrutura de Arquivos

```
kaizen-mcp/
├── src/
│   ├── index.ts              ← entry point
│   ├── client.ts             ← kaizenFetch, KaizenApiError, helpers
│   └── tools/
│       ├── sites.ts
│       ├── restrictions.ts
│       ├── actionPlans.ts
│       ├── supply.ts
│       ├── deviations.ts
│       └── measurements.ts
├── dist/                     ← compilado (gitignored)
├── package.json
├── tsconfig.json
└── .env.example
```

---

## 3. Configuração e Instalação

### Pré-requisitos

- Node.js ≥ 18
- Uma `KAIZEN_API_KEY` válida (obtida junto ao time Kaizen)

### Passos

```bash
cd kaizen-mcp
npm install
cp .env.example .env
# Editar .env com as credenciais
npm run build
```

### Variáveis de Ambiente

| Variável | Obrigatória | Padrão | Descrição |
|---|---|---|---|
| `KAIZEN_API_KEY` | **SIM** | — | API key do tenant; enviada como header `x-api-key` |
| `KAIZEN_BASE_URL` | não | `https://us-central1-kaizen-1ee16.cloudfunctions.net/api` | URL base da API Kaizen |
| `KAIZEN_CLIENT_ID` | não | — | `clientId` padrão; usado quando a ferramenta não receber um |

> **Atenção:** Se `KAIZEN_API_KEY` não estiver definida, o servidor loga um erro descritivo e encerra com código de saída `1`.

---

## 4. Ferramentas Disponíveis

O servidor registra **8 ferramentas MCP** organizadas em 6 domínios:

| Ferramenta | Domínio | Descrição |
|---|---|---|
| `list_sites` | Obras | Lista todas as obras (obras) de um cliente |
| `list_restrictions` | Restrições | Lista restrições com filtros por status, responsável e data |
| `get_restriction` | Restrições | Retorna detalhes completos de uma restrição específica |
| `list_action_plans` | Planos de Ação | Lista planos de ação por obra ou por cliente |
| `get_supply_lighthouse` | Suprimentos | Snapshot do Farol de Contratações (forecast de procurement) |
| `list_supply_processes` | Suprimentos | Lista processos de suprimento ativos |
| `list_deviations` | Custos | Lista desvios orçamentários (planejado vs. realizado) |
| `list_physical_measurements` | Medições | Lista medições físicas para construção de curvas S |

---

## 5. Integração com Claude Desktop

Adicione ao arquivo `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "kaizen": {
      "command": "node",
      "args": ["/caminho/absoluto/para/kaizen-mcp/dist/index.js"],
      "env": {
        "KAIZEN_API_KEY": "sua_api_key_aqui",
        "KAIZEN_CLIENT_ID": "seu_client_id_aqui"
      }
    }
  }
}
```

Localização do arquivo de configuração por SO:

| Sistema Operacional | Caminho |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

Após salvar e reiniciar o Claude Desktop, as ferramentas Kaizen aparecem automaticamente no painel de ferramentas disponíveis.

---

## 6. Referência de Ferramentas

### `list_sites`

Lista todas as obras de um cliente Kaizen.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente (tenant). Usa `KAIZEN_CLIENT_ID` se omitido. |

**Retorna:** `KaizenListResponse<Site>` — array de obras com `id`, `name`, `status`, `address`.

---

### `list_restrictions`

Lista restrições (impedimentos) de uma obra com suporte a filtros.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | **sim** | ID da obra. |
| `status` | `pending` \| `in_progress` \| `done` \| `cancelled` | não | Filtro por status. |
| `responsible` | string | não | Filtro por nome ou ID do responsável. |
| `updatedAt_gte` | string (ISO 8601) | não | Restrições atualizadas a partir desta data. |
| `updatedAt_lte` | string (ISO 8601) | não | Restrições atualizadas até esta data. |

**Retorna:** `KaizenListResponse<Restriction>` — inclui campos de caminho crítico como `hasCriticalActivities`, `timesRescheduled`, `escalationLevel`.

---

### `get_restriction`

Retorna todos os detalhes de uma restrição específica, incluindo `linkedActivities`.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | **sim** | ID da obra. |
| `restrictionId` | string | **sim** | ID do documento da restrição. |

**Retorna:** objeto `Restriction` completo (não envelopado em lista).

---

### `list_action_plans`

Lista planos de ação de uma obra ou de todas as obras de um cliente.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | não | ID da obra. Se omitido, retorna planos de **todas** as obras. |
| `status` | `pending` \| `in_progress` \| `done` \| `cancelled` | não | Filtro por status. |
| `responsible` | string | não | Filtro por responsável. |
| `updatedAt_gte` / `updatedAt_lte` | string (ISO 8601) | não | Intervalo de última atualização. |
| `dueDate_gte` / `dueDate_lte` | string (ISO 8601) | não | Intervalo de prazo. |

**Retorna:** `KaizenListResponse<ActionPlan>` — inclui `milestones`, `percentComplete`, `budgetEstimate`, `budgetActual`.

---

### `get_supply_lighthouse`

Retorna o snapshot mais recente do **Farol de Contratações** de uma obra.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | **sim** | ID da obra. |

**Retorna:** `LighthouseSnapshot` — objeto com array `items` contendo cada insumo com datas planejadas, datas forecast, lead time e contador de SLA.

---

### `list_supply_processes`

Lista os processos de suprimento/contratação ativos de uma obra.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | **sim** | ID da obra. |

**Retorna:** `KaizenListResponse<SupplyProcess>` — com `status`, `currentSector`, `startedAt`, `finishedAt`.

---

### `list_deviations`

Lista desvios orçamentários (planejado vs. realizado) de uma obra ou de todas as obras.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | não | ID da obra. Se omitido, retorna desvios de **todas** as obras. |
| `updatedAt_gte` / `updatedAt_lte` | string (ISO 8601) | não | Filtro por data de atualização. |

**Retorna:** `KaizenListResponse<BudgetDeviation>` — com `plannedAmount`, `actualAmount`, `deviationAmount`, `deviationPercent`.

---

### `list_physical_measurements`

Lista medições físicas de progresso de uma obra. Dados usados para construção de curvas S.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `clientId` | string | não | ID do cliente. |
| `siteId` | string | **sim** | ID da obra. |
| `period_gte` / `period_lte` | string (ISO 8601) | não | Intervalo de período de medição. |

**Retorna:** `KaizenListResponse<PhysicalMeasurement>` — com `plannedPercent`, `actualPercent`, `unit`, `period`.

---

## 7. Segurança

- **Autenticação via API Key:** toda requisição à Kaizen API inclui o header `x-api-key`. A chave nunca é exposta ao cliente MCP.
- **Sem escrita:** todas as ferramentas são somente leitura (`GET`). Nenhuma ferramenta altera dados no Firestore.
- **Isolamento multi-tenant:** a API Kaizen aplica isolamento por `clientId` em todas as queries — o MCP server não acessa dados de outros tenants.
- **Falha isolada por ferramenta:** erros em uma ferramenta retornam JSON de erro estruturado sem derrubar o servidor. O processo permanece ativo após qualquer falha de ferramenta.

---

*Documento: Kaizen MCP Server | Versão 1.0 | Kaizen Construction Management*
