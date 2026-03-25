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
  <div>
    <h3>🏗️ Arquitetura</h3>
    <p><a href="{{ '/tecnico/arquitetura/' | relative_url }}"><strong>Arquitetura Geral</strong></a><br>
    Visão geral da arquitetura do sistema, decisões de design, padrões utilizados e estrutura geral.</p>
  </div>
  <div>
    <h3>📦 Módulos</h3>
    <p><a href="{{ '/tecnico/modulos/' | relative_url }}"><strong>Módulos de Gestão de Obra</strong></a><br>
    Descrição dos módulos de gestão de obra, funcionalidades, fluxos de trabalho e integrações.</p>
  </div>
  <div>
    <h3>🔧 Assistência Técnica</h3>
    <p><a href="{{ '/tecnico/at/' | relative_url }}"><strong>Módulo AT - Assistência Técnica</strong></a><br>
    Documentação do módulo de assistência técnica, agendamento de serviços, gestão de ordens de trabalho.</p>
  </div>
  <div>
    <h3>💰 Cotações</h3>
    <p><a href="{{ '/tecnico/cotacoes/' | relative_url }}"><strong>Módulo de Cotações</strong></a><br>
    Sistema de cotações, orçamentos e gestão de propostas.</p>
  </div>
  <div>
    <h3>🔗 Integrações</h3>
    <p><a href="{{ '/tecnico/integracoes/' | relative_url }}"><strong>Integrações do Sistema</strong></a><br>
    Integrações com Salesforce, Data Lake, APIs externas e terceiros.</p>
  </div>
  <div>
    <h3>🔐 Acesso e Perfis</h3>
    <p><a href="{{ '/tecnico/acesso/' | relative_url }}"><strong>Perfis e Controle de Acesso</strong></a><br>
    Sistema de permissões, papéis de usuário, multi-tenancy e controle de acesso ao dados.</p>
  </div>
  <div>
    <h3>🤖 Inteligência Artificial</h3>
    <p><a href="{{ '/tecnico/ia/' | relative_url }}"><strong>IA Kai - Inteligência Artificial Aplicada</strong></a><br>
    Módulo de inteligência artificial para sugestões, análises e automações.</p>
  </div>
  <div>
    <h3>☁️ Cloud Functions &amp; API</h3>
    <p><a href="{{ '/tecnico/api/' | relative_url }}"><strong>Cloud Functions e API REST</strong></a><br>
    Documentação técnica de Cloud Functions, endpoints REST, autenticação e status de APIs.</p>
  </div>
</div>

## 🚀 Começar

1. **Novo no Kaizen?** Comece pela <a href="{{ '/tecnico/arquitetura/' | relative_url }}">Arquitetura Geral</a>
2. **Desenvolvedor frontend?** Veja a documentação dos <a href="{{ '/tecnico/modulos/' | relative_url }}">Módulos</a>
3. **Trabalhando com backend?** Acesse a <a href="{{ '/tecnico/api/' | relative_url }}">API REST</a>
4. **Consultando integrações?** Verifique as <a href="{{ '/tecnico/integracoes/' | relative_url }}">Integrações do Sistema</a>

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
