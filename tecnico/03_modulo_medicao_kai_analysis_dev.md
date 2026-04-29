---
layout: page
title: "Medição e Análise KAI"
permalink: /tecnico/medicao/
category: "Módulos"
order: 3
toc: true
---

# Medição e Análise KAI

O **Módulo de Medição** é o coração do controle de progresso no Kaizen. Permite registrar semanalmente o avanço físico de todas as atividades do cronograma, comparar contra o planejado e identificar problemas de forma rápida.

A **Análise KAI** aplica inteligência artificial sobre os dados de medição para:
- ✅ Destacar atividades críticas que estão atrasadas
- ✅ Sugerir planos de ação automaticamente
- ✅ Alertar o gestor sobre riscos ao cronograma
- ✅ Identificar padrões de atraso por tipo de problema

**Acesso:** Privilégio `physical` para medições | `daily_measurements` para planejamento diário

---

## 1. Como Usar a Medição

```
[Abrir medição para data X (ex: 28/04/2026)]
         ↓
[Preencher % realizado de cada atividade]
         ↓
[Adicionar comentários e problemas encontrados]
         ↓
[IA detecta sugestões automáticas]
         ↓
[Salvar medição]
         ↓
[KAI Analysis mostra: PPCC, atividades críticas, alertas]
```

---

## 2. Tela Principal

### 2.1 Escolher Data

No topo, escolha a **data de referência** da medição (ex: 28/04/2026 - final de semana):

- Calendário interativo
- Exibição automática de:
  - Data da medição anterior (para comparação)
  - Cronograma vigente
  - Quem vai registrar (seu usuário)

### 2.2 Observações Gerais

Campo de texto livre para registrar contexto da semana:

- Condições climáticas, paralisações
- Mudanças de escopo ou reprogramações
- Alertas externos que afetaram a execução
- Notas para comunicação aos stakeholders

---

## 3. Abas de Análise

A tela de medição é organizada em **4 abas** acessíveis via ícones/tabs na barra inferior ou no AppBar:

| # | Aba | Descrição | Privilégio |
|---|-----|-----------|-----------|
| 0 | **Tabela de Medição** | Grid com entrada de dados de avanço por atividade | `physical` |
| 1 | **Análise de Problemas** | Dashboard dos problemas registrados (contagem, tipo, severidade) | `physical` |
| 2 | **Planos de Ação** | Lista de planos vinculados e criação de novos | `action_plans_view` |
| 3 | **KAI** | Análise automatizada: caminho crítico, PPCC, sugestões de ação | `physical` |

---

## 4. Aba 0 — Tabela de Medição

### 4.1 Estrutura

Grade com todas as atividades do cronograma. Colunas principais:

```
Lote | Serviço | Atividade | % Acum. | Realizado | Comentário | Problemas | [meses]...
─────┼─────────┼───────────┼─────────┼───────────┼───────────┼──────────┼─────────────
1    | Funda   | Escavação | 45.2%   | [entrada] | [entrada] | [seleção]| [entrada]
1    | Funda   | Formas    | 23.1%   | [entrada] | [entrada] | [seleção]| [entrada]
...
```

#### Colunas Fixas (Esquerda)

| Coluna | O que é |
|--------|---------|
| **Lote** | Identificação do lote/pavimento |
| **Serviço** | Tipo de serviço (alvenaria, estrutura, etc.) |
| **Atividade** | Nome da atividade |
| **% Acum.** | Percentual total realizado até agora |
| **Realizado** | Campo para digitar % realizado **nesta semana** |
| **Comentário** | Observações técnicas (qualidade, dificuldades) |
| **Problemas** | Checkbox com categorias pré-definidas (Qualidade, Mão de Obra, Materiais, etc.) |

#### Colunas por Mês

Cada mês do cronograma aparece com subcoluna "Plan." (o que foi previsto) e "Real." (o que foi realizado):

| Cor da Flag | Significado |
|---------|------------|
| 🟢 Verde | Período cumprido ou adiantado |
| 🟡 Amarelo | Perto da meta (95–99%) |
| 🔴 Vermelho | Abaixo da meta |
| ⚪ Cinza | Sem dado ou não teve movimento |

### 4.2 Como Preencher

1. **Digite o % realizado** na coluna "Realizado" para cada atividade
2. **Adicione comentário** se houver dificuldades ou observações
3. **Marque problemas** encontrados (seleção múltipla)
4. **Salve** → Sistema sincroniza automaticamente

A **IA detecta automaticamente** qual é o tipo de problema baseado no seu comentário. Você confirma ou corrige se necessário.

### 4.3 Funciona Sem Internet?

Sim! Suas edições ficam salvas localmente e sincronizam quando reconectar. Um ícone no topo avisa se está offline.

---

## 5. Aba 1 — Análise de Problemas

Dashboard consolidado dos problemas registrados nesta medição.

### 5.1 Contagem por Categoria

Card supremo mostrando:

```
┌──────────────────────────────────────┐
│  Problemas Identificados             │
│                                      │
│  Qualidade          15  (45%)        │
│  Mão de Obra        8   (24%)        │
│  Materiais          6   (18%)        │
│  Método             4   (12%)        │
│  (outros)           0   (0%)         │
└──────────────────────────────────────┘
```

### 5.2 Atividades Mais Afetadas

Tabela ordenada por número de problemas registrados:

| Atividade | Serviço | Qtd Problemas | Categorias |
|-----------|---------|---------------|-----------|
| Alvenaria A | Alvenaria | 4 | Qualidade, Mão de Obra |
| Forma B | Estrutura | 3 | Materiais |
| Reboco | Acabamento | 2 | Qualidade |

### 5.3 Gráfico de Distribuição

Gráfico de barras horizontais com as categorias de problema, permitindo visão rápida do tipo de impedimento mais frequente. Útil para identificar padrões e priorizar ações corretivas.

---

## 6. Aba 2 — Planos de Ação

### 6.1 Planos Vinculados a Esta Medição

Lista de planos de ação já associados a atividades desta medição:

- Título e descrição
- Responsável e status (`aberta / em andamento / concluída / vencida`)
- Data de vencimento e impacto (atividades afetadas)
- Botões: editar, visualizar histórico, excluir vínculo

### 6.2 Criar Novo Plano

Botão `+` abre formulário rápido para criar um plano vinculado a uma ou mais atividades desta medição:

- Campo de título obrigatório
- Descrição e setor responsável
- Data de comprometimento
- Prioridade (`baixa / média / alta`)

O plano é criado, persistido no `ActionPlanController` e salvo em Firestore.

---

## 7. Aba 3 — KAI Analysis Tab

### 7.1 Visão Geral

A **aba KAI** é o coração da análise automática. Apresenta um resumo executivo do desempenho da obra e análise detalhada das atividades críticas em risco.

Estruturada em **10 seções** (scrollable):

```
[Seção 0] Resumo Executivo
[Seção 1] Índice de Produtividade
[Seção 2] Análise de Atrasos
[Seção 3] Comparativo com Baseline
[Seção 4] Variação Mensal
[Seção 5] Análise por Setor
[Seção 6] Riscos de Viabilidade
[Seção 7] Análise Temporal (Gantt analítico)
[Seção 8] Sugestões KAI Automáticas
[Seção 9] Análise de Restrições (6M)
[Seção 10] Análise de Caminho Crítico
```

---

### 7.2 Seção 10 — Análise de Caminho Crítico (Detalhe Completo)

#### 7.2.1 Propósito

Identifica as **atividades que impactam o prazo da obra** e monitora seu desempenho com métricas específicas:

- **Atividades críticas medidas:** atividades no caminho crítico que já possuem dados de medição (grupo 1)
- **Atividades críticas futuras:** próximas atividades críticas ainda não iniciadas (grupo 2)

Fornece ao gestor visibilidade clara sobre **qual é o risco real de atraso** do cronograma.

#### 7.2.2 KPIs Principais

Exibidos em card no topo da seção:

| KPI | Fórmula | Significado |
|-----|---------|-------------|
| **PPCC** | % atividades críticas com `realizado ≥ planejado (gap)` | Confiabilidade do cronograma crítico |
| **Atividades em dia** | Contagem de críticas com `done ≥ expected` | Saúde do caminho crítico |
| **Atividades atrasadas** | Contagem de críticas com `done < expected` | Número de bloqueadores |
| **Atraso médio vs baseline** | Média de dias entre `activeEnd` e `baselineEnd` | Tendência geral de slippage |
| **Atividades tardias (vs baseline)** | Contagem com `activeEnd > baselineEnd` | Atividades especificamente atrasadas vs baseline |

#### 7.2.3 PPCC — Percent Plan Complete do Caminho Crítico

**Definição:**

O PPCC mede a **confiabilidade de cumprimento das metas** das atividades críticas usando a **fórmula gap-based**:

$$PPCC = \frac{\sum \min(doneI_i,\ expectedIGap_i)}{\sum expectedIGap_i}$$

Onde:
- `expectedIGap = max(0, expected - prevDone)`: quanto a atividade precisa avançar neste período para atingir a meta, partindo de onde estava de fato
- `doneI = done - prevDone`: quanto a atividade avançou neste período
- Atividades sem gap (já adiantadas) não entram no cálculo

**Intuição:**

Em vez de medir "quanto foi feito do cronograma", PPCC mede "das tarefas que nos comprometemos a executar neste período, quantas foram cumpridas?" — metodologia alinhada com **Last Planner System (LPS)**.

**Exemplo prático:**

Suponha 3 atividades críticas medidas nesta semana:

| Atividade | prevDone | done | expected | expectedIGap | doneI | OK? |
|-----------|----------|------|----------|--------------|-------|-----|
| A         | 85%      | 89%  | 95.56%   | 10.56 pp     | 4 pp  | ❌  |
| B         | 50%      | 65%  | 70%      | 20 pp        | 15 pp | ❌  |
| C         | 40%      | 60%  | 50%      | 10 pp        | 20 pp | ✅  |

$$PPCC = \frac{4 + 15 + 10}{10.56 + 20 + 10} = \frac{29}{40.56} \approx 71.5\%$$

**Interpretação:** 71.5% das metas semanais do caminho crítico foram cumpridas.

#### 7.2.4 Cálculo On-The-Fly (Não Armazenado)

O PPCC é **computado em tempo real** no KAI Analysis Tab (não armazenado em `m.ppcc`) para garantir que reflete sempre o estado atual de `lstActivitiesInvolved`:

```dart
double _ppccTot = 0, _ppccMea = 0;
for (final ad in involvedCritical) {
  final gap = ad.expectedIGap.toDouble();
  if (gap <= 0) continue;  // Atividade já adiantada
  _ppccTot += gap;
  final di = ad.doneI.toDouble();
  _ppccMea += di > gap ? gap : di;  // Contribuição capped at target
}
final ppccPct = _ppccTot == 0 ? 0.0 : (_ppccMea / _ppccTot * 100);
```

**Por que não armazenar:** `m.ppcc` (campo salvo no Firestore) pode estar desatualizado se a medição foi criada sem dados ou sincronizada de outra fonte. Recalcular garante consistência.

#### 7.2.5 Renderização: Card Principal

**Cabeçalho com KPIs:**

```
┌──────────────────────────────────────────────────┐
│  PPCC: 71.5%  │  Atividades em dia: 1 de 3      │
│  Atrasos: 2   │  Atraso médio: +2 dias          │
└──────────────────────────────────────────────────┘
```

Cores codificadas por `flagColor()`:
- 🟢 Verde: PPCC ≥ 85%
- 🟡 Amarelo: 70% ≤ PPCC < 85%
- 🔴 Vermelho: PPCC < 70%

#### 7.2.6 Grupo 1 — Atividades que Impactaram Esta Medição

**Critérios para inclusão:**

Uma atividade é incluída se:

```
(ad.wasCritical == true) OR (ad.activtyModel.isCritical == true)
```

Onde:
- `wasCritical`: flag explícito salvo em `m.criticalActivities[]` quando a medição foi criada
- `isCritical`: propriedade do modelo de atividade do cronograma ativo (já calculado pelo Prevision ou localmente via CPM)

**Renderização: Tile por Atividade**

Cada atividade crítica é renderizada em um **tile compacto** com:

```
┌─────────────────────────────────────────────────┐
│ ✓ Atividade X — Lote Y                [Em dia]  │
│                                                 │
│ Previsto (acum.): ████████░ 95.56%              │
│ Realizado (acum.): ███████░ 89.00%              │
│                       [4.56 pp abaixo do previsto]
│                                                 │
│ No período (Δ)                                  │
│ Plan. Δ: 10.56%  │  Real. Δ: 4.00%              │
│                                                 │
│ Término planejado: 28/04/2026 (em 5d)           │
│ Fim baseline:      26/04/2026                    │
│ Atraso vs baseline: +2d                         │
└─────────────────────────────────────────────────┘
```

**Componentes do tile:**

| Componente | Descrição |
|-----------|-----------|
| **Ícone de status** | ✓ (verde) = em dia; ⚠ (vermelho) = atrasada; ⏱ (cinza) = não iniciada |
| **Nome + Lote** | Identificação da atividade |
| **Status badge** | "Em dia" / "Atrasada" / "Não iniciada" |
| **Barras de progresso (acumulado)** | Dois stacked bars: Previsto (cinza) e Realizado (verde/vermelho) em escala 0–100% |
| **Gap chip** | Vermelho com texto "X pp abaixo do previsto" se não está ok |
| **Seção "No período (Δ)"** | Valores gap-based mostrados em chips separados |
| **Datas** | Término planejado (cronograma ativo) e fim baseline |
| **Atraso vs baseline** | "+Xd" ou "-Xd" mostrando desvio em dias; oculto se não houver baseline |

**Tooltip ao passar sobre o PPCC:**

```
"PPCC = PPC do Caminho Crítico. Calculado sobre os valores 
incrementais (Δ) do período: quanto foi planejado fazer 
SÓ NESTE MÊS vs quanto foi realizado SÓ NESTE MÊS nas 
atividades críticas. Veja "No período (Δ)" em cada atividade. 
Desvio em dias: término planejado (cronograma ativo) vs baseline."
```

#### 7.2.7 Grupo 2 — Próximas Atividades Críticas

**Critérios para inclusão:**

Atividades críticas (`isCritical == true` no cronograma ativo) que **não estão em `involvedCritical`** (não têm medição de realizado).

**Ordenação:** por data de início planejada (crescente). **Limite:** primeiras 3 exibidas, com toggle "Ver todas" para expandir.

**Renderização: Tile Compacto (não iniciadas)**

```
┌──────────────────────────────────────────┐
│ ⏱ Próxima Atividade Crítica — Lote Z    │
│ Serviço: Reboco                          │
│ Status: Não iniciada                     │
│                                          │
│ Início planejado: 02/05/2026 (em 3d)     │
│ Fim baseline:     12/05/2026              │
└──────────────────────────────────────────┘
```

Mais compacto que Grupo 1 já que não há dados de progresso ainda.

---

### 7.3 Tooltip de Esclarecimento

Um **ícone de info (?)** ao lado de "PPCC" abre tooltip explicando:

- Fórmula gap-based e por quê
- Diferença entre "Previsto (acum.)" e "No período (Δ)"
- Como interpretar os valores
- Relação com baseline

---

### 7.4 Dados de Entrada

**Fonte de dados:**

- `involvedCritical`: lista filtrada de `lstActivitiesInvolved` onde `wasCritical || isCritical`
- `activeScheduleForGrid`: cronograma ativo em Hive/previsionController
- `schedule`: cronograma linkado à medição (histórico para cálculos de delta)
- `m.refDate`: data de referência da medição

**Getters de atividade utilizados:**

```
ad.expected          // % acumulado previsto até refDate
ad.prevExpected      // % acumulado previsto até período anterior
ad.expectedI         // expected - prevExpected (delta cronograma)
ad.expectedIGap      // max(0, expected - prevDone) (gap-based)
ad.done              // % acumulado realizado
ad.prevDone          // % acumulado realizado até período anterior
ad.doneI             // done - prevDone (delta realizado)
ad.baselineEndAt     // data de término de baseline (se existe)
ad.activtyModel.endAt // data de término do modelo de atividade
```

---

### 7.5 Ordenação Padrão

Tanto Grupo 1 quanto Grupo 2 podem ser ordenados por:

1. **Status de progresso** (atrasadas primeiro)
2. **Atraso em dias** (máximo atraso primeiro)
3. **Data de início** (próximas primeiro)
4. **Prioridade** (crítica máxima risco primeiro)

Padrão exibido: **status + atraso decrescente**.

---

## 8. Integração com Outros Módulos

### 8.1 Planos de Ação

Ao identificar atividade crítica atrasada, o botão "Criar Plano de Ação" no tile abre o formulário pré-preenchido com:

- Serviço e atividade referência
- Descrição sugerida: "Recuperar atraso de X dias em [atividade]"
- Prioridade: `alta` (crítica)
- Data de comprometimento: data planejada + X dias

### 8.2 Restrições (6M)

Seção 9 do KAI (não detalhada aqui) lista restrições do modelo LPS que podem estar causando os atrasos críticos. Integra com `MidTermController` para sugerir categorias 6M baseado em problemas registrados.

### 8.3 Alertas e Notificações

Se PPCC < 70% na medição, o sistema:
1. Envia notificação push ao gestor: "⚠️ PPCC crítico: 65% — 2 atividades atrasadas"
2. Deep link abre direto na aba KAI seção 10
3. Cria sugestão de restrição automática

---

## 9. Processamento Offline e Sincronização

### 9.1 Cache Local (Hive)

- `lstActivitiesInvolved`: lista de `ActivityData` com estados locais
- Edições diretas às propriedades (realizado, comentários, problemas)

### 9.2 Flush ao Salvar

```
fieldUpdates['measurement.{activityId}'] = value
fieldUpdates['expected.{activityId}'] = value
fieldUpdates['comments.{activityId}'] = value
fieldUpdates['measurementProblems.{activityId}'] = []
fieldUpdates['ppcc'] = ppccRecalculado
fieldUpdates['lastUpdate'] = DateTime.now()
```

**Atomicidade:** Firestore `update()` garante que todas as mudanças são aplicadas ou nenhuma (transação implícita).

---

## 10. Exportação e Relatórios

### 10.1 PDF

Gera relatório formatado com:
- Tabela de medição com todas as colunas
- Gráficos: Curva S, distribuição de problemas
- Resumo KAI: PPCC, atrasos, restrições
- Observações gerais

Enviável diretamente via e-mail.

### 10.2 Excel

Grid completo da tabela de medição com formatação (cores de flag, larguras otimizadas).

---

## 11. Referências no Código

**Arquivos principais:**

- [`kai_analysis_tab_widget.dart`](../../lib/app/modules/obra/phisycal_analysis/widgets/kai_analysis_tab_widget.dart) — Interface KAI e seção 10
- [`phisycal_analysis_controller.dart`](../../lib/app/modules/obra/phisycal_analysis/controllers/phisycal_analysis_controller.dart) — Lógica de cálculo e getters de `ActivityData`
- [`physical_table.dart`](../../lib/app/modules/obra/phisycal_analysis/widgets/physical_table.dart) — Renderização da tabela de medição
- [`phisycal_controller.dart`](../../lib/app/modules/obra/phisycal/controllers/phisycal_controller.dart) — Persistência em Firestore

**Modelos:**

- [`activity_model.dart`](../../lib/app/data/models/activity_model.dart) — `ActivityModel` com `isCritical`, `expectedPercentageCompleted`, `percentageCompleted`
- [`physical_measurement_model.dart`](../../lib/app/data/models/physical_measurement_model.dart) — `PhysicalMeasurement` com mapas `measurement`, `expected`, `comments`

---

## 12. Fluxo Típico de Uso

1. **Engenheiro abre Medição** para data X
2. **Preenche avanços** na tabela (colunas mensais)
3. **Registra problemas** via dropdown ou comentário (IA detecta automaticamente)
4. **Salva medição** → Firestore persiste, cálculos rodados
5. **Abre aba KAI** → vê PPCC, atividades críticas, próximas em risco
6. **Identifica atraso crítico** → clica "Criar Plano de Ação"
7. **Cria plano** vinculado a atividade, com responsável e data
8. **Exporta PDF** → distribui para time

---

## 13. Dicas de Performance

- **Grid lazy:** apenas atividades com medição (subset do cronograma) são renderizadas
- **Cálculos incrementais:** PPCC, impactos e desvios recalculados apenas para dirty rows
- **Cache de expectedIGap:** computado uma única vez durante build de `ActivityData`
- **Paginação de meses:** colunas dinâmicas renderizadas sob demanda conforme scroll horizontal

