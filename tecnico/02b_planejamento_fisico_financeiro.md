---
layout: page
title: "Planejamento Físico-Financeiro"
permalink: /tecnico/modulos/planejamento/
category: "Módulos"
order: 2.5
toc: true
---

> **Parte de:** [Módulos de Gestão de Obra](../02_modulos_gestao_obra) — Bloco 2

O módulo de **Planejamento Físico-Financeiro** oferece o conjunto completo de ferramentas para visualizar, analisar e reprogramar o cronograma da obra. Todos os dados fluem da integração com o [Prevision](05_integracoes.md#2-prevision-planejamento-de-obras) (API GraphQL) e são persistidos no Firestore por obra.

**Chaves de privilégio:** `baseline_view` (leitura e análise) · `physical` (gestão de cronogramas e importação)

---

## Sumário

| Sub-módulo | Rota / Widget | Privilégio |
|-----------|--------------|-----------|
| [2.1 Atividades (tabela)](#21-atividades--tabela-do-cronograma) | `BaselineView` (mode=`""`) | `baseline_view` |
| [2.2 Macaquinho](#22-macaquinho--visão-matricial) | `BaselineMonkey` (mode=`'monkey'`) | `baseline_view` |
| [2.3 Gantt Reprogramável](#23-gantt-reprogramável) | `BaselineGantt` | `baseline_view` |
| [2.4 Curva S — Acumulada](#241-aba-0--curva-s-acumulada) | `ScurveView` aba 0 | `baseline_view` |
| [2.5 Gráfico Mensal de Deltas](#25-gráfico-mensal-de-deltas) | `ScurveView` aba 1 | `baseline_view` |
| [2.6 Análise de Serviços](#26-análise-de-serviços--grade-mensal) | `ScurveView` aba 2 | `baseline_view` |
| [2.7 CFF](#27-cff--cronograma-físico-financeiro) | `ScurveView` aba 3 | `baseline_view` |
| [2.8 Budget Weights](#28-vinculação-de-orçamento--budget-weights) | Prevision import | `baseline_view` |
| [2.9 Gestão de Cronogramas](#29-gestão-de-cronogramas) | `PhisycalView` | `physical` |

---

## Arquitetura do Módulo

### Controladores

| Controlador | Responsabilidade |
|------------|-----------------|
| `BaselineController` | Estado central do cronograma: atividades carregadas, filtros, pilha de desfazer, modo de visualização (`activity table` vs `monkey` vs `gantt`) |
| `ScurveController` | Análise da Curva S e CFF; recebe `ScheduleModel` como argumento e coordena os quatro gráficos/abas |
| `PhisycalController` | Gestão do ciclo de vida dos cronogramas: criação, seleção de baseline e importação |
| `OutterPrevisionController` | Integração com a API Prevision; carrega cronogramas, atividades, baseline, pesos de orçamento e medições |

### Fluxo de dados

```
Prevision API (GraphQL)
        │
        ▼
OutterPrevisionController
  ├── activateBaselineSchedule(siteId)  ←── baseline imutável de referência
  ├── loadSchedule(id)                  ←── cronograma atual (reprogramável)
  └── applySCurveBaselineFlagAdjustment()
              │
              ▼
       ScheduleModel
  ├── daySCurve            Map<DateTime, {s,b,d}>  — dados acumulados diários
  ├── daySCurveActivities  Map<DateTime, {actId → {s,b,d}}>  — por atividade
  └── lstActivityModels    List<ActivityModel>
              │
     ┌────────┴─────────┐
     ▼                  ▼
BaselineController    ScurveController
(filtros/pilha/edição) (gráficos/CFF)
```

### Modelo de dados — chaves do `daySCurve`

Cada entrada de `daySCurve[DateTime]` possui três valores fracionários (0.0 a 1.0):

| Chave | Série | Significado |
|-------|-------|------------|
| `"b"` | **Base** (baseline) | Progresso acumulado esperado segundo a baseline imutável |
| `"s"` | **Previsto** (scheduled) | Progresso acumulado previsto pelo cronograma atual (reprogramável) |
| `"d"` | **Realizado** (done) | Progresso real acumulado a partir das medições físicas; pode ser `null` para datas futuras |

---

## 2.1 Atividades — Tabela do Cronograma

**Widget:** `ActivityTableScreen` + `ActivityEditDataSource` (SfDataGrid/Syncfusion)

### Colunas configuráveis

| Coluna | Editável | Descrição |
|--------|---------|-----------|
| Serviço | Admins | Nome do serviço / pacote de trabalho |
| Lote | Admins | Lote, pavimento ou setor da obra |
| Início | Admins | Data de início planejada |
| Término | Admins | Data de término planejada |
| Duração | — | Calculada em dias úteis (respeita feriados da obra) |
| Custo | Admins | Custo orçado da atividade |
| Peso | — | Participação percentual no custo total |
| % Base | — | Progresso esperado pela baseline na `activeDate` |
| % Previsto | — | Progresso previsto pelo cronograma atual na `activeDate` |
| % Realizado | — | Progresso real registrado em medições |
| Crítica | — | `isCritical`: pertencimento ao caminho crítico |
| Fornecedor | — | Empresa contratada; carregada sob demanda via Prevision |

**Cabeçalhos agrupados (Stacked Headers):** as colunas visíveis são agrupadas visualmente em faixas por categoria — Identificação / Custo / Baseline / Previsto / Realizado / Qualidade.

### Filtros rápidos

| Filtro | Flag no controlador |
|--------|-------------------|
| Ocultar concluídas (≥ 100%) | `showFinishedActivities` |
| Somente mês corrente | — |
| Somente semana corrente | — |
| Baseline > Realizado | `filterBaselineGtRealizado` |
| Previsto > Realizado | `filterPrevistoGtRealizado` |
| Somente caminho crítico | `filterCriticalOnly` |

### Busca e debounce

A busca de atividades usa `_activitySearchDebounceTimer` com janela de **500 ms** antes de aplicar o filtro, evitando reconstruções excessivas do SfDataGrid durante digitação.

### Pilha de desfazer

```dart
maxUndoSteps = 15  // snapshots máximos
saveUndoSnapshot() → undoStack.add(deepCopy(lstActivityModels))
undo()             → undoStack.removeLast() → restaura snapshot anterior
```

O botão de desfazer aparece na AppBar com cor laranja **apenas quando há alterações pendentes** (`hasSomethingChanged == true`).

### Salvamento em nuvem

`saveScheduleToFirebase()` persiste atomicamente no Firestore:
- Custos e datas das atividades modificadas
- Nome do serviço, lote e cronograma
- Feriados customizados da obra

**Salvar Como:** `saveScheduleAs()` cria uma cópia do cronograma com novo nome, preservando o original intacto.

### Exportação

| Formato | Biblioteca |
|---------|-----------|
| Excel (`.xlsx`) | `syncfusion_flutter_datagrid_export` / `syncfusion_flutter_xlsio` |
| PDF | `printing` + `pdf` |

---

## 2.2 Macaquinho — Visão Matricial

**Acionamento:** rota que passa `mode: 'monkey'` ao `BaselineController`. O mesmo controller serve as três views (tabela, macaquinho, Gantt); o `mode` determina qual `View` é renderizada.

### Estrutura da grade

```
         Serv. A    Serv. B    Serv. C   ...
Lote 01 [MonkeyTile][MonkeyTile][MonkeyTile]
Lote 02 [MonkeyTile][     —    ][MonkeyTile]
Lote 03 [     —    ][MonkeyTile][MonkeyTile]
```

Cada célula é um `MonkeyTile` exibindo:

```
┌──────────────────────────────────┐
│  Serviço — Lote                  │
│                                  │
│  Base:    ##.#%  (laranja escuro) │
│  Previsto: ##.#% (preto)         │
│  Realizado: ##.#% (azul)        │
│                                  │
│  [barra de progresso colorida]   │
│  Início → Término                │
└──────────────────────────────────┘
```

### Cor de fundo do tile

| Cor | Condição |
|-----|---------|
| Verde | `realizado ≥ 95%` do previsto |
| Azul claro | `85% ≤ realizado < 95%` do previsto |
| Amarelo | `70% ≤ realizado < 85%` do previsto |
| Vermelho | `realizado < 70%` do previsto OU atividade atrasada |
| Roxo / outline | `isCritical == true` (caminho crítico) |
| Cinza | Sem dados de progresso |

### Modos de Heatmap

Sobreposição opcional que substitui a cor de status por uma paleta de concentração temporal:

| Modo | Algoritmo de cor |
|------|----------------|
| **Mensal** | Hue por mês ISO com stride-7 — cada mês tem matiz bem distinto |
| **Semanal** | Gradiente 0°→300° distribuído pelas 52 semanas do ano |
| **Proximidade** | Quente (vermelho) = próximo de `activeDate`; frio (azul) = distante |

### Filtros e controles

| Controle | Variável | Detalhe |
|----------|---------|---------|
| Seletor de data | `activeDate` | Recalcula `expectedAtDate` para todos os tiles sem recarregar do servidor |
| Zoom de célula | `monkeyCellSize` | Slider de 20 a 240 px; escala fontes e barras proporcionalmente |
| Filtro de lotes | `selectedLots` | Multi-select com agrupamento por tags favoritas |
| Filtro de serviços | `serviceSearchQuery` | Busca textual debounced; exibe contador de selecionados |
| Ocultar finalizados | `showServicesWithAllActivitiesDone` | Remove serviços onde todos os lotes estão a 100% |
| Caminho crítico | `filterCriticalOnly` | Restringe a grade apenas às atividades `isCritical = true` |

### Ações por célula (menu de contexto)

Ao pressionar / clicar com botão direito sobre um `MonkeyTile`:

| Ação | Integração |
|------|-----------|
| Abrir / criar Restrição | `restrictionDialog` via `MidTermController` |
| Listar Restrições vinculadas | Bottom sheet com filtro por `activityId` |
| Abrir / criar Plano de Ação | `showActionPlanDialog` via `ActionPlanController` |
| Listar Planos de Ação | Bottom sheet |
| Status de inspeção | `_showInspectionStatusMenuAtPosition` via módulo FVS |

### KPIs

- **Global:** % realizado × previsto × baseline ponderados por custo de todas as atividades visíveis
- **Por serviço:** tile de estatísticas ao lado do cabeçalho de cada serviço com médias B/P/R para aquele serviço em todos os lotes visíveis
- **Painel mobile:** ícone na AppBar expande bottom sheet com KPIs detalhados

### Exportação

| Formato | Função | Conteúdo |
|---------|--------|---------|
| PDF | `printMonkey()` | Grid completo com cores replicando os tiles |
| Excel | `exportMonkeyToExcel()` | Matriz lote×serviço com valores B/P/R e fundo colorido |

---

## 2.3 Gantt Reprogramável

**Widgets:** `BaselineGantt` (scaffold) + `GanttChart` (CustomPainter)

As barras do Gantt são desenhadas diretamente em canvas via `CustomPainter` para máxima performance. São sobrepostos até três cronogramas (base, previsto, realizado).

### Camadas do canvas

| Painter | Responsabilidade |
|---------|----------------|
| `GanttGridPainter` | Grade de datas, rótulos do eixo X e linhas de hoje / `activeDate` |
| `GanttActivityPainter` | Barras de cada atividade, linhas de dependência e overlay de caminho crítico |

### Reprogramação por drag & drop

```
[usuário arrasta barra da atividade A]
  → tempDragStartAt / tempDragEndAt atualizados em tempo real no canvas
  → dragOffsetX armazena deslocamento em pixels
  → ao soltar: duração original mantida, datas deslocadas
  → saveUndoSnapshot() salva estado anterior (max 15 passos)
  → applyFilters() + refreshChart() atualizam a view
```

- A duração é **preservada** no arrasto (move sem esticar)
- Dependências TI / II / TT / IT com lag configurável são verificadas: o sistema alerta se a nova data viola um predecessor
- O cálculo usa `addWorkingDays()` e `nextWorkingDay()` respeitando feriados da obra

### Controles da pilha de desfazer / salvar

| Controle | Detalhe |
|----------|---------|
| Desfazer | Restaura snapshot completo; ícone laranja aparece só com alterações pendentes |
| Salvar em nuvem | `saveScheduleToFirebase()` — persiste toda a lista de atividades no Firestore |
| Salvar Como | Cria cópia com novo nome via `saveScheduleAs()` |
| Travar edições | `isEditLocked` — ícone de cadeado; roxo = travado, bloqueia qualquer drag |

### Caminho crítico

| Origem | Comportamento |
|--------|--------------|
| Prevision | `isCritical` já calculado pela API; barras críticas em vermelho sem recalcular localmente |
| Cronograma local | Botão de triângulo dispara `calculateCriticalPath()` (algoritmo CPM); barras críticas destacadas após o cálculo |

Toggle na AppBar mostra / oculta o overlay de caminho crítico independentemente da fonte.

### Filtros disponíveis

| Filtro | Variável | Detalhe |
|--------|---------|---------|
| Por serviço | `activeServiceIndex` | Paginação com setas ← →; exibe um serviço por vez |
| Por lote | `selectedLots` | Multi-select com tags favoritas |
| Intervalo de datas | `filterStartDate` / `filterEndDate` | `DateRangePicker`; restringe o canvas ao período |
| Ocultar concluídas | `showFinishedActivities` | Remove atividades a 100% |
| Caminho crítico | `filterCriticalOnly` | Exibe só `isCritical = true` |
| Tags favoritas de lotes | — | Dropdown na AppBar; muda lotes + serviços em um clique |

### Navegação temporal

- **Scroll para hoje:** botão na AppBar centraliza o canvas na data corrente
- **Scroll para término previsto:** toque duplo no campo de término na AppBar rola até a última barra
- **Data de término previsto** exibida em tempo real na AppBar; atualizada quando barras são arrastadas

---

## 2.4 Curva S — Módulo de Análise (`ScurveView`)

**Controlador:** `ScurveController`

O `ScurveView` é organizado em **quatro abas** acessíveis por ícones na AppBar:

| # | Ícone | Aba |
|---|-------|-----|
| 0 | `show_chart` | Curva S Acumulada |
| 1 | `bar_chart` | Gráfico Mensal de Deltas |
| 2 | `home_repair_service_rounded` | Análise de Serviços |
| 3 | `attach_money` | CFF |

### Inicialização (`ScurveController.onInit`)

```dart
if (activeBaselineSchedule == null) {
  await previsionController.activateBaselineSchedule(siteId)
  await schedule.calculateSCurve(activeBaselineSchedule)  // preenche "b"
  previsionController.applySCurveScheduledFlagAdjustment(schedule)
}
// Sempre, independente de cache:
previsionController.applySCurveBaselineFlagAdjustment(schedule, baseline)
```

O `applySCurveBaselineFlagAdjustment` garante que a série `"b"` não ultrapasse `"d"` em datas passadas onde a baseline seria teoricamente maior que o realizado — evitando gráficos confusos.

---

### 2.4.1 Aba 0 — Curva S Acumulada

**Widget:** `SCurveChart` (`SfCartesianChart`)

#### Séries do gráfico

| Série | React. | Cor |
|-------|--------|-----|
| **Base** (`"b"`) | Baseline imutável | Laranja escuro (`Colors.orange[800]`) |
| **Previsto** (`"s"`) | Cronograma atual | Preto (`Colors.black`) |
| **Realizado** (`"d"`) | Progresso real | Azul (`Colors.blueAccent[700]`); série `nullable` — pode ter lacunas |

#### Comportamentos

- **Eixo X:** `DateTimeAxis` cobrindo o ciclo completo da obra
- **Eixo Y:** `NumericAxis` 0–100% com `labelFormat: '{value}%'`
- **Trackball** em `groupAllPoints`: toque exibe tooltip com os três valores simultâneos na mesma data
- **Zoom / pan** horizontal (`ZoomMode.x`); botão `zoom_out_map` na barra roxa para reset
- **`lastValidDone`:** armazena o último valor não-nulo de `"d"` para exibição correta no tooltip mesmo em datas sem dado novo

#### Tratamento da série `"d"` (realizado)

```dart
double? doneValue = (done != null && done is double)
    ? (done * 100)
    : (done != null && done is int ? done.toDouble() * 100 : null);
```

Strings vazias (`""`) são convertidas para `null`, tornando a série naturalmente `nullable` e evitando pontos zerados indevidos após a última medição.

#### Exportação

Ícone de download na barra roxa chama `schedule.exportSCurveAsCSV()` — gera um arquivo CSV com as três séries diárias e permite download direto no web ou compartilhamento no mobile.

---

## 2.5 Gráfico Mensal de Deltas

**Widget:** `MonthlyDeltaChart` (`SfCartesianChart`)

Exibe o incremento mensal de cada série (delta em vez de acumulado), facilitando a comparação de ritmo mês a mês.

### Cálculo dos deltas mensais (`getMonthlyDeltas`)

```
Para cada mês M:
  delta_s(M) = max(s) do mês M  −  max(s) do mês M−1
  delta_b(M) = max(b) do mês M  −  max(b) do mês M−1
  delta_d(M) = max(d) do mês M  −  max(d) do mês M−1
```

O `max` é usado porque `daySCurve` é acumulado — o valor mais alto do mês representa o acumulado ao final daquele mês.

### Drill-down por serviço

Ao tocar em uma barra de mês, o gráfico de colunas alterna para um **detalhe por serviço** daquele mês:

```dart
_switchToServiceChart(selectedDate)
  → lê daySCurveActivities filtrado pelo mês
  → agrega por serviceName usando activityIdToServiceName
  → ordena por previsto decrescente
  → exibe SfCartesianChart com barras B/P/R por serviço
```

Os pesos de custo (`budgetCost / totalBudget`) são aplicados para converter percentuais unitários em contribuição ponderada do serviço.

### Controles

| Controle | Variável | Detalhe |
|----------|---------|---------|
| Rótulos sobre barras | `areLabelsVisible` | Toggle que alterna `DataLabelSettings.isVisible` |
| Zoom / pan | `_zoomPanBehavior` | `ZoomMode.x`; botão de reset na barra superior |
| Bottom sheet de serviços | `isBottomSheetVisible` | `ScurveController.showMonthlyServicesDetails()` — exibe também via bottom sheet ao tocar no gráfico acumulado |

---

## 2.6 Análise de Serviços — Grade Mensal

**Widget:** `ServiceMonthlyGrid` (`SfDataGrid`)

Grade tabular que cruza **serviços (linhas)** com **meses (colunas)**, mostrando o progresso previsto incremental de cada serviço por período. Expandindo uma linha de serviço, exibe as atividades individuais que o compõem.

### Pré-computação (`_precompute`)

Ao inicializar, o widget constrói três estruturas em memória:

| Estrutura | Descrição |
|-----------|-----------|
| `_serviceMonthMap` | `service → 'yyyy-MM' → fração prevista` (soma de `s` de todas as atividades do serviço no mês) |
| `_activityMonthMap` | `service → actId → 'yyyy-MM' → fração` (breakdown por atividade individual) |
| `_totalPerMonth` | `'yyyy-MM' → delta total` (derivado de `daySCurveK`, mesma lógica do `getMonthlyDeltas`) |

### Filtros de período

| Filtro | Variável | Detalhe |
|--------|---------|---------|
| Ocultar meses passados | `_hidePastMonths` | Padrão `true`; remove meses anteriores a hoje |
| Próximos N meses | `_nextMonthsFilter` | 0 = todos; 1–12 = próximos N meses a partir de hoje |
| Range customizado | `_customRange` | `DateRangePicker` para seleção livre de período |
| Meses visíveis (global) | `lstVisibleMonths` | `RxList<DateTime>` compartilhada com o `CFFGrid` para sincronizar seleção de período entre abas |

### Expansão de serviços

Ao tocar no nome de um serviço, a linha se expande exibindo as atividades do serviço com seus dados de `inicio`, `termino` e percentual previsto mensal individual. O `_activityById` fornece lookup O(1) para enriquecer cada linha de atividade com metadados do `ActivityModel`.

---

## 2.7 CFF — Cronograma Físico-Financeiro

**Widget:** `CFFGrid` (`SfDataGrid`)

O CFF cruza **itens de orçamento WBS (linhas)** com **períodos mensais (colunas)**, mostrando a distribuição temporal dos custos planejados (Base / Previsto) e realizados.

### Estrutura da tabela

```
Colunas fixas:
  Código WBS | Descrição | Custo Material | Custo MO | Total

Colunas dinâmicas (uma por mês visível):
  jan/26 | fev/26 | mar/26 | ... | Total
```

Cada célula mensal exibe o valor em **R$** ou **%** (toggle `showPercent`) e pode ser apresentada em modo **incremental** ou **acumulado** (toggle `showAcc`).

### Modos de visualização

| Modo (`cffViewMode`) | Mapa usado | Descrição |
|---------------------|-----------|-----------|
| 0 — **Base** | `itemMonthMapBase` | Distribuição conforme baseline imutável |
| 1 — **Previsto** (padrão) | `itemMonthMapPrevisto` | Distribuição conforme cronograma atual |
| 2 — **Realizado** | `itemMonthMapRealizado` | Distribuição dos custos efetivamente incorridos |

### Hierarquia WBS e expansão

`CFFDataSource.toggleExpand(code)` alterna a expansão de uma linha WBS pai, exibindo os subitens filhos. O código WBS é usado como chave do mapa hierárquico — ex. `"01"` expande `["01.01", "01.02", "01.03"]`.

### Cache no Firestore

```
Path: {client}/default_sites/{siteId}/budgets/{budgetId}/cff/
```

| Campo | Descrição |
|-------|-----------|
| `cffVersion` | Incrementado a cada recálculo; `CFFGrid` se reconstrói quando a versão muda |
| `cffRefDate` | Data de corte usada no cálculo; exibida no tooltip do botão "Recalcular" |
| `_cffChunkSize` | `150` — chunks usados no batch write para respeitar limite do Firestore |

**Fluxo de recálculo:**

```dart
calculateAndSaveCff()
  → calculateCFFAsync()  // cálculo async em isolate
  → persiste em batches de 150 itens no Firestore
  → incrementa cffVersion  → CFFGrid reconstrói automaticamente
```

**Carregamento do cache:**

```dart
_loadCffCache()  // onInit do ScurveController
  // se cffVersion > 0: exibe imediatamente sem recalcular
```

### Filtro de meses visíveis

`lstVisibleMonths` é um `RxList<DateTime>` **compartilhado** entre `ServiceMonthlyGrid` e `CFFGrid`. Alterar a seleção de meses em uma aba reflete imediatamente na outra.

---

## 2.8 Vinculação de Orçamento — Budget Weights

Liga cada item do orçamento WBS a uma ou mais atividades do cronograma com percentuais de peso, permitindo o cálculo do CFF e a projeção do progresso físico-financeiro.

### Modelo

```
BudgetItem "01.03 Alvenaria" (R$ 300 k)
  └── BudgetPlanBinding[]
        ├── activityId: "act_042"  percentage: 40%  → R$ 120 k
        └── activityId: "act_091"  percentage: 60%  → R$ 180 k
```

**Classe:** `BudgetPlanBinding` (campo `budgetWeights` em `BudgetItem`)

**Origem dos dados:** importado do Prevision via `budgetItemsPage` com `budgetWeights { percentage, activity { id } }`.

### Impacto nos cálculos

- `item.monthlyCFF[month]` = soma de `activity.expectedAtMonth * binding.percentage * item.totalCost` para todas as bindings do item
- `item.monthlyBaselineCFF[month]` = idem usando `activity.baselineExpectedAtMonth`
- `item.monthlyRealizedCFF[month]` = idem usando `activity.realizedAtMonth`

---

## 2.9 Gestão de Cronogramas

**View:** `PhisycalView` | **Controlador:** `PhisycalController`

Gerencia o ciclo de vida completo dos cronogramas da obra: criação, seleção do baseline de referência, importação de novas versões e histórico de revisões.

### Fontes de importação

| Fonte | Formato | Fluxo |
|-------|---------|-------|
| **Prevision** | API GraphQL | Sincronização automática pelo `previsionProjectId` da obra; sem ação manual |
| **Excel** | `.xlsx` | Upload → Cloud Function `excelToJson` → mapeamento manual de colunas (ID, serviço, lote, início, fim, custo, % concluído) |
| **JSON** | `.json` | Importação direta no formato interno do Kaizen (ex.: exportação de outra obra) |

### Ciclo de vida de um cronograma

```
Rascunho / Importado
       ↓
  [Definir como Baseline]   ← baseline imutável de referência
       ↓
  Ativo (reprogramável)     ← drag & drop no Gantt; edições na tabela
       ↓
  [Salvar Como]             → nova versão (mantém histórico)
       ↓
  Arquivado                 ← versões anteriores consultáveis
```

### Campos de um cronograma

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `name` | String | Nome da versão (ex.: "Rev. 3 — Atualização Março") |
| `createdAt` | DateTime | Timestamp de criação (server time) |
| `createdBy` | String | Nome / ID do usuário que criou |
| `scheduleSource` | String | `'prevision'`, `'excel'` ou `'json'` |
| `previsionProjectId` | String | ID do projeto no Prevision (se origem = Prevision) |
| `isBaseline` | bool | `true` para o cronograma de baseline imutável |
| `lstActivityModels` | List | Lista completa de atividades com todos os campos |
| `daySCurve` | Map | Série diária acumulada (B/P/D) para os gráficos |
| `daySCurveActivities` | Map | Série diária por atividade (B/P/D) para drill-down |

### Seleção do baseline

Ao definir um cronograma como baseline, o `OutterPrevisionController` o carrega em `activeBaselineSchedule`. Todos os gráficos e indicadores usam essa referência para calcular a série `"b"`. O baseline é imutável — edições no cronograma ativo não o afetam.

---

## Relatórios Exportáveis

| Relatório | Formato | Fonte |
|-----------|---------|-------|
| Atividades completas | Excel / PDF | Tabela de atividades do `BaselineController` |
| Curva S do projeto | Gráfico PDF | `SCurveChart` via `schedule.exportSCurveAsCSV()` |
| CFF por período | Excel / PDF | `CFFGrid` via `SfDataGrid` export |
| Análise de desvios | Excel | Desvios por rubrica categorizados (ORC, CST, PRJ, ENG, PRZ) |
| Macaquinho | PDF / Excel | `printMonkey()` / `exportMonkeyToExcel()` |

---

## Considerações de Performance

| Técnica | Onde | Detalhe |
|---------|------|---------|
| Canvas CustomPainter | Gantt | Evita rebuild de widgets por barra; renderiza centenas de atividades sem lag |
| Lazy loading Prevision | Tabela de atividades | `Fornecedor` carregado sob demanda, não no load inicial |
| Chunks no Firestore | CFF save | Máx. 150 docs por batch para respeitar limite de 500 ops/commit |
| CFF cache | ScurveController onInit | Carrega do Firestore sem recalcular se `cffVersion > 0` |
| Debounce 500 ms | Buscas | `_activitySearchDebounceTimer`, `_searchDebounceTimer` |
| `daySCurveActivities` separado | Drill-down | Evita processar o mapa completo ao navegar na Curva S acumulada |

---

[↑ Módulos de Gestão de Obra](../02_modulos_gestao_obra)
