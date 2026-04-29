---
layout: page
title: "Medição e Análise KAI"
permalink: /tecnico/medicao/
category: "Módulos"
order: 3
toc: true
---

# Medição e Análise KAI

O **Módulo de Medição** é o coração do controle de progresso no Kaizen. Permite registrar semanalmente o avanço físico de todas as atividades do cronograma, comparar contra o planejado e identificar problemas.

A **Análise KAI** aplica inteligência artificial para:
- ✅ Destacar atividades críticas que estão em risco
- ✅ Sugerir planos de ação automaticamente
- ✅ Alertar sobre riscos ao cronograma
- ✅ Identificar padrões de atraso

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
- Cronograma vigente
- Data da medição anterior (para comparação)

### 2.2 Observações Gerais

Campo de texto livre para registrar contexto da semana:

- Condições climáticas, paralisações
- Mudanças de escopo
- Alertas externos que afetaram a execução
- Notas para stakeholders

---

## 3. Abas de Análise

A medição tem **4 abas**:

| Aba | Descrição | Uso |
|-----|-----------|-----|
| **0 — Tabela** | Grade com entrada de dados por atividade | Preencher % realizado, comentários, problemas |
| **1 — Problemas** | Dashboard dos problemas registrados | Ver contagem, categorias e atividades afetadas |
| **2 — Planos de Ação** | Listar e criar planos vinculados | Gerenciar ações para resolver atrasos |
| **3 — KAI** | Análise automática e caminho crítico | Ver PPCC, atividades críticas, alertas |

---

## 4. Aba 0 — Tabela de Medição

### 4.1 Estrutura

Grade com todas as atividades. Colunas principais:

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
| **Problemas** | Checkbox com categorias (Qualidade, Mão de Obra, Materiais, etc.) |

#### Colunas por Mês

Cada mês do cronograma aparece com "Plan." (previsto) e "Real." (realizado):

| Cor da Flag | Significado |
|---------|------------|
| 🟢 Verde | Período cumprido ou adiantado |
| 🟡 Amarelo | Perto da meta (95–99%) |
| 🔴 Vermelho | Abaixo da meta |
| ⚪ Cinza | Sem dado ou sem movimento |

### 4.2 Como Preencher

1. **Digite o % realizado** para cada atividade
2. **Adicione comentário** se houver dificuldades
3. **Marque problemas** encontrados (seleção múltipla)
4. **Salve** → Sistema sincroniza automaticamente

**A IA detecta automaticamente** qual é o tipo de problema baseado no seu comentário. Você confirma ou corrige.

### 4.3 Funciona Sem Internet?

Sim! Suas edições ficam salvas localmente e sincronizam quando reconectar. Um ícone no topo avisa se está offline.

### 4.4 Exportar

**PDF:** Tabela completa com cores e resumo de KPIs (enviar por e-mail)

**Excel:** Grid completo (download direto)

---

## 5. Aba 1 — Análise de Problemas

### 5.1 Contagem por Categoria

Card mostrando contagem de problemas por tipo:

```
Qualidade          15  (45%)
Mão de Obra        8   (24%)
Materiais          6   (18%)
Método             4   (12%)
```

### 5.2 Atividades Mais Afetadas

Tabela ordenada por número de problemas:

| Atividade | Serviço | Qtd Problemas | Categorias |
|-----------|---------|---------------|-----------|
| Alvenaria A | Alvenaria | 4 | Qualidade, Mão de Obra |
| Forma B | Estrutura | 3 | Materiais |
| Reboco | Acabamento | 2 | Qualidade |

### 5.3 Gráfico

Gráfico de barras mostrando distribuição de problemas por categoria. Útil para identificar padrões e priorizar ações.

---

## 6. Aba 2 — Planos de Ação

### 6.1 Planos Vinculados

Lista de planos de ação já associados a atividades desta medição:

- Título, descrição
- Responsável e status (aberta, em andamento, concluída)
- Data de vencimento
- Botões: editar, visualizar, excluir vínculo

### 6.2 Criar Novo Plano

Botão `+` abre formulário para criar um plano:

- Título obrigatório
- Descrição e setor responsável
- Data de comprometimento
- Prioridade (baixa, média, alta)

O plano é criado e vinculado a uma ou mais atividades desta medição.

---

## 7. Aba 3 — KAI Analysis Tab

### 7.1 O que é?

A **aba KAI** mostra um resumo automático do desempenho da obra com foco em **atividades críticas** (aquelas que podem atrasar todo o projeto).

Estruturada em 10 seções (scroll):

```
[Seção 0] Resumo Executivo
[Seção 1] Índice de Produtividade
[Seção 2] Análise de Atrasos
[Seção 3] Comparativo com Cronograma Original
[Seção 4] Variação Mensal
[Seção 5] Análise por Setor
[Seção 6] Riscos de Viabilidade
[Seção 7] Gantt Analítico
[Seção 8] Sugestões Automáticas
[Seção 9] Restrições (6M)
[Seção 10] Caminho Crítico ← Detalhe abaixo
```

---

### 7.2 Seção 10 — Caminho Crítico

#### O que é?

Esta seção mostra quais são as **atividades que podem atrasar o cronograma todo** e como estão progredindo.

Se uma atividade crítica atrasa, toda a obra atrasa. Por isso é importante monitorá-las semanalmente.

#### KPIs Exibidos

Card no topo com 5 indicadores:

| Indicador | Significado |
|-----------|------------|
| **PPCC** | % das metas semanais do cronograma crítico que foram cumpridas |
| **Atividades em dia** | Quantas atividades críticas estão no prazo |
| **Atividades atrasadas** | Quantas críticas estão atrasadas |
| **Atraso médio** | Quantos dias em atraso vs cronograma original |
| **Atividades tardias** | Quantas críticas ultrapassaram a data planejada |

**Cores:**
- 🟢 Verde (≥ 85%): Cronograma crítico saudável
- 🟡 Amarelo (70–84%): Atenção, alguns atrasos
- 🔴 Vermelho (< 70%): Alerta! Risco de atraso geral

#### O que é PPCC?

**PPCC** = "Percent Plan Complete do Caminho Crítico"

É uma **nota de 0 a 100%** que responde:

> "Das metas que nos comprometemos a fazer SÓ NESTA SEMANA nas atividades críticas, quantas foram cumpridas?"

**Exemplo:**

```
Semana de 28/04 a 04/05

Atividade A (crítica)
  → Meta da semana: 10%  |  Realizado: 4%   → ❌ Não cumpriu

Atividade B (crítica)  
  → Meta da semana: 20%  |  Realizado: 15%  → ❌ Não cumpriu

Atividade C (crítica)
  → Meta da semana: 10%  |  Realizado: 20%  → ✅ Cumpriu

PPCC = 72.5%  ← 72.5% das metas foram cumpridas
```

Significa: há 2 atividades críticas atrasadas que precisam de atenção.

#### Visualização: Atividades Críticas Medidas

Cada atividade crítica mostra um **card com:**

```
┌─────────────────────────────────────────────────────┐
│ ✓ Atividade X — Lote Y                  [Em dia]   │
│                                                     │
│ Previsto (acum.): ████████░ 95.56%                 │
│ Realizado (acum.): ███████░ 89.00%                 │
│           [6.56% abaixo do previsto]                │
│                                                     │
│ Nesta semana                                        │
│ Planejado: 10.56%  │  Realizado: 4%                │
│                                                     │
│ Término planejado: 28/04/2026 (em 5 dias)          │
│ Data original: 26/04/2026                          │
│ Atraso: +2 dias                                    │
└─────────────────────────────────────────────────────┘
```

**O que significa:**

- **Barras coloridas:** linha cinza = "meta acumulada", linha verde/vermelha = "realizado acumulado"
- **"Nesta semana":** quanto foi planejado fazer SÓ NOS ÚLTIMOS 7 DIAS vs quanto foi feito
- **Atraso:** diferença entre data planejada (cronograma atual) vs cronograma original

#### Grupos de Atividades

**Grupo 1 — Atividades que Impactaram Esta Medição:**

Atividades críticas que já têm dados de medição registrados (estão em andamento ou concluídas).

**Grupo 2 — Próximas Atividades Críticas:**

Atividades críticas que ainda não começaram, mas virão em breve. Mostra as **3 próximas** com opção de ver todas.

Cada uma exibe:

```
┌──────────────────────────────────────────┐
│ ⏱ Atividade Crítica Futura — Lote Z     │
│ Serviço: Reboco                          │
│                                          │
│ Início planejado: 02/05/2026 (em 3d)     │
│ Fim original: 12/05/2026                 │
└──────────────────────────────────────────┘
```

---

## 8. Integração com Outros Módulos

### 8.1 Planos de Ação

Ao identificar atividade crítica atrasada, botão "Criar Plano de Ação" no card abre formulário pré-preenchido com:

- Serviço e atividade
- Descrição sugerida: "Recuperar atraso de X dias em [atividade]"
- Prioridade: alta
- Data de comprometimento: data planejada + X dias

### 8.2 Restrições (6M)

Seção 9 do KAI lista restrições que podem estar causando os atrasos críticos. Integra com modelo LPS para sugerir categorias baseado em problemas registrados.

### 8.3 Alertas e Notificações

Se PPCC < 70%:
1. Notificação push ao gestor: "⚠️ PPCC crítico: 65% — 2 atividades atrasadas"
2. Deep link abre direto na aba KAI seção 10
3. Sugestão automática de restrição

---

## 9. Fluxo Típico de Uso (Semanal)

1. **Segunda-feira:** Gestor abre medição para sexta-feira anterior
2. **Preenche avanços:** % realizado em cada atividade
3. **Registra problemas:** Comentário → IA sugere categorias
4. **Salva medição** → Sistema calcula automaticamente
5. **Abre aba KAI:** Vê PPCC, atividades críticas em risco
6. **Identifica atraso crítico:** Clica "Criar Plano de Ação"
7. **Cria plano:** Responsável, data, descrição
8. **Exporta PDF:** Distribui para time (opcional)
9. **Segue acompanhando:** Próxima semana repete o processo

---

## 10. Dúvidas Comuns

**P: PPCC 70% é bom ou ruim?**  
R: Acima de 85% é verde (excelente). Entre 70–84% é amarelo (atenção). Abaixo de 70% é vermelho (alerta).

**P: Por que atividade crítica A com 89% realizado aparece "atrasada"?**  
R: Porque o planejado para esta semana era 95.56%, e só 89% foi realizado. Faltaram 6.56%.

**P: Funciona sem internet?**  
R: Sim. Edições ficam locais e sincronizam quando reconectar. Ícone no topo indica se está offline.

**P: Posso exportar o relatório?**  
R: Sim, em PDF (com email) ou Excel (download).

