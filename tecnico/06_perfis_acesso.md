---
layout: page
title: "Perfis de Acesso e Controle de Permissões"
permalink: /kaizen/tecnico/acesso/
category: "Segurança"
order: 6
toc: true
---

# Kaizen — Perfis de Acesso e Controle de Permissões

## Sumário
1. [Tipos de Usuário](#1-tipos-de-usuário)
2. [Controle por Filial](#2-controle-por-filial)
3. [Sistema de Privilégios por Módulo](#3-sistema-de-privilégios-por-módulo)
4. [Mapa Completo de Privilégios](#4-mapa-completo-de-privilégios)
5. [Gestão de Usuários](#5-gestão-de-usuários)
6. [Segurança em Camadas](#6-segurança-em-camadas)
7. [Matriz Resumida de Acesso](#7-matriz-resumida-de-acesso)

---

## 1. Tipos de Usuário

O Kaizen possui quatro tipos de usuário, com hierarquia crescente de permissões:

| Tipo | Código | Descrição |
|------|--------|-----------|
| **Usuário Padrão** | `normal` | Acesso limitado à própria filial e módulos autorizados via chaves de privilégio |
| **Administrador** | `admin` | Acesso a todas as filiais do tenant; pode gerenciar usuários e configurações |
| **Super Administrador** | `super` | Acesso a funcionalidades avançadas de configuração; visibilidade de todos os tenants |
| **Desenvolvedor** | `dev` | Acesso irrestrito para suporte técnico e desenvolvimento |

### 1.1 Comportamento por Tipo

**Usuário Padrão (`normal`):**
- Visualiza dados apenas da sua filial
- Acessa somente os módulos para os quais possui chave de privilégio
- Não pode trocar de filial
- Não vê dados de outros usuários de outras filiais

**Administrador (`admin`):**
- Pode selecionar qualquer filial no dropdown global (ou "Todas as filiais")
- Enxerga dados de todas as filiais do tenant
- Pode gerenciar usuários, filiais e configurações do tenant
- Acessa todos os módulos (independente de chaves de privilégio individuais)

**Super Administrador (`super`) e Desenvolvedor (`dev`):**
- Todos os privilégios do `admin` mais funcionalidades de diagnóstico e configuração de baixo nível
- Acesso a logs de sistema, configurações de integrações, toggle de sandbox Salesforce
- Usados principalmente pela equipe Kaizen para suporte

---

## 2. Controle por Filial

### 2.1 BranchAccessService

Serviço central que determina o escopo de acesso de dados para qualquer operação:

```dart
// Para usuário normal: retorna a filial do usuário
// Para admin com filial selecionada: retorna a filial selecionada
// Para admin visualizando todas: retorna null (sem filtro)
String? getActiveBranchNameForFilter()
```

### 2.2 Aplicação do Filtro de Filial

O filtro de filial é aplicado em **todas as queries críticas** do banco de dados:

```dart
Query query = _collection
  .where('tenantId', isEqualTo: activeClientId)
  .where('branch', isEqualTo: activeBranch); // null = sem filtro (admin all)
```

O mesmo filtro é passado como parâmetro nas chamadas às Cloud Functions que consultam o Salesforce, garantindo que o backend também aplique a restrição.

### 2.3 Hierarquia de Filial

```
Tenant (clientId)
├── Filial A (branchId/branchName)
│   ├── Usuários da Filial A
│   ├── Colaboradores AT da Filial A
│   ├── Materiais da Filial A
│   └── Dados específicos da Filial A
├── Filial B
│   └── ...
└── Admin (enxerga todas as filiais)
```

---

## 3. Sistema de Privilégios por Módulo

### 3.1 Como Funciona

Além do tipo de usuário, cada módulo do menu lateral pode ser restringido por uma **chave de privilégio**. Apenas usuários que possuem aquela chave em seu perfil veem o módulo no menu.

No código, a verificação é feita antes de renderizar cada item de navegação:

```dart
NavItem(
  key: 'nav_fvs',
  label: 'Gestão de FVS',
  route: Routes.FVS,
  requiredPrivilege: 'fvs_approval',  // chave de privilégio
)

// No controlador de usuário:
bool hasPrivilege(String key) {
  if (userType == 'admin' || userType == 'super' || userType == 'dev') return true;
  return userPrivileges.contains(key);
}
```

### 3.2 Atribuição de Privilégios

- Privilégios são armazenados no perfil do usuário
- Administradores podem conceder/revogar privilégios via painel de gestão de usuários
- Um usuário pode ter zero ou múltiplos privilégios
- Usuários `admin`/`super`/`dev` têm todos os privilégios automaticamente (sem verificação por chave)

---

## 4. Mapa Completo de Privilégios

### 4.1 Módulos de Gestão de Obra

| Chave de Privilégio | Módulo / Funcionalidade | Seção |
|--------------------|------------------------|-------|
| `daily_measurements` | Medições Kaizen (medição diária de produção) | Produtividade |
| `daily_kaizen` | Kaizen Diário (dashboard de produção) | Produtividade |
| `prod_bi` | Estatísticas BI (analytics avançado de produtividade) | Produtividade |
| `physical` | Avanço Físico (gantt, medições físicas, análise) | Planejamento |
| `baseline_view` | Gantt de Controle com linha de base | Planejamento |
| `mid_term` | Plano de Médio Prazo / Lookahead | Planejamento |
| `action_plans_view` | Planos de Ação | Rotinas de Gestão |
| `cost_control_view` | Controle de Custo e Orçamento | Custo |
| `fvs_fill` | Preenchimento de FVS em campo | Qualidade |
| `fvs_approval` | Aprovação de FVS (nível gerencial) | Qualidade |
| `quality_manager` | Gestão geral de qualidade (acesso completo FVS) | Qualidade |
| `warehouse_view` | Almoxarifado Digital (estoque e pedidos) | Materiais |
| `units` | Controle de Unidades (personalizações, vistorias) | Controle |
| `lean_inspection` | Estabilização Lean (checklists de rotina) | Rotinas |
| `supply_schedule` | Farol de Contratações (cronograma de suprimentos) | Suprimentos |
| `vehicle_reservation` | Reserva de Veículos e controle de frota | Logística |
| `teams` | Gestão de Equipes de produção | Configuração |
| `project_manager` | Acesso de gerente de projeto (visão consolidada) | Geral |
| `request_manager` | Gestão de Pedidos e aprovações de compra | Compras |

### 4.2 Módulo de Assistência Técnica

O módulo AT tem seu próprio controle de acesso via o tipo de usuário e filial. Não usa chaves de privilégio individuais por sub-módulo — o acesso é concedido ou negado ao módulo AT como um todo, e a filial do usuário determina o escopo dos dados.

Funcionalidades com restrição adicional dentro do AT:
| Funcionalidade | Restrição |
|---------------|-----------|
| Trocar de filial no AT | Apenas `admin`/`super`/`dev` |
| Exportar Work Orders | Usuários autenticados com acesso ao AT |
| Excluir colaborador | Verificação de agendamentos; qualquer usuário AT |
| Toggle Sandbox Salesforce | Apenas `dev` |
| Configurações de integração | Apenas `admin`/`super`/`dev` |

### 4.3 Módulo de Cotações

| Funcionalidade | Restrição |
|--------------|-----------|
| Criar mapa de cotação | Usuários com acesso ao módulo |
| Registrar preços de fornecedores | Estágio 2, usuário responsável ou admin |
| Aprovar mapa (Estágio 4) | Usuário aprovador designado ou admin |
| Excluir mapa | Apenas admin (mapa em qualquer estágio) |
| Ver todos os mapas da filial | Usuários com acesso ao módulo |
| Ver mapa de outra filial | Apenas admin |

---

## 5. Gestão de Usuários

### 5.1 Ciclo de Vida do Usuário

```
Criação (admin) → Ativação (email de boas-vindas) → 
Configuração de privilégios → Uso → 
Desativação (admin) → Exclusão (opcional)
```

### 5.2 Campos do Perfil de Usuário

| Campo | Descrição |
|-------|-----------|
| `uid` | ID único Firebase Auth |
| `name` | Nome completo |
| `email` | E-mail (usado para login) |
| `userType` | `normal` \| `admin` \| `super` \| `dev` |
| `clientId` | Tenant ao qual o usuário pertence |
| `branchId` | ID da filial do usuário |
| `branchName` | Nome da filial (usado para filtros) |
| `privileges` | Array de chaves de privilégio concedidas |
| `favorites` | Array de módulos favoritados no menu lateral |
| `isActive` | Se o usuário pode fazer login |
| `photoUrl` | Foto de perfil (opcional) |
| `phone` | Telefone de contato (opcional) |
| `createdAt` | Data de criação do usuário |
| `lastLogin` | Última data de acesso |

### 5.3 Favoritos do Menu Lateral

Usuários `normal` podem favoritar módulos no menu lateral para acesso rápido. O menu exibe primeiro os módulos favoritos, depois os demais. Esta personalização é salva e persiste entre sessões e dispositivos.

Apenas módulos para os quais o usuário tem privilégio (ou que são públicos) aparecem para favoritar.

### 5.4 Gestão de Usuários pelo Administrador

O painel de administração permite:
- Criar novos usuários (envia e-mail de convite via Firebase Auth)
- Editar tipo de usuário e filial
- Conceder/revogar chaves de privilégio com interface visual (toggles por módulo)
- Desativar usuários (sem excluir dados históricos)
- Ver último acesso e atividade recente
- Resetar senha (envia e-mail de redefinição via Firebase Auth)

---

## 6. Segurança em Camadas

O controle de acesso do Kaizen opera em múltiplas camadas independentes e complementares:

### Camada 1 — Firebase Authentication
- Toda sessão exige token JWT válido emitido pelo Firebase Auth
- Tokens expiram em 1 hora e são renovados automaticamente pelo SDK
- Sem token válido = nenhuma operação Firestore ou Cloud Function é executada

### Camada 2 — Firebase Security Rules (Firestore)
Regras definidas em `firestore.rules` validadas pelo servidor antes de qualquer leitura/escrita:
```javascript
// Exemplo simplificado das regras
match /clients/{clientId}/{document=**} {
  allow read, write: if request.auth != null
    && request.auth.token.clientId == clientId;
}
```
- Usuário só acessa dados do próprio `clientId` (tenant)
- Nenhum acesso cross-tenant é possível, mesmo com manipulação do cliente

### Camada 3 — Validação nas Cloud Functions
Todas as Cloud Functions validam o `context.auth` (Firebase Auth context):
```javascript
if (!context.auth) throw new functions.https.HttpsError('unauthenticated', ...);
const clientId = context.auth.token.clientId;
const userType = context.auth.token.userType;
// ... validações de filial, tipo de ação, etc.
```
- Filial validada no servidor para chamadas ao Salesforce
- Ações destrutivas requerem tipo de usuário adequado

### Camada 4 — BranchAccessService (Flutter)
- Filtros de filial aplicados a todas as operações antes de enviá-las ao servidor
- Mesmo que um usuário manipule o estado local, as regras de segurança garantem o isolamento

### Camada 5 — hasPrivileges() (UI Flutter)
- Módulos não autorizados não aparecem na interface (invisível ao usuário)
- Não é uma barreira de segurança por si só (é cosmética), mas melhora a experiência
- A segurança real está nas camadas 1–4 acima

---

## 7. Matriz Resumida de Acesso

### 7.1 Módulos de Obra por Tipo de Usuário

| Módulo | `normal` | `admin` | `super`/`dev` |
|--------|---------|---------|--------------|
| Medições Kaizen | Com privilégio `daily_measurements` | ✅ | ✅ |
| Kaizen Diário | Com privilégio `daily_kaizen` | ✅ | ✅ |
| Gantt de Controle | Com privilégio `physical` + `baseline_view` | ✅ | ✅ |
| Planos de Ação | Com privilégio `action_plans_view` | ✅ | ✅ |
| Quadro de Restrições | Com privilégio `mid_term` | ✅ | ✅ |
| FVS Preenchimento | Com privilégio `fvs_fill` | ✅ | ✅ |
| FVS Aprovação | Com privilégio `fvs_approval` | ✅ | ✅ |
| Controle de Custo | Com privilégio `cost_control_view` | ✅ | ✅ |
| Almoxarifado | Com privilégio `warehouse_view` | ✅ | ✅ |
| Controle de Unidades | Com privilégio `units` | ✅ | ✅ |
| Estabilização Lean | Com privilégio `lean_inspection` | ✅ | ✅ |
| Farol de Contratações | Com privilégio `supply_schedule` | ✅ | ✅ |
| Reserva de Veículos | Com privilégio `vehicle_reservation` | ✅ | ✅ |
| BI/Estatísticas | Com privilégio `prod_bi` | ✅ | ✅ |

### 7.2 Módulo AT e Cotações

| Ação | `normal` | `admin` | `super`/`dev` |
|------|---------|---------|--------------|
| Visualizar Work Orders (própria filial) | ✅ | ✅ | ✅ |
| Visualizar Work Orders (qualquer filial) | ❌ | ✅ | ✅ |
| Criar agendamento | ✅ | ✅ | ✅ |
| Exportar Work Orders Excel | ✅ | ✅ | ✅ |
| Gerenciar colaboradores AT | ✅ | ✅ | ✅ |
| Toggle ambiente Salesforce | ❌ | ❌ | ✅ (dev) |
| Criar mapa de cotação | ✅ | ✅ | ✅ |
| Aprovar mapa de cotação | Por designação | ✅ | ✅ |
| Excluir mapa de cotação | ❌ | ✅ | ✅ |
| Gerenciar usuários | ❌ | ✅ | ✅ |
| Configurar privilégios de usuário | ❌ | ✅ | ✅ |

---

*Documento: Perfis de Acesso e Controle de Permissões | Versão 1.0 | Kaizen Gerenciamento de Obras*
