# Do SLM agents screen papers as well as LLMs?

## Autores [João Goulart Mendes de Freitas Filho](mailto:joao.filho63204@alunos.ufersa.edu.br) e [Sílvio R. Fernandes](mailto:silvio@ufersa.edu.br)

 
### Universidade Federal Rural do Semi-Árido (UFERSA), Mossoró - RN - Brasil.

### Departamento de Computação
### Group of Embedded Systems and Computer Architecture 

## [ENIAC 2025 - Encontro Nacional de Inteligência Artifial E Computacional](https://bracis.sbc.org.br/2025)

## Resumo

A Revisão Sistemática da Literatura (RSL) é um método crucial para a sumarização do conhecimento científico, especialmente em pesquisas baseadas em evidências. Este processo, no entanto, é complexo e demorado. Uma das etapas mais críticas e que consome mais tempo é a triagem de artigos, onde uma grande porcentagem dos trabalhos inicialmente retornados é descartada.

Este trabalho investiga o uso de Modelos de Linguagem Pequenos (SLMs), executados localmente, para a tarefa de triagem de artigos, uma área com poucos estudos disponíveis, apesar dos resultados promissores com Modelos de Linguagem Grandes (LLMs) comerciais. Analisamos três abordagens distintas utilizando SLMs e as comparamos com um LLM comercial como linha de base.

Os resultados demonstram que os SLMs podem atingir um bom desempenho quando as tarefas são simplificadas e divididas em subtarefas. O modelo Qwen 3-8B alcançou uma acurácia de até 94,35%. Em uma abordagem multiagente, o Phi 4-14B e o Qwen 3-4B obtiveram, respectivamente, 79,5% e 78,8% de acurácia em relação ao LLM comercial.

## Metodologia

Neste estudo, foram utilizados diversos SLMs de código aberto, executados através da plataforma Ollama em suas versões quantizadas de 4 bits. O framework LangChain foi empregado para orquestrar os modelos, seus prompts e interações.

### Tabela de SLMs

<img src="assets/tabela_slms.jpg">

A experimentação foi baseada em uma RSL sobre a geração de dados sintéticos para o reconhecimento de atividade humana. Foram definidos três critérios de inclusão para a triagem de 619 artigos, selecionados a partir das bases de dados Springer, IEEE, ACM e Elsevier.

As três abordagens implementadas foram:

1.  **Agente Único (2 classes):** Um único agente classificava os artigos como "incluído" ou "excluído".
2.  **Agente Único (5 classes):** Um único agente classificava os artigos em "incluído", "excluído 1", "excluído 2", "excluído 3" ou "indeterminado".
3.  **Multiagentes:** Três agentes analisavam cada um dos critérios de inclusão individualmente, e um quarto agente realizava a classificação final com base nas respostas dos anteriores.

### Prompts Utilizados

Abaixo estão os prompts detalhados para cada abordagem.

#### Abordagem 1: Agente Único (2 classes)
```
You are an expert in article screening and should follow the inclusion criteria described below:
The inclusion criteria are:
1. The article proposes data synthesis;
2. The article uses generative neural networks to generate synthetic data;
3. The article addresses the recognition of human activities, regardless of whether the term "Human Activity Recognition" is explicitly used.

Your goal is to read the title and abstract of an article, and then provide a response indicating the classification (’INCLUDED’ or ’EXCLUDED’) and a brief justification for your classification. The article under review should be classified as INCLUDED only if ALL inclusion criteria are met, otherwise it should be classified as EXCLUDED.

Your output should be a JSON object with two keys: "classification" and "justification".
For example:
{"classification": "INCLUDED", "justification": "This article meets all three inclusion criteria."}
or
{"classification": "EXCLUDED", "justification": "This article does not meet inclusion criterion 1 as it does not propose data synthesis."}

Title: {title}
Abstract: {abstract}
```

#### Abordagem 2: Agente Único (5 classes)
```
You are an expert in article screening and should follow the inclusion criteria described below:
The inclusion criteria are:
1. The article proposes data synthesis;
2. The article uses generative neural networks to generate synthetic data;
3. The article addresses the recognition of human activities, regardless of whether the term "Human Activity Recognition" is explicitly used.

Your goal is to read the title and abstract of an article and then provide a response indicating the classification (’INCLUDED’, ’EXCLUDED 1’, ’EXCLUDED 2’, ’EXCLUDED 3’ or ’UNDETERMINED’) and a brief justification for your classification. 

The article under review should be classified as ’INCLUDED’ only if ALL inclusion criteria are met; otherwise, it should be classified as ’EXCLUDED 1’ if it does not meet inclusion criterion 1, ’EXCLUDED 2’ if it does not meet inclusion criterion 2 and ’EXCLUDED 3’ if it does not meet inclusion criterion 3. If you cannot classify it in any of the above, classify it as ’UNDETERMINED’.

Your output should be a JSON object with two keys: "classification" and "justification".
For example:
{{"classification": "INCLUDED", "justification": "This article meets all three inclusion criteria because it proposes data synthesis using generative neural networks for human activity recognition."}}
or
{{"classification": "EXCLUDED 1", "justification": "This article does not meet inclusion criterion 1 as it does not propose data synthesis."}}
or
{{"classification": "UNDETERMINED", "justification": "For this article I cannot determine whether it can be included or does not meet one of the criteria for exclusion."}}

Title: {title}
Abstract: {abstract}
```

#### Abordagem 3: Multiagentes

**Prompt para os Agentes Verificadores de Critério):**
```
You are an expert in screening scientific articles for literature review on the topic: Human Activity Recognition (HAR).

You must follow the following criteria to indicate whether it should be included in a literature review.
Inclusion Criteria:
{inclusion_criteria}

Your task is to classify the article strictly according to the above criteria.
- If the article perfectly meets the inclusion criteria, classify it as "TRUE".
- If you are certain that the article does not meet the inclusion criteria, classify it as "FALSE".
- If there is insufficient information or you have doubts whether the article meets the inclusion criteria, classify it as "UNDETERMINATED".

Be clear and decisive. Avoid "UNDETERMINATED" unless absolutely necessary.
Your final answer should be ONLY a JSON object with the keys: "classification" and "justification".

Example output for you to follow:
{"classification": "TRUE", "justification": "The article proposes an approach to data synthesis."}
{"classification": "FALSE", "justification": "The article uses synthetic data but does not propose its synthesis."}
{"classification": "UNDETERMINATED", "justification": "There is not enough information in the title and abstract to determine."}

Now analyze the title and abstract of the following article and classify it.

Title: {title}
Abstract: {abstract
```
*(Nota: O `{incluscion_criteria}` foi substituído por por cada critério específico para cada um dos três agentes verificadores)*

## Resultados

Os resultados indicam que a **Abordagem 1** apresentou a maior acurácia, com destaque para os modelos Qwen3_8b (94,35%), Qwen2.5-7b (93,7%) e Phi4_14b (93,38%). No entanto, a falta de rastreabilidade nas exclusões torna essa abordagem insuficiente para o processo de RSL.


<img src="assets/acuracia_ap1.jpg">


A **Abordagem 2** demonstrou uma queda na acurácia em comparação com a primeira, com os melhores desempenhos sendo do Deepseek-r1_14b (64,01%) e Qwen2.5-14b (59,45%).

<img src="assets/acuracia_ap2.jpg">

A **Abordagem 3**, de "dividir para conquistar", mostrou que a tarefa de verificação final se tornou complexa para um SLM. Os melhores F1-Scores foram obtidos pelos modelos Gemma3_12b, Llama3.1_8b e Qwen3_4b. A acurácia geral, no entanto, foi menor em comparação com a Abordagem 1, com os modelos Phi4_14b e Qwen3_4b apresentando os melhores resultados (79,5% e 78,8%, respectivamente).

<img src="assets/acuracia_ap3.jpg">

### Outras métricas para avaliaçãp da abordagem 3
<img src="assets/f1-prec-recall.jpg">


## Citation

```
@inproceedings{freitasfilho2025slm,
  title={Do SLM agents screen papers as well as LLMs?},
  author={Freitas Filho, João Goulart Mendes and Fernandes, Síilvio R.},
  booktitle={Anais do Encontro Nacional de Inteligência Artificial e Computacional},
  year={2025},
  organization={Sociedade Brasileira de Computação}
}
```

## Contato

Para mais informações, entre em contato com os autores:

* **João Goulart Mendes de Freitas Filho:** joao.filho63204@alunos.ufersa.edu.br
* **Sílvio R. Fernandes:** silvio@ufersa.edu.br