---
layout: page
title: "Módulo de Assistência Técnica"
permalink: /kaizen/tecnico/at/
category: "Módulos"
order: 3
toc: true
---

# Kaizen — Módulo de Assistência Técnica (AT)

## Sumário
1. [Visão Geral do Módulo AT](#1-visão-geral-do-módulo-at)
2. [Home / Dashboard AT](#2-home--dashboard-at)
3. [Work Orders (Ordens de Trabalho)](#3-work-orders-ordens-de-trabalho)
4. [Cases (Casos Salesforce)](#4-cases-casos-salesforce)
5. [Colaboradores](#5-colaboradores)
6. [Agendamentos](#6-agendamentos)
7. [Status dos Colaboradores](#7-status-dos-colaboradores)
8. [Materiais AT](#8-materiais-at)
9. [Fornecedores](#9-fornecedores)
10. [Analytics e Relatórios](#10-analytics-e-relatórios)
11. [Controle Multi-Filial](#11-controle-multi-filial)

---

## 1. Visão Geral do Módulo AT

O Módulo de Assistência Técnica é o sistema de gestão de serviços de pós-obra do Kaizen, desenvolvido como substituto às funcionalidades anteriores do Salesforce Field Service. Ele centraliza a operação de todas as equipes técnicas que atendem chamados e ordens de serviço nos empreendimentos entregues.

### Fontes de Dados
- **Salesforce CRM/FSL:** Work Orders e Cases criados no Salesforce são sincronizados para o Kaizen via Cloud Functions (incremental sync a cada 15 min)
- **Azure Synapse:** fallback para Work Orders em lote quando a API Salesforce está indisponível
- **Banco de dados:** agendamentos, colaboradores, status de colaboradores, materiais e fornecedores são gerenciados diretamente no Kaizen

### Isolamento Multi-Filial
O módulo AT opera com isolamento completo por filial. O controlador principal (`AtMainController`) mantém a filial ativa selecionada e propaga o filtro para todos os sub-controladores:
```dart
void setSelectedBranch(String branch) {
  selectedBranch.value = branch;
  _scheduleController.refetchForBranch(branch);
  _collaboratorsController.refetchForBranch(branch);
  _materialsController.refetchForBranch(branch);
  // ... todos os sub-controladores
}
```

---

## 2. Home / Dashboard AT

### 2.1 Painel de Hoje
- Lista de todos os agendamentos do dia corrente para a filial ativa
- Exibição em tempo real com sincronização automática
- Agrupamento por técnico ou por horário
- Status colorido em cada cartão: Agendado, Em Rota, Em Atendimento, Concluído, Cancelado

### 2.2 KPIs do Dia
| KPI | Descrição |
|-----|-----------|
| Agendamentos Hoje | Total de compromissos para o dia corrente |
| Concluídos | Atendimentos finalizados no dia |
| Pendentes | Agendamentos ainda não iniciados ou em andamento |
| Cancelados | Compromissos cancelados no dia |
| Horas Trabalhadas | Somatório de horas efetivas de atendimento |
| Taxa de Conclusão | % de agendamentos concluídos vs. total do dia |

### 2.3 Modo Kanban
- Visualização em colunas Kanban por status dos agendamentos
- **Agrupamento configurável:**
  - Por torre/empreendimento
  - Por status do agendamento
  - Por tipo de serviço
- Drag & drop para mover agendamentos entre status
- Cards com informações resumidas: técnico, cliente, endereço, horário

### 2.4 Suporte Offline
- O dashboard armazena localmente os dados da última sincronização (Hive)
- Indicador visual quando operando em modo offline
- Sincronização automática ao recuperar conectividade

---

## 3. Work Orders (Ordens de Trabalho)

### 3.1 Descrição
Tela principal de gestão de todas as Ordens de Trabalho sincronizadas do Salesforce. O trabalho de campo de assistência técnica gira em torno das Work Orders.

### 3.2 Listagem e Paginação
- Carregamento paginado de até **300 Work Orders por vez**
- Paginação server-side para grandes volumes
- Botão "Carregar mais" para paginas adicionais
- Contador de total de registros disponíveis

### 3.3 Colunas Configuráveis (30+ colunas disponíveis)
O usuário seleciona quais colunas deseja exibir e a ordem, salvando a preferência por perfil. Exemplos de colunas disponíveis:

| Coluna | Campo Salesforce |
|--------|-----------------|
| Número da OT | `WorkOrderNumber` |
| Status | `Status` |
| Prioridade | `Priority` |
| Tipo de Trabalho | `WorkType.Name` |
| Cliente | `Account.Name` |
| Empreendimento | `Account.Name` (filtro) |
| Torre | `Torre__c` |
| Unidade | `Unidade__c` |
| Descrição | `Description` |
| Data Criação | `CreatedDate` |
| Data Agendada | `ScheduledStartTime` |
| Técnico Designado | `ServiceResource.Name` |
| Filial | `Branch__c` |
| Área | `Area__c` |
| Projeto | `Project__c` |
| Tipo de Defeito | `DefectType__c` |
| Causa Raiz | `RootCause__c` |
| Resolução | `Resolution__c` |
| Horas Planejadas | `PlannedHours` |
| Horas Reais | `ActualHours` |

### 3.4 Filtros Avançados
- **Status:** múltipla seleção (Aberto, Em Execução, Concluído, Cancelado, Em Espera)
- **Prioridade:** múltipla seleção (Baixa, Média, Alta, Crítica)
- **Período:** data de criação ou data agendada (de/até)
- **Tipo de Trabalho:** dropdown de tipos disponíveis
- **Empreendimento:** por obra/conta
- **Técnico:** por recurso designado
- **Filial:** por filial (admin pode trocar; usuário comum fixo na própria filial)
- **Pesquisa livre:** busca textual por número, cliente, descrição, endereço

### 3.5 Detalhe da Work Order
- Todas as informações da OT no Salesforce
- Histórico de alterações de status
- Service Appointments vinculados
- Fotos e anexos (via servidor de armazenamento)
- Comentários internos
- Botões de ação: Iniciar, Pausar, Concluir, Cancelar (atualiza status no Salesforce via Cloud Function)

### 3.6 Busca Avançada e Exportação
- Busca combinada com múltiplos filtros simultâneos
- **Exportação para Excel (.xlsx)** de todos os Work Orders filtrados
- Relatório inclui todos os campos selecionados nas colunas visíveis

### 3.7 Histórico por Work Order
- Timeline completa de eventos: criação, mudanças de status, conclusão
- Data/hora e usuário que realizou cada ação
- Integrado com os logs do Salesforce via sincronização

---

## 4. Cases (Casos Salesforce)

### 4.1 Descrição
Gerenciamento dos Cases abertos no Salesforce — chamados técnicos de clientes que podem ou não resultar em uma Work Order.

### 4.2 Funcionalidades
- Listagem paginada de Cases sincronizados do Salesforce
- Filtros por: Status, Prioridade, Origem, Filial, Divisão
- Pesquisa por número do caso, cliente, descrição

### 4.3 Campos Exibidos
| Campo | Descrição |
|-------|-----------|
| Número do Case | `CaseNumber` |
| Status | Aberto, Em Andamento, Aguardando Cliente, Fechado |
| Prioridade | Baixa, Média, Alta, Urgente |
| Origem | Telefone, E-mail, Web, Presencial |
| Cliente | Account/Contact vinculado |
| Descrição | Relato do problema |
| Data de Abertura | `CreatedDate` |
| Data de Resolução | `ClosedDate` |
| Técnico Responsável | `Owner.Name` |
| Filial | Campo customizado de filial |
| Divisão | Divisão de negócio responsável |

### 4.4 Paginação
- Paginação server-side com cursor para grandes volumes
- Limite configurável por carregamento

---

## 5. Colaboradores

### 5.1 Cadastro de Colaboradores AT
Gerenciamento da equipe técnica que executa os serviços de campo:

| Campo | Descrição |
|-------|-----------|
| Nome | Nome completo do colaborador |
| Matrícula | Código interno / matrícula de funcionário |
| Função | Cargo/especialidade (ex.: Técnico Eletricista, Encanador) |
| Custo/Hora | Valor hora para cálculo de custo de mão de obra |
| Salário | Salário mensal bruto |
| Encargos | Percentual de encargos sobre o salário |
| Filial | Filial à qual o colaborador está vinculado |
| Status | Ativo, Férias, Licença, Inativo |
| Data de Admissão | Para cálculo de benefícios e vigência |
| Telefone | Contato direto do técnico |

### 5.2 Importação em Massa via Excel
- **Upload de planilha Excel (.xlsx) com múltiplos colaboradores**
- Template de importação disponível para download
- Validação de campos obrigatórios antes do import
- Relatório de erros por linha em caso de dados inválidos
- Atualização incremental: colaboradores existentes são atualizados, novos são criados

### 5.3 Validação de Exclusão
- Antes de excluir um colaborador, o sistema verifica a existência de agendamentos vinculados
- Se houver agendamentos futuros, a exclusão é bloqueada com exibição dos agendamentos conflitantes
- O usuário pode reatribuir os agendamentos antes de prosseguir com a exclusão

### 5.4 Cálculo de Custo de Mão de Obra
- Custo real = (Horas trabalhadas × Custo/hora) + (Encargos calculados sobre salário)
- Integrado ao relatório de custos do módulo AT e ao controle de orçamento da obra

---

## 6. Agendamentos

### 6.1 Descrição
Gerenciamento dos Service Appointments — compromissos agendados entre técnicos e clientes para execução de Work Orders de assistência técnica.

### 6.2 Stream em Tempo Real
- Dados de agendamentos são atualizados em **tempo real** com sincronização automática
- Alterações feitas por qualquer usuário aparecem instantaneamente para todos
- Filtro de filial aplicado server-side na query do stream (`.where('branch', isEqualTo: activeBranch)`)
- Singleton controller (`ScheduleController._instance`) mantém estado entre navegação de telas

### 6.3 Visão de Agenda
- **Lista diária:** agendamentos do dia selecionado por técnico
- **Visão Gantt:** timeline horizontal de agendamentos por técnico por período
- Cores por status de agendamento
- Indicação de conflitos de horário

### 6.4 Gestão de Agendamentos
- Criação de agendamento: vinculação de Work Order + Técnico + Data/Hora + Duração
- Edição com validação de conflito de agenda do técnico
- Status do agendamento:
  - Agendado
  - Confirmado (pelo técnico)
  - Em Rota
  - Em Atendimento
  - Concluído
  - Cancelado / Remarcado
- Log de todas as mudanças de status com autor e timestamp

### 6.5 Suporte Offline e SyncQueue
- Criação/edição de agendamentos offline é enfileirada na `SyncQueueService`
- Ao conectar, a fila é processada em ordem e o servidor é atualizado
- Conflitos (outro usuário alterou o mesmo agendamento) são flagueados para revisão manual

---

## 7. Status dos Colaboradores

### 7.1 Gestão de Períodos de Ausência
Controle dedicado de períodos especiais que afetam a disponibilidade dos técnicos:

| Tipo de Período | Exemplos |
|----------------|---------|
| Férias | Férias regulares (programadas de baixo CLT) |
| Licença Médica | Afastamento por atestado |
| Licença Maternidade/Paternidade | — |
| Licença Especial | Qualquer outro afastamento justificado |
| Treinamento | Cursos e capacitações fora do campo |

### 7.2 Funcionalidades
- Cadastro de períodos com data início/fim e tipo
- **Stream em tempo real** — alterações refletem imediatamente na agenda do colaborador
- Filtragem automática de colaboradores de ausência: ao criar agendamento, técnicos em férias/licença não aparecem como disponíveis
- Histórico completo de ausências por colaborador
- Exportação de relatório de ausências por período

---

## 8. Materiais AT

### 8.1 Estoque de Materiais AT
Controle de peças e materiais específicos para atendimento de assistência técnica:
- Catálogo de peças por categoria e filial
- Saldo em estoque por material e localização (almoxarifado da filial)
- Registro de uso de material por Work Order executada
- Rastreabilidade: qual peça foi usada em qual atendimento, por qual técnico

### 8.2 Pedidos de Compra AT
- Solicitação de reposição de materiais com aprovação interna
- **Notificações push** para aprovadores via FCM (tópico `pedidos_{clientId}_{branchId}`)
- Status do pedido: Solicitado, Aprovado, Em Compra, Recebido, Cancelado
- Histórico de pedidos por material e período

### 8.3 Integração Mega ERP
- Consulta de disponibilidade de materiais no estoque central do ERP
- Fields sincronizados: `CodigoProduto`, `NomeProduto`, `NomeGrupo`, `UnidadeMedida`
- Requisições do Kaizen AT podem ser enviadas ao Mega ERP para processamento
- Alertas de materiais em falta cruzando saldo Kaizen vs. saldo ERP

---

## 9. Fornecedores

### 9.1 Cadastro de Fornecedores
Gerenciamento de prestadores de serviço terceirizados que atuam na assistência técnica:

| Campo | Descrição |
|-------|-----------|
| Razão Social | Nome jurídico do fornecedor |
| CNPJ | Cadastro Nacional de Pessoa Jurídica |
| Especialidade | Tipo de serviço prestado (elétrica, hidráulica, etc.) |
| Responsável | Contato principal na empresa |
| Telefone / E-mail | Meios de contato |
| Filial | Filial(is) em que presta serviço |
| Status | Ativo, Inativo, Homologado, Em análise |
| Avaliação | Score de desempenho (gerado pelo histórico de atendimentos) |

### 9.2 Gestão por Filial
- Cada filial tem seu próprio catálogo de fornecedores aprovados
- Usuários veem apenas fornecedores da própria filial
- Administradores podem gerenciar fornecedores de todas as filiais

---

## 10. Analytics e Relatórios

### 10.1 Filtros do Dashboard Analytics
| Filtro | Opções |
|--------|--------|
| Período | De/até (datas com seletor) |
| Colaborador | Specific technician or all |
| Empreendimento | Obra/empreendimento specific |
| Filial | Specific branch or all (admin only) |

### 10.2 Gráficos Disponíveis
| Gráfico | Tipo | Descrição |
|---------|------|-----------|
| Atendimentos por Status | Pizza / Barra | Distribuição de Work Orders por status no período |
| Atendimentos por Dia da Semana | Barra | Volume de atendimentos por dia (Seg-Dom) |
| Atendimentos por Tipo de Serviço | Pizza | Breakdown por categoria de serviço |
| Atendimentos por Área | Barra horizontal | Quais áreas do empreendimento mais acionam AT |
| Atendimentos por Empreendimento | Barra | Comparativo de volume por obra |
| Evolução Temporal | Linha | Tendência de volume de chamados ao longo do período |
| Tempo Médio de Atendimento | Indicador | TMA por tipo de serviço |
| Taxa de Resolução no 1º Atendimento | Indicador | % de Work Orders concluídas sem revisita |

### 10.3 Exportação
- **Exportação para Excel (.xlsx)** de todos os dados do período filtrado
- Inclui todos os campos das Work Orders filtradas
- Formato estruturado para análise em ferramentas de BI externas (Power BI, Excel)

---

## 11. Controle Multi-Filial

### 11.1 Seletor de Filial
- Dropdown no cabeçalho do módulo AT para selecionar a filial ativa
- **Usuários comuns:** campo desabilitado, fixo na própria filial
- **Admins/Supers/Devs:** dropdown livre para selecionar qualquer filial ou "Todas"

### 11.2 Propagação do Filtro
Ao trocar de filial, o `AtMainController.setSelectedBranch()` dispara `refetchForBranch()` em todos os sub-controladores:
- `ScheduleController` — recarrega agendamentos da nova filial
- `CollaboratorsController` — recarrega equipe técnica da nova filial
- `MaterialsController` — recarrega estoque da nova filial
- `VendorsController` — recarrega fornecedores da nova filial
- `WorkOrdersController` — aplica filtro de filial na próxima consulta

### 11.3 Isolamento Garantido
- Nenhuma consulta retorna dados de filial diferente da selecionada
- Chamadas às Cloud Functions Salesforce incluem o parâmetro de filial no payload
- Logs e auditoria incluem sempre a filial do contexto da operação

---

## Resumo de Capacidades do Módulo AT

| Capacidade | Suporte |
|------------|---------|
| Sincronização com Salesforce Work Orders | ✅ (incremental, 15 min) |
| Sincronização com Salesforce Cases | ✅ |
| Agendamentos em tempo real | ✅ (sincronização automática) |
| Dashboard Kanban | ✅ |
| Colunas configuráveis por usuário | ✅ (30+ colunas) |
| Exportação Excel | ✅ (Work Orders + Analytics) |
| Importação Excel de colaboradores | ✅ |
| Funcionamento offline (campo) | ✅ (SyncQueueService + Hive) |
| Isolamento multi-filial | ✅ |
| Notificações push (materiais) | ✅ (FCM por tópico) |
| Integração Mega ERP (materiais) | ✅ |
| Analytics com gráficos | ✅ |

---

*Documento: Módulo de Assistência Técnica | Versão 1.0 | Kaizen Gerenciamento de Obras*
