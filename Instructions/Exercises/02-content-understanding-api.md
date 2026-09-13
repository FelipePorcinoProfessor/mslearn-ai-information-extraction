---
lab:
  title: Desenvolver um aplicativo cliente do Content Understanding
  description: Use o Azure Content Understanding Python SDK para criar e usar analisadores programaticamente.
  duration: 30
  level: 300
  islab: true
  status: 'released'
  primarytopics:
    - Azure
    - Azure Content Understanding
---

# Desenvolver um aplicativo cliente do Content Understanding

Neste exercício, você usará o Azure Content Understanding Python SDK para criar um analisador que extrai informações de cartões de visita. Em seguida, você desenvolverá um aplicativo cliente que usa o analisador para extrair detalhes de contato de cartões de visita digitalizados.

Este exercício leva aproximadamente 30 minutos.

## Criar um recurso e um projeto do Microsoft Foundry

Os recursos que vamos usar neste exercício exigem um recurso e um projeto do Microsoft Foundry.

1. Em um navegador da web, abra o [portal do Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez que você entrar.
1. Verifique se a alternância **New Foundry** está ativada para que você esteja usando o **Foundry (new)**.
1. Selecione o nome do projeto no canto superior esquerdo e, em seguida, selecione **Create new project**.
1. Dê um nome ao seu projeto e expanda **Advanced options** para especificar as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Location**: Escolha uma das seguintes regiões compatíveis:\*
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

    > \*O Azure Content Understanding está disponível em regiões selecionadas. Consulte a [documentação de suporte por região](https://learn.microsoft.com/azure/ai-services/content-understanding/language-region-support) para saber a disponibilidade mais recente.

1. Selecione **Create** e aguarde a criação do seu projeto. Isso criará um projeto e o recurso pai.
1. Depois de criado, selecione o nome do projeto na parte superior da página e selecione **Project details**. Nessa página, siga o link para o recurso pai. Deixe essa guia do navegador aberta.

## Configurar modelos e conexão do Content Understanding



Conectar o Foundry por meio do portal do Content Understanding

O Content Understanding usa modelos OpenAI para análise que são implantados em seu projeto. Você precisa implantar esses modelos antes de usar analisadores e configurar a conexão entre o Content Understanding e seu recurso do Foundry. A maneira mais fácil é por meio do Content Understanding Studio.

1. Em uma nova guia, navegue para o [Content Understanding Studio](https://contentunderstanding.ai.azure.com/home) em `https://contentunderstanding.ai.azure.com/home` e entre com suas credenciais.
1. Selecione o ícone de engrenagem de configurações na barra de navegação superior e selecione **+ Add resource**.
1. Selecione sua assinatura e o grupo de recursos onde você criou seu recurso do Foundry e, em seguida, selecione o nome do recurso do Foundry no menu. Esse recurso é o recurso pai do projeto que você criou anteriormente.
1. Verifique se a caixa **Enable auto-deployment** está marcada e selecione **Next** e **Save** para criar a configuração.
1. Aguarde enquanto os modelos necessários para o Content Understanding são implantados.

## Preparar o ambiente de desenvolvimento

Você usará o Visual Studio Code como seu ambiente de desenvolvimento.

1. Inicie o **Visual Studio Code**.
1. Abra a Paleta de Comandos (pressione **Ctrl+Shift+P**), digite **Git: Clone** e selecione a opção.
1. Na barra de URL, cole o seguinte URL do repositório e pressione **Enter**:

    ```
    https://github.com/microsoftlearning/mslearn-ai-information-extraction
    ```

1. Escolha uma pasta local para clonar e, quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.
1. No painel do Explorador do VS Code, navegue até **Labfiles/02-content-understanding-api**. A pasta contém duas imagens de cartões de visita digitalizados, bem como os arquivos de código Python necessários para criar seu aplicativo.
1. Abra um novo terminal e navegue até a pasta do aplicativo:

    ```
   cd Labfiles/02-content-understanding-api
    ```

1. Instale as bibliotecas necessárias:

    ```
   python -m venv labenv
   labenv\Scripts\activate
   pip install -r requirements.txt azure-ai-contentunderstanding
    ```

1. No painel do Explorador do VS Code, abra o arquivo **.env** na pasta **Labfiles/02-content-understanding-api**.
1. No arquivo, substitua os placeholders **YOUR_ENDPOINT** e **YOUR_KEY** pelo endpoint e pela chave de API do seu recurso do Microsoft Foundry (copiados da guia do portal que você deixou aberta) e verifique se **ANALYZER_NAME** está definido como `businesscardanalyzer`.

    > Dica: você também pode encontrar o endpoint e as chaves no [portal do Azure](https://portal.azure.com) navegando até o seu recurso do Microsoft Foundry e visualizando **Resource Management** > **Keys and Endpoint**.

1. Salve o arquivo (**CTRL+S**).

## Criar um analisador com o Python SDK

Agora você usará o Content Understanding Python SDK para criar um analisador que pode extrair informações de imagens de cartões de visita.

1. No painel do Explorador do VS Code, abra o arquivo **biz-card.json** e revise seu conteúdo. Esse JSON define um esquema de analisador para um cartão de visita, especificando os campos a serem extraídos (Company, Name, Title, Email, Phone).

1. Abra o arquivo **create-analyzer.py** no VS Code.

1. Revise o código, que:
    - Importa `ContentUnderstandingClient` e `AzureKeyCredential` do [Azure Content Understanding SDK](https://learn.microsoft.com/python/api/overview/azure/ai-contentunderstanding-readme?view=azure-python-preview).
    - Carrega o esquema do analisador do arquivo **biz-card.json**.
    - Recupera o endpoint, a chave e o nome do analisador do arquivo de configuração de ambiente.
    - Chama uma função chamada **create_analyzer**, que atualmente não está implementada.

1. Na função **create_analyzer**, localize o comentário **Create a Content Understanding analyzer** e adicione o seguinte código (tendo cuidado para manter a indentação correta):

    ```python
    # Create a Content Understanding analyzer
    print(f"Creating {analyzer}")

    # Create the Content Understanding client
    client = ContentUnderstandingClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )

    # Parse the schema JSON into a ContentAnalyzer object
    analyzer_definition = json.loads(schema)

    # Create the analyzer using the SDK (long-running operation)
    poller = client.begin_create_analyzer(
        analyzer_id=analyzer,
        resource=analyzer_definition,
        allow_replace=True
    )

    # Wait for the operation to complete
    result = poller.result()
    print(f"Analyzer '{analyzer}' created successfully.")
    print(f"Status: {result['status'] if isinstance(result, dict) else 'Succeeded'}")
    ```

1. Revise o código que você adicionou, que:
    - Cria uma instância de `ContentUnderstandingClient` com o endpoint e a chave de API.
    - Analisa o JSON do esquema do analisador.
    - Usa `begin_create_analyzer` para iniciar a operação de longa duração para criar o analisador.
    - Chama `.result()` para aguardar a conclusão da operação.

    > Observação: o SDK gerencia a sondagem automaticamente por meio do padrão `LROPoller` — nenhuma sondagem manual é necessária!

1. Salve o arquivo (**CTRL+S**).
1. No terminal do VS Code (verifique se o ambiente virtual ainda está ativado e se você está na pasta **Labfiles/02-content-understanding-api**), execute o código Python:

    ```
    python create-analyzer.py
    ```

1. Revise a saída do programa, que deve indicar que o analisador foi criado.

## Analisar conteúdo usando o Python SDK

Agora que você criou um analisador, pode consumi-lo a partir de um aplicativo cliente por meio do Content Understanding Python SDK.

1. No VS Code, abra o arquivo **read-card.py**.

1. Revise o código, que:
    - Importa `ContentUnderstandingClient` e `AzureKeyCredential` do SDK.
    - Identifica o arquivo de imagem a ser analisado, com padrão de **biz-card-1.png**.
    - Recupera o endpoint e a chave do arquivo de configuração de ambiente.
    - Chama uma função chamada **analyze_card**, que atualmente não está implementada.

1. Na função **analyze_card**, localize o comentário **Use Content Understanding to analyze the image** e adicione o seguinte código (tendo cuidado para manter a indentação correta):

    ```python
    # Use Content Understanding to analyze the image
    print(f"Analyzing {image_file}")

    # Create the Content Understanding client
    client = ContentUnderstandingClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )

    # Read the image data
    with open(image_file, "rb") as file:
        image_data = file.read()

    # Submit the image for analysis
    print("Submitting request...")
    poller = client.begin_analyze_binary(
        analyzer_id=analyzer,
        binary_input=image_data
    )

    # Wait for the analysis to complete
    result = poller.result()
    print("Analysis succeeded:\n")

    # Save JSON results to a file
    output_file = "results.json"
    with open(output_file, "w") as json_file:
        json.dump(dict(result), json_file, indent=4, default=str)
        print(f"Response saved in {output_file}\n")

    # Iterate through the contents and extract fields
    for content in result.contents:
        if hasattr(content, 'fields') and content.fields:
            for field_name, field_data in content.fields.items():
                value = field_data.value if hasattr(field_data, 'value') else None
                print(f"{field_name}: {value}")
    ```

1. Revise o código que você adicionou, que:
    - Cria uma instância de `ContentUnderstandingClient`.
    - Lê o conteúdo do arquivo de imagem como bytes.
    - Chama `begin_analyze_binary` para enviar a imagem ao analisador (o SDK gerencia a sondagem assíncrona automaticamente).
    - Chama `.result()` para aguardar e recuperar os resultados da análise.
    - Salva a resposta JSON e analisa os campos extraídos.

1. Salve o arquivo (**CTRL+S**).
1. No terminal do VS Code, execute o código Python:

    ```
    python read-card.py biz-card-1.png
    ```

1. Revise a saída do programa, que deve mostrar os valores dos campos no seguinte cartão de visita:

    ![Um cartão de visita de Roberto Tamburello, um funcionário da Adventure Works Cycles.](./media/biz-card-1.png)

1. Execute o programa novamente com um cartão de visita diferente:

    ```
    python read-card.py biz-card-2.png
    ```

1. Revise os resultados, que devem refletir os valores deste cartão de visita:

    ![Um cartão de visita de Mary Duartes, uma funcionária da Contoso.](./media/biz-card-2.png)

1. Para ver a resposta JSON completa que foi retornada, abra o arquivo **results.json** no VS Code ou execute o seguinte comando no terminal:

    ```
    cat results.json
    ```

## Limpar

Se você terminou de trabalhar com o serviço Content Understanding, deve excluir os recursos criados neste exercício para evitar incorrer em custos desnecessários do Azure.

1. No [portal do Azure](https://portal.azure.com), exclua o grupo de recursos que você criou para este exercício.
