---
lab:
    title: 'Integrate Azure OpenAI into your app'
---

# Integrate Azure OpenAI into your app

With the Azure OpenAI Service, developers can create chatbots, language models, and other applications that excel at understanding natural human language. The Azure OpenAI provides access to pre-trained AI models, as well as a suite of APIs and tools for customizing and fine-tuning these models to meet the specific requirements of your application. In this exercise, you'll learn how to deploy a model in Azure OpenAI and use it in your own application to summarize text.

This exercise will take approximately **30** minutes.

## Before you start

You will need an Azure subscription that has been approved for access to the Azure OpenAI service.

- To sign up for a free Azure subscription, visit [https://azure.microsoft.com/free](https://azure.microsoft.com/free).
- To request access to the Azure OpenAI service, visit [https://aka.ms/oaiapply](https://aka.ms/oaiapply).

## Provision an Azure OpenAI resource

Before you can use Azure OpenAI models, you must provision an Azure OpenAI resource in your Azure subscription.

1. Sign into the [Azure portal](https://portal.azure.com).
2. Create an **Azure OpenAI** resource with the following settings:
    - **Subscription**: An Azure subscription that has been approved for access to the Azure OpenAI service.
    - **Resource group**: Choose an existing resource group, or create a new one with a name of your choice.
    - **Region**: Choose any available region.
    - **Name**: A unique name of your choice.
    - **Pricing tier**: Standard S0
3. Wait for deployment to complete. Then go to the deployed Azure OpenAI resource in the Azure portal.
4. Navigate to **Keys and Endpoint** page, and save those to a text file to use later.

## Deploy a model

To use the Azure OpenAI API, you must first deploy a model to use through the **Azure OpenAI Studio**. Once deployed, we will reference that model in our app.

1. On the **Overview** page for your Azure OpenAI resource, use the **Explore** button to open Azure OpenAI Studio in a new browser tab.
2. In Azure OpenAI Studio, on the **Deployments** page, view your existing model deployments. If you don't already have one, create a new deployment of the **gpt-35-turbo-16k** model with the following settings:
    - **Model**: gpt-35-turbo-16k
    - **Model version**: Auto-update to default
    - **Deployment name**: *A unique name of your choice*
    - **Advanced options**
        - **Content filter**: Default
        - **Tokens per minute rate limit**: 5K\*
        - **Enable dynamic quota**: Enabled

    > \* A rate limit of 5,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.

> **Note**: In some regions, the new model deployment interface doesn't show the **Model version** option. In this case, don't worry and continue without setting the option

## Set up an application in Cloud Shell

To show how to integrate with an Azure OpenAI model, we'll use a short command-line application that runs in Cloud Shell on Azure. Open up a new browser tab to work with Cloud Shell.

1. In the [Azure portal](https://portal.azure.com?azure-portal=true), select the **[>_]** (*Cloud Shell*) button at the top of the page to the right of the search box. A Cloud Shell pane will open at the bottom of the portal.

    ![Screenshot of starting Cloud Shell by clicking on the icon to the right of the top search box.](../media/cloudshell-launch-portal.png#lightbox)

2. The first time you open the Cloud Shell, you may be prompted to choose the type of shell you want to use (*Bash* or *PowerShell*). Select **Bash**. If you don't see this option, skip the step.  

3. If you're prompted to create storage for your Cloud Shell, select **Show advanced settings** and select the following settings:
    - **Subscription**: Your subscription
    - **Cloud shell regions**: Choose any available region
    - **Show VNET isolation setings** Unselected
    - **Resource group**: Use the existing resource group where you provisioned your Azure OpenAI resource
    - **Storage account**: Create a new storage account with a unique name
    - **File share**: Create a new file share with a unique name

    Then wait a minute or so for the storage to be created.

    > **Note**: If you already have a cloud shell set up in your Azure subscription, you may need to use the **Reset user settings** option in the ⚙️ menu to ensure the latest versions of Python and the .NET Framework are installed.

4. Make sure the type of shell indicated on the top left of the Cloud Shell pane is *Bash*. If it's *PowerShell*, switch to *Bash* by using the drop-down menu.

5. Once the terminal starts, enter the following command to download the sample application and save it to a folder called `azure-openai`.

    ```bash
   rm -r azure-openai -f
   git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```
  
6. The files are downloaded to a folder named **azure-openai**. Navigate to the lab files for this exercise using the following command.

    ```bash
   cd azure-openai/Labfiles/02-nlp-azure-openai
    ```

7. Open the built-in code editor by running the following command:

    ```bash
    code .
    ```

8. In the code editor, expand the **text-files** folder and select **sample-text.txt** to see the text that you will use your model to summarize.

    > **Tip**: Consult the [documentation for the Azure cloud shell code editor](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor) for more details about using it to work with files in the Azure cloud shell environment.

## Configure your application

For this exercise, you'll complete some key parts of the application to enable using your Azure OpenAI resource. Applications for both C# and Python have been provided. Both apps feature the same functionality.

1. In the code editor, expand the **CSharp** or **Python** folder, depending on your language preference.

2. Open the configuration file for your language

    - C#: `appsettings.json`
    - Python: `.env`
    
3. Update the configuration values to include the **endpoint** and **key** from the Azure OpenAI resource you created, as well as the model name that you deployed. Save the file.

4. In the console pane, enter the following commands to navigate to the folder for your preferred language and install the necessary packages

    **C#**

    ```bash
   cd CSharp
   dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.9
    ```

    **Python**

    ```bash
    cd Python
    pip install python-dotenv
    pip install openai==1.2.0
    ```

5. Navigate to your preferred language folder, select the code file, and add the necessary libraries.

    **C#**

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

6. Open up the application code for your language and add the necessary code for building the request, which specifies the various parameters for your model such as `prompt` and `temperature`.

    **C#**

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    
    // Build completion options object
    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatMessage(ChatRole.System, "You are a helpful assistant."),
            new ChatMessage(ChatRole.User, "Summarize the following text in 20 words or less:\n" + text),
        },
        MaxTokens = 120,
        Temperature = 0.7f,
        DeploymentName = oaiModelName
    };
    
    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);
    string completion = response.Choices[0].Message.Content;
    
    Console.WriteLine("Summary: " + completion + "\n");
    ```

    **Python**

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_model,
        temperature=0.7,
        max_tokens=120,
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Summarize the following text in 20 words or less:\n" + text}
        ]
    )
    
    print("Summary: " + response.choices[0].message.content + "\n")
    ```

## Run your application

Now that your app has been configured, run it to send your request to your model and observe the response.

1. In the Cloud Shell bash terminal, navigate to the folder for your preferred language.
1. Run the application.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

1. Observe the summarization of the sample text file.
1. Navigate to your code file for your preferred language, and change the *temperature* value to `1`. Save the file.
1. Run the application again, and observe the output.

Increasing the temperature often causes the summary to vary, even when provided the same text, due to the increased randomness. You can run it several times to see how the output may change. Try using different values for your temperature with the same input.

## Clean up

When you're done with your Azure OpenAI resource, remember to delete the deployment or the entire resource in the [Azure portal](https://portal.azure.com?azure-portal=true).
