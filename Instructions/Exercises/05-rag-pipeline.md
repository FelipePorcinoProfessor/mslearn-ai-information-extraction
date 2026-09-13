---
lab:
  title: Crie um pipeline de ingestão RAG automatizado com Content Understanding
  description: Use Azure Content Understanding, Azure AI Search e Azure OpenAI para criar um pipeline contínuo de ingestão RAG multimodal.
  duration: 45
  level: 300
  islab: true
  status: 'released'
  primarytopics:
    - Azure
    - Azure Content Understanding
---

# Crie um pipeline de ingestão RAG automatizado com Content Understanding

Retrieval-augmented generation (RAG) é um método que aprimora Large Language Models (LLMs) ao integrar dados de fontes de conhecimento externas. Em cenários de produção, novos documentos chegam continuamente e precisam ser extraídos, embedded e indexados para ficarem disponíveis para pesquisa quase em tempo real.

Neste exercício, você criará um pipeline automatizado de ingestão RAG que usa Azure Content Understanding para extrair conteúdo de documentos multimodais, faz o embedding do conteúdo com Azure OpenAI e o indexa no Azure AI Search. O pipeline acompanha quais arquivos já foram processados e pode ser executado em **watch mode** para detectar e ingerir automaticamente novos documentos assim que eles chegarem. Você finalizará criando um agente conversacional que responde a perguntas com base nos seus dados indexados.

Este exercício leva aproximadamente **45** minutos.

## Criar recursos do Azure

Você precisa de vários recursos do Azure para este pipeline: um recurso Microsoft Foundry (para Content Understanding e Azure OpenAI) e um recurso Azure AI Search.

### Criar um recurso e projeto Microsoft Foundry

1. Em um navegador da Web, abra o [portal Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre com suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido abertos na primeira vez em que você entrar.
1. Verifique se a alternância **New Foundry** está ativada para que você esteja usando o **Foundry (new)**.
1. Selecione o nome do projeto no canto superior esquerdo e, em seguida, selecione **Create new project**.
1. Dê um nome ao projeto e expanda **Advanced options** para especificar as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Location**: Escolha uma das seguintes regiões com suporte:\*
        - Australia East
        - East US
        - East US 2
        - Japan East
        - North Europe
        - South Central US
        - Southeast Asia
        - Sweden Central
        - UK South
        - West Europe
        - West US
        - West US 3

    > \*Azure Content Understanding está disponível em regiões selecionadas. Consulte a [documentação de suporte de regiões](https://learn.microsoft.com/azure/ai-services/content-understanding/language-region-support) para a disponibilidade mais recente.

1. Selecione **Create** e aguarde a criação do projeto. Isso criará um projeto e o recurso pai.
1. Após a criação, selecione o nome do projeto na parte superior da página e selecione **Project details**. Nessa página, siga o link para o recurso pai. Deixe essa guia do navegador aberta.

### Configurar modelos e conexão do Content Understanding

O Content Understanding usa modelos OpenAI para análise que são implantados no seu projeto. Você precisa implantar esses modelos antes de usar analisadores e configurar a conexão entre o Content Understanding e seu recurso Foundry. A maneira mais fácil é por meio do Content Understanding Studio.

1. Em uma nova guia, acesse o [Content Understanding Studio](https://contentunderstanding.ai.azure.com/home) em `https://contentunderstanding.ai.azure.com/home` e entre com suas credenciais.
1. Selecione o ícone de engrenagem de configurações na barra de navegação superior e selecione **+ Add resource**.
1. Selecione sua assinatura e o grupo de recursos onde você criou seu recurso Foundry e, em seguida, selecione o nome do seu recurso Foundry na lista. Esse recurso é o recurso pai do projeto que você criou anteriormente.
1. Certifique-se de que a caixa **Enable auto-deployment** esteja marcada, depois selecione **Next** e **Save** para criar a configuração.
1. Aguarde enquanto os modelos necessários para o Content Understanding são implantados.

### Criar um recurso Azure AI Search

1. Em uma nova guia do navegador, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre com suas credenciais do Azure.
1. Selecione **&#65291;Create a resource**, pesquise por `Azure AI Search` e crie um recurso **Azure AI Search** com as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *O mesmo grupo de recursos do seu recurso Microsoft Foundry*
    - **Service name**: *Um nome exclusivo válido*
    - **Location**: *A mesma localização do seu recurso Microsoft Foundry*
    - **Pricing tier**: Free ou Basic
1. Aguarde a conclusão da implantação.

### Coletar credenciais

Você precisará dos seguintes valores para configurar o pipeline. Anote-os no portal do Azure:

- **Foundry endpoint**: Na guia do recurso pai que você deixou aberta, copie o **Endpoint** da página **Overview** (por exemplo, `https://<name>.services.ai.azure.com/`).
- **Foundry API key**: Na mesma página, selecione **Resource Management** > **Keys and Endpoint** e copie uma das **Keys**.
- **Azure AI Search endpoint**: Na página **Overview** do seu recurso AI Search no portal do Azure (por exemplo, `https://<name>.search.windows.net`).
- **Model deployments**: Na página Foundry Home, selecione **Build** > **Deployments** para ver seus modelos implantados. Observe que há um número no final do nome do seu modelo de embedding, que você precisará atualizar no arquivo `.env`.
- **Azure AI Search admin key**: Na página **Settings** > **Keys** do seu recurso AI Search.

    > **Observação**: O endpoint e a chave do Foundry são usados tanto para o Content Understanding quanto para o Azure OpenAI, já que ambos os serviços estão incluídos no mesmo recurso Foundry.

## Preparar o ambiente de desenvolvimento

Você usará o Visual Studio Code como seu ambiente de desenvolvimento.

1. Inicie o **Visual Studio Code**.
1. Abra a Paleta de Comandos (pressione **Ctrl+Shift+P**), digite **Git: Clone** e selecione.
1. Na barra de URL, cole o seguinte repositório e pressione **Enter**:

    ```
    https://github.com/microsoftlearning/mslearn-ai-information-extraction
    ```

1. Escolha uma pasta local para clonar e, quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.
1. Abra um novo terminal e navegue até a pasta do pipeline RAG:

    ```
    cd Labfiles/05-rag-pipeline
    ```

1. Instale os pacotes Python necessários:

    ```
    python -m venv labenv
    labenv\Scripts\activate
    pip install -r requirements.txt
    ```

    > **Observação**: O requirements.txt instala o SDK [azure-ai-contentunderstanding](https://learn.microsoft.com/python/api/overview/azure/ai-contentunderstanding-readme?view=azure-python-preview), o SDK [azure-search-documents](https://learn.microsoft.com/python/api/overview/azure/search-documents-readme?view=azure-python) e o pacote [openai](https://pypi.org/project/openai/).

1. No painel Explorer do VS Code, abra o arquivo **.env** em **Labfiles/05-rag-pipeline**.
1. Substitua os valores de placeholder no arquivo `.env` pelas credenciais que você coletou anteriormente:
    - `FOUNDRY_ENDPOINT` — O endpoint do seu recurso Microsoft Foundry
    - `FOUNDRY_KEY` — A API key do seu recurso Microsoft Foundry
    - `AZURE_OPENAI_CHAT_DEPLOYMENT_NAME` — O nome da implantação do seu modelo de chat (por exemplo, `gpt-4.1-######`)
    - `AZURE_OPENAI_EMBEDDING_DEPLOYMENT_NAME` — O nome da implantação do seu modelo de embedding (por exemplo, `text-embedding-3-large-######`)
    - `AZURE_SEARCH_ENDPOINT` — O endpoint do seu Azure AI Search
    - `AZURE_SEARCH_KEY` — A admin key do seu Azure AI Search
1. Salve o arquivo (**CTRL+S**).

### Baixar documentos de exemplo

O pipeline RAG precisa de documentos para processar. Você usará os mesmos folhetos de viagem do exercício de extração de conhecimento.

1. Baixe [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) de `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip`.
1. Extraia os arquivos PDF do zip e copie-os para a pasta **Labfiles/05-rag-pipeline/data**.
1. Verifique se os arquivos estão no lugar conferindo a pasta data no VS Code Explorer ou executando no seu terminal:

    ```
    dir data
    ```

## Etapa 1: Criar um analisador do Content Understanding

A primeira etapa do pipeline é criar um analisador que extraia conteúdo estruturado dos documentos. Você usará o Content Understanding Python SDK para criar um analisador de forma programática.

1. No VS Code, abra o arquivo **create-analyzer.py**.

1. Analise o código, que:
    - Carrega variáveis de ambiente do arquivo `.env`.
    - Cria um `ContentUnderstandingClient` usando o endpoint e a API key.
    - Define um analisador de documentos com um esquema de extração de campos para capturar resumos e tópicos principais.
    - Cria o analisador chamando `begin_create_analyzer`.

1. No terminal do VS Code (verifique se o ambiente virtual está ativado), execute o script:

    ```
    python create-analyzer.py
    ```

1. Aguarde a criação do analisador. A saída deve confirmar que o analisador foi criado com sucesso.

## Etapa 2: Executar o pipeline de ingestão automatizado

Agora você executará o pipeline de ingestão automatizado. Este script único lida com todo o fluxo — extraindo conteúdo com o Content Understanding, gerando embeddings vetoriais com o Azure OpenAI e indexando no Azure AI Search. Ele também rastreia quais arquivos foram processados para poder detectar documentos novos ou atualizados em execuções subsequentes.

1. No VS Code, abra o arquivo **ingest-pipeline.py**.

1. Analise o código e observe como ele:
    - **Rastreia arquivos processados** usando um manifesto (`processed_files.json`) que registra o hash SHA-256 de cada arquivo. A cada execução, o pipeline compara o hash atual de cada arquivo na pasta `data/` com o manifesto, de modo que apenas arquivos novos ou modificados sejam processados.
    - **Garante que o índice de pesquisa exista** chamando `ensure_index()`, que cria ou atualiza o índice do Azure AI Search com o esquema necessário (campos de texto, um campo vetorial e configuração de pesquisa vetorial HNSW).
    - **Extrai conteúdo** de cada novo arquivo enviando-o ao analisador do Content Understanding via `begin_analyze_binary`, que retorna conteúdo em markdown e campos extraídos (resumo, tópicos principais).
    - **Divide o conteúdo em chunks** separando nos limites de parágrafo com limite de 2000 caracteres, mantendo cada chunk autocontido.
    - **Gera embeddings** para cada chunk usando o modelo de embedding do Azure OpenAI, produzindo um vetor de 3072 dimensões para pesquisa semântica.
    - **Indexa os chunks** no Azure AI Search usando IDs de documento determinísticos (com base no nome do arquivo e no índice do chunk), de modo que a reingestão de um arquivo atualizado substitua seus chunks antigos.
    - Oferece suporte a um sinalizador `--watch` para monitoramento contínuo e a um sinalizador `--reset` para reprocessar todos os arquivos.

1. No terminal do VS Code, execute o pipeline:

    ```
    python ingest-pipeline.py
    ```

1. Observe a saída enquanto o pipeline processa cada documento. Você verá mensagens de log com carimbo de data/hora mostrando cada arquivo sendo extraído, chunks sendo embedded e resultados sendo indexados. Por exemplo:

    ```
    [14:23:01] Verifying search index...
    [14:23:02] Search index 'rag-content-index' is ready.
    [14:23:02] Detected 5 new/updated file(s).
    [14:23:02]   Processing: Margies-Travel-Company-Info.pdf
    [14:23:08]     Embedding chunk 1/3...
    [14:23:09]     Embedding chunk 2/3...
    [14:23:09]     Embedding chunk 3/3...
    [14:23:10]     Indexed 3 chunk(s) from Margies-Travel-Company-Info.pdf.
    ...
    ```

1. Após a conclusão do pipeline, verifique se foi criado um arquivo **processed_files.json** na pasta rag-pipeline. Esse manifesto registra o hash de cada arquivo processado — se você executar o pipeline novamente, ele detectará que não há novos arquivos:

    ```
    python ingest-pipeline.py
    ```

    A saída deve dizer "Nenhum novo arquivo para ingerir — todos os documentos estão atualizados."

## Etapa 3: Consultar o índice com um agente RAG

Com o conteúdo indexado, você pode usar um agente conversacional que recupera conteúdo relevante e usa um modelo de chat do OpenAI para responder às perguntas.

1. No VS Code, abra o arquivo **rag-agent.py**.

1. Analise o código, que:
    - Cria um cliente do Azure AI Search para recuperar documentos.
    - Cria um cliente de chat do Azure OpenAI.
    - Implementa uma função de recuperação que realiza pesquisa híbrida (combinando pesquisa por palavra-chave e vetorial) para encontrar os chunks de conteúdo mais relevantes.
    - Constrói um prompt que inclui o contexto recuperado e a pergunta do usuário.
    - Envia o prompt ao modelo de chat para geração da resposta.
    - Executa um loop conversacional para que você possa fazer várias perguntas.

1. No terminal do VS Code, execute o agente:

    ```
    python rag-agent.py
    ```

1. Quando solicitado, insira uma pergunta sobre o conteúdo que você indexou. Por exemplo:
    - `What destinations are featured in the travel brochures?`
    - `What activities are recommended in Dubai?`
    - `Tell me about the Margie's Travel company`

1. Analise as respostas do agente. Elas devem estar fundamentadas no conteúdo real extraído dos documentos, com respostas que citam os nomes dos documentos de origem. Quando estiver satisfeito, digite `quit` para sair do agente.

## Etapa 4: Ingerir novos documentos automaticamente

O verdadeiro poder deste pipeline é a ingestão contínua. Agora você iniciará o pipeline em watch mode para que ele monitore a pasta `data/` e, em seguida, adicionará um novo documento e observará ele ser automaticamente extraído, embedded e indexado.

### Iniciar o pipeline em watch mode

1. No VS Code, abra um **segundo terminal** (selecione **Terminal** > **New Terminal**). Certifique-se de ativar o ambiente virtual e navegar até a pasta do pipeline:

    ```
    cd Labfiles\05-rag-pipeline
    labenv\Scripts\activate
    ```

1. Inicie o pipeline em watch mode:

    ```
    python ingest-pipeline.py --watch
    ```

    O pipeline começará a fazer polling da pasta `data/` a cada 30 segundos. Você deverá ver uma saída como:

    ```
    [14:30:00] Watching 'data/' for new documents (press Ctrl+C to stop)...

    [14:30:01] No new files. Waiting...
    ```

    Deixe este terminal em execução.

### Adicionar um novo documento

1. Mude para o painel Explorer do VS Code e clique com o botão direito na pasta **data** em **Labfiles/05-rag-pipeline**. Selecione **New File** e nomeie-o como **tokyo-guide.txt**.

1. Adicione o seguinte conteúdo ao novo arquivo e salve:

    ```text
    Tokyo Travel Guide

    Tokyo, the capital of Japan, is one of the most dynamic cities in the world,
    blending centuries-old tradition with cutting-edge technology and innovation.

    Top Attractions:
    - Senso-ji Temple: Tokyo's oldest temple, located in Asakusa, is a must-visit.
      The approach through Nakamise-dori shopping street is iconic.
    - Shibuya Crossing: The world's busiest pedestrian crossing is a symbol of
      Tokyo's energy and pace.
    - Meiji Shrine: A serene Shinto shrine set in a lush forest in the heart of
      the city, dedicated to Emperor Meiji.
    - Tokyo Skytree: At 634 meters, this broadcasting tower offers panoramic views
      of the entire metropolitan area.
    - Tsukiji Outer Market: While the inner wholesale market has moved to Toyosu,
      the outer market still offers incredible fresh seafood and street food.

    Neighborhoods to Explore:
    - Shinjuku: A vibrant district known for its nightlife, shopping, and the
      beautiful Shinjuku Gyoen National Garden.
    - Akihabara: The hub of anime, manga, and electronics culture.
    - Harajuku: Famous for its youth fashion, Takeshita Street, and trendy cafes.
    - Ginza: Tokyo's upscale shopping and dining district.

    Getting Around:
    Tokyo has one of the world's most efficient public transportation systems.
    The Tokyo Metro and JR lines connect every corner of the city. A Suica or
    Pasmo card makes travel seamless. For visitors, the Japan Rail Pass offers
    unlimited travel on JR lines.

    Best Time to Visit:
    Spring (March-May) for cherry blossoms and autumn (October-November) for
    fall foliage are the most popular seasons. Summers can be hot and humid,
    while winters are mild compared to northern Japan.
    ```

1. Volte para o terminal que está executando o pipeline em watch mode. Em até 30 segundos, você deverá ver o pipeline detectar e processar o novo arquivo:

    ```
    [14:31:00] Detected 1 new/updated file(s).
    [14:31:00]   Processing: tokyo-guide.txt
    [14:31:05]     Embedding chunk 1/1...
    [14:31:06]     Indexed 1 chunk(s) from tokyo-guide.txt.
    [14:31:06] Ingestion complete — 1 file(s), 1 chunk(s) indexed.
    ```

### Consultar o conteúdo recém-ingestido

1. Volte ao seu **primeiro terminal** (ou abra um novo com o ambiente virtual ativado) e execute o agente RAG novamente:

    ```
    python rag-agent.py
    ```

1. Faça uma pergunta sobre o documento recém-adicionado:
    - `What can you tell me about Tokyo?`
    - `What are the top attractions in Tokyo?`
    - `How do I get around in Tokyo?`

1. Agora o agente deve retornar respostas fundamentadas no guia de viagem de Tóquio — conteúdo que não estava disponível durante sua primeira sessão de consulta. Isso demonstra como o pipeline contínuo disponibiliza novos conhecimentos sem qualquer reprocessamento manual.

1. Digite `quit` para sair do agente e, em seguida, alterne para o terminal em watch mode e pressione **Ctrl+C** para parar o pipeline.

## Limpeza

Se você terminou de trabalhar com o pipeline RAG, exclua os recursos criados neste exercício para evitar custos desnecessários do Azure.

1. No [portal do Azure](https://portal.azure.com), exclua o grupo de recursos que você criou para este exercício.

## Mais informações

- [Tutorial: Crie uma solução RAG com Content Understanding](https://learn.microsoft.com/azure/ai-services/content-understanding/tutorial/build-rag-solution)
- [Retrieval-augmented generation no Azure AI Search](https://learn.microsoft.com/azure/search/retrieval-augmented-generation-overview)
- [Azure Content Understanding Python SDK](https://pypi.org/project/azure-ai-contentunderstanding/)
