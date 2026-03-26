---
layout: page
title: "Módulos de Gestão de Obra"
permalink: /tecnico/modulos/
category: "Módulos"
order: 2
toc: true
---

# Kaizen — Módulos de Gestão O Kaizen organiza a gestão de obras em **8 blocos principais**, cobrindo o ciclo completo da construção: planejamento, controle operacional, qualidade, custo e entrega. Módulos complementares ampliam a cobertura para demandas específicas.

## Navegação Rápida

| # | Bloco | Funções Principais |
|---|-------|--------------------|
| [1](#1-produtividade--planejamento-e-medições-kaizen) | **Produtividade — Planejamento e Medições Kaizen** | Planos de produção, registro diário de avanço, índice de produtividade |
| [2](#2-planejamento-físico-financeiro) | **Planejamento Físico-Financeiro** | Macaquinho (visão matricial), Curva S, CFF, Gantt reprogramável, cronogramas, vinculação de orçamento |
| [3](#3-controle-de-produção) | **Controle de Produção** | Medições físicas, painel de produção, equipes & histograma, planos de ação |
| [4](#4-rotinas-de-gestão) | **Rotinas de Gestão** | Quadro de restrições (6M/6WLA), Dashboard de restrições (análise multi-obra), Farol de contratações |
| [5](#5-qualidade) | **Qualidade** | FVS, inspeção de obra, estabilização lean |
| [6](#6-custo-e-contratos) | **Custo e Contratos** | Controle de custo, cotações, orçamento projetado |
| [7](#7-controle-de-unidades) | **Controle de Unidades** | Vistorias, personalização, avanço físico por unidade |
| [8](#8-almoxarifado-e-ativos) | **Almoxarifado e Ativos** | Almoxarifado digital, kits, reserva de veículos/ferramentas |
| [—](#módulos-complementares) | *Módulos Complementares* | Kaizen Diário, Gestão da Estrutura, Histórico, Estatísticas, Dashboard |

---

## 1. Produtividade — Planejamento e Medições Kaizen

Núcleo de produtividade lean da obra. Permite planejar o ritmo de produção atividade a atividade e registrar o avanço diário com rastreabilidade completa de mão de obra, índices de produtividade e problemas.

**Chave de privilégio:** `daily_measurements`

---

### 1.1 Planejamento Kaizen

Módulo de planejamento por atividade da EAP. Cada atividade recebe um **tipo de plano de produção**:

| Tipo | Descrição |
|------|-----------|
| **Percentual** | Avanço definido como porcentagem do total |
| **Croqui** | Associação a uma máscara de planta segmentada para medir subáreas individualmente |
| **Ciclo** | Ciclo diário de trabalho com parâmetros de ritmo e prazo |

Em todos os tipos, o plano registra a composição de equipe planejada: quantidade de profissionais e ajudantes por turno. Um plano pode ser copiado para múltiplas atividades da mesma categoria.

---

### 1.2 Medições Kaizen

Registro diário de produção. Cada medição captura:

- **Produção realizada:** quantidade no índice do plano (m², m³, peças etc. conforme máscara de croqui ou percentual)
- **Entradas de mão de obra:** total de pessoas, profissionais, ajudantes, dias trabalhados e horas
- **Problemas registrados:** seleção em lista predefinida por serviço, com descrição de causa e empresa/empreiteira responsável
- **Metas em tempo real:** indicador de cor mostrando se a produção e as pessoas entradas atingem a meta do lote

O sistema calcula automaticamente o **Índice de Produtividade** (produção ÷ trabalho) e projeta os dias restantes estimados para conclusão do lote.

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 2. Planejamento Físico-Financeiro

Ferramentas de planejamento integradas ao cronograma da obra, conectadas diretamente à plataforma **[Prevision](05_integracoes.md#2-prevision-planejamento-de-obras)** via API GraphQL. Permitem visualizar, analisar e reprogramar o cronograma com múltiplas perspectivas de progresso físico e financeiro.

> Requer cronograma ativo na obra. Itens 2.1–2.7 exigem privilégio **`baseline_view`**; item 2.8 exige **`physical`**.

---

### 2.1 Atividades

Tabela completa de todas as atividades do cronograma ativo, implementada como `ActivityTableScreen` + `ActivityDataSource` (SfDataGrid/Syncfusion). Permite visualizar e editar o cronograma em formato de planilha.

**Colunas configuráveis** (seletor de colunas na AppBar):

| Coluna | Descrição |
|--------|----------|
| Serviço | Nome do serviço / pacote de trabalho |
| Lote | Lote ou pavimento/setor da obra |
| Início / Término | Datas planejadas (editáveis inline) |
| Duração | Calculada em dias úteis |
| Custo | Custo orçado; editável por admins (validado inline) |
| Peso | Participação percentual no custo total |
| % Base | Progresso segundo a baseline de referência na data ativa |
| % Previsto | Progresso previsto pelo cronograma atual |
| % Realizado | Progresso real registrado em medições |
| Crítica | Indicador de pertencimento ao caminho crítico |
| Fornecedor | Empresa contratada, carregada sob demanda da integração Prevision |

**Operações disponíveis:**

- Edição inline de datas e custos com validação de acesso (somente admins editam custo)
- Busca de atividade por nome com debounce de 500 ms
- Filtros rápidos: ocultar concluídas, somente mês corrente, somente semana corrente, somente baseline > realizado, somente previsto > realizado
- **Cálculo do caminho crítico** sob demanda (destaca em coluna "Crítica" as atividades que definem o prazo mínimo do projeto)
- Pilha de **desfazer** (até 15 passos via `undoStack`) — restaura snapshot completo de todas as atividades
- **Salvamento em nuvem** (botão laranja quando há alterações pendentes): atualiza custos, datas, serviços, lotes, feriados e atividades no Firestore
- **Salvar Como**: cria cópia do cronograma com novo nome
- Exportação em Excel (.xlsx) e PDF via biblioteca `printing`

**Cabeçalhos agrupados (Stacked Headers):** colunas ativas são agrupadas em faixas visuais por categoria (Identificação / Custo / Baseline / Previsto / Realizado / Qualidade).

**Chave de privilégio:** `baseline_view`

---

### 2.2 Macaquinho — Visão Matricial

O **Macaquinho** é a visão matricial do cronograma da obra: um grid colorido onde cada célula representa a interseção de um **lote (linha)** com um **serviço (coluna)**. Cada célula exibe um tile de atividade com indicadores visuais de progresso, permitindo identificar de imediato onde a obra está adiantada, em dia ou atrasada.

> Acionado pela rota que passa `mode: 'monkey'` ao `BaselineController`. Ao entrar no modo macaquinho, o controller ordena atividades por data de início decrescente e carrega automaticamente o mapa de inspeções.

#### Tiles de atividade (`MonkeyTile`)

Cada célula do grid renderiza um `MonkeyTile` com as seguintes informações:

```
┌──────────────────────────────────┐
│  Serviço — Lote                  │  ← nome (truncado)
│                                  │
│  Base:    ##.#%  (laranja)       │
│  Previsto: ##.#% (preto)         │
│  Realizado: ##.#% (azul)        │
│                                  │
│  [barra de progresso colorida]   │
│  Início → Término                │
└──────────────────────────────────┘
```

**Cor de fundo do tile:**

| Cor | Condição |
|-----|---------|
| Verde | `realizado ≥ 95 %` do previsto |
| Azul claro | `85 % ≤ realizado < 95 %` do previsto |
| Amarelo | `70 % ≤ realizado < 85 %` do previsto |
| Vermelho | `realizado < 70 %` do previsto OU atividade atrasada |
| Roxo/outline | Atividade pertencente ao caminho crítico (`isCritical`) |
| Cinza | Atividade sem dados de progresso |

#### Modos de Heatmap

O tile pode ser sobreposto por um heatmap configurável, útil em análises de concentração de trabalho:

| Modo | Critério de cor |
|------|---------------|
| **Mensal** | Hue ISO-mês com stride-7 (cada mês tem cor bem distinta) |
| **Semanal** | Hue gradiente 0°→300° distribuído pelas 52 semanas do ano |
| **Proximidade** | Quente (vermelho) = próxima da `activeDate`; fria (azul) = distante |

#### Controles e filtros

| Controle | Descrição |
|----------|-----------|
| **Seletor de data** (`activeDate`) | Recalcula `expected` de todas as atividades; as cores dos tiles são atualizadas sem recarregar do servidor |
| **Zoom de célula** (`monkeyCellSize`) | Slider de 20 px a 240 px; escala fontes e barras proporcionalmente |
| **Filtro por lote** | Seleção multi-select com agrupamento por tags favoritas; ao entrar, pré-seleciona o primeiro grupo favorito |
| **Filtro por serviço** | Busca textual debounced; exibe indicador de quantos serviços selecionados |
| **Ocultar finalizados** | Remove serviços onde todos os lotes estão a 100 % |
| **Somente caminho crítico** | Restringe grid às atividades com `isCritical = true` |
| **Tags de serviço, fornecedor, status de inspeção** | Filtros adicionais |

#### Ações disponíveis por célula (menu de contexto)

Ao pressionar/clicar com botão direito sobre um tile, o usuário pode:

- **Abrir/criar Restrição** — abre `restrictionDialog` (integrado ao `MidTermController`), vinculando a restrição ao ID da atividade
- **Listar Restrições vinculadas** — bottom sheet com todas as restrições ligadas àquela atividade
- **Abrir/criar Plano de Ação** — abre `showActionPlanDialog` (integrado ao `ActionPlanController`)
- **Listar Planos de Ação vinculados** — bottom sheet com planos
- **Status de Inspeção** (`_showInspectionStatusMenuAtPosition`) — altera o status da inspeção da atividade (integrado ao módulo FVS)

#### KPIs e estatísticas

Acima do grid e em painel colapsável (mobile), são exibidos:

- **KPI global:** % realizado × previsto × baseline ponderados por custo
- **KPI por serviço:** tile de estatísticas ao lado do cabeçalho de cada serviço, mostrando médias de B/P/R para aquele serviço em todos os lotes visíveis
- **Painel de stats mobile:** ícone na AppBar expande bottom sheet com KPIs detalhados

#### Exportação

| Formato | Função | Conteúdo |
|---------|--------|----------|
| **PDF** | `printMonkey()` | Grid completo renderizado campo a campo com cores replicando os tiles |
| **Excel** | `exportMonkeyToExcel()` | Matriz lote×serviço com valores B/P/R e fundo colorido por status |

**Chave de privilégio:** `baseline_view`

---

### 2.3 Gantt Reprogramável

Visualização Gantt interativa do cronograma ativo, implementada como `BaselineGantt` + `GanttChart` (`CustomPainter`). As barras do Gantt são renderizadas diretamente em canvas para máxima performance, sobrepondo vários cronogramas.

#### Renderização e navegação

- `GanttGridPainter` desenha a grade de datas e os rótulos do eixo X; `GanttActivityPainter` pinta as barras de cada atividade e as linhas de dependência
- Eixo X scrollável horizontalmente; eixo Y: atividades filtradas
- **Scroll para hoje:** botão na AppBar centraliza a vista na data corrente
- **Scroll para término previsto:** toque duplo no campo "Término previsto" na AppBar rola até a última barra
- **Data de término previsto** exibida na AppBar em tempo real; atualizada quando barras são arrastadas

#### Reprogramação por drag & drop

```
[usuário arrasta barra da atividade A]
  → tempDragStartAt / tempDragEndAt atualizados em tempo real no canvas
  → dragOffsetX armazena deslocamento em pixels
  → ao soltar: duração original mantida; datas deslocadas
  → saveUndoSnapshot() salva estado anterior na pilha de desfazer
  → applyFilters() + refreshChart() atualizam a view
```

- Duração é preservada no arrasto (move sem esticar)
- Dependências entre atividades são verificadas: o sistema alerta se a nova data viola uma relação TI/II/TT/IT com atraso configurável
- O cálculo usa `addWorkingDays()` e `nextWorkingDay()` respeitando feriados da obra e dias úteis configurados

#### Pilha de desfazer / salvar

| Controle | Detalhe |
|----------|---------|
| **Desfazer** (ícone laranja) | Restaura snapshot completo de todas as atividades; aparece somente quando há alterações pendentes |
| **Salvar em nuvem** (ícone laranja) | `saveScheduleToFirebase()` — persiste custos, datas, serviços, lotes, feriados e a lista de atividades no Firestore |
| **Salvar Como** | Cria cópia do cronograma com novo nome via `saveScheduleAs()` |
| **Travar edições** | Ícone de cadeado (roxo = travado); bloqueia qualquer drag de barra |
| Histórico (15 passos) | `maxUndoSteps = 15`; o snapshot mais antigo é descartado automaticamente ao exceder |

#### Caminho crítico

- **Cronogramas do Prevision:** o campo `isCritical` já vem calculado pela API — barras críticas exibidas em vermelho sem recalcular localmente
- **Cronogramas locais:** botão de triângulo de aviso dispara `calculateCriticalPath()` (algoritmo de CPM); barras críticas destacadas em vermelho após o cálculo
- Toggle na AppBar mostra/oculta o overlay de caminho crítico

#### Filtros disponíveis

| Filtro | Detalhe |
|--------|---------|
| **Por serviço** | Multi-select com paginação (`activeServiceIndex`): navega um serviço por vez com setas ← → |
| **Por lote** | Seletor com agrupamento por tags; suporte a Tags Favoritas para acesso rápido |
| **Por intervalo de datas** | `DateRangePicker`; restringe o canvas ao período selecionado |
| **Ocultar concluídas** | Remove atividades a 100 % |
| **Somente caminho crítico** | Filtra apenas atividades com `isCritical = true` |
| **Tags favoritas de lotes** | Dropdown na AppBar com grupos pré-salvos; muda lotes + serviços em um clique |

#### Seletor de colunas

`ColumnSelectorDialog` permite escolher quais colunas da **tabela de atividades** (§ 2.1) são exibidas. Compartilha o estado `visibleColumns` com a view de atividades.

**Chave de privilégio:** `baseline_view`

---

### 2.4 Curva S — Análise do Planejamento

Módulo de análise do planejamento (`ScurveView` + `ScurveController`), organizado em **quatro abas** acessíveis pela barra de ícones:

| # | Aba | Widget | Descrição |
|---|-----|--------|-----------|
| 0 | Curva S Acumulada | `SCurveChart` | Linhas acumuladas B/P/R ao longo do tempo |
| 1 | Gráfico Mensal | `MonthlyDeltaChart` | Barras do incremento mensal com drill-down por serviço |
| 2 | Análise de Serviços | `ServiceMonthlyGrid` | Grade de % por serviço e mês |
| 3 | CFF | `CFFGrid` | Cronograma Físico-Financeiro (ver § 2.5) |

#### Inicialização e cálculo da curva

Ao entrar na tela, o `ScurveController.onInit()` verifica se o baseline já está carregado. Se não estiver, chama:

```
previsionController.activateBaselineSchedule(siteId)
  → schedule.calculateSCurve(baselineSchedule)
  → previsionController.applySCurveScheduledFlagAdjustment(schedule)
```

O resultado é armazenado em `schedule.daySCurve` — um `Map<DateTime, Map<String,dynamic>>` com entradas diárias contendo `{'s': ..., 'b': ..., 'd': ...}` (0.0 a 1.0, onde 1.0 = 100%).

#### Aba 0 — Curva S Acumulada (`SCurveChart`)

Gráfico de linhas com três séries simultâneas plotadas por `SfCartesianChart`:

| Série | React. | Cor |
|-------|-------|-----|
| **Base** | Baseline imutável de referência | Laranja escuro |
| **Previsto** | Cronograma atual reprogramável | Preto |
| **Realizado** | Progresso real acumulado (pode ter lacunas — série `nullable`) | Azul |

- Eixo X: `DateTimeAxis` cobrindo todo o ciclo da obra
- Eixo Y: `NumericAxis` 0–100 % com rótulo `{value}%`
- **Trackball** em modo `groupAllPoints`: toque em qualquer ponto exibe tooltip com os três valores simultâneos
- **Zoom/pan** horizontal (`ZoomMode.x`); botão de reset de zoom na barra roxa acima do gráfico
- **Exportar CSV**: ícone de download na barra roxa exporta `schedule.exportSCurveAsCSV()`

#### Aba 1 — Gráfico Mensal de Deltas (`MonthlyDeltaChart`)

Gráfico de barras agrupadas mostrando o incremento de % realizado por mês:

- Três barras por mês: Base / Previsto / Realizado
- **Drill-down**: ao tocar em uma barra, exibe gráfico de colunas com a contribuição de cada **serviço** para aquele mês (Series B/P/R por serviço, ordenado por previsto decrescente)
- O drill-down leva em conta `daySCurveActivities` (mapa por atividade), aplicando os pesos de custo (`budgetCost / totalBudget`) para converter percentuais unitários em contribuição ponderada
- `areLabelsVisible` toggle: exibe/oculta rótulos sobre as barras
- Zoom/pan horizontal compartilhado

#### Aba 2 — Análise de Serviços (`ServiceMonthlyGrid`)

Grade tabular mostrando o progresso mensal por serviço. Permite identificar quais serviços estão concentrados em quais períodos e detectar desvios por pacote de trabalho. Recalculada automaticamente quando `isTableRecalcualting` é `false`.

#### Aba 3 — CFF

Ver § 2.5.

#### Cache do CFF no Firestore

```
Path: {client}/default_sites/{siteId}/budgets/{budgetId}/cff/
```

- `calculateAndSaveCff()` executa `calculateCFFAsync()` em async, persiste em chunks de 150 itens (`_cffChunkSize`) no Firestore, e atualiza `cffVersion` para forçar rebuild do `CFFGrid`
- `cffRefDate`: data de corte usada no cálculo; exibida no tooltip do botão Recalcular CFF
- Na inicialização, `_loadCffCache()` restaura o cache salvo; se disponível, `cffVersion > 0` e a aba CFF exibe imediatamente sem recalcular

**Chave de privilégio:** `baseline_view`

---

### 2.5 Cronograma Físico-Financeiro (CFF)

O CFF é um relatório tabular que cruza **itens de orçamento (WBS)** com **períodos de execução**, mostrando a distribuição temporal dos custos planejados, esperados e realizados.

**Estrutura do CFF:**

```
Datas →   jan/26   fev/26   mar/26   ...   total
Item WBS ↓
01 Fundações    R$ ...   R$ ...   R$ ...    R$ ...
02 Estrutura    R$ ...   ...                R$ ...
```

Cada linha contém o item de orçamento (código, descrição, custo de material, custo de mão de obra, custo total) e colunas de progresso por período com três perspectivas: `basePoints` (baseline), `expectedPoints` (previsto) e `realizedPoints` (realizado). A diferença entre as colunas permite identificar desvios de cronograma e orçamento por rubrica.

**Dados importados do Prevision:** `budgetReportsPage`, `budgetReport(id).cffTable`.

---

### 2.6 Vinculação de Orçamento — Budget Weights

Liga cada item do orçamento WBS a uma ou mais atividades do cronograma com percentuais de peso. Permite calcular qual percentual do custo de um item deve ser alocado a cada atividade vinculada, tornando possível projetar o progresso físico-financeiro esperado vs. realizado por rubrica.

**Exemplo:**

```
Item WBS "01.03 Alvenaria"  (R$ 300 k)
      ↓ BudgetWeights
  Atividade A  →  40 %  (R$ 120 k)
  Atividade B  →  60 %  (R$ 180 k)
```

**Dados importados do Prevision:** `budgetItemsPage` com `budgetWeights` contendo `percentage + activity.id`.

---

### 2.7 Relatórios Operacionais

Exportações e relatórios gerados a partir do cronograma ativo:

| Relatório | Formato | Conteúdo |
|-----------|---------|----------|
| Atividades completas | Excel / PDF | Tabela de atividades com datas, custos e % |
| Curva S do projeto | Gráfico PDF | Séries S/B/D acumuladas |
| CFF por período | Excel / PDF | Cronograma físico-financeiro com colunas mensais |
| Análise de desvios | Excel | Desvios por rubrica categorizados (ORC, CST, PRJ, ENG, PRZ) |

---

### 2.8 Gestão de Cronogramas

Gerenciamento do ciclo de vida dos cronogramas da obra: criação de novas versões, definição do baseline de referência, comparação entre versões e histórico completo de revisões com autor e timestamp.

**Fontes de importação suportadas:**

| Fonte | Formato | Observação |
|---|---|---|
| **[Prevision](05_integracoes.md#2-prevision-planejamento-de-obras)** | API GraphQL | Fonte primária; sincroniza automaticamente pelo `previsionProjectId` da obra |
| **Excel** | `.xlsx` | Arquivo enviado à Cloud Function `excelToJson`; usuário mapeia as colunas manualmente (ID, serviço, lote, início, fim, custo, % concluído) |
| **JSON** | `.json` | Importa arquivo estruturado no formato interno do Kaizen (ex.: exportação de outra obra) |

**Chave de privilégio:** `physical`

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 3. Controle de Produção

Ferramentas de acompanhamento e controle da execução física da obra, com foco em medições contratuais, visualização matricial de progresso, gestão de equipes e registro de ações corretivas.

---

### 3.1 Medições Físicas

O módulo de **Medições Físicas** é o núcleo do controle de produtividade da obra. Cada **medição** é um snapshot do avanço físico real das atividades em relação ao cronograma Prevision em uma data de referência específica — captada diretamente pelo engenheiro ou estagi&aacute;rio em campo.

#### Conceito: o que é uma medição

Uma medição registra, para cada atividade do cronograma ativo, **quanto foi executado** (valor realizado) e **quanto era esperado** (valor previsto pela baseline na data de referência). A partir dessa comparação, o sistema calcula automaticamente os indicadores de desempenho da obra.

```
Baseline (Prevision)  →  expectedAtDate(refDate)  →  expected por atividade
Cronograma Atual      →  globalDone por atividade  →  measured por atividade
                                  ↓
           PPC = Σ(realizado) / Σ(esperado)   (Percent Plan Complete)
           PPCC = PPC acumulado histórico
           flagColor: 🔴 < 95% da meta | 🟡 próximo | 🟢 dentro da meta
```

#### Modelo de dados — `PhysicalMeasurement`

| Campo | Descrição |
|-------|-----------|
| `measurement` | Mapa `{actividadeId → valorRealizado}` — o dado central da medição |
| `expected` | Mapa `{actividadeId → valorEsperado}` — previsto pela baseline na data |
| `refDate` | Data de referência da medição |
| `scheduleId` | Cronograma Prevision usado como baseline |
| `comments` | Mapa `{actividadeId → comentário}` — observações por atividade |
| `measurementProblems` | Mapa `{actividadeId → [problemas]}` — problemas detectados (manual ou por IA) |
| `criticalActivities` | Lista de IDs das atividades do caminho crítico nesta data |
| `lstSCurve` | Série histórica para composição da Curva S |
| `generalComments` | Observações gerais da medição (visíveis no cabeçalho da análise) |
| `generalCommentsLastEditedBy/At` | Auditoria da última edição das obs. gerais |
| `ppc` | PPC calculado: `Σrealizado / Σesperado` |
| `ppcc` | PPC acumulado (soma histórica) |
| `expectedPercentage` / `donePercentage` | % global acumulado previsto e realizado |
| `mExpectedPercentage` / `mDonePercentage` | % do período (delta desde a medição anterior) |
| `isPreview` | Indica que é uma medição de prévia (não afeta histórico principal) |
| `media` | Metadados de mídia associados |

#### Histórico de Medições (Lista)

A tela principal mostra a lista cronológica de todas as medições com um **tile** por medição exibindo:

- **Acumulado:** % previsto vs. % realizado (desde o início da obra)
- **Do período:** delta da medição atual menos a anterior — mostra o avanço do mês
- **Diferença:** desvio acumulado `realizado − previsto`
- **PPC:** Percent Plan Complete da medição
- **Flag de desempenho:**

| Cor | Critério |
|-----|---------|
| 🔴 Vermelho | Realizado do período < 95% do previsto do período |
| 🟡 Amarelo | Realizado do período ≈ previsto (margem apertada) |
| 🟢 Verde | Realizado do período ≥ previsto do período |

**Tooltip** ao passar o cursor exibe: todos os percentuais (acumulado e período), diferença e PPC com o nome do responsável.

**Criação:** botão `+` na AppBar abre o seletor de data → compara a baseline com o cronograma atual para construir o mapa `expected` e inicia a sessão de edição.

**Importação via Prevision:** medições registradas no Prevision podem ser importadas diretamente via `measurementsTasksPage` + `measurementTask(id)`, trazendo os dados de avanço por atividade já calculados na plataforma de planejamento.

#### Tela de Análise — abas

Ao abrir uma medição (criar ou revisar), a tela de análise apresenta 4 abas:

| Aba | Conteúdo |
|-----|----------|
| **Tabela de Medição** | Grid de atividades por lote/serviço com entrada de valores, comentários e flag de desempenho por linha (ver detalhes abaixo) |
| **Análise de Problemas** | Dashboard dos problemas registrados por categoria, atividade e frequência; gráficos de distribuição |
| **Planos de Ação** | Planos de ação vinculados a esta medição — permite criar, vincular ou visualizar diretamente |
| **KAI** | Análise automatizada por IA: resumo do desempenho, atividades críticas em risco, sugestões de ação priorizadas |

No **topo da tela**, o campo de **observações gerais** é editável por admins e pelo criador da medição — serve para registrar o contexto da semana (contextos externos, decisões tomadas, etc.).

#### Grid de Atividades (Tabela de Medição)

A tabela central usa o `SfDataGrid` (Syncfusion) com as seguintes capacidades:

- **Colunas mensais** geradas dinamicamente conforme o range de datas do cronograma
- **Ordenação** por qualquer coluna
- Por linha (atividade), o usuário informa:
  - % realizado ou quantidade executada
  - Comentário / observação técnica sobre a atividade
  - Problemas vinculados (seleção de causa raiz por categoria default)
- **Detecção automática de problemas por IA:** ao salvar um comentário, o sistema envia o texto para análise e identifica a categoria de problema automaticamente, reduzindo o esforço de classificação manual
- **Planos de ação por atividade:** botão em cada linha abre o histórico de planos vinculados àquela atividade, com opção de criar novo ou vincular uno existente

#### Edição Colaborativa — `_dirtyActivityIds`

O controller mantém um conjunto de **IDs de atividades modificadas nesta sessão** (`_dirtyActivityIds`). Ao salvar, apenas esses campos são escritos no Firestore via **dot-notation** (`measurement.{actividadeId}`):

```
Engenheiro A mede Torre 1 → escreve measurement.act_001, measurement.act_002
Engenheiro B mede Torre 2  → escreve measurement.act_105, measurement.act_106
(simultâneos, sem sobrescrever os dados um do outro)
```

Esse mecanismo garante que **múltiplos engenheiros medindo simultaneamente em torres ou lotes diferentes não se sobreponham**, mesmo em cenários de reconexão offline → online.

#### Suporte Offline

O `PhysicalController` monitora conectividade em tempo real (`Connectivity`). Quando offline:
- A edição local continua normalmente sobre os dados em memória
- Ao reconectar, o dirty set é flushado para o Firestore com apenas as atividades alteradas
- O indicador `isConnected` na view alerta o usuário sobre o estado da conexão

#### Exportação

| Formato | Conteúdo |
|---------|----------|
| PDF | Tabela completa da medição com formatação, flag de desempenho e resumo de indicadores; pode ser enviado por e-mail diretamente da tela |
| Excel | Export via Syncfusion do grid completo |
| E-mail | Fluxo integrado: gera PDF → solicita lista de destinatários → envia via SendGrid |

**Chave de privilégio:** `physical` (edição) · `physical_edit` (edição nas últimas 5 medições) · admin (acesso irrestrito)

---

### 3.2 Painel de Produção — Monkey Chart

O **Painel de Produção** é a visão matricial macro do andamento da obra. Exibe o status de cada célula da EAP no formato **lote × serviço**, permitindo ao gestor identificar de relance quais frentes estão atrasadas, dentro do prazo ou concluídas.

#### Formato da grade

Cada célula da matriz representa **uma atividade** (cruzamento de um lote×pavimento com um tipo de serviço):

| Cor da célula | Significado |
|---------------|-------------|
| 🟢 Verde | Atividade concluída (100% realizado) |
| 🟡 Amarelo | Em progresso, dentro do prazo planejado |
| 🔵 Azul | Planejada — ainda não iniciada mas no prazo |
| 🔴 Vermelho | Atrasada — data prevista ultrapassada sem conclusão |

O índice de produtividade calculado para cada célula pode ser exibido diretamente sobre a cor, tornando possível identificar não apenas **se está atrasado** mas **o quanto a equipe está produzindo** em relação à meta.

#### Interação

- **Clique na célula:** abre os detalhes da atividade com dados de progresso, comentários, planos de ação e histórico de medições
- **Zoom (`monkeyTileScale`):** controle deslizante para ajustar o tamanho das células — útil em obras com muitos lotes ou serviços para visão panorâmica
- **Painel de estatísticas laterais:** colapsável; exibe contagens por status, distribuição de atraso e indicadores de produtividade agregados

#### Uso em reuniões de gestão

A grade é especialmente usada em **reuniões de produção semanais** como visual de controle: a equipe consegue identificar rapidamente quais lotes/serviços precisam de atenção e quais planos de ação estão ativos por frontem — sem precisar abrir cada atividade individualmente.

**Chave de privilégio:** `baseline_view`

---

### 3.3 Gestão de Equipes e Histograma

Cadastro e gestão das equipes de produção:

- Associação de colaboradores a equipes (compartilhado com o Módulo AT)
- Custo/hora por equipe para cálculo de mão de obra planejada vs. realizada
- Histórico de desempenho por equipe
- **Histograma de recursos:** visualização da alocação de equipes ao longo do tempo sobre o cronograma — identifica picos de demanda e períodos ociosos, apoiando o nivelamento de recursos

**Chave de privilégio:** `teams`

---

### 3.4 Planos de Ação

Gestão de itens de ação com três modos de visualização:

| Modo | Descrição |
|------|-----------|
| **Kanban** | Cartões agrupáveis por responsável, status, setor, prioridade, risco ou semana de vencimento |
| **Lista** | Visualização tabular com ordenação por 6+ critérios |
| **Dashboard** | KPIs de abertura × conclusão × vencidos |

**Campos principais:** título, descrição, prazo, status (`aberta / em andamento / concluída / cancelada / vencida`), prioridade (`baixa / média / alta`), risco (`baixo / médio / alto`), responsável, setor, áreas de impacto (multi-select), vínculos a atividades do cronograma e tags de serviço.

**Funcionalidades adicionais:**
- Notificações push automáticas ao responsável via FCM na criação e no vencimento
- Exportação em PDF e Excel
- Comentários internos por item com histórico completo
- Tags e cores de serviço configuráveis por tenant
- **IA — KAI:** Agente `kai-action-plan-analyst` disponível para geração automática de planos e sugestões

**Chave de privilégio:** `action_plans_view` (ou usuário super)

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 4. Rotinas de Gestão

Ferramentas de gestão semanal e mensal baseadas na **Metodologia Last Planner System (LPS)**, com suporte à análise de restrições pelo modelo **6M** e ao **Lookahead de 6 Semanas (6WLA)**. Requer cronograma ativo na obra.

---

### 4.1 Quadro de Restrições — 6WLA / Metodologia 6M

O Quadro de Restrições implementa o **6-Week Lookahead (6WLA)**: uma janela prospectiva configurável (padrão 8 semanas) para identificar antecipadamente os impedimentos que bloquearão a execução das atividades planejadas — antes que se tornem problemas reais.

Cada restrição é classificada pela **Metodologia 6M**, que categoriza as causas raiz de impedimentos de obra:

| Categoria 6M | Descrição | Exemplos de Restrições |
|---|---|---|
| **Mão de obra** | Disponibilidade e qualificação de equipes | Falta de técnico especializado, afastamentos |
| **Materiais** | Fornecimento e logística de insumos | Material não entregue, especificação pendente |
| **Máquinas** | Equipamentos e ferramentas | Grua indisponível, compressor em manutenção |
| **Métodos** | Processos, projetos e aprovações | Projeto não liberado, método construtivo indefinido |
| **Meio Ambiente** | Condições externas e climáticas | Chuvas intensas, acesso ao canteiro bloqueado |
| **Medição** | Controle e verificação de requisitos | Inspeção pendente, laudo atrasado |

**Ciclo de vida de uma restrição:**

```
Identificada → Em Andamento → Removida
                    ↓
               Vencida (prazo ultrapassado sem remoção)
```

**Campos por restrição:** serviço afetado, descrição do bloqueador, data de comprometimento para resolução, status, responsável, setor, categoria 6M e anexos.

**Visões disponíveis:**
- Grade semanal (serviço × semana) com lookahead configurável
- Painel de serviços com filtro de restrições ativas
- Painel de estatísticas: contagens por status e tendências

**Ordenação:** 8+ opções, incluindo data de comprometimento e data de início da próxima atividade vinculada.

**Exportação:** Excel multi-obra consolidado (`MidTermExcel`)

**Notificações:** Alertas automáticos de vencimento; deep link via FCM abre diretamente a restrição no app

**IA — KAI:** Agentes `kai-restriction-analyst` (relatório analítico 6M) e `kai-suggestion` (sugestões automáticas de novas restrições) disponíveis nesta tela

**Chave de privilégio:** `mid_term` (requer cronograma ativo)

---

### 4.2 Farol de Contratações

O **Farol de Contratações** é o módulo de gestão do cronograma de suprimentos. Seu objetivo central é garantir que nenhuma atividade do cronograma de obra seja paralisada por falta de material ou serviço contratado a tempo. O módulo usa um semáforo visual (🟢🟡🔴) para sinalizar o risco de cada processo de compra em relação às datas do cronograma físico.

#### Conceito: vinculação ao cronograma físico

Cada **item de suprimento** (material, serviço ou equipamento) é associado às **atividades do Prevision** que dele dependem. A partir da data de início da atividade mais urgente, o sistema calcula retroativamente as datas-limite de cada etapa do processo de compra, somando os prazos (SLAs) cadastrados.

```
Data de início da atividade (Prevision)
         ↑
   – margem de segurança
         ↑
   – SLA etapa N  (ex: entrega do fornecedor: 10 dias úteis)
         ↑
   – SLA etapa N-1 (ex: aprovação do pedido: 2 dias úteis)
         ↑
   – SLA etapa 1  (ex: solicitação de cotação: 1 dia útil)
         ↑
   ← DATA EM QUE O PROCESSO DEVE SER INICIADO
```

Se hoje ultrapassar a data de início calculada sem que o processo tenha sido aberto, o farol acende **vermelho**. Se estiver próximo (dentro da margem de alerta), acende **amarelo**. Caso haja tempo suficiente, permanece **verde**.

#### Entidades principais

| Entidade | Descrição |
|----------|-----------|
| **Item de suprimento** (`SupplyItem`) | O que precisa ser contratado: nome, fornecedor padrão, categoria, lista de SLAs por etapa, margem de segurança, responsável de compras (`buyerId`/`buyerName`) |
| **Processo de contratação** (`SupplyProcess`) | Instância real de compra de um item para uma obra específica: status, responsável, lista de atividades vinculadas, datas calculadas por SLA, observações |
| **SLA de etapa** | Prazo (em dias úteis) de cada passo do processo: ex. `Solicitar cotação`, `Receber propostas`, `Análise técnica`, `Aprovação`, `Emissão de pedido`, `Entrega`. Configurável por item ou por padrão global |
| **Feriados customizados** | Cada obra pode ter feriados locais que são considerados no cálculo de dias úteis |

#### Lógica do semáforo

| Cor | Condição |
|-----|----------|
| 🟢 Verde | Processo iniciado e com folga, ou prazo de início futuro com margem confortável |
| 🟡 Amarelo | Prazo de início iminente (dentro da margem de alerta) ou processo em andamento com SLA próximo do limite |
| 🔴 Vermelho | Data de início calculada já passou sem processo aberto, ou SLA de etapa vencido |
| ⚪ Cinza | Item sem atividade vinculada ou sem cronograma ativo |

#### Telas e abas

| Aba | Conteúdo |
|-----|----------|
| **Acompanhamento** | Tiles compactos por item e por obra, com semáforo e resumo do status de cada processo. Filtros por setor ativo, responsável, item e obra |
| **Cronograma Visual** | Grade de Gantt por item: barras representando janelas de cada SLA ao longo do tempo. Permite visualizar sobreposição de prazos e gargalos de compra |
| **Matriz SLA** | Tabela item × etapa de SLA com datas e status coloridos; útil para reuniões de suprimentos |
| **Grid de Processos** | Listagem tabular completa de todos os processos com colunas de status, responsável e datas; suporta ordenação multicritério |

#### Criação e gestão de processos

- Ao criar um processo, o comprador seleciona as **atividades do Prevision** que dependem do item; o sistema calcula automaticamente as datas-chave pela cadeia de SLAs
- Possibilidade de adicionar **atividades extras** fora do cronograma Prevision
- O responsável de compras é selecionado a partir do **crew da obra** (time cadastrado na obra), evitando duplicidade de cadastros
- É possível **reordenar as etapas de SLA** via drag-and-drop, e o cálculo recalcula instantaneamente
- Campo de observações por processo para registro de ocorrências, justificativas e notas de negociação

#### Alertas automáticos por e-mail

Ao criar um processo, o usuário pode configurar um **alerta automático** que avisa por e-mail quando a data de início de contratação estiver se aproximando:

- Data-alvo: primeira SLA calculada (quando o processo deve ser iniciado)
- Antecedência configurável (ex.: alertar 7 dias antes)
- Lista de destinatários por processo
- Executado pela **Cloud Function `supplyAlerts`** (agendada, verifica todos os processos com SLA em risco e dispara notificações)

#### Configurações e administração

- **SLAs padrão globais:** configura as etapas e prazos usados como base ao criar novos itens, podendo ser sobrescritas por item
- **Exceções:** regras que ajustam prazos para categorias específicas de insumo ou fornecedor (ex.: materiais importados com lead time maior)
- **Migração de processos:** permite mover um processo de uma obra para outra em caso de remanejamento de equipe ou material
- **Marcação de datas:** dialog de registro de datas reais de cada etapa, criando histórico de performance de fornecedores

#### Exportações

| Formato | Conteúdo |
|---------|----------|
| CSV (`exportDatabaseCSV`) | Dump completo de itens e processos, útil para integração com sistemas externos |
| Excel multi-obra (`SupplyScheduleExcel`) | Planilha consolidada com todos os processos agrupados por obra — para envio à equipe de suprimentos |
| PDF | Lista de processos formatada com status e responsáveis |
| CSV de processos | Exportação filtrada dos processos visíveis na tela |

**Chave de privilégio:** `supply_schedule` ou perfil admin

---

### 4.3 Dashboard de Restrições

O **Dashboard de Restrições** é o painel centralizado de monitoramento e análise do **6-Week Lookahead (6WLA)** — o quadro de impedimentos que bloqueiam a execução das atividades planejadas. Complementa a tela de edição do Quadro de Restrições (4.1) fornecendo visualizações analíticas em tempo real: estatísticas por setor responsável, análise de responsáveis externos (fornecedores), distribuição por status e nível de escalonamento, além de métricas consolidadas por obra.

O dashboard é organizado em **3 abas principais**: **Visão Geral** (Overview), **Listagem** (List) e **Kanban** (visual drag-and-drop), cada uma com sua própria lógica de filtragem e apresentação.

#### 4.3.1 Barra de Filtros Globais

Todos as três abas do dashboard compartilham uma **barra de filtros global** posicionada no topo:

| Filtro | Tipo | Função |
|--------|------|--------|
| **Setor Responsável** | Dropdown | Filtra restrições por setor (Obra, Fornecedor, Projetos, Recursos Humanos, etc.) |
| **Obra** | Dropdown | Seleciona a obra ativa para análise (multi-obra) |
| **Janela de Conclusão** | Selector | Define a janela de tempo para considerar restrições concluídas: 7, 14, 30, 60 ou 90 dias (padrão: 30 dias) |
| **Nível de Escalonamento** | Chips (N1–N4) | Filtra por nível de escalação. N1 excludo por padrão (custo otimizado); lock em N1 exige confirmação para incluir |

**Comportamento:**
- Os filtros aplicam-se **globalmente** a todas as abas (List, Kanban)
- A **Visão Geral (Overview)** tem filtro local adicional de **Status** (ver seção 4.3.2)
- Mudanças de filtro refletem imediatamente em todos os cards, gráficos e contadores
- Restrições **arquivadas** (`isArchived: true`) são **sempre excluídas** de toda análise, independentemente de filtros

**Privilege check:** O dropdown de obras exibe apenas obras acessíveis ao usuário conforme `BranchAccessService` (admin vê todas; usuários normais veem apenas sua obra)

#### 4.3.2 Visão Geral (Overview Tab)

A aba **Overview** exibe uma análise consolidada de restrições com painéis KPI e gráficos de distribuição. Inclui um **filtro local de status** (separado do controller global) para exploração granular.

##### Filtro Local de Status

Na parte superior da Overview aparecem 5 chips interativos (Material Design `FilterChip`):

| Chip | Restrições Exibidas |
|------|-------------------|
| **Todos os Status** (padrão) | Todas sem restrição de status |
| **Planejada** | Status = "Planejada" (não iniciada) |
| **Em Execução** | Status = "Em Execução" (ativa) |
| **Atrasada** | Status = "Atrasada" (vencida) |
| **Concluída** | Status = "Concluída" (finalizada) |

- Seleção exclusiva (um chip ativo por vez)
- Cores codificadas de acordo com `lstRestrictionStatus` global
- Aplicado **apenas ao conteúdo desta aba** (não afeta List ou Kanban)

##### Seção de KPIs — Resumo Geral

Card supremo com 4 estatísticas em tempo real (atualizadas a cada mudança de filtro):

| Métrica | Cálculo | Contexto |
|---------|---------|---------|
| **Total** | Contagem de todas restrições filtradas (excl. arquivadas) | Baseline de volume |
| **Abertas** | Status ∈ {Planejada, Em Execução, Atrasada} | Restrições ativas que carecem de atenção |
| **Atrasadas** | Status = "Atrasada" | Risco imediato — requer ação urgente |
| **Concluídas** | Status = "Concluída" (dentro da janela de conclusão configurada) | Métricas de resolução |

Cada métrica exibe número absoluto + variação em relação ao período anterior (se histórico disponível).

##### Gráficos de Distribuição

Abaixo dos KPIs, uma série de barras horizontais agrupadas:

**1. Restrições por Obra** — barra de contagens segmentada por nome da obra:

```
Obra A ████████ 24 restrições
Obra B ██████ 18 restrições
Obra C ████ 12 restrições
```

Útil para identificar qual obra concentra maior volume de impedimentos.

**2. Restrições por Setor Responsável** — barras coloridas por setor (cores de `lstResponsibleSectorColors`):

```
Obra              ████████ 16
Fornecedor        ██████ 12
Projetos          ████ 8
Recursos Humanos  ██ 4
```

Identifica qual departamento/categoria responde por mais bloqueadores.

**3. Responsáveis Externos** — análise específica de restrições com **setor = "Fornecedor"**, agrupadas por nome do responsável:

```
Fornecedor A ████████ 8 restrições
Fornecedor B ██████ 5 restrições
Fornecedor C ██ 2 restrições
```

Permite priorizar negociação com fornecedores críticos.

**4. Restrições por Status** — legend com contagem e cores de `lstRestrictionStatus`:

```
🟢 Concluída   42
🟡 Em Execução 18
🔴 Atrasada    7
⚪ Planejada   15
```

Visão rápida da saúde geral do quadro.

**5. Restrições por Categoria (6M)** — chips com ícones de categoria + barras:

```
Mão de obra    [ícone] █ 8 restrições
Materiais      [ícone] █ 10 restrições
Máquinas       [ícone] █ 5 restrições
Métodos        [ícone] █ 12 restrições
Meio Ambiente  [ícone] █ 3 restrições
Medição        [ícone] █ 4 restrições
```

Identifica qual fator (6M) é raiz de mais problemas — útil para direcionar estratégias de melhoria.

**6. Restrições por Nível de Escalonamento** — barras por nível:

```
N2 ████████ 24
N3 ██████ 16
N4 ██ 8
```

(N1 não exibido se `includeN1 = false`)

**7. Índice de Atraso (Concluídas)** — progress circular:

```
Concluídas no prazo: ███████ 85%
Concluídas atrasadas: 15%
```

Métrica de performance: qual % das restrições foi resolvido dentro do prazo de comprometimento.

#### 4.3.3 Listagem (List Tab)

Grade tabular de todas as restrições com suporte a **busca texto** e **filtros globais** aplicados.

##### Colunas exibidas

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| **Obra** | string | Nome da obra |
| **Serviço/Atividade** | string | Atividade da EAP afetada |
| **Status** | badge colorido | Planejada / Em Execução / Atrasada / Concluída |
| **Responsável** | chip | Departamento ou nome do responsável |
| **Setor** | string | Setor responsável (Obra, Fornecedor, etc.) |
| **Categoria 6M** | icon + label | Ícone de causa raiz |
| **Nível** | chip | N1 / N2 / N3 / N4 |
| **Data de Comprometimento** | date | Prazo acordado para resolução |
| **Há X dias** | relative time badge | Dias desde criação (verde se concluída recentemente, vermelho se atrasada) |

##### Interação

- **Clique em linha:** abre detalhes da restrição no painel lateral ou página dedicada
- **Busca por texto** (campo de pesquisa na AppBar): busca em descrição, serviço e responsável
- **Ordenação:** multicritério por coluna (status, data de comprometimento, etc.)
- **Seleção em lote:** checkboxes para exportação ou edição em massa

#### 4.3.4 Kanban Tab

Visualização em **cards por coluna de status** com suporte a **drag-and-drop** para transição de status:

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Planejada    │ Em Execução  │ Atrasada     │ Concluída    │
│ (15 cards)   │ (8 cards)    │ (3 cards)    │ (42 cards)   │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ [REST-001]   │ [REST-003]   │ [REST-005]   │ [REST-007]   │
│ Mão de obra  │ Materiais    │ Métodos      │ Máquinas     │
│ Obra A       │ Obra B       │ Obra A       │ Obra B       │
│ N2           │ N3           │ N2           │ N4           │
│              │              │              │ há 2 dias    │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

##### Card de Restrição

Cada card exibe:

- **Título/ID da restrição**
- **Obra e serviço** (desnormalizados)
- **Ícone de categoria 6M** com cor
- **Nivel de escalonamento** (chip)
- **Data de comprometimento** (se próxima do vencimento, destaca em vermelho)
- **Responsável** (chip/avatar)
- **Badge de tempo desde criação** ou "há X dias" para concluídas (verde se recente, vermelho se atrasado)

##### Funcionalidades

- **Drag-and-drop:** arrastar card de uma coluna para outra **alterna o status** da restrição (sem salvar imediatamente; exibe confirmação)
- **Clique no card:** abre sidebar ou página de detalhes
- **Menu de contexto:** opções de editar, duplicar, compartilhar, converter em plano de ação
- **Filtros globais aplicados:** colunas mostram apenas restrições que atendem aos filtros de setor, obra, escalonamento, etc.
- **Reordenação dentro da coluna:** swipe ou drag vertical para reordenar (ordem salva por usuário, preferência local)

#### 4.3.5 Modelo de Dados (`RestrictionModel`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` / doc ID | string | Salesforce WorkOrderId ou UUID interno se criado no módulo |
| `tenantId` | string | Isolamento multi-tenant |
| `siteId` | string | Obra associada |
| `serviceName` | string | Nome da atividade/serviço afetado |
| `description` | string | Descrição do bloqueador |
| `responsible` | `Map<dynamic, dynamic>` | Crew member (name, email, role, phone, uid) |
| `sector` | string | Setor responsável (Obra, Fornecedor, Projetos, etc.) — usado para identificar responsáveis externos |
| `status` | string | Planejada / Em Execução / Atrasada / Concluída |
| `categoryCode` | string | Categoria 6M (mao_de_obra, materiais, maquinas, metodos, meio_ambiente, medicao) |
| `escalationLevel` | int | 1 (N1) / 2 (N2) / 3 (N3) / 4 (N4) |
| `commitmentDate` | DateTime | Prazo acordado para resolução |
| `createdAt` | DateTime | Timestamp de criação (Firestore) |
| `completedAt` | DateTime | Timestamp de conclusão (se status = Concluída) |
| `isArchived` | bool | Controle de restrições arquivadas (sempre excluídas de análises) |
| `comments` | string | Histórico de comentários sobre a restrição |
| `attachments` | List<String> | URLs de documentos/fotos anexadas |
| `relatedPlans` | List<String> | IDs de planos de ação vinculados |

#### 4.3.6 Ações e Integrações

**Por Restrição:**
- **Criar Plano de Ação:** gera um novo plano vinculado à restrição (rota para Controle de Produção)
- **Compartilhar:** copia link deep-link ou envia por WhatsApp/e-mail
- **Anexar:** upload de fotos, documentos ou croquis
- **Editar:** modifica todos os campos (validações de privilégio por perfil)
- **Duplicar:** cria cópia com mesmos dados (útil para restrições recorrentes)
- **Arquivar:** marca `isArchived = true` (soft-delete; exclui de análises)

**Análises Globais (KAI):**
- **Relatório analítico 6M:** agente `kai-restriction-analyst` via Cloud Function analisa distribuição de categorias, identifica padrões e recomenda ações
- **Sugestões automáticas:** agente `kai-suggestion` propõe novas restrições baseado em datas de início de atividades futuras sem restrições registradas
- **Chat integrado:** usuário pode conversar com KAI para análises ad-hoc (ex.: "quais são as minhas restrições críticas por setor?")

#### 4.3.7 Privilégios e Acesso

| Privilégio | Ação | Nota |
|------------|------|------|
| `mid_term` (padrão) | Visualizar Overview, List, Kanban; editar status | Requer cronograma ativo |
| `mid_term_manager` | Editar descrição, responsável, datas, categoria 6M; criar/deletar restrições | Gerentes |
| Admin | Acesso irrestrito; editar para qualquer obra; gerenciar privilégios | Super-users |
| `restrictions_view_only` | Apenas visualizar (somente leitura) | Stakeholders |

**Branch Access:** O `BranchAccessService` filtra restrições por `siteId`:
- **Admin/Manager:** vê dropdown de todas as obras; pode filtrar ou comparar multi-obra
- **Usuário comum:** vê apenas sua obra (branch)

#### 4.3.8 Performance e Otimizações

- **N1 Exclusão por Padrão:** Query Firestore com `.where('escalationLevel', '>=', 2)` reduz volume inicial; toggle N1 refaz query
- **Computed Aggregations:** KPIs (Total, Abertas, Atrasadas, Concluídas) calculados em memória conforme filtros mudam (não engatilham queries novas)
- **Real-time Listen:** Overview, List e Kanban escutam mudanças via `snapshots()` com filtros aplicados; updates refletem em < 1s de latência
- **Lazy Load (Kanban):** Cards renderizados sob demanda conforme scroll
- **Caching de Cores:** `lstResponsibleSectorColors` e `lstRestrictionStatus` carregadas uma vez ao inicializar o controller

#### 4.3.9 Detalhes: Sidebar/Página de Restrição

Ao selecionar uma restrição (clique em List ou Kanban), abre um painel lateral (mobile) ou página dedicada exibindo:

- **Cabeçalho:** ID, título, status com progresso visual
- **Abas:**
  - **Geral:** descrição, responsável, setor, categoria 6M, datas, nível
  - **Histórico:** timeline de mudanças de status, comentários, uploads
  - **Planos de Ação:** lista de planos vinculados com links para abri-los
  - **Anexos:** fotos, croquis (viewer integrado)
- **Campos editáveis:** conforme privilégio do usuário
- **Botão de ação:** "Criar Plano", "Arquivar", "Duplicar", "KAI—Gerar Relatório"

**Chave de privilégio:** `mid_term` (visualização) · `mid_term_manager` (edição)

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 5. Qualidade

Ferramentas de controle da qualidade de execução, cobrindo verificação em campo por serviço, inspeções sistemáticas de obra e estabilização dos processos operacionais.

---

### 5.1 Fichas de Verificação de Serviço (FVS)

Dashboard central das FVS — checklists de qualidade associados a cada atividade executada. Opera em dois níveis:

| Nível | Descrição |
|-------|-----------|
| **Templates (`DefaultFVSModel`)** | Checklists padronizados por tipo de serviço, reutilizáveis entre obras; carregados com cache reativo |
| **Instâncias (`FVSModel`)** | FVS preenchida para uma atividade específica, com resultado por item de checklist |

**Fluxo de status:**

```
Em Verificação → Inspecionada → Aprovada
                      ↓
               Rejeitada → Pendência → (retrabalho) → Aprovada
```

**Campos por instância:** atividade vinculada (serviço + lote denormalizados para leitura rápida), responsável, aprovador, prioridade, lista de itens com resultado (`conforme / não conforme / N/A`), contagem de itens rejeitados.

**Filtros:** status, atividade, responsável, prioridade, lote, tag de serviço, busca por texto.

**Contadores do dashboard:** Em Verificação · Pendências · Inspecionadas · Aprovadas · Total de Itens Rejeitados (rollup).

**Exportação:** PDF com fotos, assinaturas e observações.

**Chave de privilégio:** `fvs_fill` / `fvs_approval` / `quality_manager`

---

### 5.2 Estabilização Lean — Inspeção 5S

Aplicação digital de checklists de inspeção Lean (5S) com estrutura hierárquica de seções e subseções. Cada inspeção registra data, torre/bloco inspecionado e responsável.

**Por item:** resultado (`conforme / não conforme / parcial`), foto de evidência e observação.

**Score:** calculado automaticamente por seção (roll-up dos filhos) e score total da inspeção. Histórico de até 24 meses com gráfico de evolução e média histórica. Fotos armazenadas com compressão antes do upload.

**Chave de privilégio:** `lean_inspection`

---

### 5.3 Inspeção Final de Obra e de Unidades

Módulo para inspeção técnica pré-entrega realizada por engenheiro, unidade a unidade. Cobre checklist técnico-normativo de ambientes internos, registro de não-conformidades (NCs) com evidência fotográfica e marcação em planta, acompanhamento de reinspção e painel analítico de qualidade por obra.

**Chave de privilégio:** `inspection_final_qualidade` (acesso ao Painel de Qualidade; execução de inspeção disponível para todos usuários com acesso à unidade)

#### Fluxo de Status

```
draft ──► in_progress ──► completed ──► pending_reinspection ──► closed
                                               ↑ (se há NCs abertas ao finalizar)
```

| Status | Significado |
|--------|-------------|
| `draft` | Criada mas não iniciada |
| `in_progress` | Execução ativa (ambientes sendo vistoriados) |
| `completed` | Todos itens avaliados; sem NCs pendentes (ou ignoradas) |
| `pending_reinspection` | Concluída com NCs que precisam de reinspção |
| `closed` | Reinspção encerrada (NCs aprovadas ou reaberturas resolvidas) |

#### Modelo de Dados (`InspectionFinal`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | string | UUID gerado localmente |
| `siteId / unitId` | string | Obra e unidade inspecionada |
| `towerName / unitName` | string | Desnormalizados para exibição |
| `status` | `InspectionFinalStatus` | Ver fluxo acima |
| `templateId / templateVersion` | string / int | Template de checklist base |
| `environmentsConfig` | `List<EnvironmentConfig>` | Snapshot por ambiente (nome, quantidade, ordem, habilitado) |
| `croquiRefs` | `List<InspectionCroquiRef>` | Plantas/croquis vinculados (suporta múltiplos para duplex/triplex) |
| `participants` | `List<InspectionParticipant>` | Equipe presente na inspeção |
| `workOrderSfId` | string | ID do Work Order no Salesforce (rastreabilidade) |
| `totalItems / conformCount / nonConformCount / naCount` | int | Contadores de itens |
| `openIssues` | int | NCs ainda abertas (controla badge na entrada) |
| `conformancePercent` | double | `(conform + na) / total` |
| `schemaVersion` | int | Controle de compatibilidade |
| `createdAt / updatedAt` | DateTime | Auditoria |

#### 5.3.1 Tela de Entrada (`InspectionEntryView`)

Ponto de acesso por unidade. Exibe 4 cartões de ação:

| Cartão | Rota | Condição |
|--------|------|----------|
| **Criar Inspeção** | `/inspection-final-config` | Sempre disponível |
| **Continuar Inspeção** | `/inspection-final-execute` | Se há inspeção `in_progress` |
| **Reinspecionar Pendências** | `/inspection-final-reinspect` | Se há inspeção `pending_reinspection`; exibe badge com nº de NCs abertas |
| **Atualizar Inspeções** | — | Recarrega lista da unidade |
| **Painel de Qualidade** | `/inspection-quality-dashboard` | Requer privilégio `inspection_final_qualidade` ou admin |

Histórico de inspeções da unidade listado abaixo, com status, data, contagem de NCs e opção de exclusão (admin).

#### 5.3.2 Configuração (`InspectionConfigView`)

Antes de iniciar, o engenheiro configura a inspeção:

- **Equipe (Participants):** adiciona colaboradores da obra que participam da vistoria (`InspectionParticipant` com `id`, `name`, `role`).
- **Croqui / Planta:** seleciona uma ou mais plantas da unidade (`InspectionCroquiRef`). Suporta múltiplos croquis para unidades duplex/triplex.
- **Ambientes:** lista gerada a partir do template; cada `EnvironmentConfig` pode ser habilitado/desabilitado e ter quantidade ajustada (ex.: 2 quartos). Opções de menu: *Habilitar Todos*, *Desabilitar Todos*, *Salvar como Modelo*, *Iniciar Inspeção*.
- **Template de obra:** quando há múltiplos templates disponíveis, chip de seleção permite trocar o modelo ativo antes de iniciar.

O botão FAB `rocket_launch` confirma e cria o documento `InspectionFinal` + subcoleções no Firestore, avançando para execução.

#### 5.3.3 Execução (`InspectionExecuteView`)

Tela principal de vistoria:

**Cabeçalho:** `totalAnswered / totalItems` respondidos + `LinearProgressIndicator` com percentual geral. Botão direito navega ao Resumo.

**Barra de Ambientes:** `TabBar` horizontal colorido (themePurple). Cada aba exibe ícone de status do ambiente (check verde = concluído, pending laranja = em andamento, círculo vazio = pendente) e nome. Itens carregados com *lazy-load* ao entrar na aba.

**Lista de Itens:** separada em dois grupos:
- *Itens de Liberação* (laranja) — pré-requisitos críticos
- *Itens de Verificação* (roxo) — itens padrão

Cada item pode ser um **sistema** (cabeçalho expansível com subsistemas aninhados) ou folha. O status do sistema é derivado dos subsistemas: verde se todos conformes, vermelho se algum NC, cinza + circular se nem todos avaliados.

**Ação por item (swipe/slidable):**

| Deslize | Ação |
|---------|------|
| Esquerda (verde) | Marcar **Conforme** |
| Esquerda (cinza) | Marcar **N/A** |
| Direita (vermelho) | Registrar **Não Conformidade** → abre `InspectionIssueView` |

Ao marcar Conforme em item que já possui NCs registradas, um diálogo de confirmação informa que as pendências continuarão existindo.

**Cascata de status:** ao marcar um sistema-cabeçalho, o status se propaga para todos os subsistemas filhos.

**Auto-save:** toda alteração de status é aplicada localmente de forma imediata (sem loading) e persistida no Firestore de forma assíncrona. Contadores de ambiente e inspeção também são atualizados assincronamente via `_updateEnvCounters()`.

**Barra inferior:** botão *Finalizar Inspeção* disponível quando todos os ambientes estão concluídos (`allEnvironmentsCompleted`). Ao finalizar, a inspeção transita para `completed` ou `pending_reinspection` conforme existência de NCs abertas.

#### 5.3.4 Registro de Não-Conformidades (`InspectionIssueView`)

Formulário completo para criar ou editar uma NC:

**Taxonomia assistida por IA:** campo de descrição livre dispara `TaxonomyController._suggestTaxonomyFromDescription()` via `AiTaxonomyService`. Resultado preenche cascata de pickers: *Item → Sistema → Descrição → Falha*. Usuário pode buscar manualmente ou aceitar sugestão.

**Severidade:** seletor com 4 níveis coloridos:

| Nível | Cor |
|-------|-----|
| `low` | Cinza/verde |
| `medium` | Laranja |
| `high` | Vermelho |
| `critical` | Vermelho escuro |

**Fotos + Sketch:** `PhotoSketchWidget` permite captura direta de câmera ou galeria, com anotação/desenho sobre a imagem antes de salvar.

**Marcação em croqui (`_CroquiMarkerBoard`):** croquis carregados via `croquiRefs`. Para unidades com múltiplos croquis (duplex/triplex), um `TabController` exibe uma aba por planta. O responsável pode colocar marcadores com coordenadas `(x, y)` normalizadas + foto por marcador.

**Responsável:** picker da lista `crewList` da obra.

**Comentários livres:** campo de texto opcional.

#### 5.3.5 Reinspção (`InspectionReinspectView`)

Lista as NCs com status `resolved` aguardando aprovação do engenheiro. Cabeçalho mostra `avaliadas / total` e indicador de progresso. Por NC (swipe ou botões):

- **Aprovar** → NC vai para `approved`; item avaliado some ou fica opaco.
- **Reabrir** → modal exige motivo obrigatório; NC retorna a `reopened → in_progress`.

Ao avaliar todas, a inspeção pode ser encerrada (`closed`).

#### 5.3.6 Resumo e Exportação (`InspectionSummaryView`)

Tela de resultados após execução. Exibe:
- Percentual de conformidade (`conformancePercent`) e contadores por status.
- Lista de NCs agrupadas por ambiente, com status e severidade.
- Detalhe de cada NC via `showIssueDetailSheet`.
- **Exportação PDF** (`InspectionPdfGenerator.generate`) com capa, tabela de ambientes, lista de NCs com fotos, marcadores de croqui e assinaturas. Acionado via ícone na AppBar.

#### 5.3.7 Painel de Qualidade (`InspectionQualityDashboardView`)

Dashboard analítico multi-inspeção. Suporta dois modos:
- **Escopo por unidade** (`unitId` preenchido): NCs de todas as inspeções históricas da unidade.
- **Escopo de obra** (`siteWide: true`): todas inspeções de toda a obra (visão do gestor de qualidade).

**5 abas:**

| Aba | Conteúdo |
|-----|----------|
| **Dashboard** | `QualityDashboardAnalyticsTab` — KPIs, gráficos de distribuição por severidade, evolução temporal |
| **Por Item** | NCs agrupadas por nome do item; clique filtra a aba Lista |
| **Por Cômodo** | NCs agrupadas por ambiente; clique filtra |
| **Por Unidade** | NCs agrupadas por torre/unidade; clique filtra |
| **Lista** | Tabela filtrada de todas NCs com chips de filtro ativos |

**Filtros disponíveis:** Status, Torre, Unidade, Ambiente, Nome do Item. Toggle *Mostrar aprovadas* oculta/exibe NCs já fechadas. Cada NC pode ser editada inline via `showIssueDetailSheet` com atualização reativa.

#### Integração com Salesforce

Inspeções vinculadas a Work Orders do Salesforce via `workOrderSfId`. O controller de personalizações (`InspectionFinalController`) carrega Work Orders ativos via Cloud Function `getWorkOrders`, filtrando por `assetId` e pelo `inspectionFinalWorkTypeId = '08qU40000000MjhIAE'`, com paginação de 50 registros por página.

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 6. Custo e Contratos

Módulo integrado de controle orçamentário, gestão de contratos e processo de cotação de suprimentos. Dados de custo realizado consumidos em tempo real do [Azure Synapse](05_integracoes.md#3-azure-synapse-analytics).

---

### 6.1 Controle de Custo e Orçamento

Sistema de controle orçamentário com comparativo planejado vs. comprometido vs. realizado. Cada item de orçamento contém: código WBS, descrição, referência de instalação (INS ID), quantidade, preço unitário, valor total, flag de aprovação e comentários.

**Versionamento e rastreabilidade de mudanças:**
- Snapshots de versão (`BudgetVersionModel`) com lista de alterações: campo modificado, valor anterior, valor novo, autor e timestamp
- Fluxo de aprovação para submissão de nova versão
- Dialog de revisão das mudanças pendentes antes do commit

**Sistema de comentários (distribuído, sem hotspot):**
- Comentários por item distribuídos em buckets hash para otimizar consultas
- Fontes de comentário rastreadas: `manual` (digitado), `concat` (resumo gerado pelo KAI)
- Histórico de comentários por item com carregamento incremental

**Categorias de desvio:**

| Código | Categoria |
|--------|-----------|
| `ORC` | Variação de Orçamento |
| `CST` | Variação de Custo |
| `PRJ` | Variação de Projeto |
| `ENG` | Variação de Engenharia |
| `PRZ` | Variação de Prazo |

**Fontes de custo realizado integradas:**
- `BudgetRealizedDetail` — despesas de execução com timestamp
- `StockERPDetail` — consumo de materiais do almoxarifado
- `StockPartnerEntry` — notas fiscais de fornecedores

**Integração:** Dados realizados lidos do [Azure Synapse](05_integracoes.md#3-azure-synapse-analytics) em tempo real via `SynapseController`.

**Exportação:** Excel com tabelas de desvio, vinculação a contratos e análise de custo; relatórios multi-obra consolidados.

**Chave de privilégio:** `cost_control_view`

---

### 6.2 Cotações

Módulo de cotações gerencia o processo completo de coleta e análise de propostas comerciais para aquisição de materiais e contratação de serviços. Digitaliza o processo que geralmente ocorre por e-mail e planilhas, centralizando todo o fluxo com rastreabilidade completa.

#### Conceito: Mapa de Cotação

Cada processo de cotação é representado por um **Mapa de Cotação** — documento central que agrupa todos os itens a serem cotados em um único processo de compra. Exemplos: *"Mapa de Materiais Hidráulicos — Torre A"*, *"Contratação de Mão de Obra Elétrica — Fase 2"*.

Um mapa percorre **4 estágios** sequenciais até a aprovação formal:

```
┌─────────────────────────────────────────────────────────────────┐
│  Estágio 1        Estágio 2          Estágio 3      Estágio 4   │
│  Montagem    →  Cotação com    →    Análise    →   Aprovado     │
│  do Mapa       Fornecedores                                      │
└─────────────────────────────────────────────────────────────────┘
```

| Estágio | Objetivo | Transição |
|---------|----------|-----------|
| **1 – Montagem** | Definir o que será cotado: criar mapa, adicionar itens, associar rubricas WBS, selecionar fornecedores; importação via Excel | ≥ 1 item + ≥ 1 fornecedor |
| **2 – Cotação** | Coletar propostas: modo `isSupplierSelectionMode` (tabela matricial itens × fornecedores), registrar preços, upload de propostas, controle de status por fornecedor | ≥ 2 propostas recebidas |
| **3 – Análise** | Selecionar vencedor: comparativo lado a lado, split de fornecimento por item, justificativa obrigatória se não for menor preço, categorização de variação orçamentária | Análise concluída |
| **4 – Aprovado** | Registro final: aprovação formal, atualização automática do comprometido no Controle de Custo, relatório resumo gerado; mapa fica *read-only* | — |

#### Itens de Cotação

| Campo | Descrição |
|-------|-----------|
| Nome / Descrição | O que está sendo cotado |
| Código | Código interno de referência |
| Unidade | Unidade de medida (m², kg, un, m³ etc.) |
| Quantidade | Quantidade necessária |
| Observações Técnicas | Especificações, normas ou requisitos |
| Vinculação Orçamentária | Rubrica do orçamento (`budgetItemKey`) |
| Prazo de Entrega | Data máxima necessária |

**Importação por Excel:** template disponível para download; colunas obrigatórias (descrição, unidade, quantidade); validação linha a linha com relatório de erros antes de confirmar.

#### Categorias de Variação Orçamentária

| Código | Categoria |
|--------|-----------|
| `ORC` | Variação de Orçamento (estimativa original inadequada) |
| `CST` | Variação de Custo (mercado acima do previsto) |
| `PRJ` | Variação de Projeto (escopo alterado pelo projeto) |
| `ENG` | Variação de Engenharia (solução técnica diferente) |
| `PRZ` | Variação de Prazo (impacto de urgência no preço) |

#### Gestão de Fornecedores

| Status | Descrição |
|--------|-----------|
| Convidado | Solicitação enviada, aguardando resposta |
| Aguardando | Confirmou recebimento, proposta em elaboração |
| Proposta Recebida | Valores registrados no sistema |
| Vencedor | Fornecedor selecionado na análise |
| Não Selecionado | Participou, mas não foi escolhido |
| Desistiu | Recusou participar ou não respondeu no prazo |

Comunicação: envio de e-mail de convite via Cloud Function + Resend; registro de todas as comunicações no log do mapa.

#### Auditoria e Log Automático

Todo evento relevante é automaticamente registrado no log do mapa (`addLog()`):

| Evento | Detalhes Registrados |
|--------|---------------------|
| Criação do mapa | Usuário criador, data/hora |
| Item adicionado | Nome, quantidade, unidade, quem adicionou |
| Fornecedor adicionado | Nome do fornecedor, quem adicionou |
| Preço registrado | Fornecedor, item, valor, quem registrou |
| Mudança de estágio | De qual estágio para qual, por quem, data/hora |
| Fornecedor selecionado | Item, vencedor, valor, justificativa |
| Variação categorizada | Categoria, valor do desvio, responsável |
| Aprovação final | Aprovador, data/hora, valor total aprovado |
| Comentário adicionado | Usuário, conteúdo, data/hora |
| Arquivo anexado | Nome do arquivo, uploader, data/hora |

#### Funcionalidades Adicionais

- **Anexos e documentos:** upload de propostas em PDF, especificações técnicas e desenhos; armazenamento organizado por mapa (`clientId/cotacoes/mapaId/`)
- **Comentários internos:** thread de comentários por mapa com menções de usuários (`@usuário`) e notificações push
- **Clonagem de mapa:** copia itens e estrutura; preços e fornecedores selecionados são resetados
- **Exportação:** relatório PDF com comparativo completo de propostas; Excel de cotação para análise externa

**Integração com Controle de Custo:** cotações aprovadas aparecem como *desvios aprovados* no módulo de custo, com drilldown direto do custo → mapa de cotação que gerou o desvio.

---

### 6.3 Gestão de Contratos e Orçamento Projetado *(integração SQL)*

> **Status:** Módulo em evolução — funcionalidades avançadas dependentes de integração com SQL Server / Azure Synapse.

**Gestão de Contratos:**
- Cadastro de contratos com fornecedores e empreiteiros
- Vinculação de contratos a rubricas WBS do orçamento
- Controle de aditivos, retenções e cronograma de pagamentos
- Medições contratuais por etapa com fluxo de aprovação

**Orçamento Projetado:**
- Simulação de projeção do custo final da obra (Estimate at Completion — EAC)
- Considera comprometidos (cotações aprovadas), realizados (Synapse) e saldo a contratar
- Permite cenários de variação: otimista, mais provável, pessimista

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 7. Controle de Unidades

Módulo para rastreabilidade completa de unidades habitacionais (apartamentos, lotes, lojas). Dados sincronizados do Salesforce Asset via Cloud Function autenticada.

**Hierarquia:** Empreendimento → Torre → Unidade

Leitura disponível para usuários com privilégio `units`; edição restrita a perfis `project_manager`.

---

### 7.1 Dashboard de Unidades

Tela principal com dashboard por torre: filtros de status, tipo e personalização. Acesso rápido a qualquer unidade com indicadores de progresso geral por torre.

---

### 7.2 Ficha da Unidade — Identificação

Identificação completa: número, torre, andar, tipo (kitnet, 1Q, 2Q, 3Q, cobertura, loja etc.). Dados do proprietário: nome, CPF, telefone, e-mail, data de contrato. Status global com histórico de mudanças de status (autor + timestamp). Observações sincronizadas com o registro do Salesforce Asset.

---

### 7.3 Personalização

Customizações solicitadas pelo cliente: pisos, revestimentos, cores, modificações de planta. Cada item tem:

- Status: `solicitado / em execução / concluído`
- Fotos de referência
- Data prevista de conclusão
- Alerta quando exige prazo especial de execução

---

### 7.4 Avanço Físico por Unidade (Aba Obra)

Status de cada etapa de execução na unidade (alvenaria, gesso, elétrica etc.) com percentual de conclusão, equipe responsável, fotos de evidência por etapa e pendências vinculadas a planos de ação.

---

### 7.5 Aba Checklist de Obra

Lista de verificação pré-entrega com confirmação item a item em campo:

- Foto de evidência por item
- Status: `pendente / conforme / não conforme / N/A`
- Log de quem executou cada item e quando
- Geração automática de relatório ao concluir

---

### 7.6 Vistoria do Cliente

Condução da vistoria de entrega com o cliente presente:

- Checklist digital preenchido durante a visita
- Registro de pendências com foto e prazo de resolução
- **Assinatura digital do cliente** capturada no app
- Gera **Laudo de Vistoria em PDF** com fotos, assinaturas e pendências
- Controle de revistorias com histórico completo de visitas

**Status:** Agendada → Aguardando → 1ª Vistoria → Revisoria → Entregue

---

### 7.7 Aba Inspeção Final Técnica

Inspeção técnica pré-entrega por engenheiro — distinta da vistoria do cliente, com foco em aspectos técnicos e normativos:

- Checklist específico por tipo de unidade
- Registro de pontos de atenção com fotos
- **Assinatura digital do engenheiro**
- Gera relatório de inspeção final para arquivo técnico

---

### 7.8 Gerador de Etiquetas QR Code

| Modo | Descrição |
|------|-----------|
| **Folha (2×6)** | 12 etiquetas por A4 para papel adesivo; QR Code + número, torre e andar |
| **Placa Grande (A4)** | Uma unidade por folha com QR Code amplo, logo do empreendimento e dados completos |

Seleção flexível: unidade individual, por andar, por torre ou empreendimento completo. Pré-visualização antes da geração do PDF.

---

### 7.9 Arquitetura de Cache (3 Camadas)

```
Requisição de dados
      ↓
1. Hive (local) ──────────→ Retorna imediatamente (offline-first)
      ↓ (expirado ou ausente)
2. Backend ────────────────→ Atualiza Hive e retorna
      ↓ (ausente ou refresh forçado)
3. Cloud Function / SF ───→ Atualiza backend e Hive e retorna
```

---

[↑ Voltar ao topo](#navegação-rápida)

---

## 8. Almoxarifado e Ativos

Central de gestão de materiais e ativos físicos da obra.

---

### 8.1 Almoxarifado Digital

Central de gestão de materiais da obra, organizada em seis abas via `NavigationRail`:

| Aba | Função |
|-----|--------|
| **Requisições** | Solicitações de material por usuários de campo; fluxo de aprovação (`pendente → aprovado → retirado`) |
| **Pedidos** | Ordens de compra externas ou internas; rastreio de datas prometidas e recebidas, custo e nota fiscal |
| **Saídas** | Baixas físicas de estoque geradas de requisições aprovadas, com rastreabilidade completa (quem, quando, quantidade, destino) |
| **Kits** | Pacotes pré-montados de materiais para uma atividade; convertíveis em requisições se uso parcial |
| **Consulta** | Saldo em tempo real; catálogo integrado com [Mega ERP](05_integracoes.md#4-mega-erp-materiais) (código, descrição, unidade, grupo) |
| **Auditoria** | Histórico de movimentações por tipo com filtros de data e responsável |

**Itens de estoque:** código, descrição, unidade (peça, m, m², m³, kg), quantidade em mãos, ponto de resuprimento, fornecedor, preço unitário, categoria.

**Funcionalidades transversais:**
- Etiquetas QR Code por item via `QrLabelAssistantView`
- Notificações push para aprovadores via FCM (tópico `pedidos_{clientId}_{branchId}`)
- Importação via CSV e exportação Excel

**Chave de privilégio:** `warehouse_view`

---

### 8.2 Reserva de Veículos e Ferramentas

Gestão da frota e ativos compartilhados da filial com calendário mensal de reservas (janela de 60 meses: 30 passados + atual + 30 futuros).

**Veículos / Ativos:** nome, tipo, status (`disponível / reservado / manutenção`) e especificações.

**Reservas:** veículo, solicitante, período (início/fim), justificativa, status (`pendente / aprovado / concluído / cancelado`) e custo associado.

**Gestão de custos:**
- Rastreamento de despesa por reserva
- Rateio de custo entre reservas (`calculateSplit`)
- Totais agregados por período

> O módulo suporta tanto veículos quanto equipamentos/ferramentas compartilhadas (betoneira, andaimes etc.), configuráveis por tipo de ativo.

**Chave de privilégio:** `vehicle_reservation`

---

[↑ Voltar ao topo](#navegação-rápida)

---

## Módulos Complementares

Módulos disponíveis que ampliam a cobertura de produtividade diária, histórico operacional e análise de desempenho.

---

### Kaizen Diário

Dashboard de acompanhamento diário da obra. Para a data selecionada exibe:

- Medições registradas no dia agrupadas por lote e serviço
- Status de cada atividade (planejada, em progresso, concluída)
- Resumo de problemas por serviço
- Atividades sem medição registrada para o dia corrente
- Acesso direto ao formulário de medição de cada atividade

Filtro por lote, seletor de data e versão mobile com menu lateral colapsável.

**Chave de privilégio:** `daily_measurements` ou `daily_kaizen`

---

### Gestão da Estrutura

Visualização matricial no formato **Monkey Chart**: linhas = lotes, colunas = serviços, células = atividades. Para cada célula são carregados os planos de ciclo sob demanda, com indicadores de conclusão e índice de produtividade calculado. Controles de zoom (`monkeyTileScale`) e painéis de estatísticas laterais colapsáveis. Usada para visão macro do sequenciamento e ocupação de equipes por lote/serviço.

**Chave de privilégio:** `daily_measurements`

---

### Histórico de Medições

Consulta paginada de todas as medições realizadas (20 mais recentes na abertura). Navegação retroativa por filtros (período, serviço, lote, responsável). Cada item abre a visualização completa da medição em modo leitura ou edição conforme permissão do usuário.

**Chave de privilégio:** `daily_measurements`

---

### Estatísticas e Dashboard Analítico

**Estatísticas:** Dashboard analítico com três modos de visão — **por Serviço**, **por Empresa/Empreiteira** ou **por Lote**. Métricas consolidadas: produção total, horas, dias, pessoas, medições, problemas, índice de produtividade (médio, mínimo, máximo), comparativo com meta contratual.

**Dashboard:** Análise de desempenho com gráficos em duas abas:
- **Aba Serviços:** tabela ordenável + gráfico de barras de produção ou custo por serviço; comparativo realizado vs. previsto por período
- **Aba Problemas:** gráfico de pizza com frequência de problemas por serviço e lista de ocorrências em aberto

Suporte a acúmulo mensal e exportação via impressão do browser.

**Chave de privilégio:** `prod_bi`

---

[↑ Voltar ao topo](#navegação-rápida)

---

## Consolidado de Módulos

| Módulo | Bloco | Privilégio | Exportação |
|--------|-------|-----------|-----------|
| Planejamento Kaizen | Produtividade | `daily_measurements` | — |
| Medições Kaizen | Produtividade | `daily_measurements` | — |
| Atividades | Planejamento Físico-Financeiro | `baseline_view` | Excel, PDF |
| Gantt Reprogramável | Planejamento Físico-Financeiro | `baseline_view` | — |
| Curva S | Planejamento Físico-Financeiro | `baseline_view` | — |
| CFF | Planejamento Físico-Financeiro | `baseline_view` | Excel, PDF |
| Vinculação de Orçamento | Planejamento Físico-Financeiro | `baseline_view` | — |
| Gestão de Cronogramas | Planejamento Físico-Financeiro | `physical` | — |
| Medições Físicas | Controle de Produção | `physical` | PDF |
| Painel de Produção | Controle de Produção | `baseline_view` | — |
| Equipes e Histograma | Controle de Produção | `teams` | — |
| Planos de Ação | Controle de Produção | `action_plans_view` | Excel, PDF |
| Quadro de Restrições | Rotinas de Gestão | `mid_term` | Excel |
| Farol de Contratações | Rotinas de Gestão | `supply_schedule` ou admin | CSV, Excel |
| FVS | Qualidade | `fvs_fill` / `fvs_approval` / `quality_manager` | PDF |
| Estabilização Lean | Qualidade | `lean_inspection` | PDF |
| Controle de Custo | Custo e Contratos | `cost_control_view` | Excel |
| Cotações | Custo e Contratos | — | PDF, Excel |
| Unidades | Controle de Unidades | `units` | PDF (vistoria, inspeção) |
| Almoxarifado Digital | Almoxarifado e Ativos | `warehouse_view` | Excel, CSV |
| Reserva de Veículos/Ferramentas | Almoxarifado e Ativos | `vehicle_reservation` | — |
| Kaizen Diário | Complementar | `daily_measurements` ou `daily_kaizen` | — |
| Gestão da Estrutura | Complementar | `daily_measurements` | — |
| Histórico de Medições | Complementar | `daily_measurements` | — |
| Estatísticas | Complementar | `prod_bi` | — |
| Dashboard | Complementar | `prod_bi` | Impressão |

---

*Documento: Módulos de Gestão de Obra | Versão 2.0 | Kaizen Gerenciamento de Obras*
ção Lean | Rotinas de Gestão | `lean_inspection` | PDF |
| Gestão de FVS | Qualidade | `fvs_fill` / `fvs_approval` / `quality_manager` | PDF |
| Custo e Orçamento | Custo e Orçamento | `cost_control_view` | Excel |
| Almoxarifado Digital | Gestão de Materiais | `warehouse_view` | Excel, CSV |
| Unidades | Controle de Unidades | `units` | PDF (vistoria, inspeção) |
| Reserva de Veículos | Outros Módulos | `vehicle_reservation` | — |
