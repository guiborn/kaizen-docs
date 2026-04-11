---
layout: page
title: "Inteligência Artificial - KAI"
permalink: /tecnico/ia/
category: "IA"
order: 7
toc: true
---

# Kaizen — Inteligência Artificial (KAI)

## Sumário
1. [O que é o KAI](#1-o-que-é-o-kai)
2. [Infraestrutura de IA](#2-infraestrutura-de-ia)
3. [Agentes Disponíveis](#3-agentes-disponíveis)
4. [KAI 360 — Resumo Executivo](#4-kai-360--resumo-executivo)
5. [KAI — Analista de Restrições](#5-kai--analista-de-restrições)
6. [KAI — Analista de Planos de Ação](#6-kai--analista-de-planos-de-ação)
7. [KAI — Relatório de Planejamento](#7-kai--relatório-de-planejamento)
8. [KAI — Sugestão Inteligente de Restrições](#8-kai--sugestão-inteligente-de-restrições)
9. [KAI — Identificação de Problemas](#9-kai--identificação-de-problemas)
10. [KAI Chat](#10-kai-chat)
11. [Base de Conhecimento (RAG)](#11-base-de-conhecimento-rag)
12. [Arquitetura dos Agentes KAI](#12-arquitetura-dos-agentes-kai)
13. [Segurança e Privacidade](#13-segurança-e-privacidade)

---

## 1. O que é o KAI

**KAI** é o assistente de inteligência artificial nativo do Kaizen, especializado em gestão de obras de construção civil. Diferente de assistentes genéricos, o KAI é treinado com contexto específico de construção civil: metodologia 6M, caminho crítico do cronograma, Last Planner System, análise gerencial de restrições e planos de ação.

Os agentes KAI são implementados como **Cloud Functions** no backend do Kaizen — o app Flutter dispara a análise via chamada autenticada ao backend, e o KAI retorna um relatório estruturado em markdown. Nenhum dado sensível do cliente trafega diretamente para serviços de IA externos: toda orquestração ocorre dentro do ambiente seguro das Cloud Functions, com validação de autenticação em cada requisição.

### Casos de Uso Principais

| Funcionalidade | Quando usar |
|---------------|-------------|
| Análise de Restrições | Relatório executivo + ações operacionais priorizadas do quadro de restrições |
| Análise de Planos de Ação | Dashboard gerencial de execução com alertas de vencimento e caminho crítico |
| Relatório de Planejamento | Visão analítica do cronograma para reuniões de produção |
| Sugestão de Restrições | Identificação automática de impedimentos a partir do contexto da obra |
| Identificação de Problemas | Diagnóstico de não conformidades e anomalias na execução |
| KAI Chat | Perguntas e respostas contextuais sobre restrições e outros módulos |

---

## 2. Infraestrutura de IA

### 2.1 Camada Centralizada

O **KAI** é a infraestrutura centralizada de inteligência artificial da Kaizen GDO, hospedada em servidor dedicado com disponibilidade contínua e comunicação criptografada via HTTPS. Todos os agentes operam sobre essa fundação técnica compartilhada.

### 2.2 Acesso via API

O KAI expõe um endpoint compatível com o padrão **OpenAI**, o que significa que qualquer sistema já integrado com modelos de IA consegue se conectar ao KAI com mudança mínima de configuração — basta apontar para o endpoint da Kaizen e usar a chave de acesso fornecida.

**Endpoint:** `https://kai.kaizen-gdo.com/api/chat/completions`  
**Autenticação:** Bearer Token (chave individual por tenant)  
**Protocolo:** HTTPS + WebSocket

### 2.3 Roteamento de Modelos

O KAI roteia as requisições para os melhores modelos disponíveis conforme a necessidade de cada aplicação, incluindo modelos de última geração. Novos modelos são incorporados sem necessidade de alteração no sistema do cliente.

### 2.4 Arquitetura Multi-Tenant

Cada cliente ou sistema integrado recebe uma chave de acesso própria, com rastreabilidade de uso individual. É possível definir limites de uso por chave e revogar acessos de forma independente sem impactar os demais.

### 2.5 Compatibilidade

Qualquer aplicação que consuma APIs REST consegue se integrar ao KAI — mobile (Flutter, React Native), web, ou sistemas backend.

---

## 3. Agentes Disponíveis

O KAI opera através de modelos de linguagem especializados, cada um com prompt de sistema dedicado ao seu domínio:

| Agente / Modelo | Cloud Function | Domínio |
|----------------|---------------|---------|| `kai-360-executive` | `generateKai360Report` | Resumo executivo consolidado (multi-módulo) || `kai-restriction-analyst` | `generateRestrictionReport` | Análise gerencial de restrições de obra |
| `kai-actionplan-analyst` | `generateActionPlanReport` | Análise de planos de ação e execução |
| `kai-plan-analyst` | `generatePlanReport` | Análise do planejamento e cronograma |
| `kai-suggestion` | `suggestRestrictions` | Sugestão automática de restrições a catalogar |
| `kai-problems` | `identifyProblems` | Identificação de problemas na execução |
| `kai-chat` | `kaiChat` | Chat contextual multiuso para perguntas operacionais e gerenciais |

Todos os agentes compartilham a mesma camada de autenticação e seguem o mesmo padrão de invocação.

---

## 4. KAI 360 — Resumo Executivo

### 4.1 Objetivo

O **KAI 360** é um agente agregador que consolida insights de múltiplos módulos da obra (Restrições, Planejamento Físico, Planos de Ação) em um **relatório executivo holístico**. Diferente dos agentes especializados tradicionais, o KAI 360 oferece uma visão 360° integrada, permitindo que gestores e diretores entendam o contexto completo da obra em uma única análise.

### 4.2 Modo de Operação

O KAI 360 opera em **dois modos de análise**:

#### Modo **Obra Única** (Single-Site)
- Seleção de uma obra específica
- Módulos selecionáveis (Restrições, Medições Físicas, Planos de Ação)
- Análise profunda focada em uma obra
- Relatório sem repetição de nome da obra em cada seção

#### Modo **Multi-Obra**
- Seleção de múltiplas obras da mesma filial
- Ranking consolidado por saúde geral
- Agrupamento de riscos críticos por obra
- Síntese executiva com destaques por obra
- Ideal para **Dashboard Gerencial de Restrições** e visualizações de portfólio

### 4.3 Dados Analisados

O agente coleta resumos executivos de cada módulo e sintetiza:

**De Restrições:**
- Status e categoria (6M) de cada restrição aberta
- Impacto no caminho crítico
- Vencimentos e reagendamentos
- Responsáveis e setores sobrecarregados
- KPIs: total aberto, vencidas, reagendadas 2+ vezes

**De Medições Físicas:**
- Progresso físico vs. planejado por atividade
- Desvios críticos e alertas de atraso
- Avançado/Atrasado por período
- Curva S atual vs. baseline

**De Planos de Ação:**
- Execução de planos (% completo, milestones)
- Planos vencidos e em risco
- Impacto em caminho crítico
- Extrapolações orçamentárias

### 4.4 Estrutura do Relatório Gerado

```markdown
## 🎯 Sumário Executivo
Frase única sintetizando saúde geral e principais riscos/oportunidades.

## 🚦 Semáforo de Saúde
Indicador visual (🟢 Verde / 🟡 Amarelo / 🔴 Vermelho) por obra (modo multi) ou obra única.

## 📊 Ranking de Obras (modo multi-obra)
Tabela com: Obra | Saúde | Restr. Críticas | Atraso Físico | Planos Vencidos

## 🔴 Riscos Críticos Priorizados
Até 5 riscos de maior impacto consolidados — causa, impacto potencial, obra afetada.

## ✅ Decisões Necessárias Agora
Ações imediatas para desbloquear progresso — o que precisa ser decidido nos próximos dias.

## 🎬 Próximos Passos Operacionais
Lista priorizada de atividades (máx. 5) para os próximos 7 dias.

## 🏆 Destaques Positivos (Wins)
Oportunidades, progressos notáveis ou áreas saudáveis a replicar.

## 📋 Detalhamento por Módulo (opcional)
Seções expandidas de Restrições, Medições e Planos se houver dados relevantes.
```

### 4.5 Saúde Geral (Health Status)

O agente calcula um **semáforo consolidado** considerando:

- **🔴 Vermelho:** Qualquer módulo com saúde crítica (restrições críticas vencidas, atraso > 15% no físico, planos críticos atrasados)
- **🟡 Amarelo:** Alertas significativos mas controláveis (restrições amarelas, atraso 5-15%, planos vencidos em recuperação)
- **🟢 Verde:** Todos os módulos dentro do esperado

### 4.6 Freshness (Atualidade dos Dados)

O relatório inclui informação de **quanto tempo os dados foram gerados**:

- **Fresh (0 dias):** Report da ordem de geração
- **Stale (1-3 dias):** Dados de alguns dias atrás, próximo refresh recomendado
- **Outdated (>3 dias):** Report antigo, atualização fortemente recomendada
- **Missing:** Módulo sem dados disponíveis

### 4.7 Onde Aparece no App

- **Drawer principal:** Botão "KAI 360" (gated por privilégio `kai_insights_view`)
- **View dedicada:** `KAI360View` com setup interativo (seleção de obras/módulos) e relatório em tempo real
- **Dashboard Gerencial:** Integração opcional em dashboard de restrições modo multi-obra
- **Export:** Relatório markdown pode ser copiado para clipboard ou exportado

### 4.8 Regras de Síntese

1. **Sem duplicação:** Cada fato é mencionado uma única vez no relatório consolidado
2. **Sem ficção:** Se não há dados suficientes em um módulo, o agente é honesto ("Sem restrições vencidas no momento")
3. **Linguagem executiva:** Sem nomes de campos internos (`hasCriticalActivities`, `isOverdue`); apenas descrição natural
4. **Priorização objetiva:** Risco = Severidade × Impacto × Urgência
5. **Nomes das obras:** Aparecem apenas quando necessário para clareza (multi-obra) ou para deep links

### 4.9 Deep Links e Navegação

O relatório inclui **deep links contextuais** para:

- "Ver Restrições Críticas" → abre Quadro de Restrições da obra
- "Detalhe de Medições" → abre Controle de Produção com período relevante
- "Acompanhar Planos" → abre módulo de Planos de Ação
- "Chat KAI" → abre KAI Chat com contexto pré-carregado

### 4.10 Performance e Cache

- **Tempo de resposta:** 8-20s (coleta dados + LLM synthesis)
- **Cache:** Reports são salvos em `{clientId}/kai360Reports/{docId}` com TTL de 24h
- **Regeneração:** Botão explícito para forçar nova análise (limpa cache)
- **Modo offline:** Se Firestore está indisponível, exibe "Desculpe, não consigo carregar os dados agora"

---

## 5. KAI — Analista de Restrições

### 4.1 Objetivo

Gerar um **relatório gerencial de restrições de obra** com dois públicos simultâneos:
- **Gestores/Diretores:** briefing executivo estratégico com painel de indicadores
- **Equipe de campo/Engenharia:** lista priorizada de ações operacionais concretas

### 4.2 O que o KAI analisa

O modelo recebe as restrições catalogadas no Quadro de Restrições da obra (com todos os campos estruturados) e produz uma análise orientada ao futuro — o que precisa ser desbloqueado e quando — evitando inventar dados ausentes.

O agente agora opera em dois modos:
- **Obra única:** relatório focado em uma obra, sem repetir o nome da obra em cada item.
- **Multi-obra:** usado no Dashboard Gerencial de Restrições, com agrupamento por obra e identificação explícita da obra em itens críticos e ações priorizadas.

Além disso, a resposta final é forçada a usar **linguagem executiva e operacional**, sem expor nomes técnicos de campos, flags ou parâmetros internos.

**Campos analisados por restrição:**

| Campo | Interpretação |
|-------|--------------|
| `status` | Pendente / Em andamento / Concluído / Cancelado |
| `type` | Categoria 6M (Mão de Obra, Materiais, Máquinas, Método, Medição, Meio Ambiente) |
| `responsible` / `sector` | Identificação de sobrecarga por responsável ou setor |
| `dueDate` / `isOverdue` | Alertas de vencimento e urgência |
| `openDays` | Dias desde a abertura da restrição |
| `timesRescheduled` | Détecção de problema sistêmico (reagendamentos ≥ 2) |
| `hasCriticalActivities` | Impacto no caminho crítico do cronograma |
| `linkedActivities[].isCritical` | Atividades específicas bloqueadas pelo nome |
| `escalationLevel` | Nível de escalação gerencial da restrição |
| `siteName` | Nome da obra quando a análise está em modo multi-obra |
| `priority` / `risk` | Priorização cruzada de urgência vs. impacto |

### 4.2.1 Persistência de criticidade

Em telas que não carregam o cronograma completo, o sistema passa a utilizar um **snapshot persistido** no próprio objeto da restrição para informar se há impacto no caminho crítico e quais atividades relevantes estão vinculadas. Isso permite que o KAI multi-obra mantenha qualidade analítica mesmo fora do contexto do planejamento detalhado.

### 4.3 Estrutura do Relatório Gerado

```markdown
## 📋 Painel Executivo
Indicadores-síntese: total aberto, vencidas, reagendadas 2+ vezes, impactando caminho crítico.
Semáforo geral de saúde da obra.

## 🏗️ Análise por Obra
(apenas no modo multi-obra)
Agrupa totais, vencimentos, impacto em caminho crítico e semáforo por obra.

## 🔴 Restrições Críticas Prioritárias
(restrições com impacto crítico, vencidas ou reagendadas múltiplas vezes)

## 📊 Análise por Categoria (6M)
Distribuição e tempo médio em aberto por categoria.

## 🔺 Análise por Nível de Escalação
Concentração de impedimentos por nível operacional, gerencial, diretoria e executivo.

## 👤 Responsáveis e Setores em Atenção
Identificação de sobrecarga e gargalos por área.

## ⏱️ Risco ao Cronograma
Serviços que serão bloqueados se as restrições não forem resolvidas.

## ✅ Ações Operacionais Priorizadas (máx. 5)
Ordenadas por urgência + impacto no caminho crítico.
```

### 4.4 Onde aparece no app

- Botão "Analisar com KAI" no Quadro de Restrições
- Aba KAI do Dashboard Gerencial de Restrições (modo multi-obra)
- Relatório gerado em markdown, renderizado inline na tela
- Pode ser exportado ou compartilhado

### 4.5 Regras de resposta

- O KAI não deve mencionar nomes de campos como `hasCriticalActivities`, `isOverdue` ou `timesRescheduled` no texto final.
- Quando uma seção obrigatória não tiver itens elegíveis, ele responde com uma frase curta e natural, em tom executivo.
- Em modo multi-obra, o nome da obra aparece quando for relevante para diferenciar riscos, gargalos e ações.

---

## 6. KAI — Analista de Planos de Ação

### 5.1 Objetivo

Gerar um **relatório de execução e acompanhamento de planos de ação**, com foco em:
- Planos vencidos ou em risco de vencer
- Impacto no caminho crítico do cronograma
- Progresso por etapas (milestones) e por responsável
- Alertas de extrapolação orçamentária

### 5.2 Campos Analisados

| Campo | Interpretação |
|-------|--------------|
| `status` | Pendente / Em andamento / Concluído / Cancelado |
| `responsible.name` | Responsável pelo plano |
| `dueDate` / `isOverdue` | Vencimento e urgência |
| `openDays` | Dias desde a abertura (para detectar planos antigos parados) |
| `hasCriticalActivities` | Plano impacta caminho crítico |
| `linkedActivities[].isCritical` | Cita atividade bloqueada pelo nome |
| `milestones[]` | Etapas concluídas vs. pendentes (ex: "2 de 4 etapas") |
| `percentComplete` | Progresso numérico 0–100% |
| `budgetEstimate` vs `budgetActual` | Alerta de extrapolação orçamentária |

### 5.3 Sinalizações Visuais no Relatório

| Símbolo | Significa |
|---------|-----------|
| 🔴 | Prazo vencido |
| ⚡ | Impacta caminho crítico |
| 🟡 | Em andamento, prazo próximo |
| 🟢 | Saudável, dentro do prazo |

### 5.4 Onde aparece no app

- Botão "Relatório KAI" no módulo de Planos de Ação
- Dashboard executivo por obra ou por responsável

---

## 7. KAI — Relatório de Planejamento

### 6.1 Objetivo

Análise do cronograma para reuniões de produção — destaca desvios de prazo, atividades em risco e tendências de avanço.

### 6.2 Conteúdo gerado

- Semáforo global do cronograma
- Atividades em atraso e sua criticidade
- Tendência de avanço físico vs. planejado
- Recomendações para a próxima janela de planejamento
- Síntese executiva para uso em reuniões diárias / semanais

---

## 8. KAI — Sugestão Inteligente de Restrições

### 7.1 Objetivo

A partir do contexto da obra (atividades planejadas, serviços em execução, materiais e equipes), o KAI sugere **quais restrições deveriam ser cadastradas** no Quadro de Restrições antes que se tornem problemas.

### 7.2 Funcionamento

```
Usuário clica "Sugerir Restrições" no Quadro de Restrições
      ↓
Cloud Function: suggestRestrictions
  - Recebe contexto: atividades do lookahead, serviços planejados
  - Envia para o modelo KAI com prompt especializado
  - Retorna lista de restrições sugeridas (JSON estruturado)
      ↓
App exibe sugestões para o usuário revisar/aceitar/editar
      ↓
Usuário confirma → restrições são persistidas no Firestore
```

### 7.3 Formato de retorno

O agente retorna um array JSON que o app interpreta para pré-preencher o formulário de criação de restrição. O usuário sempre revisa e confirma antes de persistir.

---

## 9. KAI — Identificação de Problemas

### 8.1 Objetivo

Análise de anomalias na execução da obra: desvios de qualidade, não conformidades recorrentes, padrões de problemas por área ou equipe.

### 8.2 Contexto de uso

- Análise pós-inspeção FVS: identifica categorias de não conformidades mais frequentes
- Correlação de problemas com equipes, períodos e tipos de serviço
- Sugestão de ações preventivas baseadas no histórico

---

## 10. KAI Chat

### 9.1 Objetivo

O **KAI Chat** é a interface conversacional do Kaizen para perguntas e respostas contextuais sobre os dados da obra. Diferente dos relatórios fixos, ele permite exploração incremental: o usuário pergunta, recebe a resposta e aprofunda o tema em novas interações.

### 9.2 Onde aparece no app

- Aba **Chat KAI** no módulo de restrições
- Aba **Chat KAI** no Dashboard Gerencial de Restrições
- Estruturado para reutilização futura em outros contextos, como planos de ação, medições e inspeções

### 9.3 Como funciona

```text
App Flutter
  ↓ envia pergunta + histórico recente
Cloud Function: kaiChat
  ↓ valida autenticação Firebase
  ↓ monta prompt de sistema conforme o contexto (restrições, ações, inspeções etc.)
  ↓ injeta o contexto factual apenas no primeiro turno
Modelo KAI Chat
  ↓ responde com texto natural, objetivo e contextual
App Flutter
  ↓ renderiza a conversa e preserva o histórico local da sessão
```

### 9.4 Características do KAI Chat

- **Contextual por módulo:** o prompt muda conforme o tipo de dado analisado
- **Primeiro turno com contexto completo:** o app envia o conjunto resumido de dados apenas na primeira pergunta
- **Turnos seguintes com histórico:** as próximas mensagens reutilizam o histórico recente sem reenviar todo o payload
- **Resposta objetiva:** o modelo é configurado para respostas factuais, baseadas somente nos dados fornecidos
- **Baixo ruído técnico:** ao citar itens específicos, o agente prioriza descrição, obra, responsável, prazo e impacto, evitando IDs internos sem necessidade

### 9.5 Payload esperado

```json
{
  "siteId": "multi_site",
  "siteName": "Dashboard Multi-obra",
  "contextType": "restrictions",
  "context": { "restrictions": [] },
  "question": "Quais obras concentram mais risco?",
  "history": [
    { "role": "user", "content": "..." },
    { "role": "assistant", "content": "..." }
  ]
}
```

### 9.6 Casos de uso típicos

- "Quais são as restrições mais críticas agora?"
- "Qual obra concentra mais risco ao cronograma?"
- "Quem está com maior carga de restrições vencidas?"
- "Explique em termos executivos o cenário atual"

### 9.7 Tratamento de timeout e indisponibilidade

Se o barramento de IA retornar timeout ou indisponibilidade temporária, a Cloud Function responde com mensagem amigável para o app, permitindo que a interface mostre erro controlado sem quebrar a conversa.

---

## 11. Base de Conhecimento (RAG)

O KAI também pode operar com base de conhecimento estruturada, usando recuperação de contexto relevante antes da geração da resposta. Esse mecanismo é aplicado quando a consulta depende de documentos, políticas ou conteúdos de referência além dos dados transacionais da obra.

### 10.1 Objetivos do RAG

- Responder perguntas com base em documentos corporativos e operacionais
- Reduzir alucinação do modelo em temas normativos ou processuais
- Permitir rastreabilidade da origem do contexto utilizado

### 10.2 Cenários típicos

- Perguntas sobre procedimentos internos
- Busca por conteúdo técnico em documentações sincronizadas
- Apoio a consultas operacionais que dependem de textos de referência

---

## 12. Arquitetura dos Agentes KAI

### 11.1 Fluxo de uma chamada KAI

```
App Flutter
  ↓ JWT token de autenticação (no header Authorization: Bearer {token})
Cloud Function (Node.js)
  ↓ Valida token com admin.auth().verifyIdToken()
  ↓ Verifica clientId e permissões
  ↓ Monta payload com: contexto dos dados + system prompt do agente
Camada de IA (modelo especializado)
  ↓ Retorna resposta em markdown estruturado
Cloud Function
  ↓ Sanitiza resposta (remove caracteres de controle, valida JSON se aplicável)
App Flutter
  ↓ Renderiza relatório em markdown inline
```

No caso do **KAI Chat**, o fluxo inclui também histórico recente da conversa e prompt especializado por contexto.

### 11.2 Parâmetros de Geração

| Parâmetro | Valor padrão | Descrição |
|-----------|-------------|-----------|
| `temperature` | 0.5 | Controle de criatividade vs. precisão |
| `max_tokens` | 8192 | Limite de tokens do relatório |
| `top_p` | 0.9 | Diversidade do sampling |
| `stream` | false | Resposta completa (não streaming) |

### 11.2.1 Parâmetros típicos do KAI Chat

| Parâmetro | Valor padrão | Descrição |
|-----------|-------------|-----------|
| `temperature` | 0.3 | Respostas mais estáveis e factuais |
| `max_tokens` | 2048 | Limite de tokens por resposta de chat |
| `top_p` | 0.9 | Diversidade controlada |
| `stream` | false | Resposta completa |

### 11.3 Tratamento de Erros

- Resposta inválida do modelo → extração de JSON válido via regex fallback
- Timeout → erro 504 retornado ao app com mensagem amigável
- Token inválido → erro 401 antes de qualquer chamada ao modelo
- Campos ausentes nos dados → o prompt instrui o modelo a omitir itens sem dados (nunca inventar)
- Timeout do barramento no KAI Chat → retry automático único antes de devolver erro amigável

### 11.4 Escalabilidade

- Cada Cloud Function KAI executa de forma independente (serverless auto-scaling)
- Tempo de resposta típico: 5–15 segundos dependendo do volume de dados
- Memória alocada: 512 MiB (RAG) a 256 MiB (geração de relatórios)
- Região: `us-central1`

---

## 13. Segurança e Privacidade

### 12.1 Autenticação obrigatória

**Nenhuma chamada ao KAI é possível sem autenticação válida.** Cada Cloud Function valida o token antes de qualquer processamento:

```javascript
async function requireAuth(req) {
  const token = getBearerToken(req);
  return admin.auth().verifyIdToken(token); // lança exceção se inválido
}
```

### 12.2 Dados enviados ao modelo

- São enviados apenas os **dados estruturados e filtrados** necessários para a análise
- Nenhuma credencial de usuário, token de autenticação ou dado sensível é transmitido ao modelo
- O `clientId` e identificadores internos **não** precisam aparecer no texto final da IA
- Dados pessoais (nomes de responsáveis) são transmitidos somente quando diretamente relevantes para o relatório (ex.: "Responsável pela restrição: João Silva")

### 12.3 Restrições de Origem (CORS)

Apenas as origens autorizadas do Kaizen (`kaizen-v3.web.app`, `localhost:5173`) podem chamar as Cloud Functions KAI — chamadas de origens desconhecidas são bloqueadas na camada de transporte.

### 12.4 Sem armazenamento de prompts externos

Os dados da obra enviados ao modelo não são armazenados fora do ambiente Kaizen além do processamento da requisição. O Firestore armazena apenas os dados originais e, se o usuário optar por salvar, o relatório gerado.

---

*Documento: Inteligência Artificial KAI | Versão 1.1 | Kaizen Construction Management*
