---
layout: page
title: "Documentação Técnica"
permalink: /tecnico/
category: "Índice"
order: 0
toc: false
---

# Kaizen — Documentação Técnica

> **Público-alvo:** Equipes de TI, para fins de avaliação técnica.
>
> **Data de referência:** Março de 2026  
> **Versão do sistema:** Flutter 3.x · Banco de dados em nuvem (Google Cloud) · Salesforce Integration

---

## Visão Geral do Produto

O **Kaizen** é uma plataforma digital de gestão de construção e assistência técnica, desenvolvida com metodologia Lean Construction. Funciona como **Progressive Web App (PWA)** acessível por navegador e como aplicativo nativo Android/iOS.

O sistema é **multi-tenant** (múltiplos clientes isolados na mesma infraestrutura) e **multi-filial** (uma empresa pode ter múltiplas obras e filiais dentro do mesmo tenant), com controle de acesso granular por perfil, privilégio e filial.

### Produtos / Módulos Principais

| Módulo | Descrição Resumida |
|---|---|
| **Gestão de Obras** | Planejamento, controle de produtividade, qualidade, custos e materiais por obra |
| **Assistência Técnica (AT)** | Gestão de ordens de serviço, agendamentos, técnicos e materiais de AT |
| **Cotações** | Gestão de mapas de cotação de suprimentos com fluxo de aprovação |
| **Inteligência Artificial (KAI)** | Agentes KAI especializados em análise de obras, RAG por obra e relatórios automáticos |

---

## Índice da Documentação

| Arquivo | Conteúdo |
|---|---|
| [01 — Arquitetura Geral](./01_arquitetura_geral.md) | Stack tecnológico, infra, multi-tenancy, offline, segurança |
| [02 — Módulos Gestão de Obras](./02_modulos_gestao_obra.md) | **Reorganizado:** 8 categorias principais de módulos; primeiros passos em cada Uma; integração Prevision |
| [03 — Módulo Assistência Técnica](./03_modulo_assistencia_tecnica.md) | AT: work orders, agendamentos, colaboradores, analytics |
| [04 — Módulo Cotações](./04_modulo_cotacoes.md) | Mapas de cotação, 4 estágios, integração com orçamento |
| [05 — Integrações Externas](./05_integracoes.md) | **Novo:** Prevision (histórico, planejamento); Salesforce, Azure Synapse, Mega ERP, FCM |
| [06 — Perfis e Controle de Acesso](./06_perfis_acesso.md) | Roles, privilege keys, controle por filial |
| [07 — Inteligência Artificial (KAI)](./07_ia_kai.md) | Agentes KAI, RAG, análise de restrições e planos de ação |
| [08 — Cloud Functions e Kaizen API](./08_cloud_functions_api.md) | Funções backend, triggers, notificações, API REST para Data Lake |

---

## Resumo de Capacidades por Domínio

### 🎯 Planejamento e Medições de Produtividade
- **Planejamento Kaizen:** ciclos de produção por atividade (percentual, croqui, ciclo)
- **Medições diárias** com registros de produção, mão de obra e problemas em tempo real
- **Índice de Produtividade** automático e projeção de conclusão
- **Dashboards de Produtividade:** análise por serviço, empreiteira, ou lote

### 📊 Planejamento: Cronograma e Controle de Custos
- **Cronograma com múltiplas versões:** baseline, previsto, realizado
- **Gantt interativo com reprogramação livre** e propagação de dependências
- **Curva S automática** (Scheduled vs. Baseline vs. Done) acumulada por período
- **Importação Prevision:** sincronização automática de cronogramas, atividades, lotes e serviços
- **Histórico de cronogramas** com comparação entre versões
- **Gestão de Equipes** com custom por hora e histograma de alocação

### 📈 Controle: Medições Físicas e Produção
- **Medições Físicas** contratuais com fluxo de aprovação
- **Painel de Produção** em matriz (Monkey Chart) com status por célula
- **Gestão de Equipes** e histograma de ocupação de recursos
- **Planos de Ação** com Kanban, lista, dashboard KPI e notificações push

### 🔧 Rotinas de Gestão Lean (Last Planner System / 6M)
- **Quadro de Restrições** (lookahead 8 semanas) com análise 6M via KAI
- **Planos de Ação** integrados com Kanban, notificações e relatórios
- **Farol de Contratações** com SLA de compra e indicador de risco (verde/amarelo/vermelho)
- **Estabilização Lean:** checklists 5S com scoring histórico (até 24 meses)

### ✅ Qualidade
- **Fichas de Verificação de Serviço (FVS):** templates por tipo de serviço, fluxo aprovação, PDF
- **Inspeção Final de Obra:** técnica (engenheiro) vs. cliente (assinatura e laudo)
- **Inspeção Final da Qualidade:** checklist de materiais, acabamentos, sistemas

### 💰 Controle de Custos e Orçamento
- **Custo vs. Orçado vs. Comprometido vs. Realizado** com desvios categorizados (5 tipos)
- **Versionamento de orçamento** com snapshots de alterações e fluxo de aprovação
- **Integração Synapse:** custo realizado em tempo real do ERP
- **Cotações integradas:** 4 estágios de coleta, análise e aprovação de propostas

### 🏢 Controle de Unidades (Imobiliário)
- **Ficha completa da unidade:** proprietário, status, histórico de mudanças
- **Aba Personalização:** customizações solicitadas com status e prazo
- **Aba Obra:** avanço físico por etapa com fotos de evidência
- **Vistoria do Cliente:** com assinatura digital e geração de Laudo em PDF
- **Inspeção Final:** técnica distinta da vistoria
- **Gerador de Etiquetas QR Code:** folha (12 etiquetas) ou placa individual A4

### 📦 Gestão de Materiais
- **Almoxarifado Digital:** requisições, pedidos, saídas, kits, consulta de saldo, auditoria
- **Integração Mega ERP:** catálogo de produtos, preço, fornecedor, unidade de medida
- **Kits pré-montados:** agregação de itens por atividade (retirada parcial gera requisição)
- **Controle de Ativos:** reserva de veículos com calendário (60 meses), custo, rateio

### Integrações Externas
- **Prevision:** sincronização automática (diária) de cronogramas, atividades, lotes, serviços, Curva S
- **Salesforce:** Work Orders, Service Appointments, Cases, Assets (telemetria a cada 15 min)
- **Azure Synapse:** custo realizado para orçamento, fallback de dados históricos
- **Mega ERP:** catálogo de materiais, saldo de estoque, pedidos de compra
- **Firebase FCM:** notificações push para pedidos de aprovação, planos, restrições, agendamentos

### 🤖 Inteligência Artificial (KAI)
- **Análise automática de restrições** com categorização 6M (Mão de Obra, Material, Máquina, Medição, Meio, Método)
- **Sugestão de novas restrições** com base em cronograma
- **Resumos de documentos** (FVS, planos de ação, relatórios) via LLM
- **RAG por obra:** análise contextual de restrições históricas e soluções passadas

---

## Requisitos de Infraestrutura do Cliente

| Requisito | Detalhe |
|---|---|
| Sistemas de origem | **Prevision** (cronograma); Salesforce CRM (obrigatório para AT/Unidades); Mega ERP (opcional, materiais); Azure Synapse (opcional, BI custo) |
| Conectividade | HTTPS para Google Cloud (backend); API REST Salesforce, Prevision, Synapse, Mega ERP |
| Dispositivos | Browser moderno (Chrome, Edge, Safari); Android 9+ ou iOS 14+ para app nativo |
| Dados exportáveis | Excel (.xlsx) para relatórios; PDF para FVS, planos, etiquetas, auditorias |
| Autenticação | E-mail/senha ou SSO corporativo (Azure AD, Google Workspace para enterprise) |

---

## Contato Técnico

Para dúvidas sobre integrações (Prevision, Salesforce, Synapse, Mega ERP), SLA de API, limites de dados, customizações ou implantação técnica, consulte a equipe técnica do Kaizen.
