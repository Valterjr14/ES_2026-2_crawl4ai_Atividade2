# Eixo C: Padrões de Projeto (GoF)

---

## Introdução

Esta seção apresenta a análise dos padrões de projeto GoF identificados no repositório Crawl4AI, organizada em três categorias: padrões criacionais, estruturais e comportamentais. Para cada padrão, são apresentadas evidências diretas no código, diagnóstico técnico e classificação de risco. A análise identifica tanto os padrões corretamente aplicados quanto as ausências que representam dívidas técnicas relevantes.

---

## C.1 — Padrões Criacionais

---

### C.1.1 — Object Pool (BrowserManager) 

#### Evidência Técnica

O `BrowserManager`, definido em `crawl4ai/browser_manager.py`, mantém um pool de instâncias de browser pré-aquecidas e as reutiliza entre requisições, evitando o custo de criação de um novo processo de navegador a cada crawl.

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/browser_manager.py
```

O pool é organizado em três camadas:

|Camada|Descrição|
|-|-|
|`_permanent_pool`|Instâncias sempre ativas, reservadas para sessões fixas|
|`_hot_pool`|Instâncias prontas para uso imediato|
|`_cold_pool`|Instâncias inativas, ativadas sob demanda|

#### Diagnóstico

Implementação correta do padrão Object Pool — variação do Singleton para recursos custosos. Abrir um processo Chromium, Firefox ou WebKit do zero é uma operação cara em termos de tempo e memória. O pool reutiliza instâncias existentes, protegendo o sistema contra criação desnecessária de conexões com o navegador.

Vale destacar que a escolha por Object Pool (em vez de Singleton puro) é arquiteturalmente adequada para contextos assíncronos com múltiplas requisições paralelas, onde uma única instância global seria um gargalo.

---

### C.1.2 — Ausência de Singleton no modelo de embeddings 

#### Evidência Técnica

A `CosineStrategy`, definida em `crawl4ai/extraction_strategy.py`, utiliza `SentenceTransformer` para gerar embeddings semânticos durante a extração. O modelo é carregado via função `load_HF_embedding_model` sem qualquer mecanismo de cache ou instância compartilhada.

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/extraction_strategy.py
```

O modelo padrão utilizado é `BAAI/bge-small-en-v1.5`, carregado a partir do disco a cada nova instância de `CosineStrategy` criada.

#### Diagnóstico

Ausência do padrão Singleton. Cada instância de `CosineStrategy` carrega o modelo de embeddings de forma independente. Em cenários com múltiplas estratégias rodando em paralelo — situação comum em produção — o modelo é carregado diversas vezes, resultando em:

* Consumo duplicado de memória RAM e GPU
* Latência desnecessária na inicialização
* Risco de Out of Memory (OOM) em ambientes com recursos limitados

A correção seria um Registry ou Singleton que compartilhe a instância carregada:

```python
# Proposta de correção: Registry de modelos compartilhados
class EmbeddingModelRegistry:
    _instances: Dict[str, SentenceTransformer] = {}

    @classmethod
    def get_model(cls, model_name: str) -> SentenceTransformer:
        if model_name not in cls._instances:
            cls._instances[model_name] = SentenceTransformer(model_name)
        return cls._instances[model_name]
```

---

### C.1.3 — Ausência de Factory para providers de LLM 

#### Evidência Técnica

A criação de um `LLMConfig` é feita diretamente pelo desenvolvedor via string de provider, sem qualquer validação prévia:

```python
# Uso atual — sem validação
llm_config = LLMConfig(
    provider="openai/gpt-4o",
    api_token=os.getenv("OPENAI_API_KEY")
)
```

Uma issue aberta no repositório evidencia o problema na prática: um usuário passou um provider com formato incorreto e recebeu erro apenas em tempo de execução, quando a chamada ao LiteLLM falhou.

```
https://github.com/unclecode/crawl4ai/issues/1325
```

A issue #1446 registra um caso ainda mais crítico: o provider de embeddings estava hardcoded como `"openai/text-embedding-3-small"` no `adaptive_crawler.py`, impedindo que usuários configurassem outro provider.

```
https://github.com/unclecode/crawl4ai/issues/1446
```

#### Diagnóstico

Ausência do padrão Factory Method. Não existe uma `LLMProviderFactory` que centralize a criação e validação dos providers. Erros de configuração — como typos na string de provider ou providers não suportados — só são descobertos em tempo de execução, quando a chamada à API externa já foi realizada.

Em produção com APIs pagas (OpenAI, Anthropic), uma falha silenciosa por string incorreta pode resultar em cobranças inesperadas ou interrupção do serviço sem mensagem clara de erro.

A Refatoração Conceitual proposta pelo grupo — aplicação do padrão Factory Method com interface `ILLMProvider` — resolve diretamente esse gap.

---

## C.2 — Padrões Estruturais

---

### C.2.1 — Adapter (LiteLLM) 

#### Evidência Técnica

A `LLMExtractionStrategy`, definida em `crawl4ai/extraction_strategy.py`, delega todas as chamadas a modelos de linguagem para o LiteLLM, sem conhecer a API específica de nenhum provider.

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/extraction_strategy.py
```

O `AsyncWebCrawler` nunca importa SDKs de providers diretamente. A troca de provider é feita exclusivamente via `LLMConfig`:

```python
# Troca de provider sem alterar o crawler
llm_config = LLMConfig(provider="ollama/llama3")   # local
llm_config = LLMConfig(provider="openai/gpt-4o")   # cloud
llm_config = LLMConfig(provider="claude-3-opus-20240229")  # Anthropic
```

#### Diagnóstico

O LiteLLM é o Adapter real do projeto — converte a chamada uniforme do Crawl4AI para a interface específica de cada provider. É o padrão estrutural mais bem aplicado do repositório.

**Ponto crítico:** na versão 0.8.5, o projeto sofreu um supply chain attack no PyPI que comprometeu a biblioteca `litellm`. O hotfix da v0.8.6 substituiu a dependência por `unclecode-litellm`. Esse incidente evidencia o risco de não existir uma interface formal `ILLMProvider` no código do projeto: a dependência do LiteLLM é implícita, e uma substituição de biblioteca exigiu um hotfix urgente em vez de uma troca controlada via interface.

**Importante:** `LLMConfig` não é o Adapter — é um Value Object que encapsula apenas configuração (provider, token, parâmetros de backoff). Não realiza adaptação de interfaces.

---

### C.2.2 — Strategy na hierarquia de Deep Crawl 

#### Evidência Técnica

O sistema de deep crawl, em `crawl4ai/deep_crawling/`, oferece três estratégias intercambiáveis de traversal:

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/deep_crawling/
```

|Classe|Comportamento|
|-|-|
|`BFSDeepCrawlStrategy`|Navega nível a nível (largura)|
|`DFSDeepCrawlStrategy`|Navega ramo a ramo (profundidade)|
|`BestFirstCrawlingStrategy`|Prioriza por score de relevância|

#### Diagnóstico

Aplicação correta do padrão Strategy para as estratégias de traversal. O desenvolvedor troca o algoritmo de navegação via configuração, sem alterar o crawler. Porém, não existe um padrão Composite formal para compor crawlers em hierarquias mais complexas — como crawling multi-domínio ou agentes aninhados. Para o escopo atual, a implementação é suficiente.

---

## C.3 — Padrões Comportamentais

---

### C.3.1 — Strategy (ExtractionStrategy) 

#### Evidência Técnica

A classe abstrata `ExtractionStrategy`, em `crawl4ai/extraction_strategy.py`, define o contrato de extração com dois métodos abstratos:

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/extraction_strategy.py
```

```python
# Interface central — o crawler depende apenas desta abstração
class ExtractionStrategy:
    def extract(self, url: str, html: str) -> List[Dict]:
        raise NotImplementedError

    def run(self, url: str, sections: List[str]) -> List[Dict]:
        raise NotImplementedError
```

Quatro implementações concretas e intercambiáveis:

|Classe|Tipo|Depende de LLM?|
|-|-|-|
|`LLMExtractionStrategy`|Extração semântica|Sim|
|`JsonCssExtractionStrategy`|Seletores CSS/XPath|Não|
|`CosineStrategy`|Similaridade semântica|Não (ML local)|
|`RegexExtractionStrategy`|Padrões regex|Não|

#### Diagnóstico

O padrão mais bem aplicado do projeto. O `AsyncWebCrawler` nunca conhece a implementação concreta — depende apenas da abstração. Troca de estratégia sem modificar o crawler. Implementação alinhada ao OCP e ao DIP do SOLID.

---

### C.3.2 — Chain of Responsibility parcial (FilterChain) 

#### Evidência Técnica

O `FilterChain`, em `crawl4ai/deep_crawling/filters.py`, implementa uma cadeia de filtros aplicados sequencialmente sobre cada URL descoberta durante o deep crawl:

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/deep_crawling/filters.py
```

Filtros disponíveis: `URLPatternFilter`, `DomainFilter`, `ContentTypeFilter`, `FilterChain`.

#### Diagnóstico

Implementação correta e isolada do padrão Chain of Responsibility — cada filtro decide se passa a URL adiante ou a descarta. Porém o padrão está restrito ao subsistema de deep crawl e **não foi aplicado ao pipeline principal de processamento**, que é onde o problema é mais crítico.

---

### C.3.3 — Ausência de Chain of Responsibility no pipeline principal 

#### Evidência Técnica

O método `arun()` do `AsyncWebCrawler`, em `crawl4ai/async_webcrawler.py`, orquestra diretamente todas as etapas do pipeline dentro da mesma classe:

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/async_webcrawler.py
```

As etapas executadas sequencialmente dentro de `arun()`:

```
1. Verificação de cache
2. Alocação de browser do pool
3. Navegação e renderização da página
4. Filtragem de conteúdo (Pruning / BM25)
5. Geração de Markdown
6. Execução da ExtractionStrategy
7. Empacotamento do CrawlResult
```

Toda lógica de retry e backoff para chamadas ao LLM também está acoplada dentro de `LLMExtractionStrategy`, em vez de ser uma etapa independente.

#### Diagnóstico

O pipeline principal é um emaranhado de chamadas sequenciais dentro de um único método. Esse é o gap comportamental mais crítico do projeto:

* **Violação de SRP:** o `AsyncWebCrawler` acumula responsabilidades de cache, navegação, filtragem, geração de conteúdo e extração
* **Violação de OCP:** adicionar, remover ou reordenar etapas do pipeline exige modificar o arquivo do crawler
* **Lógica de resiliência acoplada:** o retry/backoff do LLM está misturado com a lógica de extração

#### 

---

### C.3.4 — Observer parcial (on_state_change) 

#### Evidência Técnica

O `BFSDeepCrawlStrategy` aceita o parâmetro `on_state_change` — um callback invocado após cada URL processada, permitindo persistir estado externamente (ex: Redis).

```
https://github.com/unclecode/crawl4ai/blob/main/crawl4ai/deep_crawling/
```

#### Diagnóstico

Aproximação ao padrão Observer, mas limitada ao subsistema de deep crawl. Não existe um sistema de eventos formal para o pipeline principal — eventos como "página crawleada", "extração concluída" ou "erro de provider" não são expostos de forma desacoplada. Desenvolvedores que precisam de monitoramento detalhado precisam modificar o código-fonte.

#### 

## Conclusão

O Crawl4AI demonstra maturidade arquitetural no uso do padrão Strategy — especialmente na `ExtractionStrategy`, que é o componente mais bem projetado do sistema. O Object Pool do `BrowserManager` e o Adapter via LiteLLM também são aplicações corretas dos padrões criacional e estrutural, respectivamente.

No entanto, três gaps representam dívidas técnicas relevantes: a ausência de Singleton para o modelo de embeddings, a ausência de Factory para validação de providers de LLM e a ausência de Chain of Responsibility no pipeline principal. Este último é o mais crítico — o `AsyncWebCrawler` acumula responsabilidades que deveriam estar distribuídas em handlers independentes, o que viola SRP e OCP e dificulta a evolução do sistema.

O Plano de Resgate proposto — aplicação de Chain of Responsibility no pipeline e Factory Method para providers — endereça diretamente os riscos de maior impacto identificados nesta auditoria, com alinhamento ao CMMI TS SP 2.1 e às práticas de construção do MPS.BR (CON).

---

## Ferramentas de IA Utilizadas

Para a elaboração desta seção, foi utilizado o Claude como ferramenta de apoio na estruturação e redação do texto, bem como na análise das evidências obtidas a partir do repositório GitHub do Crawl4AI.
