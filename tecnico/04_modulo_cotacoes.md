---
layout: page
title: "Módulo de Cotações"
permalink: /kaizen/tecnico/cotacoes/
category: "Módulos"
order: 4
toc: true
---

# Kaizen — Módulo de Cotações

## Sumário
1. [Visão Geral](#1-visão-geral)
2. [Mapas de Cotação](#2-mapas-de-cotação)
3. [Fluxo de Trabalho (4 Estágios)](#3-fluxo-de-trabalho-4-estágios)
4. [Itens de Cotação](#4-itens-de-cotação)
5. [Gestão de Fornecedores no Mapa](#5-gestão-de-fornecedores-no-mapa)
6. [Integração com Orçamento](#6-integração-com-orçamento)
7. [Auditoria e Rastreabilidade](#7-auditoria-e-rastreabilidade)
8. [Funcionalidades de Suporte](#8-funcionalidades-de-suporte)
9. [Resumo de Capacidades](#9-resumo-de-capacidades)

---

## 1. Visão Geral

O Módulo de Cotações gerencia o processo completo de coleta e análise de propostas comerciais para aquisição de materiais e contratação de serviços. Ele digitaliza o processo que geralmente ocorre por e-mail e planilhas, centralizando todo o fluxo de aprovação com rastreabilidade completa.

### Premissas Funcionais
- Cada processo de cotação é representado por um **Mapa de Cotação**
- Um mapa percorre 4 estágios bem definidos até a aprovação final
- Múltiplos fornecedores podem ser convidados a cotar cada item
- O módulo se integra diretamente com o **Controle de Custo** para vincular aprovações ao orçamento da obra
- Todo mapa possui um **log de auditoria** automático com todas as ações realizadas

---

## 2. Mapas de Cotação

### 2.1 Conceito
Um Mapa de Cotação é o documento central que agrupa todos os itens a serem cotados em um único processo de compra. Exemplos: "Mapa de Cotação de Materiais Hidráulicos — Torre A", "Contratação de Mão de Obra Elétrica — Fase 2".

### 2.2 Listagem de Mapas
- Listagem paginada com **filtros server-side:**

| Filtro | Descrição |
|--------|-----------|
| Estágio | Filtrar por estágio atual do mapa (1–4) |
| Setor | Setor/área responsável pelo processo |
| Responsável | Usuário responsável pelo mapa |
| Pesquisa livre | Busca por nome do mapa, descrição, fornecedor |

- Exibição de status visual por estágio (cores diferenciadas)
- Ordenação por data de criação, prazo, responsável ou estágio

### 2.3 Campos do Mapa

| Campo | Descrição |
|-------|-----------|
| Nome | Título descritivo do mapa de cotação |
| Descrição | Detalhamento do escopo da cotação |
| Setor | Área responsável (ex.: Suprimentos, Engenharia) |
| Responsável | Usuário principal responsável pelo processo |
| Obra / Empreendimento | Obra à qual a cotação se destina |
| Filial | Filial responsável |
| Prazo | Data limite para recebimento de propostas |
| Estágio | Estágio atual no fluxo (1 a 4) |
| Criado por | Usuário criador com data/hora |
| Última atualização | Timestamp da última modificação |

---

## 3. Fluxo de Trabalho (4 Estágios)

O processo de cotação é estruturado em 4 estágios sequenciais que garantem controle e rastreabilidade:

```
┌─────────────────────────────────────────────────────────────────┐
│  Estágio 1        Estágio 2          Estágio 3      Estágio 4   │
│  Montagem    →  Cotação com    →    Análise    →   Aprovado     │
│  do Mapa       Fornecedores                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Estágio 1 — Montagem do Mapa
**Objetivo:** Definir o que será cotado.

Atividades principais:
- Criação do mapa com informações básicas
- Adição de todos os **itens a cotar** (materiais, serviços)
- Definição de especificações técnicas por item
- Associação dos itens a rubricas do orçamento (`getBudgetItemByKey`)
- Importação de itens via **planilha Excel** (.xlsx)
- Seleção dos fornecedores que serão convidados a cotar

Transição para Estágio 2: ao menos 1 item cadastrado e ao menos 1 fornecedor selecionado.

---

### Estágio 2 — Cotação com Fornecedores
**Objetivo:** Coletar propostas dos fornecedores.

Atividades principais:
- Envio (ou registro) das solicitações de cotação para fornecedores
- Modo especial de entrada de dados: **`isSupplierSelectionMode`** — interface otimizada para registrar preços por fornecedor × item
- Preenchimento dos preços ofertados por cada fornecedor para cada item
- Registro de condições de pagamento, prazo de entrega e observações por proposta
- Upload de anexos (propostas em PDF, planilhas do fornecedor)
- Controle de quais fornecedores já responderam (Respondido / Aguardando / Não Respondeu)

**Interface do modo de seleção de fornecedores:**
- Tabela matricial: linhas = itens, colunas = fornecedores
- Preenchimento ágil de preços direto na célula
- Destaque automático do menor preço por item
- Cálculo instantâneo de totais por fornecedor

Transição para Estágio 3: ao menos 2 fornecedores com proposta registrada (ou por decisão do responsável).

---

### Estágio 3 — Análise
**Objetivo:** Selecionar o vencedor e definir a proposta aceita.

Atividades principais:
- Comparativo lado a lado das propostas recebidas
- Filtros de análise: por item, por fornecedor, por melhor preço
- Seleção do fornecedor vencedor (pode ser diferente por item — split de fornecimento)
- Registro de justificativa de escolha (campo obrigatório quando não é o menor preço)
- Categorização da variação orçamentária se houver desvio de custo:

| Código | Categoria de Variação |
|--------|----------------------|
| `ORC` | Variação de Orçamento (estimativa original inadequada) |
| `CST` | Variação de Custo (custo de mercado acima do previsto) |
| `PRJ` | Variação de Projeto (escopo alterado pelo projeto) |
| `ENG` | Variação de Engenharia (solução técnica diferente) |
| `PRZ` | Variação de Prazo (impacto de urgência no preço) |

- Consolidação do valor total aprovado por rubrica de orçamento
- Prévia do impacto no orçamento da obra

Transição para Estágio 4: análise concluída, fornecedor(es) selecionado(s), variação categorizada.

---

### Estágio 4 — Aprovado
**Objetivo:** Registro final do processo com aprovação formal.

Atividades:
- Aprovação do mapa pelo aprovador designado (assinatura eletrônica / confirmação digital)
- Geração do registro consolidado final: itens × fornecedores vencedores × valores aprovados
- Atualização automática no **Controle de Custo** da obra (registra compromisso orçamentário)
- Geração de relatório resumo do processo de cotação
- O mapa fica como *read-only* neste estágio; qualquer revisão necessária gera novo mapa vinculado

---

## 4. Itens de Cotação

### 4.1 Estrutura de um Item

| Campo | Descrição |
|-------|-----------|
| Nome / Descrição | O que está sendo cotado |
| Código | Código interno de referência |
| Unidade | Unidade de medida (m², kg, un, m³, etc.) |
| Quantidade | Quantidade necessária |
| Observações Técnicas | Especificações, normas ou requisitos |
| Vinculação Orçamentária | Rubrica do orçamento da obra (`budgetItemKey`) |
| Prazo de Entrega Necessário | Data máxima para entrega/execução |

### 4.2 Importação por Excel
- Template de importação disponível para download (Excel .xlsx)
- Colunas obrigatórias: descrição, unidade, quantidade
- Colunas opcionais: código, observações, código orçamentário
- Validação linha por linha com relatório de erros antes de confirmar importação
- Items importados entram no Estágio 1 prontos para cotação

### 4.3 Preços por Fornecedor
- Cada item armazena um mapa de preços: `{fornecedorId: {preco, prazo, condicao, obs}}`
- Histórico de edições de preço (audit trail por item × fornecedor)
- Campo de "Preço de Referência" (orçamento original) para comparação imediata

---

## 5. Gestão de Fornecedores no Mapa

### 5.1 Seleção de Fornecedores
- Busca e seleção no catálogo de fornecedores cadastrados da filial
- Possibilidade de adicionar fornecedor ad-hoc (sem cadastro prévio) para processos urgentes
- Mínimo e máximo de fornecedores configuráveis por tipo de processo

### 5.2 Status por Fornecedor no Mapa
| Status | Descrição |
|--------|-----------|
| Convidado | Solicitação enviada, aguardando resposta |
| Aguardando | Confirmou recebimento, proposta em elaboração |
| Proposta Recebida | Valores registrados no sistema |
| Vencedor | Fornecedor selecionado na análise |
| Não Selecionado | Participou, mas não foi escolhido |
| Desistiu | Recusou participar ou não respondeu no prazo |

### 5.3 Comunicação
- Envio de e-mail de convite para cotação (via Cloud Function + Resend)
- Registro de todas as comunicações realizadas no log do mapa

---

## 6. Integração com Orçamento

### 6.1 Vinculação de Itens ao Orçamento
- Cada item de cotação pode ser associado a uma rubrica do orçamento via `getBudgetItemByKey`
- A associação permite:
  - Comparar o orçado com o cotado por rubrica
  - Atualizar o comprometido no orçamento quando o mapa é aprovado (Estágio 4)
  - Controlar desvios por categoria de variação

### 6.2 Fluxo de Atualização do Orçamento
```
Mapa aprovado (Estágio 4)
      ↓
Para cada item com vinculação orçamentária:
  - Registra o valor comprometido na rubrica
  - Gera entrada no log de desvios (se valor > orçado original)
  - Categoriza a variação (ORC, CST, PRJ, ENG, PRZ)
      ↓
Dashboard de Custo atualizado automaticamente
```

### 6.3 Visibilidade no Controle de Custo
- Cotações aprovadas aparecem como **desvios aprovados** no módulo de custo
- Cotações em análise podem aparecer como **comprometidos pendentes** (configurável)
- Drilldown: do custo → para o mapa de cotação que gerou o desvio

---

## 7. Auditoria e Rastreabilidade

### 7.1 Log Automático por Mapa (`addLog()`)
Todo evento relevante é automaticamente registrado no log do mapa:

| Evento | Detalhes Registrados |
|--------|---------------------|
| Criação do mapa | Usuário criador, data/hora |
| Item adicionado | Nome, quantidade, unidade, quem adicionou |
| Fornecedor adicionado | Nome do fornecedor, quem adicionou |
| Preço registrado | Fornecedor, item, valor, quem registrou |
| Mudança de estágio | De qual estágio para qual, por quem, data/hora |
| Fornecedor selecionado | Item, fornecedor vencedor, valor, justificativa |
| Variação categorizada | Categoria, valor do desvio, responsável |
| Aprovação final | Aprovador, data/hora, valor total aprovado |
| Comentário adicionado | Usuário, conteúdo do comentário, data/hora |
| Arquivo anexado | Nome do arquivo, uploader, data/hora |

### 7.2 Dirty Check antes de Sair (`hasSomethingChanged()`)
- Ao tentar navegar para fora do mapa com alterações não salvas, o sistema alerta o usuário
- Confirmar: salva as alterações antes de sair
- Descartar: descarta e navega
- Cancelar: permanece na tela

### 7.3 Rastreabilidade Completa
- Toda a trilha desde a necessidade (criação do mapa) até a aprovação está registrada
- Relatório de auditoria exportável em PDF ou Excel
- Dados disponíveis para auditoria interna e análise de compliance de compras

---

## 8. Funcionalidades de Suporte

### 8.1 Anexos e Documentos
- Upload de arquivos por mapa e por item: propostas em PDF, especificações técnicas, desenhos
- Armazenamento em nuvem (organizado por `clientId/cotacoes/mapaId/`)
- Visualização inline de PDFs e imagens no app
- Download de qualquer arquivo por usuário com acesso ao mapa

### 8.2 Comentários Internos
- Thread de comentários por mapa de cotação
- Notificações push para membros do mapa ao receber novo comentário
- Suporte a menções de usuários (`@usuário`)
- Histórico completo de discussões vinculado ao mapa

### 8.3 Clonagem de Mapa
- Um mapa pode ser clonado como ponto de partida para novo processo similar
- Itens e estrutura são copiados; preços e fornecedores selecionados são resetados
- Útil para cotações recorrentes ou renovações de contrato

### 8.4 Relatórios e Exportação
- **Relatório resumo do mapa:** PDF com todos os itens, preços por fornecedor, análise comparativa e fornecedor selecionado
- **Exportação Excel:** tabela completa de cotação para análise externa
- Relatório de **custo total aprovado** por obra/filial por período

---

## 9. Resumo de Capacidades

| Capacidade | Suporte |
|------------|---------|
| Fluxo de 4 estágios | ✅ |
| Importação de itens via Excel | ✅ |
| Modo de entrada de preços por fornecedor | ✅ (isSupplierSelectionMode) |
| Análise comparativa de propostas | ✅ |
| Vinculação ao orçamento da obra | ✅ (getBudgetItemByKey) |
| Categorização de variações orçamentárias | ✅ (ORC, CST, PRJ, ENG, PRZ) |
| Log de auditoria automático | ✅ (addLog) |
| Dirty check antes de navegação | ✅ (hasSomethingChanged) |
| Upload de anexos | ✅ (armazenamento em nuvem) |
| Comentários internos | ✅ |
| Exportação Excel e PDF | ✅ |
| Filtros de listagem server-side | ✅ |
| Paginação | ✅ |
| Notificações push | ✅ (FCM) |

---

*Documento: Módulo de Cotações | Versão 1.0 | Kaizen Gerenciamento de Obras*
