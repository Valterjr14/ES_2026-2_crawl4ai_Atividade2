# Auditoria Forense de Software e Plano de Resgate Técnico
---
## Sumário
- Sobre o Repositório
- Projeto Analisado
- Equipe
- Estrutura da Auditoria
- Metodologia
- Vídeo de Apresentação
---
## Sobre o Repositório
Este repositório registra todos os passos e conclusões feitas pela equipe na Atividade Avaliativa 2 (A2), onde o objetivo é realizar um diagnóstico profundo da saúde gerencial e arquitetural do projeto Crawl4AI, identificando dívidas técnicas e propondo um Plano de Resgate baseado nos padrões da indústria (MPS.BR Nível G e padrões GoF).

---
## Projeto Analisado
O projeto analisado foi o **Crawl4AI** — https://github.com/unclecode/crawl4ai

O Crawl4AI é uma biblioteca open source em Python que realiza web crawling e scraping com saída otimizada para modelos de linguagem. Em vez de retornar HTML bruto, o projeto converte o conteúdo das páginas em Markdown limpo e estruturado, removendo scripts, anúncios e tags desnecessárias, facilitando o consumo do conteúdo por LLMs em pipelines de RAG, agentes de IA e sistemas de extração de dados.

---
- Linguagem principal - Python
- Licença - Apache 2.0
- Última versão - v0.8.7 (01 de junho de 2026)
---
## Equipe
---
- Pedro Afonso Tavares Barreto da Silva - 202300027580
- Jose Valter de Oliveira Junior - 202300083678
- Yasmin Silva Santos - 202300084058
- Beatriz Eduarda Pires da Cruz - 202200092647
- João Guilherme Santos Florêncio - 202200025546
---
## Estrutura da Auditoria
A auditoria está dividida em três eixos de investigação, uma refatoração conceitual e um roadmap de maturidade. Os arquivos estão organizados da seguinte forma:

```
ES_2026-2_Crawl4AI_Atividade2/
├── README.md
├── Eixo_A_Gestao_GPR.md
├── Eixo_B_Anatomia_Codigo_SOLID.md
├── Eixo_C_Padroes_de_Projeto_GoF.md
├── Refatoracao_Conceitual.md
├── Roadmap_de_Maturidade.md
└── imagens/
```

---
## Metodologia
A análise foi conduzida diretamente no repositório do Crawl4AI no GitHub, com inspeção de issues, pull requests, releases, histórico de commits, estrutura do código-fonte e documentação. Cada eixo foi avaliado segundo os critérios do MPS.BR Nível G e dos padrões de projeto GoF.

Eixo - Área de Investigação - Alinhamento

I. Eixo A — O Pulso da Gestão - Gerência de Projetos (GPR) - MPS.BR: Planejamento, Monitoramento e Controle

II. Eixo B — Anatomia do Código - Qualidade Arquitetural (SOLID/DRY) - Princípios de design: SRP, DIP, DRY

III. Eixo C — Padrões de Projeto - Padrões GoF - Criacionais, Estruturais e Comportamentais

IV. Refatoração Conceitual - Proposta de melhoria arquitetural - Padrão GoF aplicado com UML

V. Roadmap de Maturidade - Gerência de Projetos (GPR) - MPS.BR Nível G: GPR 2, GPR 5, GPR 6, GPR 7

---
## Vídeo de Apresentação
[Link do vídeo]()