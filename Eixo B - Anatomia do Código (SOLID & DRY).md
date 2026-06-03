# Eixo B: Anatomia do Código (SOLID & DRY)

## 1. Teste da Troca (Inversão de Dependência - DIP)

O teste avalia o esforço necessário para trocar o provedor de IA (ex: migrar do `litellm` para OpenAI nativo ou Llama-3 local). Segundo a diretriz da auditoria, se forem necessários mais de 2 ou 3 arquivos, há uma violação do **DIP**.

### 1.1 Investigação Realizada

Foi realizada uma busca sistemática pelas ocorrências da biblioteca `litellm` no código-fonte.

| Arquivo | Ocorrências | O que faz |
|---------|-------------|-----------|
| `cli.py` | 3 linhas | Interface de linha de comando |
| `legacy\llmtxt.py` | 3 linhas | Código legado (pasta legacy) |
| `utils.py` | 12 linhas | Centraliza lógica de completion e embeddings |

**Evidência:** `Captura de Tela (1).png`

### 1.2 Análise Crítica

**Diagnóstico:** O projeto **falha no Teste da Troca**. A dependência do `litellm` está concentrada em 3 arquivos, mas é rígida (*hardcoded*), especialmente no `utils.py`.

**Causa Raiz:** Falta de uma abstração formal (ex: classe `BaseLLMProvider`) que isole a lógica de criação do cliente LLM.

**Classificação de Risco:** 🟡 **Médio-Alto**

---

## 2. God Objects (Responsabilidade Única - SRP)

A investigação busca identificar classes ou arquivos que acumulam responsabilidades excessivas, violando o **S** do SOLID.

### 2.1 Resultados da Densidade de Código

A análise de tamanho revelou uma concentração crítica de lógica em poucos arquivos:

| Ranking | Arquivo | Tamanho (Bytes) | Risco |
|---------|---------|-----------------|-------|
| 🥇 1º | `utils.py` | **133.468** | 🔴 Muito Alto |
| 🥈 2º | `async_crawler_strategy.py` | **121.105** | 🔴 Alto |
| 🥉 3º | `extraction_strategy.py` | **120.722** | 🔴 Alto |

**Evidência:** `Captura de Tela (3).png`

### 2.2 Análise Crítica

**Diagnóstico:** O `utils.py` é um **God Object evidente**. Ele contém funções desconectadas (validação, formatação, chamadas de API), o que dificulta testes isolados e aumenta o risco de regressões.

**Dívida Técnica:** A presença de arquivos de backup como `async_crawler_strategy.back.py` (102 KB) indica **falta de limpeza no repositório**.

**Classificação de Risco:** 🔴 **Alto**

---

## 3. Duplicação de Código (DRY - Don't Repeat Yourself)

Busca por padrões repetidos de tratamento de exceções, um indicador de falha na abstração.

### 3.1 Investigação Realizada

Busca por padrões de tratamento de `TimeoutError` em todo o projeto.

| Arquivo | Nº de Ocorrências | Tipos de Timeout Detectados |
|---------|-------------------|-----------------------------|
| `async_crawler_strategy.py` | 5 | asyncio, Playwright, aiohttp |
| `async_url_seeder.py` | 2 | asyncio |
| `browser_manager.py` | 2 | subprocess, asyncio |
| `cache_validator.py` | 1 | httpx |

**Evidência:** `Captura de Tela (5).png`

### 3.2 Análise Crítica

**Diagnóstico:** **Violação clara do princípio DRY**. O mesmo padrão de `except asyncio.TimeoutError` aparece em **8 arquivos diferentes**.

**Causa Raiz:** Ausência de uma abstração centralizada (ex: decorator) para tratamento de resiliência.

**Classificação de Risco:** 🔴 **Alto**

---

## 4. Resumo da Avaliação do Eixo B

| Sub-eixo | Princípio Violado | Achado Principal | Classificação | Evidência |
|----------|-------------------|------------------|---------------|-----------|
| Teste da Troca | Dependency Inversion | `litellm` hardcoded no núcleo do sistema | 🟡 Médio-Alto | Print 1 |
| God Objects | Single Responsibility | `utils.py` (133KB) acumula múltiplas funções | 🔴 Alto | Print 3 |
| Duplicação | DRY | Timeout tratado repetidamente em 8+ arquivos | 🔴 Alto | Print 5 |

---

## 5. Alinhamento com Padrões de Qualidade

### SOLID - Avaliação Geral

| Princípio | Status | Justificativa |
|-----------|--------|----------------|
| **S** (SRP) | ❌ Insatisfatório | Concentração excessiva no `utils.py` |
| **O** (OCP) | 🟡 Parcial | Extensível em estratégias, mas rígido no core |
| **L** (LSP) | ✅ Satisfatório | Hierarquias consistentes |
| **D** (DIP) | ❌ Insatisfatório | Acoplamento direto com biblioteca externa (`litellm`) |

### DRY - Avaliação

| Prática | Status | Justificativa |
|---------|--------|----------------|
| Tratamento de Timeout | ❌ Insatisfatório | Espalhado por 8 arquivos |
| Validação de URL | 🟡 Parcial | Indícios de repetição de regex |

---

## 6. Conclusão

A auditoria forense realizada no **Eixo B do Crawl4AI** revela que o projeto, embora funcional, apresenta **dívidas técnicas significativas** que comprometem sua escalabilidade e manutenção em ambientes de produção real:

1. **Dependência Rígida de IA (DIP):** A integração direta com a biblioteca `litellm` em múltiplos arquivos cria um *vendor lock-in* que dificulta a migração para novos modelos ou provedores locais.

2. **Existência de God Objects (SRP):** A concentração de mais de **133 KB** de lógica heterogênea no arquivo `utils.py` viola o Princípio da Responsabilidade Única, tornando o núcleo do sistema frágil a mudanças.

3. **Duplicação Sistêmica (DRY):** A repetição de lógica de tratamento de timeouts em pelo menos **8 arquivos distintos** demonstra uma falha na criação de abstrações utilitárias.

A **implementação imediata do Plano de Resgate proposto** (eliminação de lixo técnico, criação de decorator de retry e refatoração do `utils.py`) é essencial para mitigar esses riscos arquiteturais e alinhar o repositório aos padrões de qualidade exigidos pela disciplina de Engenharia de Software.

---

## 7. Uso de IA

Este relatório de auditoria contou com o apoio da inteligência artificial **DeepSeek** para as seguintes atividades:

- **Orientação Estratégica:** Planejamento dos passos para análise do código-fonte e interpretação dos comandos de terminal (`findstr`, `dir`, `sort`).

- **Processamento de Dados:** Estruturação das evidências coletadas em tabelas de conformidade e ranking de métricas.

- **Formatação Técnica:** Organização do relatório final seguindo os padrões de documentação para GitHub (Markdown).

**Nota de Integridade:** Todo o conteúdo gerado pela IA foi validado manualmente pela equipe através da conferência direta com os arquivos do repositório e as capturas de tela anexadas como evidência.
