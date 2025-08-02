# Arquitetura de Produção Escalável para um Modelo de Machine Learning

Aqui foi descrito cada etapa de uma arquitetura pensada para operacionalizar um modelo de Machine Learning em um ambiente empresarial com alta escala.
## 🗂️ Diagrama da Arquitetura
<div align="center">
  <img width="735" height="489" alt="Captura de tela 2025-08-02 120515" src="https://github.com/user-attachments/assets/6422c100-29ed-45bf-97c2-f64d121fb753" />
</div>
## 📄 Explicação da Arquitetura

#### 1. Componentes do Sistema

1. Fontes de Dados: Representam as diversas origens de onde os dados brutos (imagens e vídeos) são coletados. Podem ser câmeras de vigilância, APIs de terceiros ou uploads de usuários.

2. Serviço de Ingestão: Uma camada de API ou um microsserviço que recebe os dados brutos das fontes, valida, padroniza e inicia o fluxo de processamento. Faz o upload dos arquivos brutos para o Armazenamento para que sejam guardados de forma durável e envia os metadados desses arquivos (como o caminho e a data/hora) para a Fila de Mensagens.

3. Fila de Mensagens: Desacopla a ingestão do processamento. Recebe os metadados de novos dados em tempo real e os armazena de forma durável. O serviço de Pré-processamento atua como consumidor que puxa as mensagens da fila.

4. Pré-processamento: Um serviço dedicado que consome as mensagens da fila e é responsável por preparar os dados para o treinamento do modelo. Lê os arquivos brutos do Armazenamento, processa as imagens e vídeos (redimensiona, extrai frames, aplica filtros, etc) e armazena os dados processados. Também registra na Base de Dados os metadados do processamento como o status, o caminho final e os parâmetros utilizados. Isso é importante para a rastreabilidade do pipeline.

5. Armazenamento: O repositório central para todos os dados do sistema. Dividido em áreas para armazenar dados brutos e dados já processados, garantindo organização e eficiência.

6. Pipeline de Treinamento: Um fluxo de trabalho automatizado que orquestra o ciclo de vida de treinamento do modelo. Lê os dados processados, obtém os rótulos da Base de Dados e, após o treinamento e a avaliação, registra o modelo treinado no Registro de Modelo.

7. Registro de Modelo: Um repositório central que armazena, versiona e gerencia os modelos de machine learning. Cada modelo treinado tem uma versão e metadados associados, como métricas de performance e os hiperparâmetros utilizados, o que é fundamental para gerenciamento e reprodutibilidade.

8.  Serviço de Inferência: Um cluster de contêineres que hospeda a API de inferência em alta escala. É o componente que consome o modelo treinado e o disponibiliza para requisições externas.

9. Monitoramento: Um sistema para coletar e visualizar métricas de performance do sistema em tempo real. Um monitora as métricas de performance do modelo (acurácia, precisão), decidindo se é necessário retreinar o modelo e outro analisa as métricas de sistema vindas do Serviço de Inferência (CPU, latência). 

10. Base de Dados: Armazena metadados estruturados, como rótulos das imagens, logs de treinamento e dados de monitoramento. Atua como fonte para os rótulos de treinamento e para o feedback do modelo.

#### 2. Justificativas das Escolhas Tecnológicas

* Apache Kafka para Fila de Mensagens: Escolhido por sua alta escalabilidade e resiliência. É uma boa solução para gerenciar o fluxo de dados de múltiplas fontes em tempo real garantindo que nenhum dado seja perdido e desacoplando os serviços.
* Amazon S3 para Armazenamento: É o padrão da indústria para armazenamento de objetos e oferece durabilidade, escalabilidade e baixo custo. É importante para armazenar grandes volumes de imagens e vídeos de alta resolução, além de suportar versionamento de dados.
* Kubeflow/AWS SageMaker para Pipeline de Treinamento: Estas plataformas fornecem acesso gerenciado a GPUs, orquestram o fluxo de trabalho de treinamento e garantem a reprodutibilidade e a automação do processo.
* MLflow para Registro do Modelo: É uma ferramenta de código aberto essencial para o gerenciamento do ciclo de vida do modelo. Sua principal vantagem é o versionamento de modelos, que permite rastrear e registrar modelos de forma organizada.
* Kubernetes e FastAPI para Serviço de Inferência: O Kubernetes gerencia a criação de múltiplos contêineres com a API do FastAPI e os escala automaticamente para lidar com milhares de requisições por segundo, enquanto um API Gateway protege o endpoint.
* PostgreSQL para Base de Dados: Escolhido por sua confiabilidade, integridade de dados e capacidade de armazenamento estruturado, sendo ideal para catalogar metadados gerados em diversas etapas do pipeline.
* Prometheus e Grafana para Monitoramento: São as ferramentas mais utilizadas para monitoramento em tempo real. O Prometheus coleta métricas do sistema e do modelo, e o Grafana as visualiza em dashboards customizáveis, permitindo o acompanhamento do sistema e da performance do modelo em produção.
