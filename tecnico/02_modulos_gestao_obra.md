---
layout: page
title: "Módulos de Gestão de Obra"
permalink: /tecnico/modulos/
category: "Módulos"
order: 2
toc: true
---

# Kaizen — Módulos de Gestão de Obra

O Kaizen organiza a gestão de obras em **8 blocos principais**, cobrindo o ciclo completo da construção: planejamento, controle operacional, qualidade, custo e entrega. Módulos complementares ampliam a cobertura para demandas específicas.

## Navegação Rápida

| # | Bloco | Funções Principais |
|---|-------|--------------------|
| [1](#1-produtividade--planejamento-e-medições-kaizen) | **Produtividade — Planejamento e Medições Kaizen** | Planos de produção, registro diário de avanço, índice de produtividade |
| [2](#2-planejamento-físico-financeiro) | **Planejamento Físico-Financeiro** | Curva S, CFF, Gantt reprogramável, cronogramas, vinculação de orçamento |
| [3](#3-controle-de-produção) | **Controle de Produção** | Medições físicas, painel de produção, equipes & histograma, planos de ação |
| [4](#4-rotinas-de-gestão) | **Rotinas de Gestão** | Farol de contratações, quadro de restrições (6M/6WLA — Last Planner) |
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

> Requer cronograma ativo na obra. Itens 2.1–2.6 exigem privilégio **`baseline_view`**; item 2.7 exige **`physical`**.

---

### 2.1 Atividades

Tabela completa de todas as atividades do cronograma ativo. Colunas: serviço, lote, início, fim, duração, custo, status, % concluído. Operações disponíveis:

- Edição inline de datas e custos diretamente na tabela
- Busca de atividade por nome com debounce
- Filtros: somente baseline, somente críticas, exibir/ocultar concluídas
- **Cálculo do caminho crítico** sob demanda (destaca as atividades que definem o prazo mínimo do projeto)
- Fila de desfazer/refazer (até 15 passos)
- **Exportação** em Excel (.xlsx) e PDF

---

### 2.2 Gantt Reprogramável

Visualização Gantt interativa do cronograma ativo. Barras coloridas por serviço sobre eixo de tempo scrollável.

| Funcionalidade | Detalhe |
|---------------|---------|
| **Reprogramação dinâmica** | Drag das barras para reprogramar; duração recalculada automaticamente |
| **Dependências** | Linhas de dependência entre atividades |
| **Caminho crítico** | Destacado em vermelho; recalculado ao mover barras |
| **Filtros** | Por serviço (dropdown na AppBar); toggle de atividades concluídas |
| **Renderização** | `CustomPainter` (`GanttGridPainter`, `GanttActivityPainter`) |

---

### 2.3 Curva S — Análise do Planejamento

Curva S acumulada com **três séries simultâneas**, comparando cronograma previsto, baseline e realizado:

| Série | Significado |
|-------|------------|
| **S** (Scheduled) | Cronograma previsto ativo |
| **B** (Baseline) | Cronograma de referência imutável |
| **D** (Done) | Realizado acumulado |

Os valores são calculados por atividade e acumulados por dia, com detalhe mensal por serviço. O resultado é cacheado no documento do cronograma (`cffCalculatedAt`, `cffVersion`) para evitar reprocessamento.

**Perspectivas disponíveis:** monetária (R$) e física (% de progresso). Suporte a exportação e zoom por período.

---

### 2.4 Cronograma Físico-Financeiro (CFF)

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

### 2.5 Vinculação de Orçamento — Budget Weights

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

### 2.6 Relatórios Operacionais

Exportações e relatórios gerados a partir do cronograma ativo:

| Relatório | Formato | Conteúdo |
|-----------|---------|----------|
| Atividades completas | Excel / PDF | Tabela de atividades com datas, custos e % |
| Curva S do projeto | Gráfico PDF | Séries S/B/D acumuladas |
| CFF por período | Excel / PDF | Cronograma físico-financeiro com colunas mensais |
| Análise de desvios | Excel | Desvios por rubrica categorizados (ORC, CST, PRJ, ENG, PRZ) |

---

### 2.7 Gestão de Cronogramas

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

Registro de medições físicas contratuais (volumes, áreas, quantidades por serviço). Vinculadas a **contratos** e **boletins de medição** com fluxo de aprovação configurável. Gera boletim de medição em PDF.

Importação de medições disponível também via [Prevision](05_integracoes.md#2-prevision-planejamento-de-obras) (`measurementsTasksPage`).

**Chave de privilégio:** `physical`

---

### 3.2 Painel de Produção

Matriz de produção no formato **Monkey Chart** orientada ao cronograma: cada célula (lote × serviço) exibe o status de conclusão com código de cor.

| Cor | Significado |
|-----|-------------|
| Verde | Concluído |
| Amarelo | Em progresso, dentro do prazo |
| Azul | Planejada |
| Vermelho | Atrasada |

Toque na célula abre os detalhes da atividade e permite edição direta.

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

Gestão do cronograma de suprimentos com semáforo visual (verde / amarelo / vermelho) baseado em SLAs configurados por fornecedor e categoria de insumo.

**Dois objetos principais:**

| Objeto | Descrição |
|--------|-----------|
| **Itens de suprimento** | Nome, fornecedor, categoria, SLA padrão, margem de segurança e estoque mínimo |
| **Processos de suprimento** | Status do processo de contratação com datas, responsáveis e vínculos às atividades do cronograma |

A matriz item × fornecedor exibe a situação de cada SLA com indicação visual de risco. O sistema antecipa alertas com base na data necessária de início da atividade vinculada menos o SLA de fornecimento — garantindo que as compras sejam iniciadas a tempo de não comprometer o cronograma.

**Exportação:** CSV (`exportDatabaseCSV`) e Excel consolidado multi-obra (`SupplyScheduleExcel`)

**Chave de privilégio:** `supply_schedule` ou perfil admin

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

A inspeção técnica pré-entrega por engenheiro é realizada no contexto de cada unidade (ver [Seção 7 — Controle de Unidades](#7-controle-de-unidades)):

- **Inspeção Final de Unidade:** checklist técnico-normativo com pontos de atenção e fotos, assinatura digital do engenheiro, geração de relatório de inspeção para arquivo técnico
- **Checklist de Obra (pré-entrega):** verificação item a item em campo, captura de foto de evidência, log de quem executou, status por item (`pendente / conforme / não conforme / N/A`)

Ver detalhes completos em [7.5 Aba Checklist de Obra](#75-aba-checklist-de-obra) e [7.7 Aba Inspeção Final Técnica](#77-aba-inspeção-final-técnica).

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
