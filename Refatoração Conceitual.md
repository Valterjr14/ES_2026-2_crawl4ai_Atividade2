# Refatoração Conceitual

---

## Evidência Técnica

Durante a análise do repositório Crawl4AI foi identificado que o sistema possui suporte a diversos provedores de modelos de linguagem (LLMs), incluindo OpenAI, Ollama, Gemini, Claude, DeepSeek e outros modelos integrados através do LiteLLM.

### Exemplo encontrado na documentação oficial

```python
llm_config = LLMConfig(
    provider="openai/gpt-4o-mini",
    api_token=os.getenv("OPENAI_API_KEY")
)
```

### Outro exemplo

```python
llm_config = LLMConfig(
    provider="ollama/llama3"
)
```

### Fontes

- Repositório Crawl4AI
- Documentação LLM Strategies
- Documentação Parameters

Observa-se que a escolha do provedor é realizada através de identificadores textuais ("strings"), passados diretamente para a configuração do sistema.

---

## Diagnóstico

Embora a arquitetura do Crawl4AI tenha sido projetada para ser independente do provedor de IA (*provider-agnostic*), a seleção dos modelos depende diretamente de valores textuais específicos.

### Exemplos

```python
provider="openai/gpt-4o"
provider="ollama/llama3"
provider="gemini/gemini-pro"
provider="deepseek/deepseek-chat"
```

Essa abordagem funciona adequadamente para projetos pequenos e médios, porém pode gerar dificuldades de manutenção à medida que o número de provedores suportados aumenta.

### Principais problemas identificados

- Dependência de nomes específicos de provedores;
- Risco de erros de configuração;
- Falta de validação centralizada;
- Crescimento do acoplamento entre módulos;
- Maior dificuldade para expansão futura.

---

## Classificação de Risco

| Critério | Avaliação |
|-----------|-----------|
| Probabilidade | Média/Alta |
| Impacto | Médio |
| Nível de Risco | Médio |

---

## Violação Identificada

### Princípio D – Dependency Inversion Principle (SOLID)

Os módulos de alto nível ainda possuem conhecimento indireto das implementações concretas dos provedores de IA.

Em vez de depender apenas de abstrações, partes do sistema precisam conhecer identificadores específicos como:

```text
openai
ollama
gemini
deepseek
```

Isso cria dependências que podem dificultar a evolução da arquitetura.

---

# Proposta de Refatoração

## Aplicação do Padrão Factory Method (GoF)

Como plano de resgate arquitetural, propõe-se a utilização do padrão criacional **Factory Method**.

A ideia consiste em centralizar a criação dos provedores de IA em uma única fábrica responsável por instanciar o componente adequado.

Dessa forma, o restante do sistema deixa de depender diretamente dos provedores específicos.

---

## Estrutura Atual

```text
Crawler
   ↓
LLMConfig
   ↓
provider="openai/gpt-4o"
```

### Problemas

- Dependência direta de strings;
- Baixa padronização;
- Expansão mais complexa;
- Maior acoplamento.

---

## Estrutura Proposta

```text
                     ILLMProvider
                           ▲
                           │
      ┌────────────────────┼────────────────────┐
      │                    │                    │
      ▼                    ▼                    ▼

OpenAIProvider     GeminiProvider     OllamaProvider

                           ▲
                           │

                     LLMFactory

                           ▲
                           │

                        Crawler
```

---

## UML Simplificada

```text
+----------------+
| ILLMProvider   |
+----------------+
| generate()     |
+----------------+
         ▲
         │

 ----------------------------
 |            |            |
 ▼            ▼            ▼

OpenAI     Gemini      Ollama

         ▲
         │

+----------------+
| LLMFactory     |
+----------------+
| create()       |
+----------------+

         ▲
         │

+----------------+
| Crawler        |
+----------------+
```

---

## Exemplo Conceitual

### Interface

```python
class ILLMProvider:

    def generate(self, prompt):
        pass
```

### Implementação OpenAI

```python
class OpenAIProvider(ILLMProvider):

    def generate(self, prompt):
        return openai.chat(prompt)
```

### Implementação Ollama

```python
class OllamaProvider(ILLMProvider):

    def generate(self, prompt):
        return ollama.chat(prompt)
```

### Factory

```python
class LLMFactory:

    @staticmethod
    def create(provider):

        providers = {
            "openai": OpenAIProvider,
            "ollama": OllamaProvider,
            "gemini": GeminiProvider
        }

        return providers[provider]()
```

---

# Benefícios Esperados

## Redução de Acoplamento

O crawler deixa de conhecer detalhes internos de cada provedor.

## Facilidade de Expansão

Novos provedores podem ser adicionados sem alterar a lógica principal.

## Melhor Manutenção

Mudanças em APIs externas ficam isoladas em suas respectivas implementações.

## Melhor Testabilidade

Possibilita criação de mocks e testes automatizados mais simples.

## Maior Aderência ao SOLID

A solução atende de forma mais adequada ao princípio da Inversão de Dependência.

## Evolução da Arquitetura

A arquitetura torna-se mais preparada para suportar novos modelos e futuras mudanças tecnológicas.

---

# Conclusão

A auditoria identificou que o Crawl4AI possui uma arquitetura moderna e flexível para integração com múltiplos modelos de IA. Entretanto, a dependência de identificadores textuais para seleção dos provedores representa um ponto de melhoria arquitetural.

Como plano de resgate técnico, foi proposta a aplicação do padrão **Factory Method**, permitindo centralizar a criação dos provedores e reduzir o acoplamento entre os módulos do sistema.

A adoção dessa solução contribuiria para maior escalabilidade, facilidade de manutenção e melhor aderência aos princípios **SOLID** e aos padrões **GoF**.

## Ferramentas de IA Utilizadas

Para a elaboração desta seção foi utilizado o ChatGPT (OpenAI) como ferramenta de apoio na análise das evidências do repositório Crawl4AI, na organização das informações coletadas e na estruturação da proposta de refatoração conceitual apresentada neste relatório.
