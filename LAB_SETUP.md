# Azure Boot Camp Development Environment Setup

In order to take part in this workshop, you'll need to download and install the following tools.

1. **Azure Subscription**
   - You will need an Azure subscription with contributor role
   - You will need access to Azure Pipelines to be able to create new pipelines

2. **Use Visual Studio 2019 (Any Edition) or Visual Studio Code**
   - [**Visual Studio 2019**](https://visualstudio.microsoft.com/vs/) (Any Edition)
   - [**Visual Studio Code**](https://code.visualstudio.com/)
3. The Chocolatey Package Manager for Windows. 
   - Open Windows PowerShell (as an Administrator) or PowerShell Core (V6 or V7) and paste the following command:

        ``` PowerShell
        Set-ExecutionPolicy Bypass -Scope Process -Force; `
        [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
        iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
        ```

4. **Install node.js** (after running the command, restart your PowerShell session)

    ```PowerShell
    choco install nodejs-lts
    ```

5. Install the [**.NET Core 5.0 SDK**](https://dotnet.microsoft.com/download/dotnet/5.0)
6. Install the Azure Functions Core Tools

    ```PowerShell
    npm install -g azure-functions-core-tools@3
    ```

7. Install Git for Windows

    ```PowerShell
    choco install git
    ```

8. Install Azure Storage Explorer

    ```PowerShell
    choco install microsoftazurestorageexplorer
    ```

9.  Install [**Postman**](https://www.postman.com/downloads/)

10. Install the [**Azure Command Line Tools (Azure CLI)**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

