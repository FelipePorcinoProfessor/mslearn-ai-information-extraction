---
lab:
  title: Extrair informações de conteúdo multimodal
  description: Usar Azure Content Understanding para extrair insights de documentos, imagens, gravações de áudio e vídeos.
  duration: 40
  level: 200
  islab: true
  status: 'released'
  primarytopics:
    - Azure
    - Azure Content Understanding
---

# Extrair informações de conteúdo multimodal

Neste exercício, você usará Azure Content Understanding para extrair informações de diversos tipos de conteúdo; incluindo uma fatura, uma imagem de um slide contendo gráficos, uma gravação de áudio de uma mensagem de voz e uma gravação de vídeo de uma chamada de conferência.

Este exercício leva aproximadamente **40** minutos.

## Criar um recurso e projeto do Microsoft Foundry

Os recursos que vamos usar neste exercício exigem um recurso e projeto do Microsoft Foundry.

1. Em um navegador da Web, abra o [portal do Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre com suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que forem abertos na primeira vez em que você entrar.
1. Verifique se a alternância **New Foundry** está ativada para que você esteja usando o **Foundry (new)**.
1. Se você não for solicitado a criar um novo projeto automaticamente, selecione o nome do projeto no canto superior esquerdo e selecione **Create new project**.
1. Dê um nome ao seu projeto e expanda **Advanced options** para especificar as seguintes configurações:
    - **Project name**: *Forneça um nome válido para seu projeto*
    - **Foundry resource**: *Use o padrão*
    - **Region**: Escolha uma das seguintes regiões com suporte:\*
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
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    

    > \*Azure Content Understanding está disponível em regiões selecionadas. Consulte a [documentação de suporte por região](https://learn.microsoft.com/azure/ai-services/content-understanding/language-region-support) para ver a disponibilidade mais recente.

1. Selecione **Create** e aguarde a criação do projeto.

## Baixar conteúdo

O conteúdo que você vai analisar está em um arquivo .zip. Baixe-o e extraia-o em uma pasta local.

1. Em uma nova guia do navegador, baixe o [content.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/content/content.zip) em `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/content/content.zip` e salve-o em uma pasta local.
1. Extraia o arquivo *content.zip* baixado e veja os arquivos que ele contém. Você usará esses arquivos para explorar os analisadores do Content Understanding neste exercício.

> Observação: Se você estiver interessado apenas em explorar a análise de uma modalidade específica (documentos, imagens, vídeo ou áudio), poderá pular para a tarefa relevante abaixo. Para a melhor experiência, percorra cada tarefa para aprender a extrair informações de diferentes tipos de conteúdo.

## Experimentar analisadores predefinidos no Microsoft Foundry

Azure Content Understanding inclui os analisadores predefinidos **Read** e **Layout** que podem extrair texto e elementos estruturais de documentos sem exigir qualquer configuração personalizada. Esses analisadores predefinidos estão disponíveis diretamente no portal Foundry (new) como modelos de AI Services.

### Usar o analisador Layout no playground

1. No [portal do Microsoft Foundry](https://ai.azure.com), verifique se a alternância **New Foundry** está ativada.
1. Selecione **Build** no menu superior direito e, em seguida, selecione **Deployments** no painel esquerdo.
1. Selecione a guia **AI Services** para ver os modelos predefinidos fornecidos pelo Foundry Tools.
1. Localize e selecione **Azure Content Understanding - Layout**.

    Isso abre a página do playground do analisador Layout, onde você pode testar o modelo de layout em dados de exemplo ou em seus próprios arquivos.

1. No playground, use a opção para fazer upload dos seus próprios dados e carregue o arquivo **invoice-1234.pdf** da pasta onde você extraiu os arquivos de conteúdo. Este arquivo contém a seguinte fatura:

    ![Imagem de uma fatura número 1234.](./media/invoice-1234.png)

1. Execute o analisador e aguarde a conclusão da análise.
1. Revise os resultados. Você pode ver o conteúdo extraído como saída formatada ou como dados JSON sem formatação. Observe que o analisador Layout extrai texto, tabelas e elementos estruturais, como parágrafos e seções do documento.

    > Observação: Os analisadores predefinidos **Read** e **Layout** extraem conteúdo de documentos sem exigir um modelo de IA generativa. **Read** extrai elementos de texto (palavras, parágrafos, fórmulas e códigos de barras), enquanto **Layout** extrai adicionalmente tabelas, figuras, estrutura do documento, hiperlinks e anotações. Esses analisadores são úteis para extração de conteúdo de propósito geral, mas não extraem campos personalizados específicos, como valores de fatura ou nomes de fornecedores.

1. Opcionalmente, volte para a guia **AI Services** e experimente **Azure Content Understanding - Read** com o mesmo arquivo para comparar os resultados. Observe que Read extrai texto sem análise de layout.

## Configurar o Content Understanding Studio para analisadores personalizados

Para extrair campos específicos do seu conteúdo (como valores de fatura, nomes de quem ligou ou participantes de reunião), você precisa criar analisadores personalizados. Os analisadores personalizados são criados no **Content Understanding Studio**, uma ferramenta separada baseada na Web para criar e testar analisadores com esquemas personalizados.

1. Em uma nova guia do navegador, abra o [Content Understanding Studio](https://contentunderstanding.ai.azure.com) em `https://contentunderstanding.ai.azure.com`.
1. Se solicitado, entre com as mesmas credenciais do Azure que você usou para o portal Foundry.
1. Na página **Settings** (ou se redirecionado para configurar seu recurso), selecione o botão **+ Add resource**.
1. Selecione o recurso do Foundry que você criou anteriormente e selecione **Next** > **Save**.

    > Dica: Verifique se a caixa de seleção **Enable autodeployment for required models if no defaults are available** está marcada. Isso garante que seu recurso esteja configurado com os modelos `GPT-4.1`, `GPT-4.1-mini` e `text-embedding-3-large` necessários para analisadores personalizados.

1. Depois que seu recurso estiver conectado, você estará pronto para criar analisadores personalizados. Selecione **Content Understanding** na navegação superior para ir para a página inicial.

## Extrair informações de documentos de fatura

Você criará um analisador personalizado do Azure Content Understanding que pode extrair campos específicos de faturas. Você criará um projeto no Content Understanding Studio, definirá um esquema com base em uma fatura de exemplo e, em seguida, criará um analisador reutilizável.

### Criar uma conta de armazenamento

O Content Understanding Studio requer uma conta do Azure Blob Storage para armazenar os dados usados na criação de analisadores personalizados. Você precisa criar uma na mesma grupo de recursos do seu recurso Foundry.

1. Em uma nova guia do navegador, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre com suas credenciais do Azure.
1. Selecione **+ Create a resource**, pesquise `Storage account` e crie um novo recurso **Storage account** com as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *O mesmo grupo de recursos do seu recurso Foundry*
    - **Storage account name**: *Insira um nome globalmente exclusivo*
    - **Region**: *A mesma região do seu recurso Foundry*
    - **Preferred storage type**: Azure Blob Storage ou Azure Data Lake Storage Gen 2
    - **Performance**: Standard
    - **Redundancy**: Locally-redundant storage (LRS)
1. Selecione **Review + create** e depois **Create**. Aguarde a conclusão da implantação.

### Definir um esquema para análise de faturas

1. No Content Understanding Studio, selecione o botão **Get started** na seção de projetos personalizados e selecione **Create**.
1. Selecione **Extract content and fields with a custom schema** e crie um projeto com as seguintes configurações:
    - **Project name**: `Invoice analysis`
    - **Description**: `Extract data from an invoice`
    - **Advanced settings**
        - **Connected resource**: *Confirme se o seu recurso Foundry está selecionado*
        - **Connect storage account**: *Selecione a conta de armazenamento que você acabou de criar*
        - **Blob container**: *Crie um novo contêiner chamado* `content-understanding`
1. Aguarde a criação do projeto.

    > Dica: Se ocorrer um erro de acesso ao armazenamento, aguarde um minuto e tente novamente. As permissões para um novo recurso podem levar alguns minutos para se propagar.

1. Faça upload do arquivo **invoice-1234.pdf** da pasta onde você extraiu os arquivos de conteúdo.

    O Content Understanding classifica seus dados e recomenda modelos de analisador com base no conteúdo carregado.

1. Na janela **Choose a template**, selecione o modelo **Invoice** e selecione **Save**.

    O modelo *Invoice* inclui campos comuns encontrados em faturas. Você pode usar o editor de esquema para excluir qualquer um dos campos sugeridos que não precisar e adicionar quaisquer campos personalizados de que precisar.

1. Na lista de campos sugeridos, selecione **BillingAddress**. Esse campo não é necessário para o formato de fatura que você carregou, portanto, use o ícone **Delete field** (**&#128465;**) que aparece ao final na linha do campo selecionado para excluí-lo.
1. Na barra superior da guia do esquema, selecione **Suggest**. Isso analisará a fatura de exemplo e sugerirá quais campos devem fazer parte do seu esquema. Expanda o campo **Items** para ver quais subcampos são sugeridos. Adicionar esses campos substituirá seu esquema existente, portanto, tenha cuidado em seus projetos se você tiver editado um esquema. Selecione **Save**.
1. Use o botão **+ Add new field** para adicionar o seguinte campo, selecionando **Save** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `TotalQuantity` | `Total number of items on the invoice` | String | Auto |

1. Verifique se seu esquema concluído se parece com este e selecione **Save**.

    ![Captura de tela do esquema do analisador de fatura no Content Understanding Studio mostrando campos como VendorName, InvoiceDate, SubTotal, Items e TotalQuantity.](./media/invoice-schema.png)

1. Selecione a guia **Test**, depois selecione **Run analysis** para testar seu esquema. Aguarde a conclusão da análise.

1. Revise os resultados da análise, que devem ser semelhantes a estes:

    ![Captura de tela dos resultados do teste de análise de fatura no Content Understanding Studio mostrando valores de campos extraídos da fatura de exemplo.](./media/invoice-analysis.png)

1. Veja os detalhes dos campos que foram identificados no painel **Fields**.

### Criar e testar um analisador para faturas

Agora que você definiu um esquema para extrair campos de faturas, pode criar um analisador para usar com documentos semelhantes.

1. Selecione o botão **Build analyzer** na parte superior e crie um novo analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Name**: `invoiceanalyzer`
    - **Description**: `Invoice analyzer`
1. Quando o analisador for criado, selecione **Jump to analyzer list** para ver todos os analisadores criados e, em seguida, selecione o link **invoiceanalyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **invoiceanalyzer**, selecione a guia **Test**.
1. Carregue **invoice-1235.pdf** da pasta onde você extraiu os arquivos de conteúdo e execute a análise para extrair dados de campos da fatura.

    A fatura que está sendo analisada é assim:

    ![Imagem de uma fatura número 1235.](./media/invoice-1235.png)

1. Revise o painel **Fields** e verifique se o analisador extraiu os campos corretos da fatura de teste.
1. Revise o painel **Results** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Feche a página **invoiceanalyzer** para voltar à lista de analisadores.

## Extrair informações de uma imagem de slide

Você criará um analisador personalizado do Azure Content Understanding que pode extrair informações de um slide contendo gráficos.

### Definir um esquema para análise de imagem

1. Na guia **Project list**, selecione **Create** e selecione **Extract content and fields with a custom schema**; em seguida, crie um projeto com as seguintes configurações:
    - **Project name**: `Slide analysis`
    - **Description**: `Extract data from an image of a slide`
    - **Advanced settings**: *Verifique se as configurações são as mesmas do último projeto*
1. Aguarde a criação do projeto.

1. Carregue o arquivo **slide-1.jpg** da pasta onde você extraiu os arquivos de conteúdo. Em seguida, selecione o modelo **Image analysis** e selecione **Save**.

    O modelo *Image analysis* não inclui campos predefinidos. Você deve definir campos para descrever as informações que deseja extrair.

1. Use o botão **+ Add new field** para adicionar os seguintes campos, selecionando **Save changes** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Title` | `Slide title` | String | Generate |
    | `Summary` | `Summary of the slide` | String | Generate |
    | `Charts` | `Number of charts on the slide` | Integer | Generate |

1. Use o botão **+ Add new field** para adicionar um novo campo chamado `QuarterlyRevenue` com a descrição `Revenue per quarter` com o tipo de valor **List of objects**. Em seguida, selecione o ícone de tabela ao lado da lista suspensa de tipo de valor. Na nova página de subcampos da tabela que se abre, adicione os seguintes subcampos:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Quarter` | `Which quarter?` | String | Generate |
    | `Revenue` | `Revenue for the quarter` | Number | Generate |

1. Selecione **Back** para retornar ao nível superior do seu esquema e use o botão **+ Add new field** para adicionar um novo campo chamado `ProductCategories` com a descrição `Product categories` com o tipo de valor **List of objects**. Em seguida, selecione o ícone de tabela ao lado do tipo de valor para abrir uma nova página para os subcampos da tabela e adicione os seguintes subcampos:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `ProductCategory` | `Product category name` | String | Generate |
    | `RevenuePercentage` | `Percentage of revenue` | Number | Generate |

1. Selecione **Back** para retornar ao nível superior do seu esquema e verifique se ele está como este. Em seguida, selecione **Save**.

    ![Captura de tela do esquema do analisador de imagem no Content Understanding Studio mostrando campos para Title, Summary, Charts, QuarterlyRevenue e ProductCategories.](./media/slide-schema.png)

1. Selecione a guia **Test**, depois **Run analysis** e aguarde a conclusão da análise.
1. Revise os resultados da análise, que devem se parecer com isto:

    ![Captura de tela dos resultados do teste de análise de imagem no Content Understanding Studio mostrando campos extraídos do slide, incluindo dados de receita e categorias de produtos.](./media/slide-analysis.png)

1. Veja os detalhes dos campos que foram identificados no painel **Fields**, expandindo os campos **QuarterlyRevenue** e **ProductCategories** para ver os valores dos subcampos.

### Criar e testar um analisador

Agora que você definiu um esquema para extrair campos de slides, pode criar um analisador para usar com imagens de slides semelhantes.

1. Selecione o botão **Build analyzer** na parte superior e crie um novo analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Name**: `slideanalyzer`
    - **Description**: `Slide image analyzer`
1. Quando o analisador for criado, selecione **Jump to analyzer list** e, em seguida, selecione o link **slideanalyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **slideanalyzer**, selecione a guia **Test**.
1. Use o botão **+ Upload test files** para carregar **slide-2.jpg** da pasta onde você extraiu os arquivos de conteúdo e execute a análise para extrair dados de campo da imagem.
1. Revise o painel **Fields** e verifique se o analisador extraiu os campos corretos da imagem do slide.

    > Observação: O slide 2 não inclui um detalhamento por categoria de produto, portanto, os dados de receita por categoria de produto não são encontrados.

1. Revise o painel **Results** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Feche a página **slideanalyzer**.

## Extrair informações de uma gravação de áudio de correio de voz

Você criará um analisador personalizado do Azure Content Understanding que pode extrair informações de uma gravação de áudio de uma mensagem de voz.

### Definir um esquema para análise de áudio

1. Na guia **Project list**, selecione **Create** e selecione **Extract content and fields with a custom schema**; em seguida, crie um projeto com as seguintes configurações:
    - **Project name**: `Voicemail analysis`
    - **Description**: `Extract data from a voicemail recording`
    - **Advanced settings**: *Verifique se as configurações são as mesmas do último projeto*
1. Aguarde a criação do projeto.

1. Carregue o arquivo **call-1.mp3** da pasta onde você extraiu os arquivos de conteúdo. Em seguida, selecione o modelo **Audio analysis** e selecione **Save**.
1. No painel **Content** à direita, selecione **Get transcription preview** para ver uma transcrição da mensagem gravada.

    O modelo *Audio analysis* não inclui campos predefinidos. Você deve definir campos para descrever as informações que deseja extrair.

1. Use o botão **+ Add new field** para adicionar os seguintes campos, selecionando **Save** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Caller` | `Person who left the message` | String | Generate |
    | `Summary` | `Summary of the message` | String | Generate |
    | `Actions` | `Requested actions` | String | Generate |
    | `CallbackNumber` | `Telephone number to return the call` | String | Generate |
    | `AlternativeContacts` | `Alternative contact details` | List of Strings | Generate |

1. Selecione **Run analysis** e aguarde a conclusão da análise.

    A análise de áudio pode levar algum tempo. Enquanto aguarda, você pode reproduzir o arquivo de áudio abaixo:

    <video controls src="./media/call-1.mp4" title="Call 1" width="300">
        <track src="./media/call-1.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este áudio foi gerado usando IA.

1. Revise os resultados da análise e veja os detalhes dos campos que foram identificados no painel **Fields**, expandindo o campo **AlternativeContacts** para ver os valores listados.

### Criar e testar um analisador

Agora que você definiu um esquema para extrair campos de mensagens de voz, pode criar um analisador para usar com gravações de áudio semelhantes.

1. Selecione o botão **Build analyzer** na parte superior e crie um novo analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Name**: `voicemailanalyzer`
    - **Description**: `Voicemail audio analyzer`
1. Quando o analisador for criado, selecione **Jump to analyzer list** e, em seguida, selecione o link **voicemailanalyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **voicemailanalyzer**, selecione a guia **Test**.
1. Use o botão **+ Upload test files** para carregar **call-2.mp3** da pasta onde você extraiu os arquivos de conteúdo e execute a análise para extrair dados de campo do arquivo de áudio.

    A análise de áudio pode levar algum tempo. Enquanto aguarda, você pode reproduzir o arquivo de áudio abaixo:

    <video controls src="./media/call-2.mp4" title="Call 2" width="300">
        <track src="./media/call-2.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este áudio foi gerado usando IA.

1. Revise o painel **Fields** e verifique se o analisador extraiu os campos corretos da mensagem de voz.
1. Revise o painel **Results** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Feche a página **voicemail-analyzer**.

## Extrair informações de uma gravação de videoconferência

Você criará um analisador personalizado do Azure Content Understanding que pode extrair informações de uma gravação de vídeo de uma chamada de conferência.

### Definir um esquema para análise de vídeo

1. No Content Understanding Studio, selecione **Create project** na página inicial (ou use a navegação para voltar primeiro à página inicial).
1. Selecione **Extract content and fields with a custom schema** e crie um projeto com as seguintes configurações:
    - **Project name**: `Conference call video analysis`
    - **Description**: `Extract data from a video conference recording`
1. Aguarde a criação do projeto.

1. Carregue o arquivo **meeting-1.mp4** da pasta onde você extraiu os arquivos de conteúdo. Em seguida, selecione o modelo **Video analysis** e selecione **Create**.
1. No painel **Content** à direita, selecione **Get transcription preview** para ver uma transcrição da reunião gravada.

    O modelo *Video analysis* extrai dados para cada segmento. Ele não inclui campos predefinidos. Você deve definir campos para descrever as informações que deseja extrair.

1. Use o botão **+ Add new field** para adicionar os seguintes campos, selecionando **Save** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Summary` | `Summary of the discussion` | String | Generate |
    | `Participants` | `Count of meeting participants` | Integer | Generate |
    | `ParticipantNames` | `Names of meeting participants` | List of Strings | Generate |
    | `SharedSlides` | `Descriptions of any PowerPoint slides presented` | List of Strings | Generate |
    | `AssignedActions` | `Tasks assigned to participants` | List of Objects | Generate |

1. Ao inserir o campo **AssignedActions**, na tabela de subcampos, crie os seguintes subcampos:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Task` | `Description of the task` | String | Generate |
    | `AssignedTo` | `Who the task is assigned to` | String | Generate |

1. Selecione **Back** para retornar ao nível superior do seu esquema e verifique se ele está como este. Em seguida, selecione **Save**.

1. Selecione **Run analysis** e aguarde a conclusão da análise.

    A análise de vídeo pode levar algum tempo. Enquanto aguarda, você pode assistir ao vídeo abaixo:

    <video controls src="./media/meeting-1.mp4" title="Meeting 1" width="480">
        <track src="./media/meeting-1.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este vídeo foi gerado usando IA.

1. Quando a análise for concluída, revise os resultados.

1. No painel **Fields**, veja os dados extraídos.

### Criar e testar um analisador

Agora que você definiu um esquema para extrair campos de gravações de chamadas de conferência, pode criar um analisador para usar com vídeos semelhantes.

1. Selecione o botão **Build analyzer** na parte superior e crie um novo analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Name**: `meetinganalyzer`
    - **Description**: `Meeting video analyzer`
1. Aguarde o novo analisador ficar pronto (use o botão **Refresh** para verificar).
1. Quando o analisador for criado, selecione **Jump to analyzer list** e, em seguida, selecione o link **meetinganalyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **meetinganalyzer**, selecione a guia **Test**.
1. Use o botão **+ Upload test files** para carregar **meeting-2.mp4** da pasta onde você extraiu os arquivos de conteúdo e execute a análise para extrair dados de campo do arquivo de vídeo.

    A análise de vídeo pode levar algum tempo. Enquanto aguarda, você pode assistir ao vídeo abaixo:

    <video controls src="./media/meeting-2.mp4" title="Meeting 2" width="480">
        <track src="./media/meeting-2.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este vídeo foi gerado usando IA.

1. Revise o painel **Fields** e veja os campos que o analisador extraiu para cada tomada no vídeo da chamada de conferência.
1. Revise o painel **Results** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Feche a página **meetinganalyzer**.

## Limpeza

Se você terminou de trabalhar com o serviço Content Understanding, deve excluir os recursos que criou neste exercício para evitar custos desnecessários do Azure.

1. No [portal do Azure](https://portal.azure.com), exclua o grupo de recursos que você criou para este exercício.
