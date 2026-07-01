# Biohub Cell Tracking
### Pipeline de Inferência para Rastreamento Celular 3D sob Restrição de Recursos

> **Estudo de engenharia aplicado ao desafio Biohub – Cell Tracking During Development (Kaggle 2026).**
>
> Desenvolvimento de um pipeline modular para detecção, rastreamento e reconstrução de linhagem celular em microscopia 3D+t, projetado para operar de forma **offline**, **reprodutível**, **auditável** e dentro de um **SLA de 12 horas**, conforme as restrições da competição.

---

<!-- Badges -->

![Python](https://img.shields.io/badge/Python-3.13+-3776AB?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-Latest-EE4C2C?logo=pytorch&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Code Style](https://img.shields.io/badge/Code%20Style-Black-black)
![Lint](https://img.shields.io/badge/Lint-Ruff-blue)
![Type%20Checking](https://img.shields.io/badge/Type%20Checking-MyPy-blueviolet)
![Tests](https://img.shields.io/badge/Tests-PyTest-success)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-success)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-Code%20Competition-20BEFF?logo=kaggle)

---

## O Problema

O rastreamento celular em microscopia tridimensional é um dos principais gargalos da biologia do desenvolvimento.

Pesquisadores precisam acompanhar milhares de células ao longo do tempo para compreender processos como crescimento embrionário, diferenciação celular, migração e divisão celular.

Embora existam ferramentas automatizadas, muitas ainda apresentam limitações quando aplicadas a cenários reais, especialmente em ambientes caracterizados por:

- alta densidade celular;
- baixo contraste entre células vizinhas;
- ruído de aquisição;
- deformações estruturais;
- eventos de mitose;
- grandes volumes de dados 3D.

Como consequência, boa parte desse trabalho ainda depende de rastreamento manual realizado por especialistas, tornando experimentos complexos demorados, caros e pouco escaláveis.

---

## A Competição

Este projeto foi desenvolvido tomando como estudo de caso a competição **Biohub – Cell Tracking During Development**, promovida no Kaggle.

O desafio consiste em detectar células em volumes de microscopia 3D ao longo do tempo e reconstruir corretamente sua linhagem celular, produzindo um grafo contendo:

- nós (detecções celulares);
- conexões temporais entre células;
- identificação correta de eventos de divisão celular.

A avaliação considera simultaneamente:

- qualidade do rastreamento (Edge Jaccard);
- qualidade da reconstrução das divisões celulares (Division Jaccard).

Mais do que maximizar uma métrica, este projeto busca responder a uma questão de engenharia:

> **Como construir um pipeline de Machine Learning robusto, reproduzível e auditável capaz de operar sob restrições rígidas de tempo, memória e ausência de conexão com a internet?**

---

# Objetivos

Este projeto possui quatro objetivos principais.

### Científico

Investigar arquiteturas modernas para rastreamento celular em imagens biomédicas tridimensionais.

### Engenharia

Projetar um pipeline modular que possa ser compreendido, auditado e evoluído de maneira independente.

### Competição

Implementar uma solução compatível com todas as restrições técnicas da competição do Kaggle.

### Portfólio

Construir um projeto open source que demonstre competências em:

- Machine Learning
- Computer Vision
- Deep Learning
- Data Engineering
- MLOps
- Engenharia de Software
- Arquitetura de Sistemas
- Python

---

# Filosofia do Projeto

Este repositório foi concebido seguindo três princípios fundamentais.

## 1. Engenharia antes da otimização

Antes de buscar o melhor score, o projeto prioriza:

- arquitetura;
- organização;
- rastreabilidade;
- documentação;
- reprodutibilidade.

---

## 2. Decisões técnicas explícitas

Toda decisão arquitetural possui:

- contexto;
- alternativas consideradas;
- trade-offs;
- justificativa técnica.

Nenhuma escolha importante permanece implícita.

---

## 3. Ciência reproduzível

Todo resultado apresentado deverá ser reproduzível utilizando apenas:

- código versionado;
- dados públicos permitidos;
- notebooks;
- documentação do projeto.

Sem dependências externas ocultas.

---

# Arquitetura de Alto Nível

```text
Volumes 3D (.zarr)
          │
          ▼
Pré-processamento
          │
          ▼
Detecção Celular
          │
          ▼
Extração de Centroides
          │
          ▼
Linking Temporal
          │
          ▼
Reconstrução de Linhagem
          │
          ▼
Validação
          │
          ▼
submission.csv
```

Toda a arquitetura foi projetada para operar:

- offline;
- sem acesso à internet;
- dentro do limite de execução do Kaggle;
- com execução reproduzível;
- com baixo acoplamento entre módulos.

---

# Sumário

- O Problema
- A Competição
- Objetivos
- Filosofia do Projeto
- Arquitetura
- Metodologia
- Estrutura do Repositório
- Instalação
- Quick Start
- Pipeline de Machine Learning
- Decisões Técnicas
- Resultados
- Benchmark
- Roadmap
- Contribuição
- Licença
- Citação
- Agradecimentos

---
# Metodologia O desenvolvimento da solução segue uma abordagem modular inspirada em pipelines de Machine Learning utilizados em ambientes de produção. Em vez de um modelo único fim-a-fim, o problema é decomposto em etapas independentes, permitindo: - validação isolada de cada componente; - depuração mais simples; - substituição de módulos sem reescrever todo o pipeline; - maior auditabilidade; - melhor controle de consumo computacional. --- # Pipeline de Machine Learning O fluxo completo da solução é composto por seis estágios. ## 1. Ingestão dos dados Os volumes de microscopia são fornecidos no formato `.zarr`. Nesta etapa o pipeline: - lê os volumes 3D+t; - valida dimensões e metadados; - identifica escala física do voxel; - prepara os dados para processamento em blocos. ```text .zarr → Volume 4D (t, z, y, x) ``` --- ## 2. Pré-processamento Objetivos: - normalização de intensidade; - redução de ruído; - padronização espacial; - preparação para inferência. Esta etapa é executada localmente e não depende de acesso à internet. --- ## 3. Detecção celular Responsável por localizar células em cada timepoint. Saída: ```text (t, z, y, x) ``` A implementação inicial utiliza um detector calibrado para priorizar **precisão** sobre **recall absoluto**, alinhando-se à métrica oficial da competição, que penaliza excesso de nós previstos. --- ## 4. Linking temporal As detecções de frames consecutivos são associadas utilizando custo baseado em distância física. Características: - voxel anisotrópico; - custo em micrômetros; - assignment bipartido; - restrição temporal local. Saída: ```text célula_t → célula_t+1 ``` --- ## 5. Reconstrução de linhagem Nesta etapa o pipeline: - identifica divisões celulares; - cria nós-filhos; - monta o grafo completo de linhagem; - remove inconsistências estruturais. Uma divisão é representada quando um nó possui duas ou mais arestas de saída válidas. --- ## 6. Geração da submissão O resultado final é convertido para o formato exigido pelo Kaggle: ```text nodes + edges → submission.csv ``` --- # Diagrama Detalhado ```text Volumes .zarr │ ▼ ┌──────────────────────┐ │ Pré-processamento │ └──────────────────────┘ │ ▼ ┌──────────────────────┐ │ Detecção celular │ └──────────────────────┘ │ ▼ Centroides (t,z,y,x) │ ▼ ┌──────────────────────┐ │ Linking temporal │ └──────────────────────┘ │ ▼ Arestas candidatas │ ▼ ┌──────────────────────┐ │ Reconstrução │ │ de linhagem │ └──────────────────────┘ │ ▼ Grafo final │ ▼ submission.csv ``` --- # Decisões Técnicas ## Pipeline modular vs. modelo fim-a-fim **Escolha:** pipeline modular. ### Motivos - depuração simplificada; - troca de componentes; - melhor observabilidade; - menor acoplamento; - compatibilidade com ambientes regulados. ### Trade-off Possível perda de alguns ganhos que um modelo fim-a-fim poderia capturar. --- ## Offline-first **Escolha:** toda a inferência deve funcionar sem internet. ### Motivos - requisito da competição; - reprodutibilidade; - previsibilidade operacional. ### Trade-off Maior esforço de empacotamento de dependências e modelos. --- ## SLA de 12 horas **Escolha:** throughput é tratado como requisito funcional. ### Motivos Um pipeline que não termina dentro do prazo é inutilizável na competição. ### Trade-off Arquiteturas extremamente pesadas podem ser descartadas mesmo sendo potencialmente mais precisas. --- ## Uso do Ultrack como baseline **Escolha:** utilizar o método de referência publicado pelo grupo responsável pelo dataset. ### Motivos - benchmark sólido; - comparação justa; - foco em engenharia do pipeline. ### Trade-off Dependência de uma biblioteca externa. --- # Arquiteturas Avaliadas O projeto considera três famílias principais de modelos. | Arquitetura | Papel | Status | |-------------|--------|--------| | U-Net 3D | Detecção | Baseline | | GraphSAGE (GNN) | Linking/Linhagem | Pesquisa | | Transformer espaçotemporal | Refinamento temporal | Roadmap | No escopo inicial, o foco está em obter um pipeline completo, reproduzível e compatível com o SLA antes de introduzir componentes de maior custo computacional. --- # Estrutura do Repositório ```text biohub-cell-tracking/ ├── src/ ├── tests/ ├── configs/ ├── docs/ ├── notebooks/ ├── experiments/ ├── scripts/ ├── models/ └── assets/ ``` A documentação detalhada de arquitetura e decisões encontra-se em: - `docs/ARCHITECTURE.md` - `docs/DECISIONS.md` - `docs/ADR/` 


---


---

# Instalação

## Pré-requisitos

Antes de executar o projeto, certifique-se de possuir o seguinte ambiente configurado.

### Sistema Operacional

O projeto foi desenvolvido para ser compatível com:

- Linux
- macOS
- Windows (WSL2 recomendado)

### Python

- Python 3.13 ou superior

### Gerenciador de Pacotes

Recomenda-se utilizar um ambiente virtual.

Exemplo utilizando `venv`:

```bash
python -m venv .venv

source .venv/bin/activate        # Linux / macOS

.\.venv\Scripts\activate         # Windows
```

---

## Clonando o repositório

```bash
git clone https://github.com/Santosdevbjj/biohub-cell-tracking.git

cd biohub-cell-tracking
```

---

## Instalando as dependências

```bash
pip install --upgrade pip

pip install -r requirements.txt

pip install -r requirements-dev.txt
```

ou

```bash
make install
```

---

# Estrutura do Projeto

```
biohub-cell-tracking/

.github/
│
├── workflows/
│
assets/
│
configs/
│
data/
│
docker/
│
docs/
│
├── adr/
├── architecture/
├── benchmark/
├── diagrams/
│
models/
│
notebooks/
│
scripts/
│
src/
│
└── biohub_tracking/
│
tests/

README.md
LICENSE
Dockerfile
Makefile
pyproject.toml
requirements.txt
requirements-dev.txt
```

Cada diretório possui uma responsabilidade única, reduzindo acoplamento e facilitando manutenção.

---

# Quick Start

Após instalar todas as dependências, execute:

```bash
make check
```

Esse comando executa automaticamente:

- Ruff
- Black
- MyPy
- Pytest

Caso todos os testes sejam aprovados, o ambiente estará pronto para desenvolvimento.

---

## Primeiro Pipeline

Executar:

```bash
python scripts/run_pipeline.py
```

Fluxo esperado:

```
Leitura dos dados

↓

Pré-processamento

↓

Detecção

↓

Tracking

↓

Reconstrução

↓

submission.csv
```

---

# Executando os Testes

Todos os testes:

```bash
pytest
```

Cobertura:

```bash
pytest --cov
```

Testes rápidos:

```bash
pytest -m unit
```

Testes de integração:

```bash
pytest -m integration
```

---

# Qualidade de Código

O projeto segue ferramentas utilizadas em projetos profissionais.

## Formatação

```bash
black .
```

---

## Lint

```bash
ruff check .
```

---

## Type Checking

```bash
mypy src
```

---

## Hooks

Instalação:

```bash
pre-commit install
```

Execução:

```bash
pre-commit run --all-files
```

---

# Docker

Construção da imagem:

```bash
docker build -t biohub-cell-tracking .
```

Execução:

```bash
docker run biohub-cell-tracking
```

---

# GitHub Actions

Toda Pull Request executará automaticamente:

- Instalação das dependências

- Ruff

- Black

- MyPy

- Pytest

- Build do projeto

Nenhum código será incorporado à branch principal sem aprovação da pipeline de integração contínua.

---

# Configuração

Todos os parâmetros do projeto ficam centralizados em:

```
configs/
```

Exemplos:

```
configs/

dataset.yaml

training.yaml

tracking.yaml

submission.yaml
```

Essa abordagem evita alterar código para modificar experimentos.

---

# Reprodutibilidade

Um dos principais objetivos deste projeto é garantir experimentos reproduzíveis.

Para isso:

- dependências possuem versionamento;

- configurações ficam separadas do código;

- seeds são controladas;

- resultados possuem rastreabilidade;

- notebooks não são utilizados como fonte principal da implementação.

Toda lógica do projeto reside dentro do pacote Python localizado em:

```
src/biohub_tracking/
```

Os notebooks funcionam apenas como ambiente de experimentação e visualização.

---

# Executando no Kaggle

A submissão oficial será realizada exclusivamente por meio de um Kaggle Notebook.

O notebook será responsável apenas por:

1. carregar os modelos;

2. executar o pipeline de inferência;

3. gerar o arquivo:

```
submission.csv
```

Toda a lógica de negócio permanecerá encapsulada no pacote Python do projeto, permitindo reutilização tanto localmente quanto na plataforma do Kaggle.

---

# Estratégia de Desenvolvimento

O desenvolvimento segue uma abordagem incremental.

Cada funcionalidade passa pelas seguintes etapas:

1. definição do problema;

2. implementação;

3. testes unitários;

4. testes de integração;

5. benchmark;

6. documentação;

7. publicação.

Nenhuma funcionalidade é considerada concluída sem documentação e testes automatizados.

---

# Organização dos Experimentos

Todos os experimentos serão documentados.

Para cada experimento serão registrados:

- hipótese;

- configuração utilizada;

- métricas;

- tempo de execução;

- consumo de memória;

- conclusão.

O objetivo é garantir que toda decisão técnica seja sustentada por evidências reproduzíveis, e não apenas por resultados isolados. 

---

---

# Baseline

Antes de otimizar qualquer componente do pipeline, é fundamental estabelecer um ponto de comparação.

Neste projeto utilizamos três níveis de baseline.

## Baseline Científico

O método de referência adotado é o **Ultrack**, desenvolvido pelo grupo responsável pelo dataset e publicado na literatura científica.

Ele representa o estado da arte disponível publicamente para rastreamento celular em microscopia 3D+t e serve como referência técnica para comparação da arquitetura proposta.

---

## Baseline Humano

Em muitos laboratórios, parte significativa do rastreamento celular ainda é realizada manualmente por especialistas.

Embora esse processo produza resultados de alta qualidade, apresenta limitações importantes:

- elevado custo operacional;
- baixa escalabilidade;
- grande consumo de tempo;
- variabilidade entre anotadores.

Um dos objetivos deste projeto é investigar como pipelines automatizados podem reduzir esse gargalo mantendo rastreabilidade e reprodutibilidade.

---

## Baseline Operacional

Além da qualidade científica, existe uma restrição operacional imposta pela competição:

- execução offline;
- ausência de acesso à internet;
- limite máximo de 12 horas;
- memória limitada;
- ambiente controlado do Kaggle.

Por esse motivo, throughput e consumo de recursos são tratados como requisitos de projeto, e não apenas como métricas secundárias.

---

# Métricas de Avaliação

O desempenho do pipeline será analisado em duas dimensões complementares.

## Métricas da Competição

As métricas oficiais incluem:

- Edge Jaccard
- Division Jaccard

Essas métricas avaliam simultaneamente a qualidade do rastreamento temporal e a reconstrução correta das divisões celulares.

---

## Métricas Operacionais

Além da pontuação da competição, o projeto registra indicadores de engenharia.

### Tempo total de execução

Tempo necessário para processar integralmente um conjunto de dados.

---

### Throughput

Quantidade de volumes processados por unidade de tempo.

---

### Consumo de memória

Uso máximo de memória RAM e GPU durante a inferência.

---

### Reprodutibilidade

Capacidade de reproduzir exatamente os mesmos resultados utilizando:

- mesma configuração;
- mesmas dependências;
- mesma seed;
- mesmo conjunto de dados.

---

# Estratégia Experimental

Cada melhoria implementada segue um protocolo padronizado.

## 1. Hipótese

Exemplo:

> "Substituir o algoritmo de linking reduzirá ambiguidades em regiões de alta densidade celular."

---

## 2. Implementação

Desenvolvimento isolado da modificação proposta.

---

## 3. Validação

Execução no conjunto de validação.

---

## 4. Comparação

Comparação direta com o baseline.

---

## 5. Conclusão

Registro dos resultados obtidos.

Nenhuma alteração será incorporada ao pipeline principal sem evidências experimentais que justifiquem sua adoção.

---

# Experimentos

Todos os experimentos serão registrados em:

```
experiments/
```

Cada experimento possuirá:

```
experiments/

experiment_001/

config.yaml

results.json

metrics.csv

notes.md

plots/

logs/
```

Essa organização garante rastreabilidade completa das decisões tomadas durante o desenvolvimento.

---

# Decisões Arquiteturais (ADR)

As decisões técnicas relevantes serão documentadas utilizando o padrão **Architecture Decision Records (ADR)**.

Exemplos:

```
docs/

ADR/

ADR-001-pipeline-modular.md

ADR-002-ultrack-baseline.md

ADR-003-offline-first.md

ADR-004-runtime-budget.md

ADR-005-linking-strategy.md
```

Cada ADR responderá às seguintes perguntas:

- Qual problema motivou a decisão?
- Quais alternativas foram consideradas?
- Por que esta solução foi escolhida?
- Quais trade-offs foram aceitos?
- Quais impactos futuros são esperados?

---

# Benchmark

Os resultados obtidos serão comparados de forma transparente.

| Pipeline | Edge Jaccard | Division Jaccard | Runtime | Memória | Status |
|-----------|--------------|------------------|---------|----------|--------|
| Ultrack | — | — | — | — | Baseline |
| Pipeline SDI v1 | — | — | — | — | Em desenvolvimento |
| Pipeline SDI v2 | — | — | — | — | Planejado |

Todos os valores publicados possuirão origem rastreável em notebooks, logs e arquivos de configuração.

---

# Business Performance

Embora este projeto tenha origem em uma competição científica, sua arquitetura foi concebida para refletir desafios encontrados em ambientes produtivos.

Além das métricas técnicas, serão analisados indicadores como:

- redução do tempo de processamento;
- previsibilidade de execução;
- facilidade de manutenção;
- escalabilidade do pipeline;
- reprodutibilidade dos experimentos.

A intenção é demonstrar que desempenho operacional também faz parte da qualidade de uma solução de Machine Learning.

---

# Limitações Conhecidas

Até o momento, algumas limitações já são conhecidas.

- O pipeline inicial prioriza modularidade em relação ao desempenho máximo.
- Componentes de Deep Learning mais sofisticados serão introduzidos gradualmente.
- O foco inicial está na construção de uma base sólida e reproduzível antes da otimização de métricas.
- O desempenho final dependerá das características dos volumes disponibilizados na competição.

Essas limitações são documentadas de forma explícita para garantir transparência durante toda a evolução do projeto.

---

# Roadmap

## Foundation

- [ ] Estrutura profissional do repositório
- [ ] Pipeline de integração contínua
- [ ] Ambiente Docker
- [ ] Ferramentas de qualidade

---

## Data

- [ ] Leitura de arquivos Zarr
- [ ] Data Loader
- [ ] Pré-processamento
- [ ] Visualização dos volumes

---

## Detection

- [ ] Detector inicial
- [ ] Extração de centroides
- [ ] Pós-processamento

---

## Tracking

- [ ] Linking temporal
- [ ] Reconstrução de linhagem
- [ ] Detecção de divisões

---

## Evaluation

- [ ] Implementação das métricas
- [ ] Benchmark
- [ ] Perfil de desempenho

---

## Submission

- [ ] Notebook Kaggle
- [ ] Inferência offline
- [ ] Geração automática do submission.csv

---

## Research

- [ ] U-Net 3D
- [ ] Graph Neural Networks
- [ ] Spatiotemporal Transformers
- [ ] Min-Cost Flow
- [ ] Self-Supervised Learning


---


---

# Arquitetura da Solução

O pipeline foi projetado seguindo princípios de engenharia de software para sistemas críticos:

- Componentes desacoplados;
- Facilidade de auditoria;
- Reprodutibilidade;
- Possibilidade de substituir módulos individualmente;
- Execução offline;
- Cumprimento do SLA imposto pelo Kaggle.

Ao invés de construir um único modelo "end-to-end", o projeto divide claramente cada responsabilidade.

```text
                    Dados Brutos (.zarr)
                             │
                             ▼
┌──────────────────────────────────────────────────────┐
│                Data Loader / Zarr Reader             │
└──────────────────────────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────┐
│          Pré-processamento dos Volumes 3D            │
│ • Normalização                                       │
│ • Padronização                                       │
│ • Denoising                                          │
│ • Preparação para Inferência                         │
└──────────────────────────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────┐
│           Detector de Células (Baseline)             │
│               Ultrack / Detector 3D                  │
└──────────────────────────────────────────────────────┘
                             │
                             ▼
                   Centroides (t,z,y,x)
                             │
                             ▼
┌──────────────────────────────────────────────────────┐
│          Linking Temporal entre Frames               │
│      Assignment Bipartido + Distância Física         │
└──────────────────────────────────────────────────────┘
                             │
                             ▼
                Grafo de Arestas Candidatas
                             │
                             ▼
┌──────────────────────────────────────────────────────┐
│       Resolução de Divisões Celulares                │
└──────────────────────────────────────────────────────┘
                             │
                             ▼
                 Grafo Final de Linhagens
                             │
                             ▼
                  submission.csv (Kaggle)
```

Todo o pipeline foi desenhado para operar inteiramente offline, respeitando as limitações impostas pela competição.

---

# Decisões Arquiteturais

Este projeto segue um princípio simples:

> **Cada decisão técnica precisa possuir uma justificativa de engenharia claramente documentada.**

Ao longo do desenvolvimento, cada mudança importante será registrada em um Architecture Decision Record (ADR), permitindo compreender:

- qual problema existia;
- quais alternativas foram avaliadas;
- por que determinada solução foi escolhida;
- quais trade-offs foram aceitos.

Essa prática é comum em projetos de software de larga escala e facilita manutenção, auditoria e evolução futura.

---

# Decisões Técnicas e Trade-offs

## Decisão 1 — Pipeline Modular

### Alternativa considerada

Construir um modelo único responsável por detectar, rastrear e reconstruir toda a linhagem celular.

### Solução adotada

Separar o pipeline em três módulos independentes:

- Detecção
- Linking
- Reconstrução

### Justificativa

Cada componente pode ser:

- testado isoladamente;
- substituído futuramente;
- otimizado individualmente;
- comparado com novas abordagens.

### Trade-off

O pipeline pode perder parte da otimização global de um modelo fim-a-fim, mas ganha:

- manutenção;
- reprodutibilidade;
- facilidade de depuração.

---

## Decisão 2 — Uso do Ultrack como Baseline

### Alternativa considerada

Desenvolver toda a solução do zero.

### Solução adotada

Utilizar inicialmente o Ultrack como baseline científico.

### Justificativa

O objetivo deste projeto não é reinventar um método consolidado, mas construir uma plataforma de engenharia sobre um baseline reconhecido pela comunidade científica.

Isso permite concentrar esforços em aspectos igualmente importantes:

- arquitetura;
- desempenho;
- organização do código;
- automação;
- reprodutibilidade.

### Trade-off

Dependência de uma biblioteca externa.

Em compensação, ganha-se um ponto de partida sólido para futuras pesquisas.

---

## Decisão 3 — Execução Offline

A competição proíbe acesso à internet durante a submissão.

Em vez de tratar isso como uma limitação apenas no momento da entrega, o projeto adota esse requisito como princípio de arquitetura desde o primeiro commit.

Todos os componentes deverão funcionar completamente offline.

Isso inclui:

- pesos dos modelos;
- bibliotecas;
- configuração;
- inferência.

Essa abordagem aumenta significativamente a confiabilidade do pipeline.

---

## Decisão 4 — SLA de 12 horas

Todo componente introduzido no projeto deverá responder à seguinte pergunta:

> **Ele cabe dentro do orçamento computacional da competição?**

Não basta melhorar a métrica.

Também é necessário garantir:

- tempo de execução;
- consumo de memória;
- estabilidade;
- previsibilidade.

Sempre que houver conflito entre ganho marginal de acurácia e inviabilidade operacional, a decisão deverá ser documentada.

---

## Decisão 5 — Evolução Progressiva do Pipeline

Este projeto seguirá uma estratégia incremental.

Primeiro será construída uma solução simples, reproduzível e totalmente funcional.

Somente depois serão avaliadas arquiteturas mais sofisticadas, como:

- U-Net 3D
- Graph Neural Networks
- Spatio-Temporal Transformers

Essa estratégia reduz risco técnico, facilita experimentação e permite medir claramente o ganho obtido por cada evolução introduzida. 


---

---

# Estrutura do Repositório

O projeto foi organizado para separar claramente responsabilidades, facilitar manutenção, incentivar reutilização de código e permitir evolução incremental do pipeline.

```text
biohub-cell-tracking/
│
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── lint.yml
│   │   ├── tests.yml
│   │   ├── docs.yml
│   │   └── release.yml
│   │
│   ├── ISSUE_TEMPLATE/
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── CODEOWNERS
│
├── configs/
│   ├── default.yaml
│   ├── training.yaml
│   ├── inference.yaml
│   ├── logging.yaml
│   └── paths.yaml
│
├── data/
│   ├── raw/
│   ├── interim/
│   ├── processed/
│   ├── external/
│   └── samples/
│
├── docs/
│   ├── architecture.md
│   ├── adr/
│   ├── benchmark.md
│   ├── decisions.md
│   ├── experiments.md
│   ├── roadmap.md
│   ├── references.md
│   └── diagrams/
│
├── models/
│   ├── checkpoints/
│   ├── pretrained/
│   ├── exported/
│   └── metadata/
│
├── notebooks/
│   ├── exploratory/
│   ├── experiments/
│   ├── benchmarks/
│   └── kaggle/
│
├── reports/
│   ├── figures/
│   ├── tables/
│   ├── logs/
│   └── metrics/
│
├── scripts/
│   ├── setup.sh
│   ├── train.sh
│   ├── inference.sh
│   ├── benchmark.sh
│   ├── validate.sh
│   ├── package.sh
│   └── delete.sh
│
├── src/
│   └── biohub_cell_tracking/
│       ├── data/
│       ├── preprocessing/
│       ├── detection/
│       ├── tracking/
│       ├── lineage/
│       ├── models/
│       ├── metrics/
│       ├── visualization/
│       ├── utils/
│       ├── pipelines/
│       ├── config/
│       └── cli/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── regression/
│   ├── performance/
│   └── fixtures/
│
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── CITATION.cff
├── Dockerfile
├── Makefile
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── README.md
└── .pre-commit-config.yaml
```

---

# Organização das Pastas

## `.github/`

Centraliza toda a automação do repositório.

Inclui:

- Integração Contínua (CI)
- execução automática de testes;
- lint;
- geração de documentação;
- publicação de releases;
- templates para Issues e Pull Requests.

Essa estrutura garante qualidade contínua do projeto.

---

## `configs/`

Armazena todos os arquivos de configuração utilizados pelo pipeline.

Nenhum parâmetro crítico deve permanecer codificado diretamente em Python.

Exemplos:

- localização dos datasets;
- hiperparâmetros;
- caminhos dos modelos;
- configuração de logging;
- parâmetros de inferência.

Isso aumenta reprodutibilidade e facilita experimentos.

---

## `data/`

Responsável pela organização dos dados utilizados durante o projeto.

As subdivisões seguem uma convenção amplamente utilizada em projetos de Ciência de Dados.

### raw/

Dados originais.

Nunca sofrem modificações.

São a fonte oficial do projeto.

### interim/

Dados parcialmente tratados.

Utilizados durante o pré-processamento.

### processed/

Dados prontos para treinamento ou inferência.

### external/

Bases externas autorizadas pela competição.

### samples/

Pequenos subconjuntos utilizados para testes rápidos e desenvolvimento local.

---

## `docs/`

Toda documentação complementar fica concentrada nesta pasta.

Enquanto o README apresenta uma visão geral do projeto, esta pasta documenta aspectos específicos.

Inclui:

- arquitetura;
- benchmarks;
- decisões técnicas;
- experimentos;
- roadmap;
- diagramas;
- referências bibliográficas.

Também abriga os **Architecture Decision Records (ADR)**, que registram formalmente as principais decisões arquiteturais tomadas durante o desenvolvimento.

---

## `models/`

Armazena modelos treinados e seus respectivos metadados.

A separação entre checkpoints, modelos exportados e pesos pré-treinados facilita versionamento e reprodução dos experimentos.

---

## `notebooks/`

Reúne notebooks utilizados exclusivamente para exploração, experimentação e validação.

Nenhum notebook é considerado código de produção.

Toda lógica consolidada deve ser migrada para o pacote Python localizado em `src/`.

Essa prática reduz duplicação de código e facilita testes automatizados.

---

## `reports/`

Concentra todos os artefatos gerados durante análises e experimentos.

Inclui:

- gráficos;
- tabelas;
- métricas;
- logs;
- relatórios técnicos.

Esses materiais servem como evidência dos resultados obtidos e apoiam a escrita de artigos, apresentações e documentação científica. 

---


---

# Organização do Código-Fonte

Todo o código de produção está concentrado no diretório `src/`.

Essa organização segue a convenção moderna utilizada por projetos Python profissionais, evitando problemas de importação, facilitando empacotamento e incentivando uma separação clara entre código de produção, experimentos e documentação.

```text
src/
└── biohub_cell_tracking/
    ├── __init__.py
    │
    ├── cli/
    ├── config/
    ├── data/
    ├── preprocessing/
    ├── detection/
    ├── tracking/
    ├── lineage/
    ├── metrics/
    ├── visualization/
    ├── pipelines/
    ├── utils/
    └── models/
```

Cada módulo possui responsabilidade única, reduzindo acoplamento e facilitando evolução incremental do projeto.

---

# Descrição dos Módulos

## `cli/`

Implementa a interface de linha de comando do projeto.

Permite executar operações como:

- treinamento;
- inferência;
- benchmark;
- validação;
- geração da submissão;
- exportação de resultados.

O objetivo é evitar que usuários precisem executar scripts Python manualmente.

Exemplo:

```bash
biohub-track infer
```

---

## `config/`

Centraliza o carregamento das configurações YAML.

Responsabilidades:

- leitura dos arquivos em `configs/`;
- validação;
- valores padrão;
- resolução de caminhos.

Toda configuração do pipeline passa por esse módulo.

---

## `data/`

Responsável pelo acesso aos datasets.

Inclui funcionalidades para:

- leitura dos arquivos `.zarr`;
- organização dos volumes;
- carregamento eficiente;
- criação de datasets para treinamento;
- preparação para inferência.

Nenhuma lógica de Machine Learning deve existir aqui.

---

## `preprocessing/`

Implementa todas as transformações realizadas antes da inferência.

Exemplos:

- normalização;
- padronização;
- remoção de ruído;
- interpolações;
- conversão entre formatos;
- ajustes de resolução;
- preparação dos tensores.

Esse módulo torna o pipeline reproduzível.

---

## `detection/`

Responsável exclusivamente pela detecção das células.

Nesta fase do projeto serão implementadas diferentes estratégias.

Exemplos:

- Ultrack (baseline);
- detectores clássicos;
- U-Net 3D;
- futuras arquiteturas de segmentação.

A saída desse módulo consiste em centroides e atributos associados às células detectadas.

---

## `tracking/`

Recebe as detecções produzidas anteriormente e resolve o problema de associação temporal.

Entre suas responsabilidades:

- cálculo de custos;
- distância física;
- assignment bipartido;
- reconstrução inicial do grafo;
- resolução de ambiguidades.

Este módulo é completamente independente da etapa de detecção.

---

## `lineage/`

Transforma o grafo temporal em uma árvore de linhagem celular.

Responsabilidades:

- identificação de mitoses;
- validação estrutural;
- construção das relações pai-filha;
- preparação para submissão.

Ao final desta etapa, o pipeline possui um grafo consistente.

---

## `metrics/`

Implementa métricas utilizadas durante experimentação.

Exemplos:

- Edge Jaccard;
- Division Jaccard;
- precisão;
- recall;
- F1-score;
- métricas internas de benchmarking.

Sempre que possível, as implementações buscarão reproduzir fielmente a metodologia oficial da competição.

---

## `visualization/`

Centraliza todas as rotinas de visualização.

Inclui:

- projeções volumétricas;
- sobreposição de segmentações;
- grafos de linhagem;
- mapas de calor;
- gráficos de desempenho.

Esse módulo é utilizado apenas para análise e interpretação dos resultados.

---

## `pipelines/`

Representa o coração do projeto.

Cada pipeline orquestra diversos módulos independentes.

Exemplos:

- pipeline de treinamento;
- pipeline de inferência;
- pipeline de benchmark;
- pipeline de validação;
- pipeline de geração do `submission.csv`.

Essa separação permite reutilização e reduz duplicação de código.

---

## `utils/`

Contém componentes compartilhados por todo o projeto.

Exemplos:

- logging;
- temporizadores;
- manipulação de caminhos;
- serialização;
- gerenciamento de seeds;
- funções auxiliares.

Nenhuma regra de negócio específica deve ficar nesse módulo.

---

## `models/`

Define abstrações para os modelos utilizados no projeto.

Sua função é padronizar a interface entre diferentes arquiteturas.

Isso permitirá comparar facilmente diferentes abordagens como:

- Ultrack;
- U-Net 3D;
- Graph Neural Networks;
- Transformers;
- futuras arquiteturas experimentais.

---

# Estratégia de Evolução

A arquitetura foi planejada para permitir evolução incremental.

Cada componente pode ser substituído sem alterar o restante do pipeline.

Por exemplo:

```text
Ultrack
      │
      ▼
U-Net 3D
      │
      ▼
Foundation Models
```

O mesmo princípio vale para o módulo de tracking:

```text
Hungarian Assignment
         │
         ▼
GraphSAGE
         │
         ▼
Graph Transformer
```

Essa estratégia reduz risco técnico e facilita experimentação científica.

---

# Estratégia de Testes

Todo componente implementado deverá possuir testes automatizados.

Os testes estão organizados em diferentes níveis.

## Testes Unitários

Validam funções isoladas.

Objetivo:

- detectar regressões rapidamente;
- facilitar refatorações.

---

## Testes de Integração

Validam comunicação entre módulos.

Exemplos:

- detector → tracker;
- tracker → reconstrução;
- pipeline → geração da submissão.

---

## Testes de Regressão

Garantem que melhorias futuras não alterem resultados já validados.

São particularmente importantes para projetos científicos.

---

## Testes de Performance

Monitoram continuamente:

- tempo de execução;
- consumo de memória;
- throughput;
- utilização de GPU.

Esses testes garantem que o pipeline continue respeitando o SLA imposto pela competição.

---

---

# Pipeline Completo de Machine Learning

Este projeto implementa um pipeline completo de inferência para rastreamento celular em microscopia 3D+t.

Todas as etapas foram desenhadas para serem reproduzíveis, auditáveis e independentes entre si.

O objetivo é que cada componente possa evoluir sem comprometer a estabilidade do restante da arquitetura.

```text
                Dados Brutos (.zarr)
                         │
                         ▼
               Ingestão dos Dados
                         │
                         ▼
                Pré-processamento
                         │
                         ▼
              Detecção de Células
                         │
                         ▼
             Associação Temporal
                         │
                         ▼
        Reconstrução de Linhagens
                         │
                         ▼
          Avaliação das Métricas
                         │
                         ▼
            Geração da Submissão
```

---

# Etapa 1 — Ingestão dos Dados

O pipeline inicia com a leitura dos volumes tridimensionais disponibilizados pela competição.

Principais responsabilidades:

- leitura dos arquivos `.zarr`;
- validação da estrutura dos diretórios;
- carregamento incremental dos volumes;
- gerenciamento eficiente de memória;
- preparação para processamento em lote.

Nesta etapa nenhuma transformação é aplicada aos dados.

---

# Etapa 2 — Pré-processamento

Antes da inferência, os volumes passam por uma etapa padronizada de preparação.

Entre as transformações previstas estão:

- normalização de intensidade;
- padronização dos dados;
- remoção de ruído;
- ajuste de resolução quando necessário;
- preparação dos tensores para inferência.

Todo processamento realizado nesta etapa deverá ser completamente determinístico.

Isso garante reprodutibilidade entre diferentes execuções.

---

# Etapa 3 — Detecção das Células

A etapa de detecção identifica as células presentes em cada volume tridimensional.

A arquitetura inicial utilizará o **Ultrack** como baseline técnico.

Ao longo da evolução do projeto poderão ser avaliadas outras abordagens, como:

- detectores clássicos;
- U-Net 3D;
- modelos híbridos;
- arquiteturas baseadas em Foundation Models.

A saída desta etapa consiste em:

- centroides;
- posição espacial;
- tempo (`t`);
- atributos auxiliares da célula.

---

# Etapa 4 — Associação Temporal (Tracking)

Uma vez detectadas as células em cada instante de tempo, inicia-se o problema de associação temporal.

Nesta etapa o sistema responde perguntas como:

- Qual célula no frame seguinte corresponde à célula atual?
- A célula desapareceu?
- Houve divisão celular?
- Existe ambiguidade na associação?

Inicialmente será utilizada associação baseada em distância física e assignment bipartido.

No futuro poderão ser avaliadas abordagens baseadas em grafos.

---

# Etapa 5 — Reconstrução da Linhagem

O objetivo desta etapa é transformar as associações temporais em uma árvore consistente de linhagem celular.

Responsabilidades:

- identificar eventos de divisão;
- construir relações pai-filha;
- validar continuidade temporal;
- remover inconsistências estruturais;
- gerar o grafo final.

Essa estrutura representa o principal resultado científico produzido pelo pipeline.

---

# Etapa 6 — Avaliação

O pipeline será continuamente avaliado utilizando métricas oficiais e métricas auxiliares.

Entre elas:

- Edge Jaccard;
- Division Jaccard;
- precisão;
- recall;
- F1-score;
- tempo de execução;
- consumo de memória;
- throughput.

Além das métricas científicas, o projeto também monitora indicadores de engenharia.

---

# Etapa 7 — Geração da Submissão

Após a reconstrução da linhagem, o pipeline produz automaticamente o arquivo exigido pela competição.

Formato:

```text
submission.csv
```

O arquivo contém:

- nós (cells);
- arestas (links);
- identificadores;
- coordenadas;
- relações temporais.

Toda validação estrutural deverá ocorrer antes da geração do arquivo final.

---

# Fluxo de Execução

O pipeline completo poderá ser representado da seguinte maneira:

```text
Volume 3D
     │
     ▼
Data Loader
     │
     ▼
Pré-processamento
     │
     ▼
Detector
     │
     ▼
Centroides
     │
     ▼
Tracker
     │
     ▼
Lineage Builder
     │
     ▼
Validation
     │
     ▼
submission.csv
```

Cada etapa registra informações detalhadas em log para facilitar auditoria e depuração.

---

# Estratégia de Evolução do Pipeline

O projeto seguirá uma abordagem incremental.

A cada evolução, apenas um componente será alterado por vez.

Exemplo:

```text
Versão 1

Ultrack
+
Assignment Bipartido

↓

Versão 2

U-Net 3D
+
Assignment Bipartido

↓

Versão 3

U-Net 3D
+
Graph Neural Network

↓

Versão 4

U-Net 3D
+
Graph Neural Network
+
Spatio-Temporal Transformer
```

Essa estratégia permite medir objetivamente o impacto de cada alteração sobre:

- qualidade da reconstrução;
- custo computacional;
- consumo de memória;
- tempo de execução;
- reprodutibilidade.

---

# Filosofia de Engenharia

Este projeto adota uma premissa simples:

> **Toda melhoria precisa ser mensurável.**

Nenhuma alteração arquitetural será incorporada apenas por representar uma tecnologia mais recente.

Cada evolução deverá demonstrar benefícios concretos em pelo menos um dos seguintes aspectos:

- qualidade científica;
- desempenho computacional;
- simplicidade de manutenção;
- reprodutibilidade;
- escalabilidade;
- robustez operacional.

Essa abordagem reduz complexidade desnecessária e mantém o projeto alinhado às boas práticas de Engenharia de Software, Machine Learning e MLOps.

--- 


---

# Metodologia Experimental

Este projeto adota uma abordagem científica para desenvolvimento de modelos de Machine Learning.

Nenhuma alteração arquitetural é considerada uma melhoria até que seja validada experimentalmente.

Cada experimento deverá ser:

- reproduzível;
- documentado;
- comparável;
- versionado;
- auditável.

Essa metodologia garante que todas as decisões técnicas sejam sustentadas por evidências quantitativas.

---

# Ciclo Experimental

Todos os experimentos seguirão o mesmo fluxo de trabalho.

```text
Hipótese
    │
    ▼
Planejamento
    │
    ▼
Implementação
    │
    ▼
Execução
    │
    ▼
Avaliação
    │
    ▼
Documentação
    │
    ▼
Decisão
```

Nenhuma hipótese será considerada válida antes da documentação completa dos resultados.

---

# Hipóteses de Pesquisa

Durante o desenvolvimento serão avaliadas diversas hipóteses.

Exemplos:

### Hipótese 1

Uma etapa adicional de remoção de ruído melhora a qualidade da detecção.

---

### Hipótese 2

A utilização de distância física anisotrópica reduz erros de associação entre frames.

---

### Hipótese 3

Graph Neural Networks aumentam a precisão da reconstrução de linhagens.

---

### Hipótese 4

Transformers espaço-temporais reduzem ambiguidades em regiões de alta densidade celular.

---

### Hipótese 5

Modelos mais leves apresentam melhor relação entre desempenho e custo computacional sob o SLA da competição.

---

Cada hipótese será validada individualmente.

---

# Registro dos Experimentos

Todos os experimentos possuirão identificação única.

Exemplo:

```text
EXP-001
EXP-002
EXP-003
```

Cada registro deverá conter:

- objetivo;
- hipótese;
- configuração utilizada;
- dataset;
- hardware;
- parâmetros;
- métricas;
- conclusão.

Essa padronização facilita comparações futuras.

---

# Benchmarking

Cada nova arquitetura será comparada com um baseline previamente definido.

O objetivo não é apenas obter maior pontuação, mas compreender os custos associados a cada solução.

Os benchmarks considerarão:

- Edge Jaccard;
- Division Jaccard;
- tempo de inferência;
- consumo de memória;
- utilização de GPU;
- throughput;
- estabilidade.

---

# Baselines

O projeto utiliza diferentes categorias de baseline.

## Baseline Científico

Métodos publicados na literatura.

Exemplo:

- Ultrack.

---

## Baseline Operacional

Pipeline simples executando dentro das restrições do Kaggle.

---

## Baseline Humano

Rastreamento manual realizado por especialistas.

Embora não seja reproduzido neste projeto, serve como referência conceitual para avaliar ganhos operacionais.

---

# Controle de Variáveis

Para garantir comparabilidade entre experimentos, sempre que possível serão mantidos constantes:

- dataset;
- hardware;
- versões das bibliotecas;
- sementes aleatórias;
- parâmetros não relacionados ao experimento.

Dessa forma, qualquer diferença observada poderá ser atribuída principalmente à alteração investigada.

---

# Reprodutibilidade

Um experimento somente será considerado válido quando puder ser reproduzido.

Para isso serão registrados:

- commit do Git;
- versão do Python;
- versão das dependências;
- configuração utilizada;
- hardware;
- sistema operacional;
- tempo de execução.

Essa documentação permitirá repetir experimentos meses após sua realização.

---

# Monitoramento de Performance

Além das métricas científicas, serão acompanhados indicadores de engenharia.

Entre eles:

- tempo total de execução;
- tempo por etapa;
- utilização de CPU;
- utilização de GPU;
- consumo máximo de memória;
- tamanho do modelo;
- tempo de inicialização.

Esses indicadores são fundamentais para validar o atendimento ao SLA da competição.

---

# Critérios para Aprovação de Melhorias

Uma alteração somente será incorporada ao pipeline principal quando atender aos seguintes critérios:

- manter ou melhorar a qualidade científica;
- preservar a reprodutibilidade;
- não comprometer a estabilidade;
- respeitar o limite de 12 horas da competição;
- possuir documentação completa.

Caso contrário, permanecerá registrada apenas como experimento.

---

# Registro das Decisões Arquiteturais

Toda decisão relevante será registrada em um **Architecture Decision Record (ADR)**.

Cada ADR responderá às seguintes perguntas:

1. Qual problema motivou a decisão?
2. Quais alternativas foram avaliadas?
3. Qual solução foi escolhida?
4. Quais trade-offs foram aceitos?
5. Quais consequências são esperadas?

Essa prática facilita a evolução do projeto e torna explícito o raciocínio por trás de cada escolha.

---

# Filosofia do Projeto

Este repositório não busca apenas alcançar uma boa colocação na competição.

Seu propósito é demonstrar como um problema científico pode ser tratado com práticas modernas de Engenharia de Software, Machine Learning e MLOps.

O foco está na construção de um pipeline:

- modular;
- reproduzível;
- auditável;
- escalável;
- bem documentado;
- orientado por evidências.

Mais do que um conjunto de notebooks, este projeto pretende servir como um estudo de caso completo sobre o desenvolvimento de sistemas de inferência para visão computacional em ambientes com restrições reais de operação.

--- 



---

# Lições de Engenharia

Uma competição de Machine Learning não é apenas uma disputa por métricas.

Ela também representa um ambiente controlado para avaliar decisões de arquitetura, engenharia de software, gerenciamento de recursos computacionais e reprodutibilidade.

Ao longo deste projeto, todas as decisões relevantes serão registradas, incluindo aquelas que não produziram os resultados esperados.

O objetivo é transformar este repositório em um estudo de caso completo sobre desenvolvimento de pipelines de inferência para Visão Computacional.

---

# Diário de Engenharia

Durante o desenvolvimento serão registrados os principais marcos técnicos do projeto.

Cada entrada deverá responder perguntas como:

- Qual problema surgiu?
- Como o problema foi investigado?
- Quais hipóteses foram levantadas?
- Quais alternativas foram avaliadas?
- Qual solução foi escolhida?
- Quais resultados foram obtidos?
- O que foi aprendido?

Essa documentação complementa os experimentos quantitativos com o contexto necessário para compreender a evolução do projeto.

---

# Lições Aprendidas

Ao longo do desenvolvimento serão documentados aprendizados relacionados a diferentes áreas.

## Engenharia de Software

Exemplos:

- arquitetura modular;
- desacoplamento entre componentes;
- organização de projetos científicos;
- automação de tarefas;
- integração contínua.

---

## Machine Learning

Exemplos:

- seleção de modelos;
- calibração de hiperparâmetros;
- impacto do pré-processamento;
- estratégias de validação;
- análise de erros.

---

## Visão Computacional

Exemplos:

- segmentação volumétrica;
- representação tridimensional;
- rastreamento de objetos;
- reconstrução de linhagens;
- tratamento de ruído.

---

## Engenharia de Dados

Exemplos:

- leitura eficiente de arquivos `.zarr`;
- processamento incremental;
- gerenciamento de memória;
- organização de datasets;
- preparação para inferência.

---

## MLOps

Exemplos:

- reprodutibilidade;
- empacotamento;
- versionamento;
- monitoramento;
- automação do pipeline.

---

# Erros que Não Voltaremos a Cometer

Nem toda decisão produz o resultado esperado.

Quando uma abordagem falhar, ela será documentada.

Cada registro deverá conter:

- contexto;
- hipótese original;
- motivo da falha;
- impacto observado;
- solução adotada.

Registrar erros evita repetir experimentos improdutivos e acelera a evolução do projeto.

---

# Decisões Revertidas

Algumas decisões poderão ser revistas ao longo do desenvolvimento.

Sempre que isso ocorrer, o histórico será preservado.

Essa prática permite compreender a evolução arquitetural do projeto e os fatores que motivaram cada mudança.

---

# Evolução da Arquitetura

O pipeline deverá evoluir em ciclos sucessivos.

```text
Baseline
      │
      ▼
Pipeline Modular
      │
      ▼
Otimizações de Performance
      │
      ▼
Novos Modelos
      │
      ▼
Pipeline Híbrido
      │
      ▼
Versão Final
```

Cada evolução será acompanhada por benchmarks e documentação técnica.

---

# Roadmap

O desenvolvimento está organizado em fases incrementais.

## Fase 1 — Foundation

- Estrutura profissional do repositório.
- Automação.
- Qualidade de código.
- Documentação.

---

## Fase 2 — Data Pipeline

- Leitura dos dados.
- Pré-processamento.
- Organização dos datasets.

---

## Fase 3 — Baseline

- Implementação do pipeline utilizando o Ultrack.
- Primeira submissão válida no Kaggle.

---

## Fase 4 — Benchmark

- Avaliação detalhada de desempenho.
- Profiling.
- Otimizações.

---

## Fase 5 — Pesquisa

Exploração de arquiteturas como:

- U-Net 3D;
- Graph Neural Networks;
- Spatio-Temporal Transformers.

---

## Fase 6 — Produção

- Pipeline totalmente reproduzível.
- Documentação final.
- Artigo técnico.
- Publicação dos resultados.

---

# Contribuições

Contribuições são bem-vindas.

Antes de abrir uma Pull Request, recomenda-se:

- ler o guia de contribuição;
- executar todos os testes;
- verificar o padrão de formatação;
- atualizar a documentação quando necessário.

Toda contribuição será revisada buscando preservar:

- qualidade;
- consistência;
- reprodutibilidade;
- clareza.

---

# Referências

Este projeto é fundamentado em artigos científicos, documentação técnica e bibliotecas amplamente utilizadas pela comunidade.

As referências completas encontram-se em:

```text
docs/references.md
```

Sempre que possível, novas decisões arquiteturais serão acompanhadas por referências técnicas que fundamentem sua adoção.

---

# Agradecimentos

Agradecemos à comunidade científica e de software livre que torna projetos como este possíveis.

Em especial:

- aos organizadores da competição Biohub;
- aos pesquisadores que disponibilizaram os dados;
- aos mantenedores das bibliotecas utilizadas;
- à comunidade Python;
- à comunidade de Visão Computacional e Machine Learning.

---

# Licença

Este projeto é distribuído sob a licença MIT.

Consulte o arquivo `LICENSE` para mais informações.

---

# Como Citar

Caso este repositório contribua para pesquisas, estudos ou projetos derivados, utilize as informações disponíveis em:

```text
CITATION.cff
```

---

# Considerações Finais

Este projeto nasceu com um objetivo que vai além da competição.

Mais do que desenvolver um algoritmo para rastreamento celular, buscamos construir um exemplo completo de como problemas científicos podem ser tratados com práticas modernas de Engenharia de Software, Ciência de Dados, Machine Learning e MLOps.

Cada experimento, cada decisão arquitetural e cada melhoria implementada será documentada de forma transparente, permitindo que outros profissionais reproduzam os resultados, compreendam os trade-offs envolvidos e utilizem este repositório como referência para projetos futuros.

Independentemente da colocação obtida na competição, o maior resultado esperado é a construção de um pipeline robusto, reproduzível e tecnicamente consistente, capaz de demonstrar a importância da engenharia na transformação de pesquisas científicas em soluções computacionais confiáveis.

> **"O objetivo não é apenas construir modelos melhores, mas construir sistemas melhores."**

--- 


## Qualidade, Reprodutibilidade e Engenharia

Uma solução de Machine Learning não deve ser avaliada apenas pela métrica obtida no leaderboard. Em ambientes de produção, requisitos como reprodutibilidade, testabilidade, observabilidade e facilidade de manutenção costumam ser tão importantes quanto a acurácia do modelo.

Este projeto foi estruturado para que qualquer execução possa ser reproduzida de forma determinística, desde a preparação do ambiente até a geração do arquivo submission.csv.

## Reprodutibilidade

Todo o ambiente é definido por código.

Entre as práticas adotadas estão:

gerenciamento de dependências por versão;

ambiente isolado via Docker;

configuração centralizada através do pyproject.toml;

execução padronizada via Makefile;

integração contínua (GitHub Actions);

versionamento completo do código-fonte.


## Isso reduz o risco clássico de:

> "Funciona na minha máquina."



Cada experimento deve poder ser reproduzido por outro pesquisador utilizando exatamente a mesma versão do projeto.


---

## Qualidade de Código

O projeto adota ferramentas amplamente utilizadas em projetos Open Source profissionais.

## Ferramenta	Objetivo

Ruff	Linter extremamente rápido
Black	Padronização automática de código
MyPy	Verificação estática de tipos
Pytest	Testes automatizados
pre-commit	Validação antes de cada commit
GitHub Actions	Execução automática da pipeline de CI


Essa combinação reduz defeitos antes mesmo da execução dos experimentos.


---

## Arquitetura Modular

Cada componente do pipeline possui responsabilidade única.

Detecção
     │
     ▼
Tracking
     │
     ▼
Reconstrução
     │
     ▼
Submission

Essa separação permite:

substituir apenas o detector;

testar novos algoritmos de linking;

comparar diferentes estratégias de reconstrução;

reutilizar componentes em outros projetos.



---

## Observabilidade

Todos os experimentos devem produzir informações suficientes para posterior auditoria.

São registrados, entre outros:

versão do código;

parâmetros utilizados;

tempo de execução;

utilização de memória;

versão das bibliotecas;

hardware utilizado;

seed aleatória;

métricas obtidas.


Esse histórico torna possível reproduzir exatamente qualquer resultado apresentado no repositório.


---

## Filosofia de Engenharia

Este projeto segue um princípio simples:

> Todo resultado deve ser reproduzível, toda decisão deve ser justificável e todo componente deve poder ser substituído sem reescrever o restante do sistema.



Essa filosofia reduz acoplamento, facilita manutenção e torna o pipeline adequado tanto para pesquisa quanto para ambientes produtivos.


---

## Compatibilidade com o Kaggle

Toda a arquitetura foi desenhada respeitando as restrições da competição.

execução totalmente offline;

ausência de chamadas externas;

geração automática do submission.csv;

processamento dentro do limite de tempo permitido;

dependências empacotadas previamente;

comportamento determinístico.


Essas restrições foram consideradas desde o início do projeto, evitando adaptações de última hora para submissão.


---



