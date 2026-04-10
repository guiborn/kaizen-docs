---
layout: page
title: Changelog
permalink: /tecnico/changelog/
toc: true
---

Histórico de alterações e melhorias do sistema Kaizen.

## Sumário

Este documento registra todas as versões e mudanças implementadas no sistema Kaizen, incluindo novos recursos, melhorias e correção de bugs.

---
## Versão 3.7.10
*09/04/2026*

### 1. Painel de Notificações
- **1.1.** Busca e filtro

### 2. Medições Físicas
- **2.1.** Correção de bugs
- **2.2.** Novo gráfico na análise por grupo de serviços
- **2.3.** Otimização do carregamento

### 3. Quadro / Dashboard de Restrições
- **3.1.** Correção de bugs
- **3.2.** Responsável nos marcos
- **3.3.** Editar marcos
- **3.4.** Criar novas restrições a partir da dash

### 4. Controle de Custos
- **4.1.** Liberação do comparativo de versões para usuários normais

### 5. Assistência Técnica
- **5.1.** Novo privilégio de acesso para preenchimento da presença

---

## Versão 3.7.9
*08/04/2026*

### 1. Medição Física / Restrições
- **1.1.** Correção de Bugs

---

## Versão 3.7.8
*08/04/2026*

### 1. Medição Física
- **1.1.** Correção de Bugs

### 2. Restrições
- **2.1.** Correção de Bugs

---

## Versão 3.7.7
*08/04/2026*

### 1. Medições Físicas
- **1.1.** Correção de bugs
- **1.2.** Novas visões e colunas: Previsto Ponderado e Baseline Ponderado
- **1.3.** Finalização e publicação da aba "Relatório"
- **1.4.** Baseline Acumulada e Mensal no tooltip da medição (dentro da tela de análise de medição)
- **1.5.** Nova aba KAI
- **1.6.** Transformação em navigationRail

---

## Versão 3.7.6
*07/04/2026*

### 1. Medições Físicas
- **1.1.** Melhorias no treinamento do modelo de análise
- **1.2.** Melhorias no payload de dados para análise via IA
- **1.3.** Correção de Bugs

### 2. Inspeção Final
- **2.1.** Correção de Bugs
- **2.2.** Configurações - Requisitos (campos obrigatórios em NCs)
- **2.3.** Contador de reinspeções

---

## Versão 3.7.5
*06/04/2026*

### 1. IA
- **1.1.** Barramento oficial KAI
- **1.2.** Clientes com barramento OpenWebUi poderão conectar seus próprios agentes

### 2. Programação Semanal
- **2.1.** Em desenvolvimento

---

## Versão 3.7.4
*02/04/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs

### 2. Restrições
- **2.1.** Correção de bugs (anexos)

### 3. Notificações
- **3.1.** Atalhos para notificações do farol de contratações

---

## Versão 3.7.3
*02/04/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs

### 2. Restrições
- **2.1.** Correção de bugs (anexos)

---

## Versão 3.7.2
*01/04/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs

### 2. Análise Física / Curva S
- **2.1.** Correção de Bugs
- **2.2.** Alteração na lógica de disposição da baseline na curva S
- **2.3.** Novos filtros

### 3. Painel de Produção
- **3.1.** Correção de bugs

### 4. Cronograma / Tabela de Atividades
- **4.1.** Novo campo "A executar no período"
- **4.2.** Novo campo "Previsto Ponderado"

### 5. Gantt
- **5.1.** Correção de Bugs
- **5.2.** Seletor de grupo de serviços
- **5.3.** Filtro por fornecedor

### 6. KAI
- **6.1.** Melhorias no agente de análise de restrições (v6)

---

## Versão 3.7.1
*30/03/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs
- **1.2.** Melhorias na UI e UX
- **1.3.** Melhorias no relatório
- **1.4.** Assinatura de Cliente
- **1.5.** Integração Salesforce

### 2. Dashboard Gerencial de Restrições
- **2.1.** Correção de Bugs
- **2.2.** Edição de restrições
- **2.3.** Por padrão restrições concluídas não carrегam mais na primeira abertura da dashboard

### 3. Assistência Técnica (AT)
- **3.1.** Visualização para usuários read-only

### 4. Farol de Contratações
- **4.1.** Novo conceito de fluxo de contratação (etapas)
- **4.2.** Nova visão Matriz de SLAs

### 5. Login e Rotas
- **5.1.** Correção de bugs e otimizações

### 6. Cache de Usuários
- **6.1.** Otimizações e ranking de interações para ordenamento

---

## Versão 3.7.0
*26/03/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs
- **1.2.** Melhorias na UI e UX
- **1.3.** Novas etapas no fluxo de inspeção:
  - **in_progress** (Em inspeção) - Iniciada pela abertura da inspeção
  - **completed** (Pendências em aberto) - Finalizar com NCs abertos (automático)
  - **awaiting_reinspection** (Aguardando reinspeção) - Acionado pelo time de obra via tela de resumo
  - **pending_reinspection** (Em reinspeção) - Inspetor via card "Iniciar Reinspeção"
  - **closed** (Encerrada) - Inspetor finaliza (sem NCs ou após reinspeção)

### 2. Medição Física
- **2.1.** Warning para medições parciais não marcadas como prévia
- **2.2.** Visão Baseline, Previsto e filtros na tabela de atividades
- **2.3.** Vínculo de Restrições direto pela medição

### 3. KAI
- **3.1.** Sugestão de restrições pelos desvios apontados na medição física

---
## Versão 3.6.9
*26/03/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Correção de Bugs
- **1.2.** Integração Salesforce

### 2. Farol de Contratações
- **2.1.** Correção de Bugs

### 3. Dashboard Gerencial de Restrições
- **3.1.** Auto-escalonamento por prazo determinado
- **3.2.** Filtros múltiplos

### 4. KAI
- **4.1.** Melhorias na análise de restrições

### 5. Controle de Custos
- **5.1.** Correção de Bugs
- **5.2.** Data base para estoque com parceiros

---

## Versão 3.6.8
*25/03/2026*

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Integração Salesforce
- **1.2.** Melhorias na UX da criação de máscaras e gestão de pendências
- **1.3.** Mini Dash por inspeção

### 2. Farol de Contratações
- **2.1.** Correção de Bugs

### 3. Dashboard Gerencial de Restrições
- **3.1.** Desenvolvimento

### 4. KAI
- **4.1.** Melhorias no KAI Restriction Analyst
- **4.2.** Lançamento do KAI Chat
- **4.3.** Concepção do KAI 360

---

## Versão 3.6.7

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Sistema de Notificações
- **1.2.** Dashboard de NCs por empreendimento

### 2. Assistência Técnica
- **2.1.** Correção de Bugs

### 3. Quadro de Restrições
- **3.1.** Filtro por status na dashboard
- **3.2.** Matriz de risco
- **3.3.** Escalonamento
- **3.4.** Marcos, Progresso, Vínculos, Chat e Anexos

### 4. Cronograma Físico-Financeiro (CFF)
- **4.1.** Extensão de prazos

---

## Versão 3.6.6

### 1. Controle de Custos
- **1.1.** Estoque e estoque com parceiros

### 2. Medições Físicas
- **2.1.** Criação da API de Medições Físicas e documentação

### 3. Cronograma Físico-Financeiro (CFF)
- **3.1.** Persistência de dados do CFF para otimização do carregamento

### 4. Inspeção Final da Qualidade (IFQ)
- **4.1.** Subsistemas e melhorias na criação de máscaras
- **4.2.** Gestão de Máscaras e vínculo de unidades
- **4.3.** Classificação rápida e auditoria de NCs
- **4.4.** Melhorias no relatório de IFQ (PDF)

### 5. Unidades
- **5.1.** Persistência

---

## Versão 3.6.5

### 1. Análise de Cronograma e Vínculos com Orçamento
- **1.1.** Correção de Bugs

### 2. Assistência Técnica
- **2.1.** Anexos no pedido

---

## Versão 3.6.4

### 1. Acesso
- **1.1.** Correção de Bugs
- **1.2.** Grupos de Usuários
- **1.3.** Otimização de convites
- **1.4.** Controle de acesso
- **1.5.** Remoção de users inativos do cache de menções

### 2. Assistência Técnica
- **2.1.** Correção de Bugs
- **2.2.** Notificações de Pedidos (apenas admins e almoxarife)
- **2.3.** Solicitante dos Pedidos
- **2.4.** "Marcar minha Ausência" para colaboradores
- **2.5.** KAI agora roda em segundo plano na criação de itens de linha

---

## Versão 3.6.3

### 1. Módulo de Acesso
- **1.1.** Correção de Bugs

### 2. Farol de Contratações
- **2.1.** Correção de Bugs
- **2.2.** Logs de Ações
- **2.3.** Otimização do carregamento de processos
- **2.4.** Notificações do Chat e mudança de responsável

### 3. Médio Prazo
- **3.1.** Correção de Bugs
- **3.2.** Melhorias na criação de restrições em massa

### 4. Almoxarifado Digital
- **4.1.** Gerador de QR Code
  - **4.1.1.** Logo da empresa
  - **4.1.2.** Grupos de Itens
  - **4.1.3.** Itens que não estão no estoque

### 5. Medição Física
- **5.1.** Correção de Bugs
- **5.2.** Liberação para marcação das últimas 5 medições como prévia e aplicação ao cronograma (apenas o ativo)
- **5.3.** Melhorias na exportação do relatório de medição (csv → xlsx)
- **5.4.** Carregar realizado da última prévia

---

## Versão 3.6.2

### 1. Inspeção Final da Qualidade (IFQ)
- **1.1.** Aba de gerenciamento de pendências

### 2. Medição Física
- **2.1.** Correção de Bugs

---

## Versão 3.6.1

### 1. Planos de Ação
- **1.1.** Criação de novo agente KAI ActionPlan Analyst, especializado em analisar planos de ação e gerar recomendações de melhoria, com base em dados históricos da obra, melhores práticas do setor e diretrizes estratégicas da empresa. O agente é capaz de identificar gaps nos planos de ação, sugerir ações mais eficazes e priorizar as recomendações com base no impacto esperado e na facilidade de implementação.

### 2. Restrições
- **2.1.** Criação de novo agente KAI Restriction Analyst, especializado em analisar as restrições identificadas no projeto e gerar recomendações de mitigação, com base em dados históricos da obra, melhores práticas do setor e diretrizes estratégicas da empresa. O agente é capaz de identificar gaps nas restrições, sugerir estratégias mais eficazes de mitigação e priorizar as recomendações com base no impacto esperado e na facilidade de implementação.

### 3. Módulo de Medições Físicas
- **3.1.** Correção de Bugs

### 4. Módulo Consolidador de Relatórios IA
- **4.1.** Em desenvolvimento

---

## Versão 3.6.0

### 1. Controle de Custos
- **1.1.** Correção de Bugs

### 2. Assistência Técnica
- **2.1.** Central de Notificações
- **2.2.** Possibilidade do colaborador marcar a própria ausência

### 3. Painel de Produção
- **3.1.** Zoom

---

## Versão 3.5.9

### 1. Módulo de Medições
- **1.1.** Correção de Bugs

### 2. Painel de Produção
- **2.1.** Heatmap Mensal
- **2.2.** Heatmap Semanal
- **2.3.** Gestão de Restrições
- **2.4.** Gestão de Planos de Ação
- **2.5.** Totalizador Base Previsto Realizado
- **2.6.** Filtros rápidos Base Previsto Realizado
- **2.7.** Filtro por fornecedor

---

## Versão 3.5.8

### 1. Módulo de Controle de Custos (HPO)
- **1.1.** Apontamento e acompanhamento de distratos a partir da lista de contratos

### 2. Painel de Produção
- **2.1.** Correção de Bugs
- **2.2.** Filtro por grupo de serviços

### 3. Análise da Medição Física
- **3.1.** Correção de Bugs
- **3.2.** Impacto com relação à baseline (desabilitada por padrão)

### 4. Farol de Contratações
- **4.1.** Correção de Bugs
- **4.2.** Novo Layout de filtro
- **4.3.** Data da versão
- **4.4.** Funcionalidade de marcação de itens como concluídos mesmo sem processo vinculado
- **4.5.** Melhorias no fluxo de carregamento das atividades e etapas de itens antes do carregamento de seus processos
- **4.6.** Otimização do Exportável do Farol

---

## Versão 3.5.7

### 1. Módulo de Controle de Custos (HPO)
- **1.1.** Revisão e divisão dos níveis de acesso
  - **a)** Acesso total
  - **b)** Acesso de edição
  - **c)** Apenas visualização
- **1.2.** Script para apontamento de desvios múltiplos
- **1.3.** Melhoria no relatório de distratos
- **1.4.** Persistência dos dados do Synapse (todos os dias 4 AM) para agilizar o carregamento

### 2. Painel de Produção
- **2.1.** Ordenamento de Lotes e serviços ao clicar na linha de cabeçalho
- **2.2.** Correção de Bugs

### 3. Análise da Medição Física
- **3.1.** Correção de Bugs

### 4. Medição Física
- **4.1.** Correção de Bugs
- **4.2.** Otimização da medição simultânea em diferentes dispositivos

---

## Versão 3.5.6

### 1. Módulo de Controle de Custos (HPO)
- **1.1.** Correção de Bugs
- **1.2.** Possibilidade de apontamento de distratos completos
- **1.3.** Assistente de vinculação de contratos e planejamento

### 2. Módulo de Inspeção Final da Qualidade
- **2.1.** Desenvolvimento do módulo

### 3. Módulo de Medições Físicas
- **3.1.** Importar múltiplas medições do Prevision
- **3.2.** Exportação do Cronograma Excel melhorado

### 4. Planos de Ação
- **4.1.** Correção de Bugs

---

## Versão 3.5.5

### 1. Módulo de Planejamento / Medições Físicas
- **1.1.** Correção de bugs

---

## Versão 3.5.4

### 1. Módulo de Planejamento / Medições Físicas
- **1.1.** Importação do caminho crítico
- **1.2.** Separação de acessos ao módulo de planejamento

### 2. Módulo de Custos
- **2.1.** Correção de bug para expansão do realizado
- **2.2.** Importação de vínculos de orçamento do Prevision

### 3. Página Inicial
- **3.1.** Atualização da UI da tela de seleção de obras
- **3.2.** Atualização da UI da tela de menu da obra

### 4. Sistema de Notificações Otimizado
- **4.1.** Vínculo com Planos de Ação
- **4.2.** Vínculo com Restrições
- **4.3.** Vínculo com AT

### 5. Quadro de Restrições
- **5.1.** Otimização smartphones
- **5.2.** Repaginação do Kanban de Restrições

### 6. Módulo de Qualidade
- **6.1.** Tela de gestão de FVS

---

## Versão 3.5.3

### 1. Módulo Almoxarifado Digital
- **1.1.** Correção de Bugs
- **1.2.** Permitir conversão automática de kits
- **1.3.** Erro ao dar saída no 2º item no mobile (textfield disposed antes da hora)
- **1.4.** Agrupamento de baixas por insumo
- **1.5.** Consulta de estoque considerar baixas em aberto

### 2. Módulo de Medições Físicas
- **2.1.** Correção de bugs
- **2.2.** Observação Geral da Medição
- **2.3.** Possibilidade de abertura de planos de ação a partir de observações da medição e de cada atividade atrasada (vinculando o serviço ou atividade)
- **2.4.** Importação de medições diretamente do Prevision com opção de escolha de cronograma vinculado

### 3. Módulo de Planos de Ação
- **3.1.** Criação da API de leitura de planos de ação para integração com datalakes
- **3.2.** Possibilidade de vincular planos de ação a serviços ou atividades específicas dentro de uma medição, para análises mais precisas

### 4. Módulo de Assistência Técnica
- **4.1.** Correção de Bugs
- **4.2.** Simplificação do diálogo de abertura de itens

### 5. Módulo Farol de Contratações
- **5.1.** Correção de Bugs
- **5.2.** Repaginação do diálogo de informações do processo
- **5.3.** Exportação no diálogo de informações do processo

---

## Versão 3.5.2

### 1. Módulo Almoxarifado Digital

### 2. Módulo de Medições
- **2.1.** Correção de Bugs

### 3. Módulo de Planos de Ação
- **3.1.** Correção de Bugs

### 4. Módulo de Medição Diária
- **4.1.** Otimização do layout para web

### 5. Integrações
- **5.1.** Correção de Bugs - Integração Salesforce
- **5.2.** Otimizações da integração com Synapse
- **5.3.** Otimizações da integração com Mega

---

## Versão 3.5.1

### 1. Módulo Almoxarifado Digital
- **1.1.** Requisições
- **1.2.** Pedidos de Suprimentos
- **1.3.** Controle de Kits
- **1.4.** Gerador de Etiquetas
- **1.5.** Painel de Acompanhamento de Requisições e Kits (TV de Almoxarifado)
- **1.6.** Consulta Estoque
- **1.7.** Relatório de Saídas
- **1.8.** Saídas por QR Code

---

## Versão 3.5.0

### 1. Módulo Assistência Técnica
- **1.1.** Correção de Bugs - Solicitação

---

## Versão 3.4.9

### 1. Integrações Salesforce
- **1.1.** Método para troca de ambiente

### 2. Farol de Contratações
- **2.1.** Correção de Bugs

### 3. Módulo de Planos de Ação
- **3.1.** Correção de Bugs

---

## Versão 3.4.8

### 1. Módulo de Medições Físicas
- **1.1.** Correção de Bugs
- **1.2.** Análise de Problemas
- **1.3.** Indicadores Preditivos
- **1.4.** Categorização dos problemas via IA - KAI ProblemIdentifier
- **1.5.** Relatório de IA - KAI PlanReporter

### 2. Módulo de AT
- **1.1.** Correção de Bugs - Criação de OT
- **1.2.** Correção de Bugs - Presença e agendamento de férias/afastamentos
- **1.3.** Correção de Bugs - Destravada a finalização de OT com agendamento cancelado

---

## Versão 3.4.7

### 1. Módulo Kaizen Diário
- **1.1.** Correção de Bugs

### 2. Módulo de AT
- **2.1.** Ordenamento das OTs na visão Kanban
- **2.2.** Travamento do cabeçalho no Gantt
- **2.3.** Gestão de status nos cartões de agendamento ou dentro da aba da janela de informações da OT
- **2.4.** Novo status "A confirmar" para agendamentos

### 3. Módulo Cronograma de Contratações
- **3.1.** Ajustes na UI
- **3.2.** Otimização do carregamento (separação das funcionalidades "Gerar Farol" e "Abrir Farol")
- **3.3.** Versionamento de Farol

### 4. Módulo de Restrições
- **4.1.** Otimização no modelo de IA para geração mais rápida de restrições
- **4.2.** Criação de restrições em massa a partir de recomendações de IA

### 5. Módulo de Controle de Custos
- **5.1.** Correção de Bugs
- **5.2.** Trava para criação de desvios em níveis de resumo
- **5.3.** Correção de campos da API Kaizen (desvios → dataRealizado representará a data de aprovação do desvio)

### 6. Módulo de Planos de Ação
- **6.1.** Correção de Bugs
- **6.2.** Otimizações Mobile
- **6.3.** Melhoria das buscas e filtros
- **6.4.** Início dos Testes

### 7. Módulo de Unidades
- **7.1.** Vinculação com cronograma
- **7.2.** Personalização (conectado ao Salesforce)
- **7.3.** Vistoria da Qualidade (conectado ao Salesforce) - em desenvolvimento

---

## Versão 3.4.6

### 1. Módulo Plano de Ação
- **1.1.** Finalização do Módulo (pronto para testes)
- **1.2.** Otimização Mobile
- **1.3.** Visão Kanban em diferentes filtros (Risco, Setor, Status)
- **1.4.** Matriz de Risco

### 2. Módulo de AT
- **2.1.** Possibilidade de exclusão de mensagens e fotos do feed
- **2.2.** Possibilidade de exclusão de itens de linha da OT (apenas para admins)
- **2.3.** Correção de Bugs

---

## Versão 3.4.5

### 1. Módulo de AT
- **1.1.** Correção de Bugs
- **1.2.** Reorganização e filtro de cards de agendamento dentro da tela de detalhes da OT
- **1.3.** Otimização do uso do Modelo de IA para categorização dos itens da OT, tornando o preenchimento mais rápido e eficiente

---

## Versão 3.4.4

### 1. Módulo de AT
- **1.1.** Possibilidade de Agendamento Multi-dia
- **1.2.** Férias e afastamentos agendados agora aparecem no Gantt
- **1.3.** Justificativas de cancelamento ao cancelar um agendamento
- **1.4.** Agendamentos cancelados não aparecerão, por padrão na tela inicial (mas ainda podem ser acessados clicando no filtro nos cards acima)
- **1.5.** Agrupamento por unidade no Kanban
- **1.6.** Filtro por Supervisores na tela inicial
- **1.7.** Horário de término nos cards

### 2. Módulo de Pendências de Obra
- **2.1.** Início da refatoração do módulo, que englobará também planos de ação

---

## Versão 3.4.3

### 1. Módulo de AT
- **1.1.** Correção de Bugs
- **1.2.** Atualizações na dashboard (Legendas, custo por chamado)
- **1.3.** Possibilidade de criar OTs dentro do diálogo de detalhes de caso
- **1.4.** Gantt agora considera agendamentos que englobam dois turnos
- **1.5.** Presença: opção de edição em massa de registros de presença
- **1.6.** Presença: opção de agendar férias e afastamentos
- **1.7.** Multi apropriação de insumos na solicitação
- **1.8.** Tela de gestão de pedidos
- **1.9.** Seção "Novidades"

### 2. Módulo de Controle de Custos de Obra (HPOV2)
- **2.1.** Gestão de Contratos - Exportação

---

## Versão 3.4.2

### 1. Módulo de AT
- **1.1.** Correção de Bugs (inclusive bug crítico de UTC)
- **1.2.** Início dos testes de solicitação (Mega)
- **1.3.** Visões de agendamentos "Para Mim" e "Por Mim"

---

## Versão 3.4.1

### 1. Módulo de AT
- **1.1.** Finalização da Integração com o Mega
- **1.2.** Início do desenvolvimento do formulário de solicitação (Mega)

### 2. Módulo de Controle de Custos (HPOV2)
- **2.1.** Correção de Bugs
- **2.2.** Tela de Gestão de Contratos
- **2.3.** Análise de insumos

### 3. Módulo de Unidades
- **3.1.** Macaquinho
- **3.2.** Personalização
- **3.3.** Integração Salesforce

---

## Versão 3.4.0

### 1. Módulo de AT
- **1.1.** Correção de bugs
- **1.2.** Horário de conclusão da OT para o Salesforce
- **1.3.** Conclusão da OT para não-admins
- **1.4.** Fotos dos itens da OT agora vão para o feed

### 2. Módulo de Controle de Custos (HPOV2)
- **2.1.** Correção de Bugs
- **2.2.** Atualização da máscara de orçamento
- **2.3.** Análise de insumos

---

## Versão 3.3.9

### 1. Módulo de AT
- **1.1.** Correção de bugs
- **1.2.** Divisão de colaboradores por filial

---

## Versão 3.3.8

### 1. Módulo de AT
- **1.1.** Correção de bugs
- **1.2.** Modo Offline
- **1.3.** Busca por número da OT na home (cartões de agendamento)
- **1.4.** Otimização de views mobile

---

## Versão 3.3.7

### 1. Módulo de Farol de Contratações
- **1.1.** Correção de bugs
- **1.2.** Novo filtro de pesquisa de atividades na tela de planejamento

### 2. Melhorias Gerais
- **2.1.** Correção de bugs gerais

### 3. Tela de Planejamento
- **3.1.** Seleção de atividades na tela de planejamento (para selecionar, dar o toque/clique duplo sobre a atividade desejada)
- **3.2.** Novo menu de ações para as atividades selecionadas na tela de planejamento (agora a liberação de serviços e impressão dos mapas de produção é feita por este menu)
- **3.3.** Otimização da sincronização de planos de trabalho
- **3.4.** Filtro por torre
- **3.5.** Possibilidade de ocultar atividades (apenas para o dispositivo)
- **3.6.** Diferenciação de atividades vinculadas aos serviços de produtividade controlada

### 4. Sincronização e Croquis
- **4.1.** Lembrete sincronização para todos os dispositivos conectados ao projeto do Prevision (os usuários administradores devem entrar no menu de configurações da obra e pressionar o botão "Solicitar Sincronização Prevision")
- **4.2.** Botão para sincronizar todos os croquis do projeto (aba de croquis das configurações da obra)

### 5. Módulo de Checklist de Estabilização Lean
- **5.1.** Introdução de versionamento e atualização dos checklists para o novo modelo proposto no Guia Lean
- **5.2.** Nova visão de desempenho de acordo com a categoria dos itens
- **5.3.** Mudanças gerais na aparência do módulo
- **5.4.** Emissão de relatório da inspeção em PDF

### 6. Dashboard de Estabilização Lean
- **6.1.** Caixa de seleção de versão do checklist

### 7. Tela de Criação de Mapas de Medição
- **7.1.** Inserção da unidade e critério de medição nas informações

### 8. Impressão de Mapas de Medição
- **8.1.** Agora é possível utilizar tags ao invés do nome do lote no mapa de medição (ex: 1P1 → Lote 1)

---

## Notas Importantes

Este changelog é um registro histórico do desenvolvimento do sistema Kaizen. Cada versão marca um conjunto de melhorias, novas funcionalidades e correções implementadas, permitindo aos usuários acompanhar a evolução do produto.

Para mais informações sobre módulos específicos, consulte a [documentação dos módulos de gestão de obra]({{ '/tecnico/modulos/' | relative_url }}).
