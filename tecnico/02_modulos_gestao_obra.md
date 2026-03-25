---
layout: page
title: "Módulos de Gestão de Obra"
permalink: /tecnico/modulos/
category: "Módulos"
order: 2
toc: true
---

# Kaizen — Módulos de Gestão de Obra

## Visão Geral Estruturada

O Kaizen organiza a gestão de construção em **oito categorias principais** e **múltiplas subcategorias**, cada uma com sua própria interface, privilégios de acesso e integrações. Os módulos são acessados via menu de navegação na "Seção no Menu" indicada.

**Integração com Prevision:** O sistema suporta importação automática de cronogramas da plataforma de planejamento **Prevision**, sincronizando atividades, lotes, serviços, curvas S e análises de caminho crítico.

## Sumário Estruturado

### **Planejamento e Medições Kaizen** (Produtividade)
- Planejamento Kaizen | Medições Kaizen | Kaizen Diário | Gestão da Estrutura | Histórico | Estatísticas | Dashboard

### **Planejamento: Cronograma e Controle de Custos**
- Atividades | Gantt de Controle | Painel de Produção | Análise do Planejamento (Curva S) | Gestão de Cronogramas | Equipes

### **Controle: Medições Físicas e Produção**
- Medições Físicas | Painel de Produção | Gestão de Equipes e Histograma | Planos de Ação

### **Rotinas de Gestão Lean** (6M/6WLA)
- Planos de Ação | Quadro de Restrições | Farol de Contratações | Estabilização Lean

### **Qualidade**
- Gestão de FVS | Inspeção Final de Obra | Inspeção Final da Qualidade

### **Controle de Custos e Orçamento**
- Custo e Orçamento | Cotações | Desvios Categorizados

### **Controle de Unidades** (Imobiliário)
- Ficha de Unidade | Personalização | Avanço de Obra | Inspeção Final | Vistoria do Cliente | Gerador de Etiquetas QR

### **Gestão de Materiais**
- Almoxarifado Digital | Controle de Kits | Reserva de Ativos (Veículos e Ferramentas)

---

## I. PLANEJAMENTO E MEDIÇÕES KAIZEN (Produtividade)

---

## 1. Produtividade

### 1.1 Planejamento Kaizen
Módulo de planejamento por atividade da EAP. Cada atividade recebe um tipo de plano de produção:

- **Percentual:** avanço definido como porcentagem do total
- **Croqui:** associação a uma máscara de planta segmentada para medir subáreas individualmente
- **Ciclo:** ciclo diário de trabalho com parâmetros de ritmo e prazo

Em todos os tipos, o plano registra a composição de equipe planejada: quantidade de profissionais e ajudantes por turno. Um plano pode ser copiado para múltiplas atividades da mesma categoria.

**Chave de privilégio:** `daily_measurements`

---

### 1.2 Medições Kaizen
Registro diário de produção. Cada medição captura:

- **Produção realizada:** quantidade no índice do plano (m², m³, peças etc. conforme máscara de croqui ou percentual)
- **Entradas de mão de obra:** total de pessoas, profissionais, ajudantes, dias trabalhados e horas (campo opcional)
- **Problemas registrados:** seleção em lista predefinida por serviço, com descrição de causa e empresa/empreiteira responsável
- **Metas em tempo real:** indicador de cor mostrando se a produção e as pessoas entradas atingem a meta do lote

O sistema calcula automaticamente o Índice de Produtividade (produção ÷ trabalho) e projeta os dias restantes para conclusão do lote.

**Chave de privilégio:** `daily_measurements`

---

### 1.3 Kaizen Diário
Dashboard de acompanhamento diário da obra. Para a data selecionada exibe:

- Medições registradas no dia agrupadas por lote e serviço
- Status de cada atividade (planejada, em progresso, concluída)
- Resumo de problemas por serviço
- Atividades sem medição registrada para o dia corrente
- Acesso direto ao formulário de medição de cada atividade

Possui filtro por lote, seletor de data e versão mobile com menu lateral colapsável.

**Chave de privilégio:** `daily_measurements` ou `daily_kaizen`

---

### 1.4 Gestão da Estrutura
Visualização matricial da obra no formato Monkey Chart: linhas = lotes, colunas = serviços, células = atividades. Para cada célula são carregados os planos de ciclo sob demanda, com indicadores de conclusão e índice de produtividade calculado.

Controles de zoom (`monkeyTileScale`) e painéis de estatísticas laterais colapsáveis. Usada para visão macro do sequenciamento e ocupação de equipes por lote/serviço.

**Chave de privilégio:** `daily_measurements`

---

### 1.5 Histórico de Medições
Consulta paginada de todas as medições realizadas. Carrega as 20 mais recentes na abertura; navegação retroativa por filtros (período, serviço, lote, responsável). Cada item abre a visualização completa da medição — modo leitura ou edição conforme permissão do usuário.

**Chave de privilégio:** `daily_measurements`

---

### 1.6 Estatísticas
Dashboard analítico com três modos de visão selecionáveis: **por Serviço**, **por Empresa/Empreiteira** ou **por Lote**. Métricas consolidadas:

- Produção total, horas totais, dias totais e pessoas totais
- Quantidade de medições e de problemas registrados
- Índice de produtividade médio, mínimo e máximo
- Comparativo com meta de produção do contrato da obra

Filtros de data e serviço via `FilterController` compartilhado.

**Chave de privilégio:** `prod_bi`

---

### 1.7 Dashboard de Desempenho
Análise de desempenho com visualizações gráficas de duas abas:

- **Aba Serviços:** tabela ordenável + gráfico de barras de produção ou custo por serviço; comparativo realizado vs. previsto por período
- **Aba Problemas:** gráfico de pizza com frequência de problemas por serviço e lista de ocorrências em aberto

Suporte a acúmulo mensal e exportação via impressão do browser.

**Chave de privilégio:** `prod_bi`

---

## II. PLANEJAMENTO: CRONOGRAMA E CONTROLE DE CUSTOS

---

### 2.1 Atividades
Tabela completa de atividades do cronograma ativo. Colunas: serviço, lote, início, fim, duração, custo, status, % concluído. Operações disponíveis:

- Edição inline de datas e custos diretamente na tabela
- Busca de atividade por nome com debounce
- Filtros: somente baseline, somente críticas, exibir/ocultar concluídas
- Cálculo do caminho crítico sob demanda
- Fila de desfazer/refazer (até 15 passos)
- Exportação em Excel (.xlsx) e PDF

**Chave de privilégio:** `baseline_view`

---

### 2.2 Gantt de Controle — Reprogramação Livre
Visualização Gantt interativa do cronograma ativo com **reprogramação em tempo real**. Barras coloridas por serviço sobre eixo de tempo scrollável. Funcionalidades:

- Linhas de dependência com caminho crítico destacado em vermelho
- **Drag das barras para reprogramar** com impacto cascata automático; duração recalculada automaticamente
- Filtro por serviço (dropdown na AppBar)
- Toggle para exibir/ocultar atividades concluídas e serviços 100% finalizados
- Undo/Redo de mudanças de datas
- Relatório de impacto de reprogramações

Renderizado com `CustomPainter` (`GanttGridPainter`, `GanttActivityPainter`) para performance otimizada.

**Chave de privilégio:** `baseline_view`

---

### 2.3 Painel de Produção — Visão por Status
Matriz de produção no formato Monkey Chart orientada ao cronograma: cada célula (lote × serviço) exibe o status de conclusão com código de cor:
- 🟢 Planejada
- 🟡 Em progresso
- 🔵 Concluída
- 🔴 Atrasada

Toque na célula abre os detalhes e permite edição da atividade, visualização de equipes vinculadas e histórico de medições.

**Chave de privilégio:** `baseline_view`

---

### 2.4 Análise do Planejamento — Curva S
Curva S acumulada (cronograma × realizado) com três séries simultâneas sobre dois eixos:

| Série | Significado |
|-------|------------|
| **S** (Scheduled) | Cronograma previsto ativo |
| **B** (Baseline) | Cronograma de referência (baseline) |
| **D** (Done) | Realizado acumulado |

Os valores são calculados por atividade (% ou dias conforme configuração) e acumulados por dia, com detalhe mensal por serviço. O resultado é cacheado no documento do cronograma (`cffCalculatedAt`, `cffVersion`) para evitar reprocessamento.

**Chave de privilégio:** `baseline_view`

---

### 2.5 Medições Físicas
Registro de medições físicas contratuais (volumes, áreas, quantidades por serviço). Vinculadas a contratos e boletins de medição com fluxo de aprovação configurável. Gera boletim de medição em PDF com assinatura digital.

**Chave de privilégio:** `physical`

---

### 2.6 Gestão de Cronogramas e Importação
Gerenciamento de versões do cronograma: criação de nova versão, definição do baseline de referência, comparação entre versões e histórico completo de revisões com autor e timestamp.

**Fontes de importação suportadas:**

| Fonte | Formato | Observação |
|---|---|---|
| **Prevision** | GraphQL | Fonte primária – veja seção "A. Integração Prevision" abaixo para detalhes |
| **Excel** | `.xlsx` | Arquivo enviado à Cloud Function `excelToJson`; usuário mapeia as colunas manualmente |
| **JSON** | `.json` | Importa arquivo JSON estruturado no formato interno do Kaizen |

**Chave de privilégio:** `physical`

---

### 2.7 Equipes — Gestão e Histograma
Cadastro e gestão das equipes de produção com múltiplas visualizações:

**Equipes (Cadastro):**
- Associação de colaboradores a equipes (compartilhado com AT)
- Custo/hora por equipe para cálculo de mão de obra
- Histórico de desempenho por equipe

**Histograma de Equipes:**
- Visualização de alocação de pessoas por período
- Mostrar/ocultar equipes para análise de capacidade
- Projeção de necessidade vs. disponibilidade

**Chave de privilégio:** `teams`

---

## III. CONTROLE: MEDIÇÕES FÍSICAS E PRODUÇÃO

### 3.1 Medições Físicas (Detalhes)
Complementa a seção 2.5; foco em registro contratual de volumes executados.

- Vinculação a contratos e boletins de medição
- Fluxo de aprovação configurável por obra
- Relatório de medição em PDF

**Chave de privilégio:** `physical`

---

### 3.2 Gestão de Equipes com Histograma
Veja seção 2.7 acima para detalhes de histograma de alocação.

---

### 3.3 Planos de Ação (Kanban e Dashboard)
Veja seção "IV. Rotinas de Gestão Lean" (item 4.1) para detalhes completos.

---

## IV. ROTINAS DE GESTÃO LEAN (6M / Last Planner / 6WLA)

> Metodologia de Gestão de Médio Prazo usando Last Planner System (LPS) e análise de restrições pelo método 6M.

### 4.1 Planos de Ação — Gestão Integrada
Gestão de itens de ação com três modos de visualização coordenados:

**Kanban:**
- Cartões agrupáveis por responsável, status, setor, prioridade, risco ou semana de vencimento
- Drag & drop entre colunas
- Filtros: responsável, setor, prioridade, tags de serviço

**Lista:**
- Visualização tabular com 8+ critérios de ordenação
- Busca de texto integrada
- Paginação de 50+ itens

**Dashboard KPIs:**
- Total de ações abertas, em andamento, concluídas, canceladas, vencidas
- Gráfico de trend de abertura × conclusão
- Alertas de ações vencidas

**Campos do Plano de Ação:**
- Título, descrição (texto livre)
- Prazo, status, prioridade (`baixa / média / alta`), risco (`baixo / médio / alto`)
- Responsável (usuário), setor, áreas de impacto (multi-select)
- Vínculos a atividades do cronograma (multi-link)
- Tags de serviço  (categorizadas por cor)

**Funcionalidades:**
- Notificações push automáticas ao responsável via FCM (criação, vencimento)
- Comentários internos por item com @mentions
- Anexos (fotos, PDFs)
- Exportação em PDF e Excel
- Histórico de mudanças

**Chave de privilégio:** `action_plans_view` (ou usuário admin/super)

---

### 4.2 Quadro de Restrições — Last Planner System
Lookahead de restrições (LPS — Last Planner) com janela configurável, visões múltiplas e análise 6M via agente KAI.

**Conceito:**
Restrições são bloqueadores que impedem o início de uma atividade no cronograma. O quadro organiza as semanas à frente identificando e removendo essas barreiras.

**Estrutura de Dados por Restrição:**
- **Serviço afetado:** qual atividade do cronograma será impactada
- **Descrição do bloqueador:** (ex.: "Aguardando inspeção de estrutura", "Falta de material")
- **Categoria 6M:** método de Ishikawa — Mão de Obra, Material, Máquina, Medição, Meio Ambiente, Método
- **Data de comprometimento:** quando a restrição deve ser removida
- **Status:** `aberta / em andamento / removida / vencida`
- **Responsável, setor, anexos**

**Visões Disponíveis:**
- **Grade Semanal:** serviço × semana (lookahead: 8 semanas padrão, configurável)
- **Painel de Serviços:** filtro de restrições ativas compartilhadas por serviço
- **Painel de Estatísticas:** contagens por status, tendências, serviços mais restringidos

**Ordenação:** 8+ opções (por data de comprometimento, serviço, status, ou data de início da atividade vinculada)

**Exportação:** Excel multi-obra consolidado com formatação de cores por status

**Notificações:** Alertas automáticos de vencimento (configurável) via FCM; deep link abre diretamente a restrição no app (`pendingOpenRestrictionId`)

**IA — KAI:** 
- Agente `kai-restriction-analyst` gera relatório analítico detalhado por categoria 6M (ex.: "70% das restrições são por falta de Material; 20% por Mão de Obra")
- Agente `kai-suggestion` propõe novas restrições com base em atividades do cronograma próximas com dependências

**Chave de privilégio:** `mid_term` (requer cronograma ativo)

---

### 4.3 Farol de Contratações — Supply Schedule
Cronograma de suprimentos com semáforo de SLA (verde/amarelo/vermelho) integrado ao cronograma da obra.

**Objetos Gerenciados:**

| Objeto | Descrição |
|--------|-----------|
| **Itens de Suprimento** | Nome do item, fornecedor, categoria, SLA padrão (ex.: 30 dias), margem de segurança, estoque mínimo |
| **Processos de Suprimento** | Rastreamento do status: data de solicitação, data prometida, data recebida, responsável, status (`em aberto / em andamento / recebido / vencido`) |

**Matriz de Status:**
- Cada célula (item × fornecedor) exibe a situação do SLA em cores
- 🟢 Verde: dentro do SLA
- 🟡 Amarelo: atenção (últimos 10 dias)
- 🔴 Vermelho: vencido ou atrasado

**Integração com Cronograma:**
- Cada item vinculado a atividades que dependem da entrega
- Alerta automático quando prazo ameaça impactar o cronograma

**Exportação:** 
- CSV com matriz de status
- Excel consolidado multi-obra com análise de tendências

**Chave de privilégio:** `supply_schedule` ou perfil admin

---

### 4.4 Estabilização Lean — 5S Digital
Aplicação de checklists de inspeção Lean com foco em estabilização de obra.

**Estrutura:**
- Checklists com estrutura hierárquica (seções e subseções)
- Cada inspeção registra: data, torre/bloco inspecionado, responsável
- Por item: resultado (`conforme / não conforme / parcial`), foto de evidência, observação

**Scoring:**
- Score por seção (roll-up automático dos filhos)
- Score total da inspeção em porcentagem
- Histórico de até 24 meses com gráfico de evolução
- Média histórica calculada para comparativo

**Fotos:**
- Compressão automática antes do upload
- Armazenamento no Firebase Storage
- Exibição em galeria por inspeção

**Chave de privilégio:** `lean_inspection`

---

## V. QUALIDADE

### 5.1 Gestão de Fichas de Verificação de Serviço (FVS)
Dashboard central de FVS com fluxo de preenchimento, inspeção e aprovação.

**Dois Níveis de Hierarquia:**

| Nível | Descrição |
|-------|-----------|
| **Templates (`DefaultFVSModel`)** | Checklists padronizados por tipo de serviço; reutilizáveis entre obras; gerenciados pelo admin tenant |
| **Instâncias (`FVSModel`)** | FVS preenchida para uma atividade específica na obra |

**Fluxo de Status:**
```
Em Verificação (preenchimento em campo)
       ↓
Inspecionada (verificação concluída)
       ↓
Aprovada (aprovador validou)
  ou ↘
    Rejeitada → Pendência (volta para campo)
```

**Dados por Instância:**
- Atividade vinculada (serviço + lote denormalizados)
- Responsável pelo preenchimento, aprovador
- Prioridade (`baixa / normal / alta`)
- Lista de itens com resultado por item (`conforme / não conforme / N/A`)
- Contagem de itens rejeitados

**Filtros no Dashboard:**
- Status (Em Verificação, Inspecionada, Aprovada, Rejeitada)
- Atividade, lote, tag de serviço
- Responsável, aprovador
- Prioridade
- Busca por texto livre

**Ordenações:** prioridade, recência, quantidade de pendências, nome da atividade

**Contadores:**
- Em Verificação · Pendências · Inspecionadas · Aprovadas
- Total de Itens Rejeitados (rollup)

**Exportação:** PDF da FVS com:
- Dados completos do checklist
- Fotos de cada item (se preenchidas)
- Assinaturas digitais (responsável + aprovador)
- Data e hora de preenchimento

**Chave de privilégio:** `fvs_fill` (preenchimento) | `fvs_approval` (aprovação) | `quality_manager` (admin)

---

### 5.2 Inspeção Final de Obra
Inspeção técnica pré-entrega (distinta da vistoria do cliente) com foco em conformidade técnica e normativa.

- Checklist específico por tipo de obra (residencial, comercial, etc.)
- Registro de pontos de atenção com fotos de comprovação
- Assinatura digital do engenheiro/inspetor
- Relatório de inspeção final em PDF para arquivo técnico
- Histórico de inspeções posteriores (revisitas)

**Chave de privilégio:** `quality_manager`

---

### 5.3 Inspeção Final da Qualidade
Inspeção complementar focada em aspectos de qualidade pós-entrega (materiais, acabamentos, funcionamento de sistemas).

- Checklist de itens finais (vidros, louças, tintura, sistemas elétricos/hidráulicos)
- Aprovação item a item ou pendência registrada
- Relatório consolidado de pendências per unidade
- Integração com vistoria do cliente (dados compartilhados)

**Chave de privilégio:** `quality_manager`

---

## VI. CONTROLE DE CUSTOS E ORÇAMENTO

### 6.1 Custo e Orçamento — Sistema Integrado

Controle orçamentário completo com comparativo planejado × realizado × comprometido.

**Estrutura de Orçamento:**
- Códigos WBS hierárquicos (ex.: 01 → 01.01 → 01.01.01)
- Por item: descrição, referência de instalação (INS ID), quantidade, preço unitário, valor total
- Campos de aprovação e comentários distribuídos

**Comparativos Exibidos:**
| Campo | Significado |
|-------|-----------|
| **Orçado** | Valor original do contrato ou revisão aprovada |
| **Comprometido** | Valores de cotações aprovadas (Módulo de Cotações) |
| **Realizado** | Despesas lançadas no ERP (Azure Synapse) + consumo de estoque + notas fiscais |
| **Desvio** | Realizado − Orçado (positivo = estouro, negativo = economizado) |

**Versionamento e Rastreabilidade:**

- **Snapshots de Versão (`BudgetVersionModel`):** cada revisão mantém lista de alterações documentadas
  - Campo modificado, valor anterior, valor novo, autor, timestamp
  - Fluxo de aprovação para submissão de nova versão
  - Dialog de revisão das mudanças pendentes antes do commit

**Sistema de Comentários (Distribuído):**
- Comentários por item armazenados em buckets hash para otimizar queries
- Fontes rastreadas: `manual` (digitado pelo usuário) | `concat` (resumo gerado por KAI)
- Histórico de comentários com carregamento incremental

**Categorias de Desvio (para análise de variações):**

| Código | Categoria | Exemplo |
|--------|-----------|---------|
| `ORC` | Variação de Orçamento | Estimativa do projeto inadequada |
| `CST` | Variação de Custo | Custo de mercado acima do previsto |
| `PRJ` | Variação de Projeto | Escopo alterado pela engenharia |
| `ENG` | Variação de Engenharia | Solução técnica diferente da prevista |
| `PRZ` | Variação de Prazo | Urgência impactou preço ou escopo |

**Fontes de Custo Realizado Integradas:**

1. **`BudgetRealizedDetail`** — Despesas de execução com timestamp
2. **`StockERPDetail`** — Consumo de materiais do almoxarifado
3. **`StockPartnerEntry`** — Notas fiscais de fornecedores/subcontratado

**Exportação:**
- Excel com tabelas de desvio, listagem de contratos vinculados, análise de custo por serviço
- Relatórios multi-obra consolidados
- PDF para apresentação em reuniões

**Integração:**
- Dados realizados lidos do Azure Synapse em tempo real via `SynapseController`
- Sincronização de comprometidos com Módulo de Cotações aprovadas

**Chave de privilégio:** `cost_control_view`

---

### 6.2 Módulo de Cotações — Gestão de Mapas de Cotação

*Documentação completa em arquivo dedicado: [04 — Módulo de Cotações](./04_modulo_cotacoes.md)*

Resumo: o Módulo de Cotações digitaliza o processo de coleta de propostas comerciais, organizado em 4 estágios (Montagem → Cotação → Análise → Aprovado). Cada mapa vincula diretamente ao Controle de Custo, registrando compromissos orçamentários e categorias de variação.

---

## VII. INTEGRAÇÃO PREVISION

### A. Visão Geral da Integração

O **Prevision** ([Prevision.com.br](https://prevision.com.br)) é uma plataforma especializada de planejamento de obras que o Kaizen integra para importação automática de cronogramas, atividades, lotes, serviços e análises (curva S, caminho crítico, CFF).

**Pontos-chave:**
- **Sincronização automática:** cronogramas importados via API GraphQL do Prevision
- **Campo de vínculo:** cada obra no Kaizen possui um campo `previsionProjectId` que identifica o projeto correspondente no Prevision
- **Duas fases de importação:** incremental (diária) + sincronização completa (semanal ou manual)
- **Dados suportados:** Atividades, Lotes, Serviços, Curva S, Caminho Crítico, CFF (Cronograma Físico Financeiro)

---

### B. Fluxo de Sincronização

**Arquitetura:**

```
Obra no Kaizen (previsionProjectId = "ABC123")
       ↓
Cloud Function: syncPrevisionSchedules() [agendada daily]
       ↓
GraphQL Query ao endpoint Prevision
       ↓
Prevision API retorna: Activities, Lots, Services, S-Curve
       ↓
Mapeamento para ScheduleModel, ActivityModels, LotModels
       ↓
Firestore: atualiza em batch com { ScheduleVersion: N }
       ↓
PrevisionController no Flutter recarrega (reatividade GetX)
```

**Sincronização Incremental:**
- Consulta apenas registros modificados desde últim syncronização (via `LastModifiedDate`)
- Operações: `set(..., { merge: true })` no Firestore para idempotência
- Mantém histórico de versões de cronograma

**Sincronização Completa (Fallback):**
- Executada manualmente ou periodicamente (ex.: semanal)
- Recarrega todos os dados do Prevision para validação de integridade
- Útil se sincronização incremental falhou previamente

---

### C. Dados Importados

**Atividades (`ActivityModel`):**

| Campo Prevision | Campo Kaizen | Tipo |
|-----------------|-------------|------|
| `id` | `activityId` | String |
| `name` / `description` | `activityName` | String |
| `service` | `service` (link) | String (service ID) |
| `floor` / `lot` | `lot` (link) | String (lot ID) |
| `startAt` | `dataInicio` | DateTime |
| `endAt` | `dataFim` | DateTime |
| `workDuration` | `duracao` | Number (dias) |
| `cost` | `custo` | Number |
| `withinPeriod.percentageCompleted` | `percentConcluido` | Double (0-1) |

**Lotes (`LotModel`):**

| Campo | Significado |
|-------|-----------|
| `id`, `name` | Identificador único e nome do lote (ex.: "Lote A",  "Torre 3") |
| `floor` (referência) | Andar ou pavimento |

**Serviços (`ServiceModel`):**

| Campo | Significado |
|-------|-----------|
| `id`, `name` | Identificador e nome do serviço (ex.: "Alvenaria", "Elétrica") |

**Curva S (`Curve`):**

| Série | Significado |
|-------|-----------|
| `basePoints` | Valores dos pontos de base (custos/dias acumulados conforme cronograma) |
| `expectedPoints` | Valores esperados (planejado vs. data) |
| `realizedPoints` | Valores realizados (p/ futuro: quando integrar medições) |

**Caminho Crítico (`criticalPath`):**

- Array de IDs de atividades que fazem parte do caminho crítico
- Destacadas em vermelho no Gantt

**CFF (Cronograma Físico Financeiro):**

- Tabela com datas, itens de orçamento, custos base/esperado/realizado
- Pontos para cada perspectiva (monetária, física, de prazo)

---

### D. GraphQL Queries Utilizadas

O controller `PrevisionController` executa várias consultas GraphQL ao Prevision:

```dart
// Atividades com filtro por período
readPrevisionActivitiesV2(projectId, startDate, endDate)

// Lotes (Floors)
readPrevisionLots(startAt, projectId)

// Serviços
readPrevisionServices(startAt, projectId)

// Curva S
readPrevisionSCurve(projectId)

// CFF (Cronograma Físico Financeiro)
readPrevisionCffTable(projectId, budgetReportId)

// Caminho Crítico
readPrevisionCriticalPath(projectId)

// Pesos de Budget (ligação entre atividades e itens de orçamento)
readPrevisionBudgetWeights(budgetReportId, after)
```

Todas as queries usam a biblioteca `graphql_flutter` para execução HTTP contra o endpoint GraphQL do Prevision.

---

### E. Autenticação Prevision

A autenticação com o Prevision ocorre via **Token Bearer** armazenado em variáveis de ambiente:

```dart
// No Firebase: PREVISION_API_KEY ou similar
HttpLink httpLink = HttpLink(
  'https://api.prevision.com.br/graphql',
  httpClient: http.Client(),
  defaultHeaders: {
    'Authorization': 'Bearer $previsionToken',
  },
);
```

- Credenciais armazenadas de forma segura (não no código-fonte)
- Refresh automático se token expirar

---

### F. Tratamento de Falhas e Fallback

Se a sincronização Prevision falhar:
1. Log do erro no Cloud Logging
2. Cronograma existente mantém versão anterior
3. Próxima sincronização (agendada) tenta novamente
4. Se falha repetir por 3x consecutivamente → alerta ao admin

**Fallback manual:** usuários com privilégio `physical` podem importar cronograma via Excel ou JSON como alternativa.

---

### G. Campos de Configuração na Obra

Cada obra (`SiteModel`) inclui:

```dart
String? previsionProjectId;    // ID do projeto no Prevision
DateTime? lastPrevisionSync;   // Último sync bem-sucedido
int? previsionScheduleVersion; // Versão atual importada
```

---

### H. Cache e Performance

- Dados do Prevision são armazenados em Firestore com cache em Hive para offline
- Queries incremental limitadas a atividades modificadas (50-200 registros por sync típico)
- Curva S e CFF cacheados após cálculo (`cffCalculatedAt`, `cffVersion`) para evitar reprocessamento

---

## VIII. CONTROLE DE UNIDADES (Imobiliário)

### 8.1 Visão Geral
Módulo para rastreabilidade completa das unidades habitacionais (apartamentos, lotes, lojas) em empreendimentos imobiliários. Dados sincronizados do Salesforce Asset via Cloud Function autenticada com controle de acesso por filial.

**Hierarquia:** Empreendimento → Torre → Unidade

Leitura disponível para usuários com privilégio `units`; edição restrita a perfis `project_manager`.

---

### 8.2 Ficha da Unidade — Aba Principal
Identificação completa: número, torre, andar, tipo (kitnet, 1Q, 2Q, 3Q, cobertura, loja, etc.). Dados do proprietário: nome, CPF, telefone, e-mail, data de contrato. **Status global** com histórico de mudanças (autor + timestamp). Observações sincronizadas com registro Salesforce Asset.

---

### 8.3 Aba Personalização
Customizações solicitadas pelo cliente: pisos, revestimentos, cores, modificações de planta. Por item:
- Status: `solicitado / em execução / concluído`
- Fotos de referência (antes/depois)
- Data prevista de conclusão
- Alerta se exigir prazo especial

---

### 8.4 Aba Obra (Avanço de Obra)
Status de cada etapa de execução na unidade (alvenaria, gesso, elétrica, hidráulica, pintura, etc.)
- Percentual de conclusão por etapa
- Equipe responsável
- Fotos de evidência por etapa
- Pendências vinculadas a Planos de Ação

---

### 8.5 Aba Checklist de Obra
Lista de verificação **pré-entrega** com confirmação item a item em campo. Por item:
- Status: `pendente / conforme / não conforme / N/A`
- Foto de evidência
- Log de quem executou + timestamp
- Geração automática de relatório ao concluir

---

### 8.6 Aba Vistoria do Cliente
Condução da vistoria de **entrega** com o cliente. Inclui:
- Checklist digital preenchido durante a visita
- Registro de pendências com foto e prazo de resolução
- **Assinatura digital do cliente** capturada no app (e-assinatura)
- Gera **Laudo de Vistoria em PDF** com fotos, assinaturas e pendências
- Controle de **revistorias** com histórico completo de visitas

**Status de vistoria:** Agendada → Aguardando → 1ª Vistoria → Revisoria → Entregue

---

### 8.7 Aba Inspeção Final
Inspeção técnica **pré-entrega** por engenheiro (distinta da vistoria do cliente; foco em conformidade técnica). Inclui:
- Checklist específico de aspectos técnicos e normativos
- Registro de pontos de atenção com fotos
- Assinatura digital do engenheiro
- Gera relatório de inspeção final para arquivo técnico

---

### 8.8 Gerador de Etiquetas QR Code

**Modo Folha (2×6):**
- 12 etiquetas por folha A4 em formato para papel adesivo
- Por etiqueta: QR Code + número, torre, andar

**Modo Placa Grande (A4):**
- Uma unidade por folha com QR Code grande
- Logo do empreendimento + dados completos

Seleção flexível: unidade individual | por andar | por torre | empreendimento completo | com pré-visualização antes da geração em PDF.

---

### 8.9 Arquitetura de Cache Tri-Camada (3 Níveis)

```
Requisição de dados da Unidade
       ↓
1️⃣  Hive (Local) ————→ Retorna imediatamente (offline-first)
       ↓ (expirado ou ausente)
2️⃣  Backend Kaizen —→ Atualiza Hive e retorna
       ↓ (ausente ou refresh forçado)
3️⃣  Cloud Function / Salesforce ——→ Atualiza backend e Hive
```

Essa estratégia mantém performance offline e consistência entre camadas.

---

## IX. GESTÃO DE MATERIAIS

### 9.1 Almoxarifado Digital
Central de gestão de materiais da obra com **restrição de filial**: cada usuário vê apenas materiais de sua filial (via campo `branch` no documento).

**Seis Abas Principais (via `NavigationRail`):**

#### Requisições
Solicitações de material por usuários de campo. Fluxo de status:
- `pendente` → Aguardando aprovação
- `aprovado` → Liberado para retirada
- `retirado` → Baixa do estoque realizada

#### Pedidos
Ordens de compra (externas a fornecedores ou internas entre filiais). Rastreio:
- Data prometida vs. data recebida
- Custo unitário e total
- Número de nota fiscal (NF)
- Status: `em aberto / em andamento / recebido / vencido`

#### Saídas
Baixas físicas de estoque geradas a partir de requisições aprovadas. Rastreabilidade completa:
- Quem solicitou, quem aprovou
- Quando foi retirado
- Quantidade e destino (atividade/local)

#### Kits
Pacotes pré-montados de materiais para atividade. Características:
- Definição de itens componentes (código, quantidade)
- Uso total ou parcial (uso parcial converte em requisição)
- Reutilizáveis entre obras

#### Consulta (Estoque)
Saldo em tempo real por item. Integração com **Mega ERP**:
- Código, descrição, unidade (peça, m, m², m³, kg)
- Quantidade em mãos
- Ponto de resuprimento
- Fornecedor padrão
- Preço unitário
- Categoria

#### Auditoria
Histórico de **todas as movimentações** por tipo. Filtros:
- Por tipo: requisição, saída, pedido
- Por período (data início/fim)
- Por responsável
- Por item

**Itens de Estoque (Estrutura):**

| Campo | Tipo | Uso |
|-------|------|-----|
| `código` | String | Identificador único (ERP) |
| `descrição` | String | Nome do produto |
| `unidade` | Enum | peça, m, m², m³, kg, litro, etc. |
| `qtdEmMaos` | Number | Estoque atual |
| `pontoResuprimento` | Number | Quantidade mínima de alerta |
| `fornecedor` | String | Fornecedor padrão (FK) |
| `precoUnitario` | Number | Preço de custo |
| `categoria` | String | Grupo de material (ex.: Hidráulica, Elétrica) |

**Funcionalidades Transversais:**

1. **Etiquetas QR Code por Item**
   - Geração via `QrLabelAssistantView`
   - Código bidimensional armazena `itemId`
   - Scanning rápido para requisições/saídas

2. **Notificações Push para Aprovadores**
   - Novo pedido criado → notificação via FCM
   - Tópico: `pedidos_{clientId}_{branchId}`
   - Deep link abre diretamente o pedido

3. **Importação via CSV**
   - Upload de arquivo com itens para carga em massa
   - Validação linha a linha
   - Logging de erros

4. **Exportação Excel**
   - Relatório completo do estoque
   - Relatório de movimentações por período
   - Análise ABC (quantidade × valor)

**Chave de privilégio:** `warehouse_view`

---

### 9.2 Controle de Kits
Subconjunto da gestão de Almoxarifado dedicado a kits pré-montados. Um kit é uma **agregação de itens** para uma atividade específica:

- Definição em projeto: "Kit Hidráulico Apt. — Bloco A" = 5 registros + conexões + ferramentas
- Retirada em campo: usuário retira o kit completo ou parcialmente
- Se **uso parcial:** sistema cria requisição dos itens restantes

**Chave de privilégio:** `warehouse_view` (compartilhado com Almoxarifado)

---

### 9.3 Reserva de Ativos (Veículos e Ferramentas)
Gestão centralizada da frota da filial e ferramentas críticas com calendário de reservas e compartilhamento entre usuários.

**Objetos Gerenciados:**

#### Veículos
- Nome, tipo (carro, van, caminhão, moto), status (disponível/reservado/manutenção)
- Especificações: placa, ano, combustível, capacidade
- Custo de operação (km, diária, hourly)

#### Ferramentas
- Equipamentos: betoneira, compressor, andaime, etc.
- Status: disponível/alocado/manutenção
- Localização (qual filial)
- Custo de aluguel se terceirizada

#### Reservas
- Solicitante, período (início/fim), justificativa
- Status: `pendente / aprovado / concluído / cancelado`
- Custo associado à reserva (calculado conforme tarifa configurada)

**Funcionalidades:**

| Funcionalidade | Descrição |
|----------------|-----------|
| **Calendário Mensal** | Janela de 60 meses (30 passados + atual + 30 futuros); visualização de ocupação |
| **Rateio de Custos** | `calculateSplit()` — distribuição de custos entre múltiplas reservas no mesmo período |
| **Agregação de Custos** | Total por período, por veículo, por solicitante |
| **Notificações** | Alertas de devolução iminente; pedidos de aprovação |
| **Relatórios** | Utilização por veículo, custo acumulado, histórico de uso |

**Chave de privilégio:** `vehicle_reservation`

---

## X. MÓDULOS COMPLEMENTARES

### 10.1 Cotações (Visão Geral)
Veja [04 — Módulo de Cotações](./04_modulo_cotacoes.md) para documentação completa.

**Resumo:** 4 estágios de digitalização de processos de compra — Montagem → Cotação → Análise → Aprovado. Integração com Custo e Orçamento para registro de compromissos.

---

### 10.2 Outros Complementos
Módulos secundários mantêm as funcionalidades descritas na [ tabela consolidada abaixo](#consolidado-de-módulos).

---

## Consolidado de Módulos

| Módulo | Seção no Menu | Privilégio | Exportação | Integração |
|--------|--------------|-----------|-----------|-----------|
| **Planejamento Kaizen** | Produtividade | `daily_measurements` | — | — |
| **Medições Kaizen** | Produtividade | `daily_measurements` | — | — |
| **Kaizen Diário** | Produtividade | `daily_measurements` ou `daily_kaizen` | — | — |
| **Gestão da Estrutura** | Produtividade | `daily_measurements` | — | — |
| **Histórico de Medições** | Produtividade | `daily_measurements` | — | — |
| **Estatísticas** | Produtividade | `prod_bi` | — | — |
| **Dashboard** | Produtividade | `prod_bi` | PDF | — |
| **Atividades** | Planejamento e Avanço Físico | `baseline_view` | Excel, PDF | — |
| **Gantt de Controle** | Planejamento e Avanço Físico | `baseline_view` | — | Prevision |
| **Painel de Produção** | Planejamento e Avanço Físico | `baseline_view` | — | — |
| **Análise do Planejamento** | Planejamento e Avanço Físico | `baseline_view` | PDF | Prevision |
| **Medições Físicas** | Planejamento e Avanço Físico | `physical` | PDF | — |
| **Gestão de Cronogramas** | Planejamento e Avanço Físico | `physical` | — | Prevision, Excel, JSON |
| **Equipes** | Planejamento e Avanço Físico | `teams` | — | AT Module |
| **Planos de Ação** | Rotinas de Gestão | `action_plans_view` | PDF, Excel | KAI (sugestões) |
| **Quadro de Restrições** | Rotinas de Gestão | `mid_term` | Excel | KAI (análise 6M) |
| **Farol de Contratações** | Rotinas de Gestão | `supply_schedule` ou admin | CSV, Excel | — |
| **Estabilização Lean** | Rotinas de Gestão | `lean_inspection` | PDF | — |
| **Gestão de FVS** | Qualidade | `fvs_fill` / `fvs_approval` / `quality_manager` | PDF | — |
| **Inspeção Final de Obra** | Qualidade | `quality_manager` | PDF | — |
| **Inspeção Final da Qualidade** | Qualidade | `quality_manager` | PDF | — |
| **Custo e Orçamento** | Custo e Orçamento | `cost_control_view` | Excel, PDF | Cotações, Azure Synapse |
| **Cotações** | Custo e Orçamento | `cost_control_view` | PDF, Excel | Orçamento |
| **Almoxarifado Digital** | Gestão de Materiais | `warehouse_view` | Excel, CSV | Mega ERP |
| **Controle de Kits** | Gestão de Materiais | `warehouse_view` | — | — |
| **Controle de Unidades** | Controle de Unidades | `units` | PDF (vistoria, inspeção) | Salesforce (Assets) |
| **Personalização** | Controle de Unidades | `units` (leitura) / `project_manager` (edição) | — | — |
| **Avanço de Obra** | Controle de Unidades | `units` (leitura) / `project_manager` (edição) | — | — |
| **Vistoria do Cliente** | Controle de Unidades | `units` (leitura) / `project_manager` (edição) | PDF (Laudo) | — |
| **Reserva de Veículos** | Outros Módulos | `vehicle_reservation` | Excel | — |

---

*Documento: Módulos de Gestão de Obra | Versão 2.0 | Kaizen — Março de 2026*
