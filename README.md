
# BusLight — Monitoramento Inteligente de Pontos de Ônibus

## Integrantes

- Aline Fernandes Zeppelini - RM97966
- Camilly Breitbach Ishida - RM551474
- Julia Leite Galvão - RM550201

## Sobre o projeto

O **BusLight** é um projeto de visão computacional desenvolvido com o objetivo de explorar o monitoramento inteligente de pontos de ônibus por meio de inteligência artificial.

A proposta utiliza técnicas de detecção de objetos e processamento de imagens para identificar ônibus, pessoas e recursos de mobilidade, contribuindo para a análise de ocupação, acessibilidade e condições de utilização dos pontos de ônibus.

O projeto utiliza o modelo **YOLO11**, treinado para reconhecer diferentes objetos presentes no ambiente, e o algoritmo **ByteTrack** para realizar o rastreamento dos objetos em vídeos.

## Objetivos

- Detectar ônibus e pessoas em imagens e vídeos.
- Identificar recursos de mobilidade, como cadeiras de rodas e andadores.
- Realizar o rastreamento de objetos ao longo dos frames.
- Registrar a quantidade de objetos detectados por frame.
- Explorar estimativas de ocupação dos pontos de ônibus.
- Desenvolver indicadores relacionados à acessibilidade e segurança.
- Apresentar os resultados por meio de visualizações e dashboards.

## Modelo de visão computacional

Foi utilizado o **YOLO11n**, com transferência de aprendizado (*Transfer Learning*), treinado com um conjunto de imagens personalizado.

O treinamento foi realizado no Google Colab, utilizando GPU, com imagens organizadas e anotadas por meio da plataforma Roboflow.

### Métricas de avaliação

Na versão utilizada nos testes, foram obtidos os seguintes resultados:

| Métrica | Resultado |
|---|---:|
| Precisão (Precision) | 83,51% |
| Revocação (Recall) | 82,98% |
| mAP@50 | 87,82% |
| mAP@50-95 | 67,78% |

As métricas foram obtidas durante a avaliação do modelo no conjunto de dados de teste.

## Tecnologias utilizadas

- **Python:** linguagem de programação principal.
- **Google Colab:** ambiente de desenvolvimento e execução.
- **YOLO11 (Ultralytics):** detecção de objetos.
- **ByteTrack:** rastreamento de objetos em vídeos.
- **OpenCV:** processamento e manipulação de vídeos e imagens.
- **Roboflow:** organização, anotação e gerenciamento do conjunto de dados.
- **Pandas:** organização e análise dos resultados.
- **Matplotlib:** visualização de dados e gráficos.

## Funcionamento

O projeto está organizado em etapas:

1. **Preparação dos dados:** organização e anotação das imagens utilizadas no treinamento.
2. **Treinamento:** desenvolvimento do modelo YOLO11 utilizando transferência de aprendizado.
3. **Avaliação:** análise das métricas de desempenho e identificação de possíveis limitações.
4. **Detecção:** identificação de objetos em imagens e vídeos.
5. **Rastreamento:** utilização do ByteTrack para acompanhar os objetos ao longo dos frames.
6. **Processamento:** registro das detecções, coordenadas, classes, níveis de confiança e IDs.
7. **Análise:** geração de tabelas e gráficos para auxiliar na interpretação dos resultados.

## Processamento de vídeos

O processamento de vídeos permite executar a detecção de objetos frame a frame e utilizar o ByteTrack para acompanhar as identificações ao longo do tempo.

São gerados os seguintes resultados:

- **video_tracking.mp4:** vídeo com as detecções, classes e IDs de rastreamento.
- **contagem_por_frame.csv:** tabela com a quantidade de objetos detectados em cada frame.
- **deteccoes_detalhadas.csv:** tabela com informações individuais das detecções, incluindo classe, confiança, coordenadas e ID de rastreamento.

Esses arquivos permitem realizar análises posteriores e desenvolver indicadores para o projeto.

## Funcionalidades planejadas

- Estimativa de ocupação do ponto de ônibus.
- Classificação do nível de ocupação em vazio, moderado ou cheio.
- Identificação da presença de recursos de mobilidade.
- Desenvolvimento de indicadores de acessibilidade.
- Construção de um Índice Ponto Seguro, baseado em critérios operacionais definidos pelo projeto.
- Dashboard interativo para visualização dos resultados.

## Limitações conhecidas

Durante os testes, foram identificadas algumas limitações:

- O modelo apresentou dificuldades na detecção de pessoas em movimento em determinados vídeos.
- O desfoque de movimento, as mudanças de perspectiva e as oclusões podem comprometer as detecções.
- O rastreamento pode apresentar fragmentação ou alteração dos IDs dos objetos.
- A quantidade de detecções por frame não representa a quantidade de pessoas únicas.
- A identificação de um recurso de mobilidade não permite inferir automaticamente uma deficiência ou associar o objeto a uma pessoa específica.
- As estimativas de ocupação dependem da definição de uma área de interesse e de critérios de capacidade.

Por esse motivo, os resultados devem ser interpretados considerando as limitações do modelo e a necessidade de validação em diferentes condições reais.

## Privacidade

O projeto tem como objetivo analisar objetos e características gerais do ambiente, não identificar pessoas individualmente.

A proposta não envolve reconhecimento facial nem identificação da identidade dos passageiros.

## Execução

O desenvolvimento e os testes podem ser realizados no Google Colab, utilizando uma GPU compatível.

Para executar a interface de detecção de imagens:

1. Abra o notebook de teste do modelo.
2. Instale as dependências necessárias.
3. Carregue o arquivo de pesos `best.pt`.
4. Execute a célula responsável pela interface Gradio.
5. Envie uma imagem e visualize as detecções.

Não é necessário realizar o treinamento novamente quando os pesos do modelo já estão disponíveis.

## Link para o vídeo demonstrativo


