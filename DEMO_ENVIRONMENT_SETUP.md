# Azure Boot Camp Development Environment Setup

In order to take part in this workshop, you'll need to download and install the following tools.

1. **New Azure Subscription**
    1. Use or create a personal Microsoft Account (MSA) at https://signup.live.com
    2. Create a free Azure subscription at https://azure.microsoft.com/free/
    3. Create a free Azure DevOps organization  associated with your MSA

2. Use Visual Studio 2019 (Any Edition) or Visual Studio Code
   - Visual Studio 2019 (Any Edition) - <https://visualstudio.microsoft.com/vs/>
   - Install Visual Studio Code (https://code.visualstudio.com/)
4. The Chocolatey Package Manager for Windows. Open Windows PowerShell (as an Administrator) or PowerShell Core (V6 or V7) and paste the following command:

    ``` PowerShell
    Set-ExecutionPolicy Bypass -Scope Process -Force; `
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```

5. Install node.js (after running the command, restart your PowerShell session)

    ```PowerShell
    choco install nodejs-lts
    ```
 
6. Install the latest version of the .NET Core 5.0 SDK from <https://dotnet.microsoft.com/download/dotnet/5.0>
7. Install the Azure Functions Core Tools

    ```PowerShell
    npm install -g azure-functions-core-tools@3
    ```

8. Install Git for Windows

    ```PowerShell
    choco install git
    ```

9. Install Azure Storage Explorer

    ```PowerShell
    choco install microsoftazurestorageexplorer
    ```

10. Install Postman from <https://www.postman.com/downloads/>
    
11. Install the Azure Command Line Tools (Azure CLI) - <https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest>

