---
layout: page
title: "Documentação Técnica"
permalink: /kaizen/tecnico/
category: "Índice"
order: 0
toc: false
---

# Kaizen — Documentação Técnica para Precificação

> **Público-alvo:** Equipes de TI de empresas interessadas em adquirir o Kaizen, para fins de avaliação técnica e precificação interna.
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
| [02 — Módulos Gestão de Obras](./02_modulos_gestao_obra.md) | Todos os módulos da plataforma de obras (~25 funcionalidades) |
| [03 — Módulo Assistência Técnica](./03_modulo_assistencia_tecnica.md) | AT: work orders, agendamentos, colaboradores, analytics |
| [04 — Módulo Cotações](./04_modulo_cotacoes.md) | Mapas de cotação, estágios, integração com orçamento |
| [05 — Integrações Externas](./05_integracoes.md) | Salesforce CRM, Azure Synapse, Mega ERP |
| [06 — Perfis e Controle de Acesso](./06_perfis_acesso.md) | Roles, privilege keys, controle por filial |
| [07 — Inteligência Artificial (KAI)](./07_ia_kai.md) | Agentes KAI, RAG, análise de restrições e planos de ação |
| [08 — Cloud Functions e Kaizen API](./08_cloud_functions_api.md) | Funções backend, triggers, notificações, API REST para Data Lake |

---

## Resumo de Capacidades por Domínio

### Planejamento de Obra
- Cronograma com múltiplas versões (baseline, previsão, realizado)
- Gantt interativo com filtros por criticidade, serviço e lote
- Curva S automática (planejado vs realizado)
- Medições diárias de produtividade e avanço físico
- EAP completa (torres, pavimentos, lotes, serviços, atividades)

### Rotinas de Gestão Lean
- Planejamento de médio prazo em quadro de restrições (look-ahead 8 semanas)
- Planos de ação com Kanban, notificações push e relatório PDF
- Estabilização básica com checklists de campo por torre/pavimento
- Farol de contratações integrado ao cronograma

### Qualidade
- Fichas de Verificação de Serviço (FVS): criação, preenchimento, aprovação e PDF
- Inspeções de estabilização Lean com histórico e comparativo anual

### Custo e Suprimentos
- Orçamento paramétrico vs realizado com integração ERP
- Desvios categorizados (orçamento, custo unitário, engenharia, prazo, projeto)
- Almoxarifado digital com pedidos e integração Mega ERP
- Mapas de cotação com fluxo de 4 estágios

### Controle de Unidades (Imobiliário)
- Catálogo de unidades sincronizado do Salesforce
- Gestão de personalização de unidades com checklist
- Gerador de etiquetas QR Code (folha ou placa individual)
- Rastreamento de inspeções finais e vistoria com cliente

### Assistência Técnica
- Integração bidirecional com Salesforce (Work Orders, Cases, Service Appointments)
- Agendamentos em tempo real com suporte offline
- Analytics de desempenho por técnico, empreendimento e período

---

## Requisitos de Infraestrutura do Cliente

| Requisito | Detalhe |
|---|---|
| Sistemas de origem | Salesforce CRM (obrigatório para AT e Unidades); ERP Mega (opcional, para materiais); Azure Synapse (opcional, para BI de custo) |
| Conectividade | Acesso HTTPS para infraestrutura de backend (Google Cloud); acesso à API REST do Salesforce |
| Dispositivos | Qualquer browser moderno (Chrome, Edge, Safari); Android 9+ ou iOS 14+ para app nativo |
| Dados exportáveis | Excel (.xlsx) para relatórios; PDF para FVS, planos de ação e etiquetas |
| Autenticação | Autenticação por e-mail/senha ou SSO corporativo (para clientes enterprise) |

---

## Contato Técnico

Para dúvidas sobre integrações, SLA de API, limites de dados ou customizações, consulte a equipe técnica do Kaizen.
