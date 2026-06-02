# Relatório de Análise: Roadmap de Maturidade (MPS.BR Nível G)
 
## 1. Introdução
Esta seção apresenta o Roadmap de Maturidade do Crawl4AI, propondo as três ações prioritárias para alinhar o projeto ao Nível G do MPS.BR (GPR — Gerência de Projetos). A análise parte dos problemas identificados nos eixos anteriores: ausência de planejamento formal com cronogramas, subutilização de ferramentas de gestão do GitHub e controle de progresso realizado exclusivamente por meios técnicos (commits e Pull Requests).
 
O Nível G do MPS.BR exige, em síntese, que o projeto seja planejado, monitorado e controlado de forma sistemática, com escopo definido, estimativas documentadas e marcos temporais estabelecidos.
 
---
 
## 2. Diagnóstico de Maturidade Atual
 
Antes de propor as ações, é necessário situar o estado atual do projeto em relação aos requisitos do Nível G.
 
**O que o projeto já faz bem:**
- Versionamento semântico consistente (v0.7.x → v0.8.x), registrado no `CHANGELOG.md` e no `JOURNAL.md`, o que demonstra rastreabilidade técnica dos artefatos.
- Cadência de entregas contínua e documentada, com releases mensais que indicam um ciclo de vida ágil funcional.
- Resposta rápida a riscos técnicos críticos (vulnerabilidades RCE e LFI corrigidas em tempo hábil após divulgação responsável, padrão mantido até a v0.8.7, lançada em 01/06/2026).
**O que falta para o Nível G:**
- O `ROADMAP.md` descreve funcionalidades desejadas, mas **não possui datas, responsáveis, estimativas de esforço ou critérios de conclusão** — características exigidas pelo processo GPR do MPS.BR.

- As **Milestones do GitHub estão criadas mas imaturas**, sem associação formal de issues, datas de encerramento ou métricas de progresso.
 
- Não há uso de **GitHub Projects** nem de quadros Kanban para visualização do andamento das atividades, o que compromete o monitoramento e o controle formal exigidos pelo modelo.

 
---
 
## 3. Roadmap de Maturidade: As 3 Ações Prioritárias
 
### Ação 1 — Formalizar o Planejamento com Milestones Estruturadas
 
**Problema identificado:** O `ROADMAP.md` lista funcionalidades organizadas em quatro seções (Advanced Crawling Systems, Specialized Features, Development Tools e Community & Growth), porém sem nenhuma vinculação temporal ou de responsabilidade. As Milestones do GitHub existem, mas não possuem definições de prazo nem issues associadas.
 
**Classificação de risco:** Alto — sem planejamento formal, não é possível monitorar desvios de escopo nem demonstrar aderência ao GPR.
 
**Ação proposta:** Converter cada seção do `ROADMAP.md` em uma **Milestone formal no GitHub**, contendo:
- **Data de início e encerramento** previstas.
- **Issues associadas** para cada entregável listado (ex: "Implementar Question-Based Crawler" vira uma issue rastreável).
- **Critério de conclusão** explícito (Definition of Done).
- **Responsável** designado para cada milestone.
Esta ação corresponde diretamente ao atributo de processo **AP 1.1 (O processo é executado)** e **AP 2.1 (O processo é gerenciado)** do MPS.BR, que exigem que o trabalho seja planejado e monitorado contra um plano.
 
---
 
### Ação 2 — Implementar um Quadro de Acompanhamento com GitHub Projects
 
**Problema identificado:** O controle do projeto é realizado exclusivamente através de commits e revisões de Pull Requests. Não há visibilidade centralizada do que está em andamento, em revisão ou concluído. A aba de **GitHub Projects** está sem utilização, e não há quadro Kanban ou tabela de progresso configurados.
 
**Classificação de risco:** Médio-Alto — a ausência de um mecanismo de acompanhamento visível impede o monitoramento e o controle formal exigidos pelo GPR do MPS.BR Nível G, especialmente os resultados esperados **GPR 6** (monitorar o progresso do projeto) e **GPR 7** (tomar ações corretivas).
 
**Ação proposta:** Criar um **GitHub Project** vinculado ao repositório, configurado com um quadro Kanban com as colunas:
 
| Backlog | Em Desenvolvimento | Em Revisão (PR Aberto) | Concluído |
|---------|-------------------|------------------------|-----------|
 
Cada issue criada na Ação 1 deve ser adicionada ao quadro, permitindo rastrear o fluxo de trabalho de ponta a ponta. Adicionalmente, o projeto deve ser configurado para **atualização automática de status** com base no estado dos Pull Requests vinculados.
 
Esta ação viabiliza o monitoramento contínuo previsto no MPS.BR e reduz a dependência de conhecimento implícito da equipe para entender o estado atual do projeto.
 
---
 
### Ação 3 — Registrar Estimativas de Esforço e Documentar Riscos Formalmente
 
**Problema identificado:** O projeto não possui nenhum registro formal de estimativas de esforço associado às suas funcionalidades. Os riscos técnicos (como dependência de APIs de terceiros — OpenAI, Anthropic — e instabilidade de ambientes de crawling com anti-bot) são tratados de forma reativa no código, por meio de arquiteturas de fallback, mas **não estão documentados em um plano de riscos explícito**.
 
O único indício de gestão de riscos encontrado está na própria evolução do código: a introdução do `LLMConfig` como camada de abstração e a adição de mecanismos de detecção de anti-bot na v0.8.5 indicam consciência dos riscos, mas sem registro gerencial formal. A v0.8.7 (01/06/2026) reforça esse padrão reativo: três vulnerabilidades com CVSS 9.8 — incluindo escape de sandbox com RCE e chave JWT padrão `"mysecret"` que permitia forja de tokens — foram corrigidas somente após divulgação por pesquisadores externos, evidenciando que a identificação de riscos de segurança não ocorre de forma proativa internamente.
 
**Classificação de risco:** Médio — o conhecimento sobre riscos está concentrado no mantenedor principal, criando dependência de pessoas e dificultando a continuidade do projeto em caso de rotatividade de contribuidores.
 
**Criação de artefatos para estimativas e gerenciamento de riscos**

Outra melhoria recomendada consiste na criação de dois artefatos simples, porém alinhados às práticas de gerenciamento de projetos do MPS.BR:

1. **ESTIMATES.md** — Documento destinado ao registro das estimativas de esforço das funcionalidades previstas no ROADMAP, utilizando métricas como horas de trabalho ou story points. O documento deve ser atualizado a cada ciclo de desenvolvimento ou release, mantendo um histórico básico de planejamento compatível com a dinâmica de projetos open-source.

2. **RISK_REGISTER.md** — Documento para o registro e acompanhamento dos principais riscos do projeto. Para cada risco identificado, recomenda-se registrar:

   * Descrição do risco;
   * Probabilidade de ocorrência (Alta, Média ou Baixa);
   * Impacto potencial (Alto, Médio ou Baixo);
   * Estratégia de mitigação adotada ou planejada;
   * Responsável pelo monitoramento.

Com base na análise realizada, alguns riscos já poderiam ser registrados inicialmente, conforme apresentado na Tabela X.

| Risco                                                                                 | Probabilidade | Impacto | Estratégia de Mitigação                                           |
| ------------------------------------------------------------------------------------- | ------------- | ------- | ----------------------------------------------------------------- |
| Mudanças nas APIs de provedores de LLM (OpenAI, Anthropic, entre outros)              | Alta          | Alto    | Camada de abstração implementada por meio da classe `LLMConfig`   |
| Bloqueios por mecanismos anti-bot de sites monitorados                                | Alta          | Alto    | Utilização de mecanismos de fallback introduzidos na versão 0.8.5 |
| Vulnerabilidades de segurança relacionadas à Docker API                               | Média         | Crítico | Hooks desabilitados por padrão desde a versão 0.8.0               |
| Execução remota de código (RCE) por escape de sandbox em `eval()` (CVSS 9.8 – CWE-94) | Alta          | Crítico | Vulnerabilidade corrigida na versão 0.8.7 após reporte externo    |
| Forja de tokens JWT devido ao uso da chave padrão `"mysecret"` (CVSS 9.8 – CWE-798)   | Alta          | Crítico | Vulnerabilidade corrigida na versão 0.8.7 após reporte externo    |
| SSRF por URLs de webhook e endpoints de rastreamento (CVSS 8.6 – CWE-918)             | Média         | Alto    | Vulnerabilidade corrigida na versão 0.8.7 após reporte externo    |
| Dependência excessiva de um único mantenedor                                          | Média         | Alto    | Não há mitigação formalmente documentada                          |

A adoção desses artefatos contribuiria para institucionalizar práticas de planejamento e gerenciamento de riscos no projeto, atendendo diretamente aos resultados esperados **GPR 2** (as estimativas do projeto são estabelecidas e mantidas) e **GPR 5** (os riscos do projeto são identificados, analisados e tratados) do **MPS.BR Nível G**.
 
---
 
## 4. Conclusão
 
O Crawl4AI demonstra um nível de maturidade técnica superior ao observado em muitos projetos open-source, apresentando um ciclo de desenvolvimento ágil bem estabelecido, rastreabilidade das mudanças por meio do CHANGELOG e uma resposta eficiente a questões relacionadas à segurança. No entanto, a ausência de um planejamento formal com cronogramas definidos, mecanismos visuais de acompanhamento do progresso e registros explícitos de riscos e estimativas evidencia uma lacuna em relação às práticas exigidas pelo Nível G do MPS.BR.

Nesse contexto, as três ações propostas — a formalização de milestones, a adoção do GitHub Projects e a criação de artefatos voltados ao registro de estimativas e riscos — representam iniciativas viáveis, graduais e alinhadas ao fluxo de trabalho open-source já utilizado pelo projeto. Sua implementação demandaria baixo esforço de adaptação, mas contribuiria significativamente para o aumento da maturidade gerencial do Crawl4AI, aproximando-o dos requisitos estabelecidos pelo modelo MPS.BR.

## 5. Ferramentas de IA Utilizadas
 
Para a elaboração desta seção, foi utilizado o **Claude** como ferramenta de apoio na estruturação e redação do texto, bem como na análise das evidências obtidas a partir do repositório GitHub do Crawl4AI.