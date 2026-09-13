---
lab:
  title: Extrair dados com Azure Document Intelligence
  description: Use modelos predefinidos e personalizados do Document Intelligence para extrair dados estruturados de documentos.
  duration: 45
  level: 300
  islab: true
  status: 'released'
  primarytopics:
    - Azure
    - Azure Document Intelligence
---

# Extrair dados com Azure Document Intelligence

O Azure Document Intelligence é um serviço do Azure AI que permite criar software de processamento de dados automatizado. Esse software pode extrair texto, pares chave/valor e tabelas de formulários usando optical character recognition (OCR). O Azure Document Intelligence tem modelos predefinidos para reconhecer faturas, recibos, cartões de visita e outros tipos comuns de documentos. O serviço também oferece a capacidade de treinar modelos personalizados que podem extrair campos de dados específicos dos seus próprios formulários.

Neste exercício, você usará modelos do Document Intelligence, predefinidos e personalizados, para extrair informações de documentos.

Este exercício leva aproximadamente 45 minutos.

## Criar um recurso do Document Intelligence

O Azure Document Intelligence está incluído no Azure AI Services. Você criará um recurso do Document Intelligence diretamente no Document Intelligence Studio.

1. Em um navegador da Web, acesse o **Document Intelligence Studio** em `https://contentunderstanding.ai.azure.com/documentintelligence/studio` e entre com suas credenciais do Azure.
1. No Studio, selecione o ícone **Settings** (⚙) no canto superior direito e, em seguida, selecione a guia **Resource**.
1. Selecione **Create a new resource** e configure com as seguintes definições:
    - **Subscription**: Sua assinatura do Azure
    - **Resource group**: Crie ou selecione um grupo de recursos
    - **Name**: Um nome válido para o seu recurso do Document Intelligence
    - **Region**: Qualquer região disponível
    - **Pricing tier**: Free F0 (se você não tiver um nível Free disponível, selecione Standard S0)
1. Selecione **Create** e aguarde a implantação do recurso. O Studio se conecta automaticamente ao novo recurso.

## Usar o modelo Read no portal

Agora vamos usar o modelo Read no Studio para analisar um documento multilíngue:

1. Na página inicial do Document Intelligence Studio, em **Document analysis**, selecione o bloco **Read**.
1. Na lista de documentos à esquerda, selecione **read-german.pdf**.
1. Na barra de ferramentas superior, selecione **Analyze options**, marque a caixa de seleção **Language** (em **Optional detection**) no painel **Analyze options** e selecione **Save**.
1. No canto superior esquerdo, selecione **Run Analysis**.
1. Quando a análise for concluída, o texto extraído da imagem será exibido à direita na guia **Content**. Revise esse texto e compare-o com o texto da imagem original para verificar a precisão.
1. Selecione a guia **Result**. Essa guia exibe o código JSON extraído.
1. Role até o final do código JSON na guia **Result**. Observe que o modelo Read detectou o idioma de cada span indicado por `locale`. A maioria dos spans está em alemão (código de idioma `de`), mas você pode encontrar outros códigos de idioma nos spans (por exemplo, inglês — código de idioma `en` — em um dos primeiros spans).

## Analisar uma fatura com um modelo predefinido usando o Python SDK

Agora vamos usar o Document Intelligence Python SDK para analisar uma fatura programaticamente.

### Preparar o ambiente de desenvolvimento

1. No [Azure portal](https://portal.azure.com), localize o recurso do Document Intelligence que você criou anteriormente. Em **Resource Management**, selecione **Keys and Endpoint** e anote o **Endpoint** e uma das **Keys**. Você precisará desses valores em breve.
1. Inicie o **Visual Studio Code**.
1. Abra a Command Palette (pressione **Ctrl+Shift+P**), digite **Git: Clone** e selecione a opção.
1. Na barra de URL, cole o seguinte URL do repositório e pressione **Enter**:

    ```
    https://github.com/microsoftlearning/mslearn-ai-information-extraction
    ```

1. Escolha uma pasta local para clonar e, quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.
1. Abra um novo terminal e navegue até a pasta prebuilt do Document Intelligence:

    ```
    cd Labfiles/03-document-intelligence/prebuilt/Python
    ```

1. Instale as bibliotecas necessárias:

    ```
    python -m venv labenv
    labenv\Scripts\activate
    pip install -r requirements.txt
    ```

    > **Observação**: O requirements.txt instala o pacote do Python SDK [azure-ai-documentintelligence](https://learn.microsoft.com/python/api/overview/azure/ai-documentintelligence-readme?view=azure-python) e suas dependências.

1. No painel Explorer do VS Code, abra o arquivo **.env** em **Labfiles/03-document-intelligence/prebuilt/Python**.
1. No arquivo, substitua os placeholders **YOUR_ENDPOINT** e **YOUR_KEY** pelo endpoint e pela chave de API do seu recurso do Document Intelligence.
1. Salve o arquivo (**CTRL+S**).

### Adicionar código para analisar uma fatura

Esta é a fatura de exemplo que seu código irá analisar:

![Screenshot showing a sample invoice document.](./media/sample-invoice.png)

1. No VS Code, abra o arquivo **document-analysis.py**.

1. No arquivo de código, localize o comentário **Add references** e adicione o seguinte código:

    ```python
    # Add references
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.documentintelligence import DocumentIntelligenceClient
    from azure.ai.documentintelligence.models import AnalyzeDocumentRequest
    ```

1. Localize o comentário **Create the client** e adicione o seguinte código (tomando cuidado para manter a indentação correta):

    ```python
    # Create the client
    document_analysis_client = DocumentIntelligenceClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
    )
    ```

1. Localize o comentário **Analyze the invoice** e adicione o seguinte código:

    ```python
    # Analyze the invoice
    poller = document_analysis_client.begin_analyze_document(
        fileModelId,
        AnalyzeDocumentRequest(url_source=fileUri),
        locale=fileLocale
    )
    ```

1. Localize o comentário **Display invoice information to the user** e adicione o seguinte código:

    ```python
    # Display invoice information to the user
    result = poller.result()

    for document in result.documents:

        vendor_name = document.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.get('valueString')}, with confidence {vendor_name.get('confidence')}.")

        customer_name = document.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: {customer_name.get('valueString')}, with confidence {customer_name.get('confidence')}.")

        invoice_total = document.fields.get("InvoiceTotal")
        if invoice_total:
            amount = invoice_total.get("valueCurrency", {})
            print(f"Invoice Total: {amount.get('currencySymbol', '$')}{amount.get('amount')}, with confidence {invoice_total.get('confidence')}.")
    ```

1. Revise o código que você adicionou, que:
    - Cria um `DocumentIntelligenceClient` com seu endpoint e suas credenciais.
    - Usa o modelo `prebuilt-invoice` para analisar o documento a partir de uma URL.
    - Itera pelos resultados e imprime o nome do fornecedor, o nome do cliente e o total da fatura.

1. Salve o arquivo (**CTRL+S**).
1. No terminal do VS Code, execute o aplicativo:

    ```
    python document-analysis.py
    ```

1. Revise a saída. O programa deve exibir o nome do fornecedor, o nome do cliente e o total da fatura com níveis de confiança. Compare os valores com a fatura de exemplo mostrada acima.

## Treinar e testar um modelo personalizado

Os modelos predefinidos são úteis para tipos comuns de documentos, mas, muitas vezes, você precisa extrair dados específicos de seus próprios formulários. Você pode treinar um modelo personalizado do Document Intelligence para extrair os campos específicos de que precisa.

### Preparar os dados de treinamento

Foi fornecido um script de configuração para criar uma conta de armazenamento e fazer upload de formulários de exemplo para treinamento.

1. No terminal do VS Code, navegue até a pasta do modelo personalizado:

    ```
    cd ../../custom
    ```

    > **Dica**: Se você não tiver certeza do seu diretório atual, execute `cd` para verificar.

1. No VS Code, abra o arquivo **setup.sh** em **Labfiles/03-document-intelligence/custom**.

1. Revise os comandos no script. Ele irá:
    - Criar uma conta de armazenamento no seu grupo de recursos do Azure
    - Fazer upload de arquivos da pasta *sample-forms* para um contêiner
    - Exibir um URI de Shared Access Signature (SAS)

1. Modifique as declarações das variáveis **subscription_id**, **resource_group** e **location** com os valores apropriados para a assinatura, o grupo de recursos e o nome da região onde você implantou o recurso do Document Intelligence.

    > **Importante**: Para sua string **location**, use o formato de código (por exemplo, `eastus` para "East US"). Você pode encontrar isso na **JSON View** do seu grupo de recursos no Azure portal.

    Se a variável **expiry_date** estiver no passado, atualize-a para uma data futura, por exemplo `2026-12-31`.

1. Salve o arquivo (**CTRL+S**).
1. Para executar o script de configuração, você precisa de um shell Bash. Você pode usar uma das seguintes opções, garantindo que esteja conectado à sua conta do Azure:
    - **Azure Cloud Shell**: No [Azure portal](https://portal.azure.com), abra um Cloud Shell (Bash), navegue até a pasta e execute `./setup.sh`.
    - **Terminal do VS Code (com WSL ou Git Bash no Windows)**: Execute `bash setup.sh`.

1. Quando o script for concluído, revise a saída exibida.
1. No Azure portal, atualize seu grupo de recursos e verifique se a conta de armazenamento foi criada. Abra a conta de armazenamento e, em **Storage browser**, expanda **Blob containers** e selecione o contêiner **sampleforms** para confirmar se os arquivos foram carregados.

### Treinar o modelo no Document Intelligence Studio

Agora você usará os formulários de treinamento para criar um modelo de extração personalizado.

1. Abra uma nova guia do navegador e acesse o **Document Intelligence Studio** em `https://documentintelligence.ai.azure.com/studio`.
1. Role até a seção **Custom models** e selecione o bloco **Custom extraction model**.
1. Se solicitado, entre com suas credenciais do Azure.
1. Se for perguntado qual recurso do Azure Document Intelligence usar, selecione a assinatura e o nome do recurso que você usou ao criar o recurso.
1. Em **My Projects**, crie um novo projeto com a seguinte configuração:

    - **Enter project details**:
        - **Project name**: Um nome válido para seu projeto
    - **Configure service resource**:
        - **Subscription**: Sua assinatura do Azure
        - **Resource group**: O grupo de recursos do seu recurso do Document Intelligence
        - **Document Intelligence resource**: Seu recurso do Document Intelligence (selecione a opção *Set as default* e use a versão de API padrão)
    - **Connect training data source**:
        - **Subscription**: Sua assinatura do Azure
        - **Resource group**: Seu grupo de recursos
        - **Storage account**: A conta de armazenamento criada pelo script de configuração (selecione a opção *Set as default*, selecione o contêiner de blob `sampleforms` e deixe o caminho da pasta em branco)

1. Quando seu projeto for criado, no canto superior direito da página, selecione **Train** para treinar seu modelo. Use a seguinte configuração:
    - **Model ID**: Um nome válido para seu modelo — anote-o para uso posterior
    - **Build Mode**: Template
1. Selecione **Go to Models**.
1. O treinamento pode levar algum tempo. Aguarde até que o status do modelo seja **succeeded**.

### Testar o modelo personalizado com o Python SDK

1. No terminal do VS Code, navegue até a pasta Python do modelo personalizado:

    ```
    cd Python
    ```

1. Instale os pacotes necessários (crie um novo ambiente virtual ou reutilize o existente):

    ```
    python -m venv labenv
    labenv\Scripts\activate
    pip install -r requirements.txt
    ```

1. No VS Code, abra o arquivo **.env** em **Labfiles/03-document-intelligence/custom/Python**.

1. Atualize o arquivo com os seguintes valores:
    - Seu **endpoint** do Document Intelligence
    - Sua **key** do Document Intelligence
    - O **Model ID** que você especificou ao treinar seu modelo
1. Salve o arquivo (**CTRL+S**).
1. No VS Code, abra o arquivo **test-model.py**.

1. Revise o código, que usa o SDK [azure-ai-documentintelligence](https://learn.microsoft.com/python/api/overview/azure/ai-documentintelligence-readme?view=azure-python). Observe que ele faz referência a uma imagem de teste hospedada no repositório do GitHub. O código cria um `DocumentIntelligenceClient`, envia a imagem para análise usando seu modelo personalizado e imprime os campos extraídos.
1. No terminal do VS Code, execute o programa:

    ```
    python test-model.py
    ```

1. Revise a saída. O programa deve exibir os nomes e valores dos campos extraídos do formulário de teste, como `Merchant`, `CompanyPhoneNumber` e outros campos que você definiu durante o treinamento.

    ![An image of an invoice used in this project.](./media/Form_1.jpg)

## Limpar

Se você concluiu o uso do serviço Document Intelligence, deve excluir os recursos que criou neste exercício para evitar custos desnecessários no Azure.

1. No [Azure portal](https://portal.azure.com), exclua o grupo de recursos que você criou para este exercício.
