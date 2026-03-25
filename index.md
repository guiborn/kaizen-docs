---
layout: default
title: Documentação Técnica Kaizen
permalink: /
toc: false
---

# Bem-vindo à Documentação Técnica Kaizen

<div class="homepage-intro">
  <p>Sistema modular de gestão de construção e assistência técnica construído com Flutter, Firebase e integração com Salesforce.</p>
</div>

## 📚 Documentação Técnica

Nesta seção você encontra a documentação técnica completa sobre a arquitetura, módulos, integrações e APIs do sistema Kaizen.

<div class="docs-grid">

### 🏗️ Arquitetura

[**Arquitetura Geral**](/kaizen/tecnico/arquitetura/)
Visão geral da arquitetura do sistema, decisões de design, padrões utilizados e estrutura geral.

---

### 📦 Módulos

[**Módulos de Gestão de Obra**](/kaizen/tecnico/modulos/)
Descrição dos módulos de gestão de obra, funcionalidades, fluxos de trabalho e integrações.

---

### 🔧 Assistência Técnica (AT)

[**Módulo AT - Assistência Técnica**](/kaizen/tecnico/at/)
Documentação do módulo de assistência técnica, agendamento de serviços, gestão de ordens de trabalho.

---

### 💰 Cotações

[**Módulo de Cotações**](/kaizen/tecnico/cotacoes/)
Sistema de cotações, orçamentos e gestão de propostas.

---

### 🔗 Integrações

[**Integrações do Sistema**](/kaizen/tecnico/integracoes/)
Integrações com Salesforce, Data Lake, APIs externas e terceiros.

---

### 🔐 Acesso e Perfis

[**Perfis e Controle de Acesso**](/kaizen/tecnico/acesso/)
Sistema de permissões, papéis de usuário, multi-tenancy e controle de acesso ao dados.

---

### 🤖 Inteligência Artificial

[**IA Kai - Sistema de Sugestões**](/kaizen/tecnico/ia/)
Módulo de inteligência artificial para sugestões, análises e automações.

---

### ☁️ Cloud Functions & API

[**Cloud Functions e API REST**](/kaizen/tecnico/api/)
Documentação técnica de Cloud Functions, endpoints REST, autenticação e status de APIs.

---

</div>

## 🚀 Começar

1. **Novo no Kaizen?** Comece pela [Arquitetura Geral](/kaizen/tecnico/arquitetura/)
2. **Desenvolvedor frontend?** Veja a documentação dos [Módulos](/kaizen/tecnico/modulos/)
3. **Trabalhando com backend?** Acesse a [API REST](/kaizen/tecnico/api/)
4. **Consultando integrações?** Verifique [Integrações do Sistema](/kaizen/tecnico/integracoes/)

## ℹ️ Sobre

- **Projeto**: Kaizen - Gestão de Construção e Assistência Técnica
- **Stack**: Flutter + Firebase + Salesforce
- **Última atualização**: {{ site.time | date: "%d de %B de %Y" }}
- **Repositório**: [guiborn/kaizen](https://github.com/guiborn/kaizen)

---

<style>
.homepage-intro {
  background: linear-gradient(135deg, #7c3aed 0%, #6d28d9 100%);
  color: white;
  padding: 32px;
  border-radius: 8px;
  margin-bottom: 48px;
  font-size: 18px;
  line-height: 1.8;
}

.docs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
  margin: 32px 0;
}

.docs-grid > div {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  transition: all 0.3s ease;
}

.docs-grid > div:hover {
  border-color: #7c3aed;
  box-shadow: 0 4px 12px rgba(124, 58, 237, 0.15);
}

.docs-grid h3 {
  color: #7c3aed;
  margin-top: 0 !important;
  font-size: 18px;
}

.docs-grid a {
  color: #7c3aed;
  font-weight: 500;
  text-decoration: none;
}

.docs-grid a:hover {
  text-decoration: underline;
}

.docs-grid hr {
  border: none;
  border-top: 1px solid #e5e7eb;
  margin: 16px 0;
}
</style>
