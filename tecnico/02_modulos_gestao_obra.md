---
layout: page
title: "Módulos de Gestão de Obra"
permalink: /kaizen-docs/tecnico/modulos/
category: "Módulos"
order: 2
toc: true
---

# Kaizen — Módulos de Gestão de Obra

## Sumário
1. [Produtividade](#1-produtividade)
2. [Planejamento e Avanço Físico](#2-planejamento-e-avanço-físico)
3. [Rotinas de Gestão](#3-rotinas-de-gestão)
4. [Qualidade](#4-qualidade)
5. [Custo e Orçamento](#5-custo-e-orçamento)
6. [Gestão de Materiais](#6-gestão-de-materiais)
7. [Controle de Unidades](#7-controle-de-unidades)
8. [Outros Módulos](#8-outros-módulos)

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

### 1.7 Dashboard
Análise de desempenho com visualizações gráficas de duas abas:

- **Aba Serviços:** tabela ordenável + gráfico de barras de produção ou custo por serviço; comparativo realizado vs. previsto por período
- **Aba Problemas:** gráfico de pizza com frequência de problemas por serviço e lista de ocorrências em aberto

Suporte a acúmulo mensal e exportação via impressão do browser.

**Chave de privilégio:** `prod_bi`

---

## 2. Planejamento e Avanço Físico

> Os módulos desta seção requerem um cronograma ativo na obra. Itens 2.1–2.4 exigem privilégio **`baseline_view`**; itens 2.5–2.6 exigem **`physical`**; item 2.7 exige **`teams`**.

### 2.1 Atividades
Tabela completa de atividades do cronograma ativo. Colunas: serviço, lote, início, fim, duração, custo, status, % concluído. Operações disponíveis:

- Edição inline de datas e custos diretamente na tabela
- Busca de atividade por nome com debounce
- Filtros: somente baseline, somente críticas, exibir/ocultar concluídas
- Cálculo do caminho crítico sob demanda
- Fila de desfazer/refazer (até 15 passos)
- Exportação em Excel (.xlsx) e PDF

---

### 2.2 Gantt de Controle
Visualização Gantt interativa do cronograma ativo. Barras coloridas por serviço sobre eixo de tempo scrollável. Funcionalidades:

- Linhas de dependência com caminho crítico destacado em vermelho
- Drag das barras para reprogramar; duração recalculada automaticamente
- Filtro por serviço (dropdown na AppBar)
- Toggle para exibir/ocultar atividades concluídas e serviços 100% finalizados

Renderizado com `CustomPainter` (`GanttGridPainter`, `GanttActivityPainter`).

---

### 2.3 Painel de Produção
Matriz de produção no formato Monkey Chart orientada ao cronograma: cada célula (lote × serviço) exibe o status de conclusão com código de cor (planejada, em progresso, concluída, atrasada). Toque na célula abre os detalhes e permite edição da atividade.

---

### 2.4 Análise do Planejamento (Curva S)
Curva S acumulada com três séries simultâneas:

| Série | Significado |
|-------|------------|
| **S** (Scheduled) | Cronograma previsto ativo |
| **B** (Baseline) | Cronograma de referência (baseline) |
| **D** (Done) | Realizado acumulado |

Os valores são calculados por atividade e acumulados por dia, com detalhe mensal por serviço. O resultado é cacheado no documento do cronograma (`cffCalculatedAt`, `cffVersion`) para evitar reprocessamento.

---

### 2.5 Medições Físicas
Registro de medições físicas contratuais (volumes, áreas, quantidades por serviço). Vinculadas a contratos e boletins de medição com fluxo de aprovação configurável. Gera boletim de medição em PDF.

**Chave de privilégio:** `physical`

---

### 2.6 Gestão de Cronogramas
Gerenciamento de versões do cronograma: criação de nova versão, definição do baseline de referência, comparação entre versões e histórico completo de revisões com autor e timestamp.

**Fontes de importação suportadas:**

| Fonte | Formato | Observação |
|---|---|---|
| **Prevision** | API REST | Fonte primária; sincroniza automaticamente pelo `previsionProjectId` da obra |
| **Excel** | `.xlsx` | Arquivo enviado à Cloud Function `excelToJson`; usuário mapeia as colunas manualmente (ID, serviço, lote, início, fim, custo, % concluído) |
| **JSON** | `.json` | Importa um arquivo JSON estruturado no formato interno do Kaizen (ex.: exportação de outra obra) |

**Chave de privilégio:** `physical`

---

### 2.7 Equipes
Cadastro e gestão das equipes de produção. Associação de colaboradores a equipes (compartilhado com o Módulo AT), custo/hora por equipe para cálculo de mão de obra e histórico de desempenho por equipe.

**Chave de privilégio:** `teams`

---

## 3. Rotinas de Gestão

### 3.1 Planos de Ação
Gestão de itens de ação com três modos de visualização:

- **Kanban:** cartões agrupáveis por responsável, status, setor, prioridade, risco ou semana de vencimento
- **Lista:** visualização tabular com ordenação por 6+ critérios
- **Dashboard:** KPIs de abertura × conclusão × vencidos

Campos principais: título, descrição, prazo, status (`aberta / em andamento / concluída / cancelada / vencida`), prioridade (`baixa / média / alta`), risco (`baixo / médio / alto`), responsável, setor, áreas de impacto (multi-select), vínculos a atividades do cronograma e tags de serviço.

Funcionalidades adicionais:
- Notificações push automáticas ao responsável via FCM na criação e no vencimento
- Exportação em PDF e Excel
- Comentários internos por item com histórico completo
- Tags e cores de serviço configuráveis por tenant

**Chave de privilégio:** `action_plans_view` (ou usuário super)

---

### 3.2 Quadro de Restrições (Last Planner)
Lookahead semanal de restrições que bloqueiam as atividades do cronograma. Cada restrição registra: serviço afetado, descrição do bloqueador, data de comprometimento para resolução, status (`aberta / em andamento / removida / vencida`), responsável, setor e anexos.

**Visões disponíveis:**
- Grade semanal (serviço × semana), lookahead configurável — padrão 8 semanas
- Painel de serviços com filtro de restrições ativas
- Painel de estatísticas: contagens por status e tendências

**Ordenação:** 8+ opções, incluindo data de comprometimento e data de início da próxima atividade vinculada

**Exportação:** Excel multi-obra consolidado (`MidTermExcel`)

**Notificações:** Alertas automáticos de vencimento configuráveis; deep link via FCM abre diretamente a restrição no app (`pendingOpenRestrictionId`)

**IA — KAI:** Agentes `kai-restriction-analyst` (relatório analítico 6M) e `kai-suggestion` (sugestões automáticas de novas restrições) disponíveis nesta tela

**Chave de privilégio:** `mid_term` (requer cronograma ativo)

---

### 3.3 Farol de Contratações
Gestão do cronograma de suprimentos com semáforo (verde/amarelo/vermelho) baseado em SLAs configurados por fornecedor e categoria de insumo. Dois objetos principais gerenciados:

- **Itens de suprimento:** nome, fornecedor, categoria, SLA padrão, margem de segurança e estoque mínimo
- **Processos de suprimento:** status do processo de contratação com datas, responsáveis e vínculos às atividades do cronograma

A matrix item × fornecedor exibe a situação de cada SLA com indicação visual de risco.

**Exportação:** CSV (`exportDatabaseCSV`) e Excel consolidado multi-obra (`SupplyScheduleExcel`)

**Chave de privilégio:** `supply_schedule` ou perfil admin

---

### 3.4 Estabilização Lean
Aplicação digital de checklists de inspeção Lean (5S) com estrutura hierárquica de seções e subseções. Cada inspeção registra data, torre/bloco inspecionado e responsável. Por item: resultado (conforme / não conforme / parcial), foto evidência e observação.

Score calculado automaticamente por seção (roll-up dos filhos) e score total da inspeção. Histórico de até 24 meses com gráfico de evolução e média histórica calculada. Fotos armazenadas com compressão antes do upload.

**Chave de privilégio:** `lean_inspection`

---

## 4. Qualidade

### 4.1 Gestão de FVS
Dashboard central das Fichas de Verificação de Serviço. O sistema opera em dois níveis:

- **Templates (`DefaultFVSModel`):** checklists padronizados por tipo de serviço, reutilizáveis entre obras; carregados com cache reativo
- **Instâncias (`FVSModel`):** FVS preenchida para uma atividade específica, com resultado por item de checklist

**Fluxo de status:** Em Verificação → Inspecionada → Aprovada (ou Rejeitada → Pendência)

Campos por instância: atividade vinculada (serviço + lote denormalizados para leitura rápida), responsável, aprovador, prioridade, lista de itens com resultado (`conforme / não conforme / N/A`), contagem de itens rejeitados.

**Filtros:** status, atividade, responsável, prioridade, lote, tag de serviço, busca por texto

**Ordenações disponíveis:** prioridade, recência, quantidade de pendências, nome da atividade

**Contadores do dashboard:** Em Verificação · Pendências · Inspecionadas · Aprovadas · Total de Itens Rejeitados (rollup)

**Exportação:** PDF do FVS com fotos, assinaturas e observações

**Chave de privilégio:** `fvs_fill` ou `fvs_approval` ou `quality_manager`

---

## 5. Custo e Orçamento

### 5.1 Custo e Orçamento
Sistema de controle orçamentário com comparativo planejado vs. realizado. Cada item de orçamento contém: código WBS, descrição, referência de instalação (INS ID), quantidade, preço unitário, valor total, flag de aprovação e comentários.

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

**Exportação:** Excel com tabelas de desvio, vinculação a contratos e análise de custo; relatórios multi-obra consolidados

**Integração:** Dados realizados lidos do Azure Synapse em tempo real via `SynapseController`

**Chave de privilégio:** `cost_control_view`

---

## 6. Gestão de Materiais

### 6.1 Almoxarifado Digital
Central de gestão de materiais da obra, organizada em seis abas via `NavigationRail`:

| Aba | Função |
|-----|--------|
| **Requisições** | Solicitações de material por usuários de campo; fluxo de aprovação com status (pendente → aprovado → retirado) |
| **Pedidos** | Ordens de compra externas ou internas; rastreio de datas prometidas e recebidas, custo e nota fiscal |
| **Saídas** | Baixas físicas de estoque geradas a partir de requisições aprovadas, com rastreabilidade completa (quem, quando, quantidade, destino) |
| **Kits** | Pacotes pré-montados de materiais para uma atividade; convertíveis em requisições se uso parcial |
| **Consulta** | Saldo em tempo real por item; catálogo integrado com Mega ERP (código, descrição, unidade, grupo) |
| **Auditoria** | Histórico de todas as movimentações por tipo (requisição, saída, pedido) com filtros de data e responsável |

**Itens de estoque:** código, descrição, unidade (peça, m, m², m³, kg), quantidade em mãos, ponto de resuprimento, fornecedor, preço unitário, categoria.

**Funcionalidades transversais:**
- Etiquetas QR Code por item via `QrLabelAssistantView`
- Notificações push para aprovadores via FCM (tópico `pedidos_{clientId}_{branchId}`)
- Importação via CSV e exportação Excel

**Chave de privilégio:** `warehouse_view`

---

## 7. Controle de Unidades

### 7.1 Visão Geral
Módulo para rastreabilidade completa das unidades habitacionais (apartamentos, lotes, lojas). Dados sincronizados do Salesforce Asset via Cloud Function autenticada. A tela principal exibe um dashboard por torre com filtros de status, tipo e personalização.

**Hierarquia:** Empreendimento → Torre → Unidade

Leitura disponível para usuários com privilégio `units`; edição restrita a perfis `project_manager`.

---

### 7.2 Ficha da Unidade — Aba Principal
Identificação completa: número, torre, andar, tipo (kitnet, 1Q, 2Q, 3Q, cobertura, loja etc.). Dados do proprietário: nome, CPF, telefone, e-mail, data de contrato. Status global com histórico de mudanças de status (autor + timestamp). Observações sincronizadas com o registro do Salesforce Asset.

---

### 7.3 Aba Personalização
Customizações solicitadas pelo cliente: pisos, revestimentos, cores, modificações de planta. Cada item tem status (`solicitado / em execução / concluído`), fotos de referência, data prevista de conclusão e alerta quando exige prazo especial de execução.

---

### 7.4 Aba Obra
Status de cada etapa de execução na unidade (alvenaria, gesso, elétrica etc.) com percentual de conclusão, equipe responsável, fotos de evidência por etapa e pendências vinculadas a planos de ação.

---

### 7.5 Aba Checklist de Obra
Lista de verificação pré-entrega com confirmação item a item em campo. Cada item captura foto de evidência, status (`pendente / conforme / não conforme / N/A`) e log de quem executou. Geração automática de relatório ao concluir.

---

### 7.6 Aba Vistoria do Cliente
Condução da vistoria de entrega com o cliente. Checklist digital preenchido durante a visita, registro de pendências com foto e prazo de resolução, **assinatura digital do cliente** capturada no app. Gera **Laudo de Vistoria em PDF** com fotos, assinaturas e pendências. Controle de revistorias com histórico completo de visitas.

**Status:** Agendada → Aguardando → 1ª Vistoria → Revisoria → Entregue

---

### 7.7 Aba Inspeção Final
Inspeção técnica pré-entrega por engenheiro — distinta da vistoria do cliente, com foco em aspectos técnicos e normativos. Checklist específico, registro de pontos de atenção com fotos, aprovação com assinatura digital do engenheiro. Gera relatório de inspeção final para arquivo técnico.

---

### 7.8 Gerador de Etiquetas QR Code

**Modo Folha (2×6):** 12 etiquetas por folha A4, layout para papel adesivo; QR Code + número, torre e andar por etiqueta.

**Modo Placa Grande (A4):** Uma unidade por folha com QR Code em tamanho grande, logo do empreendimento e dados completos.

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

## 8. Outros Módulos

### 8.1 Reserva de Veículos
Gestão da frota da filial com calendário mensal de reservas (janela de 60 meses: 30 passados + atual + 30 futuros). Cada veículo registra nome, tipo, status (disponível/reservado/manutenção) e especificações. Cada reserva registra veículo, solicitante, período (início/fim), justificativa, status (`pendente / aprovado / concluído / cancelado`) e custo associado.

**Gestão de custos:**
- Rastreamento de despesa por reserva
- Rateio de custo entre reservas (`calculateSplit`)
- Totais agregados por período

**Chave de privilégio:** `vehicle_reservation`

---

## Consolidado de Módulos

| Módulo | Seção no Menu | Privilégio | Exportação |
|--------|--------------|-----------|-----------|
| Planejamento Kaizen | Produtividade | `daily_measurements` | — |
| Medições Kaizen | Produtividade | `daily_measurements` | — |
| Kaizen Diário | Produtividade | `daily_measurements` ou `daily_kaizen` | — |
| Gestão da Estrutura | Produtividade | `daily_measurements` | — |
| Histórico de Medições | Produtividade | `daily_measurements` | — |
| Estatísticas | Produtividade | `prod_bi` | — |
| Dashboard | Produtividade | `prod_bi` | Impressão |
| Atividades | Planejamento e Avanço Físico | `baseline_view` | Excel, PDF |
| Gantt de Controle | Planejamento e Avanço Físico | `baseline_view` | — |
| Painel de Produção | Planejamento e Avanço Físico | `baseline_view` | — |
| Análise do Planejamento | Planejamento e Avanço Físico | `baseline_view` | — |
| Medições Físicas | Planejamento e Avanço Físico | `physical` | PDF |
| Gestão de Cronogramas | Planejamento e Avanço Físico | `physical` | — |
| Equipes | Planejamento e Avanço Físico | `teams` | — |
| Planos de Ação | Rotinas de Gestão | `action_plans_view` | Excel, PDF |
| Quadro de Restrições | Rotinas de Gestão | `mid_term` | Excel |
| Farol de Contratações | Rotinas de Gestão | `supply_schedule` ou admin | CSV, Excel |
| Estabilização Lean | Rotinas de Gestão | `lean_inspection` | PDF |
| Gestão de FVS | Qualidade | `fvs_fill` / `fvs_approval` / `quality_manager` | PDF |
| Custo e Orçamento | Custo e Orçamento | `cost_control_view` | Excel |
| Almoxarifado Digital | Gestão de Materiais | `warehouse_view` | Excel, CSV |
| Unidades | Controle de Unidades | `units` | PDF (vistoria, inspeção) |
| Reserva de Veículos | Outros Módulos | `vehicle_reservation` | — |
