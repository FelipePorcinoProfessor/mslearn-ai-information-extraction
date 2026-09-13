---
lab:
  title: Criar uma solução de mineração de conhecimento
  description: Use o Azure AI Search para extrair informações importantes de documentos e facilitar a pesquisa e a análise.
  duration: 40
  level: 200
  islab: true
  status: 'released'
  primarytopics:
    - Azure
---

# Criar uma solução de mineração de conhecimento

Neste exercício, você usará o Azure AI Search para criar uma solução de mineração de conhecimento que indexa um conjunto de documentos de folhetos de viagem. O processo de indexação usa habilidades de IA para extrair informações importantes dos documentos, e você criará um aplicativo cliente em Python para pesquisar no índice.

Este exercício leva aproximadamente **40** minutos.

## Criar recursos do Azure

A solução requer vários recursos na sua assinatura do Azure, todos criados na mesma região.

### Criar um recurso do Azure AI Search

1. Em um navegador da Web, abra o [Azure portal](https://portal.azure.com) em `https://portal.azure.com` e entre com suas credenciais do Azure.
1. Selecione o botão **&#65291;Create a resource**, pesquise por `Azure AI Search` e crie um recurso **Azure AI Search** com as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Service name**: *Um nome válido para o seu recurso de pesquisa*
    - **Location**: *Qualquer localização disponível*
    - **Pricing tier**: Free
1. Aguarde a conclusão da implantação e vá para o recurso implantado.
1. Revise a página **Overview**. Aqui você pode usar uma interface visual para criar, testar, gerenciar e monitorar os vários componentes de uma solução de pesquisa.

### Criar uma conta de armazenamento

1. Volte para a página inicial do Azure portal e crie um recurso **Storage account** com as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *O mesmo grupo de recursos do seu recurso Azure AI Search*
    - **Storage account name**: *Um nome válido para o seu recurso de armazenamento*
    - **Region**: *A mesma região do seu recurso Azure AI Search*
    - **Primary service**: Azure Blob Storage or Azure Data Lake Storage Gen 2
    - **Performance**: Standard
    - **Redundancy**: Locally-redundant storage (LRS)
1. Aguarde a conclusão da implantação e vá para o recurso implantado.

## Carregar documentos no Azure Storage

Sua solução de mineração de conhecimento extrairá informações de documentos de folhetos de viagem armazenados no Azure Blob Storage.

1. Em uma nova guia do navegador, baixe [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) de `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` e salve-o em uma pasta local.
1. Extraia o arquivo *documents.zip* baixado e veja os arquivos de folhetos de viagem que ele contém.
1. No Azure portal, navegue até sua conta de armazenamento e selecione **Storage browser** no painel de navegação.
1. No Storage browser, selecione **Blob containers**.
1. Na barra de ferramentas, selecione **+ Container** e crie um novo contêiner com as seguintes configurações:
    - **Name**: `documents`
    - **Anonymous access level**: Private (no anonymous access)
1. Selecione o contêiner **documents** e use o botão da barra de ferramentas **Upload** para enviar os arquivos .pdf que você extraiu de **documents.zip**.

## Criar e executar um indexador

Agora que os documentos estão no lugar, você pode criar um indexador para usar habilidades de IA e extrair informações deles.

1. No Azure portal, navegue até seu recurso Azure AI Search. Na página **Overview**, selecione **Import data**.
1. Na página **Connect to your data**, na lista **Data Source**, selecione **Azure Blob Storage**.
1. Selecione **keyword search**. Em seguida, preencha os detalhes do repositório de dados com os seguintes valores:

1. Em **Connect to your data**, defina o seguinte:
    - **Storage account**: *Sua conta de armazenamento criada recentemente*
    - **Blob container**: Selecione o contêiner **documents**.
    - Deixe as demais opções com seus valores padrão e selecione **Next**.

1. Em **Apply AI enrichments**, defina o seguinte:
    - Selecione **Extract phrases**.
    - Selecione **Extract entities**, selecione o ícone de configurações, verifique se apenas **Persons** e **Locations** estão selecionados e selecione **Save**.
    - Selecione **Extract text from images**, selecione o ícone de configurações, verifique se **Generate tags** e **Categorize content** estão selecionados e selecione **Save**.
    - Se ainda não estiver selecionada, escolha a opção gratuita de recurso Foundry Tools e selecione **Next**.

    > Observação: o enriquecimento gratuito do Azure AI Services para o Azure AI Search pode ser usado para indexar no máximo 20 documentos. Em uma solução de produção, você deve criar e anexar um recurso do Azure AI Services.

1. Em **Preview mappings**, defina a seguinte configuração:
    - Os campos já estão mapeados com base nas opções selecionadas na etapa anterior.
    - Revise os seguintes campos e verifique se estão configurados conforme mostrado na tabela a seguir. Para atualizar um campo, selecione-o e, em seguida, selecione **Configure field**. Deixe todos os outros campos com as configurações padrão.

    | Target index field name | Retrievable | Filterable | Sortable | Facetable | Searchable |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &#10004; | &#10004; | &#10004; | | |
    | metadata_storage_last_modified | &#10004; | &#10004; | &#10004; | | |
    | title | &#10004; | &#10004; | &#10004; | | &#10004; |
    | locations | &#10004; | &#10004; | | | &#10004; |
    | persons | &#10004; | &#10004; | | | &#10004; |
    | keyPhrases | &#10004; | &#10004; | | | &#10004; |

    - Verifique suas seleções com atenção.
    - Selecione **Next**.

1. Em **Advanced settings**, defina o seguinte:
    - Verifique se **Enable semantic ranker** está selecionado.
    - Se ainda não estiver selecionado, defina **Schedule** como **Once**.
    - Selecione **Next**.

1. Em **Review and create**, defina **Objects name prefix** como `margies-index` e selecione **Create**.
1. Você pode fechar a notificação de sucesso.
1. No painel de navegação à esquerda, em **Search management**, veja a página **Indexers**. O **margies-index-indexer** deve aparecer. Aguarde alguns minutos e clique em **&orarr; Refresh** até que o **Status** indique **Success**.

## Pesquisar no índice

Agora que você tem um índice, pode pesquisá-lo.

1. Volte para a página **Overview** do seu recurso Azure AI Search e, na barra de ferramentas, selecione **Search explorer**.
1. No Search explorer, na caixa **Query string**, insira `*` (um único asterisco) e selecione **Search**.

    Essa consulta recupera todos os documentos no índice em formato JSON. Examine os resultados e observe os campos de cada documento, que incluem conteúdo do documento, metadados e dados enriquecidos extraídos pelas habilidades cognitivas.

1. No menu **View**, selecione **JSON view** e observe que a solicitação JSON da pesquisa é mostrada:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Os resultados incluem um campo **@odata.count** na parte superior que indica o número de documentos retornados pela pesquisa.

1. Modifique a solicitação JSON para incluir um parâmetro **select**:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "title,locations"
    }
    ```

        Desta vez, os resultados incluem apenas o nome do arquivo e quaisquer localizações mencionadas no conteúdo do documento. O nome do arquivo está no campo **title**. O campo **locations** foi gerado por uma habilidade de IA.

1. Experimente a seguinte string de consulta:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "title,keyPhrases"
    }
    ```

    Esta pesquisa encontra documentos que mencionam "New York" em qualquer campo pesquisável e retorna o nome do arquivo e as frases-chave.

1. Experimente mais uma consulta:

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "title,keyPhrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    Isso retorna documentos que mencionam "New York" e que são menores que 380.000 bytes.

## Criar um aplicativo cliente de pesquisa

Agora que você tem um índice útil, pode consultá-lo a partir de um aplicativo cliente em Python usando o SDK do Azure AI Search.

### Obter o endpoint e as chaves do seu recurso de pesquisa

1. No Azure portal, volte para a página **Overview** do seu recurso Azure AI Search. Observe o valor de **Url** (por exemplo, `https://your_resource_name.search.windows.net`). Este é o endpoint do seu recurso de pesquisa.
1. No painel de navegação à esquerda, expanda **Settings** e veja a página **Keys**. Observe a chave **query** — você precisará dela para seu aplicativo cliente.

    > Observação: o Azure AI Search cria uma chave de consulta padrão para o serviço. No Azure portal, essa chave de consulta padrão pode aparecer com um nome em branco. Esse é o comportamento esperado.

### Preparar o uso do SDK do Azure AI Search

1. Inicie o **Visual Studio Code**.
1. Abra a Command Palette (pressione **Ctrl+Shift+P**), digite **Git: Clone** e selecione a opção.
1. Na barra de URL, cole o seguinte URL de repositório e pressione **Enter**:

    ```
    https://github.com/microsoftlearning/mslearn-ai-information-extraction
    ```

1. Escolha uma pasta local para clonar e, quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.
1. Abra um novo terminal e navegue até a pasta de código Python:

    ```
    cd Labfiles/04-knowledge-mining
    ```

1. Instale os pacotes necessários:

    ```
    python -m venv labenv
    labenv\Scripts\activate
    pip install -r requirements.txt
    ```

    > Observação: o requirements.txt instala o pacote Python SDK [azure-search-documents](https://learn.microsoft.com/python/api/overview/azure/search-documents-readme?view=azure-python) e suas dependências.

1. No painel Explorer do VS Code, abra o arquivo **.env** em **Labfiles/04-knowledge-mining**.

1. Substitua os seguintes valores de placeholder:
    - **your_search_endpoint**: *O endpoint do seu recurso Azure AI Search*
    - **your_query_key**: *A chave de consulta do seu recurso Azure AI Search*
    - **your_index_name**: *O nome do seu índice (deve ser `margies-index`)*
1. Salve o arquivo (**CTRL+S**).

1. No VS Code, abra o arquivo **search-app.py**.

1. Revise o código, que:
    - Recupera as configurações de configuração do arquivo .env.
    - Cria um `SearchClient` com o endpoint, a chave e o nome do índice.
    - Solicita ao usuário uma consulta de pesquisa em um loop (até que ele digite "quit").
    - Pesquisa no índice usando a consulta, retornando os seguintes campos ordenados por título:
        - title
        - locations
        - persons
        - keyPhrases
    - Analisa os resultados da pesquisa retornados para exibir os campos retornados para cada documento no conjunto de resultados.
1. No terminal do VS Code, execute o aplicativo:

    ```
    python search-app.py
    ```

1. Quando solicitado, insira uma consulta como `London` e veja os resultados.
1. Experimente outra consulta, como `flights`.
1. Quando terminar os testes, insira `quit` para fechar o aplicativo.

## Observação sobre o knowledge store

As etapas do knowledge store foram excluídas desta versão do exercício.

O fluxo atual de pesquisa por palavra-chave de **Import data** no Azure portal não cria um knowledge store para este cenário, e a alternativa multimodal não foi adotada para este exercício.

## Limpar recursos

Se você terminou de trabalhar com o Azure AI Search, deve excluir os recursos criados neste exercício para evitar a geração de custos desnecessários no Azure.

1. No [Azure portal](https://portal.azure.com), exclua o grupo de recursos que você criou para este exercício.

## Mais informações

Para saber mais sobre o Azure AI Search, consulte a [documentação do Azure AI Search](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
