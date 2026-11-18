
# BusLight — Applied Computer Vision

**FIAP · Engenharia de Software · Global Solution 2026.2 · CitizenTech**

Projeto de visão computacional para monitoramento inteligente de pontos de ônibus, utilizando inteligência artificial para detectar pessoas, ônibus e recursos de acessibilidade, acompanhar objetos em vídeos e gerar indicadores sobre a ocupação do ponto.

## Sobre o projeto

O **BusLight** utiliza técnicas de visão computacional para analisar imagens e vídeos de pontos de ônibus, identificando passageiros, veículos e recursos de mobilidade.

A solução combina dois modelos YOLO11s com funções complementares:

- **Detecção de pessoas e ônibus:** utiliza o YOLO11s pré-treinado no conjunto de dados COCO, que possui exemplos variados de pessoas em diferentes posições e movimentos.
- **Detecção de recursos de acessibilidade:** utiliza um YOLO11s personalizado, treinado com imagens do dataset BusLight desenvolvido no Roboflow, para identificar cadeiras de rodas e andadores.

Além da detecção, o projeto utiliza o algoritmo ByteTrack para rastrear pessoas ao longo dos vídeos, estimar a lotação do ponto e gerar alertas relacionados à acessibilidade.

## Objetivos

- Detectar pessoas e ônibus em imagens e vídeos.
- Identificar cadeiras de rodas e andadores.
- Realizar o rastreamento de pessoas ao longo dos frames.
- Contabilizar a quantidade de pessoas presentes no ponto.
- Estimar o nível de lotação.
- Gerar alertas quando forem identificados recursos de acessibilidade.
- Produzir vídeos anotados, tabelas e indicadores para análise dos resultados.
- Proteger a privacidade das pessoas por meio do borramento da região da cabeça nas imagens processadas.

## Arquitetura dos modelos

O sistema utiliza dois modelos independentes que trabalham em conjunto.

### 1. Modelo de pessoas e ônibus

Foi utilizado o **YOLO11s pré-treinado no COCO**, sem treinamento personalizado para essas duas classes.

Classes utilizadas:

| ID COCO | Classe | Descrição |
|---|---|---|
| 0 | Person | Pessoas |
| 5 | Bus | Ônibus |

Esse modelo realiza a detecção de pessoas e ônibus. Durante o processamento de vídeos, suas detecções também são utilizadas pelo ByteTrack para acompanhar os objetos ao longo dos frames.

### 2. Modelo de acessibilidade

Foi utilizado o **YOLO11s com transferência de aprendizado (Transfer Learning)**, treinado especificamente para reconhecer recursos de mobilidade.

O treinamento utiliza um dataset personalizado desenvolvido no Roboflow, com imagens anotadas e organizadas em conjuntos de treinamento, validação e teste.

As classes originais foram reorganizadas em duas categorias:

| ID | Classe | Descrição |
|---|---|---|
| 0 | andador | Andadores |
| 1 | cadeira de rodas | Cadeiras de rodas |

As classes originais `wheelchair` e `wheelchairs` foram unificadas em uma única categoria, enquanto `walking-frame` foi mapeada para `andador`.

## Dataset

O dataset foi desenvolvido no Roboflow, utilizando conjuntos públicos do Roboflow Universe relacionados a ônibus, pessoas e recursos de mobilidade.

**Projeto Roboflow:** `buslight-ca7ef`

**Workspace:** `gs-ykiiq`

**Versão utilizada:** 3

O conjunto de dados foi exportado no formato YOLOv8, compatível com o treinamento realizado utilizando Ultralytics YOLO11.

Antes do treinamento, foi realizado um diagnóstico da distribuição das classes, considerando a quantidade de imagens e caixas anotadas em cada divisão.

Também foi criada uma versão específica do dataset para acessibilidade, contendo somente as anotações de cadeiras de rodas e andadores.

## Treinamento do modelo de acessibilidade

O treinamento foi realizado no Google Colab, utilizando GPU Tesla T4.

| Parâmetro | Configuração |
|---|---|
| Arquitetura | YOLO11s |
| Técnica | Transfer Learning |
| Épocas | 40 |
| Early Stopping | Patience 10 |
| Tamanho da imagem | 640 × 640 |
| Batch | Automático (-1) |
| Cache | Disco |
| GPU | Tesla T4 |

O modelo treinado é salvo no Google Drive como `best_acessibilidade.pt`.

O notebook também verifica a existência de um modelo salvo para evitar treinamentos desnecessários. Caso exista um checkpoint de treinamento interrompido, o processo pode ser retomado.

## Avaliação dos modelos

O projeto realiza a avaliação do modelo personalizado no conjunto de teste, utilizando métricas de desempenho para analisar a qualidade das detecções.

As métricas utilizadas são:

- **Precision:** proporção de detecções realizadas corretamente em relação ao total de detecções.
- **Recall:** proporção dos objetos existentes que foram identificados pelo modelo.
- **mAP@50:** média da precisão considerando uma sobreposição mínima de 50% entre a caixa prevista e a caixa real.
- **mAP@50-95:** métrica mais rigorosa, que considera diferentes níveis de sobreposição entre 50% e 95%.

Também são gerados gráficos de treinamento e matriz de confusão para auxiliar na análise dos resultados.

### Comparação com a versão anterior

O notebook realiza uma comparação entre a primeira versão do projeto e a arquitetura atual.

| Versão | Arquitetura |
|---|---|
| Versão 1 | YOLO11n treinado nas cinco classes originais |
| Versão 2 | YOLO11s COCO para pessoas e ônibus + YOLO11s personalizado para acessibilidade |

A comparação utiliza as mesmas imagens de teste e considera as métricas de Recall, mAP50 e mAP50-95 por classe.

É importante considerar que a qualidade das anotações de pessoas no conjunto de teste pode influenciar os resultados da comparação, pois imagens com pessoas não anotadas podem ser contabilizadas como erros de detecção.

## Rastreamento de objetos — ByteTrack

Para acompanhar as pessoas ao longo dos vídeos, foi utilizado o algoritmo **ByteTrack**, integrado ao YOLO11.

O rastreamento permite atribuir IDs aos objetos e acompanhar suas detecções entre os frames.

Foram utilizados ajustes nos parâmetros do rastreador:

| Parâmetro | Valor |
|---|---:|
| `track_buffer` | 60 |
| `track_low_thresh` | 0.1 |
| `conf` | 0.30 |

O `track_buffer` foi ampliado para permitir que o rastreador mantenha uma identificação por mais tempo quando uma pessoa desaparece temporariamente, por exemplo, devido a desfoque ou oclusão.

## Estimativa de lotação

O sistema utiliza a quantidade de pessoas detectadas para classificar o nível de ocupação do ponto.

Os limites definidos no notebook são:

| Quantidade de pessoas | Classificação |
|---|---|
| Até 3 pessoas | Baixa |
| De 4 a 8 pessoas | Média |
| Acima de 8 pessoas | Alta |

A classificação é realizada automaticamente durante o processamento das imagens e vídeos.

Esses limites são parâmetros configuráveis do projeto e representam uma estimativa operacional, não uma medição oficial da capacidade física do ponto.

## Alerta de acessibilidade

O modelo personalizado identifica a presença de cadeiras de rodas e andadores.

Quando pelo menos um desses recursos é detectado, o sistema apresenta um alerta visual:

**ACESSIBILIDADE: passageiro precisa de apoio no embarque**

O objetivo é demonstrar como a visão computacional pode auxiliar na identificação de situações que exigem atenção especial durante o embarque.

A detecção de um recurso de mobilidade não permite identificar automaticamente uma deficiência nem confirmar a associação do objeto a uma pessoa específica.

## Privacidade e LGPD

A privacidade é considerada durante o processamento das imagens.

O sistema utiliza uma técnica de borramento com Gaussian Blur na região superior da caixa delimitadora de cada pessoa, buscando ocultar a região da cabeça.

Medidas consideradas no projeto:

- Não utiliza reconhecimento facial.
- Não busca identificar individualmente os passageiros.
- Aplica borramento nas regiões das pessoas antes de gerar as imagens e vídeos anotados.
- Armazena os arquivos de saída necessários para análise e demonstração do projeto.

O borramento é uma medida de proteção visual e não representa, isoladamente, uma garantia de anonimização completa.

## Processamento de vídeos

O notebook permite carregar um vídeo para executar a detecção e o rastreamento de objetos frame a frame.

Durante o processamento, são realizadas as seguintes operações:

1. Leitura do vídeo utilizando OpenCV.
2. Detecção de pessoas e ônibus com o YOLO11s pré-treinado no COCO.
3. Rastreamento de pessoas utilizando ByteTrack.
4. Detecção de cadeiras de rodas e andadores com o modelo personalizado.
5. Contagem de objetos por frame.
6. Classificação do nível de lotação.
7. Geração de alertas de acessibilidade.
8. Borramento da região da cabeça das pessoas.
9. Gravação do vídeo com as detecções e informações.

O parâmetro `PULAR` permite controlar a quantidade de frames analisados. O valor padrão é `1`, ou seja, todos os frames são processados.

## Arquivos gerados

O projeto gera diferentes arquivos para documentar e analisar os resultados.

| Arquivo | Descrição |
|---|---|
| `diagnostico_dataset.csv` | Distribuição das imagens e caixas por classe e divisão do dataset |
| `metricas_teste.csv` | Métricas de avaliação do modelo de acessibilidade |
| `matriz_confusao.png` | Matriz de confusão normalizada |
| `curvas_treino.png` | Curvas de desempenho do treinamento |
| `comparacao_v1_v2.csv` | Comparação entre as versões do modelo |
| `exemplos_teste.jpg` | Exemplos de detecção em imagens do conjunto de teste |
| `antes_depois.jpg` | Comparação visual entre a versão anterior e a atual |
| `*_detectado.mp4` | Vídeo processado com detecções, rastreamento e indicadores |
| `*_contagem.csv` | Contagem de objetos por frame |
| `*_resumo.json` | Resumo dos indicadores obtidos no vídeo |
| `*_grafico.png` | Gráfico de pessoas detectadas ao longo do vídeo |
| `*_frame_*_v1_v2.jpg` | Frames de comparação entre as versões |

Os resultados são organizados na pasta `/content/BusLight` e compactados em um arquivo ZIP para download.

## Como executar

O projeto foi desenvolvido para execução no **Google Colab**.

### Requisitos

- Conta Google.
- Google Colab.
- Ambiente de execução com GPU Tesla T4 ou compatível.
- Conta no Roboflow.
- Chave de API do Roboflow.
- Google Drive para armazenamento dos pesos e resultados.

### Configuração

1. Abra o notebook `BusLight_v3.ipynb` no Google Colab.
2. Acesse **Ambiente de execução → Alterar tipo de ambiente de execução**.
3. Selecione uma GPU, preferencialmente Tesla T4.
4. Configure a variável `ROBOFLOW_API_KEY` em **Secrets** do Colab.
5. Habilite o acesso ao segredo pelo notebook.
6. Execute as células em ordem ou utilize **Executar tudo**.

Na primeira execução, o notebook realiza o download do dataset, prepara as imagens e treina o modelo de acessibilidade.

O treinamento pode levar aproximadamente duas horas, dependendo do ambiente e do dataset.

Nas próximas execuções, caso o modelo já esteja salvo no Google Drive, o treinamento será ignorado.

### Modelo personalizado

O modelo treinado é armazenado no seguinte caminho:

```text
/content/drive/MyDrive/BusLight/best_acessibilidade.pt
```

O notebook também utiliza o modelo pré-treinado `yolo11s.pt` para detectar pessoas e ônibus.

## Limitações conhecidas

Apesar dos resultados obtidos, existem limitações que devem ser consideradas:

- A qualidade das detecções depende das condições de iluminação, resolução, distância e perspectiva da câmera.
- Pessoas em movimento podem apresentar desfoque e oclusões, dificultando a identificação.
- O rastreamento pode apresentar perda ou fragmentação de IDs.
- A quantidade de pessoas detectadas em um frame não representa necessariamente a quantidade de passageiros únicos.
- A identificação de cadeiras de rodas e andadores depende da qualidade e variedade das imagens utilizadas no treinamento.
- A estimativa de lotação depende da contagem de pessoas e dos limites configurados.
- O alerta de acessibilidade é baseado na detecção dos objetos e não confirma, por si só, a necessidade individual de assistência.
- O borramento da região da cabeça não garante anonimização completa das imagens.

Os indicadores gerados devem ser interpretados como estimativas computacionais e não como medições oficiais ou decisões automatizadas definitivas.

## Trabalhos futuros

Como possibilidades de evolução do projeto, destacam-se:

- Aprimoramento do dataset com mais imagens de pessoas em movimento.
- Melhoria da detecção de recursos de acessibilidade em diferentes ângulos e condições.
- Avaliação da estabilidade dos IDs durante o rastreamento.
- Desenvolvimento de um dashboard interativo para visualização dos indicadores.
- Ampliação dos critérios de estimativa de ocupação.
- Evolução dos indicadores de acessibilidade e segurança.
- Testes em ambientes reais de pontos de ônibus.

## Equipe

Projeto desenvolvido por:

- Aline Fernandes Zeppelini - RM97966
- Camilly Breitbach Ishida - RM551474
- Julia Leite Galvão - RM550201

