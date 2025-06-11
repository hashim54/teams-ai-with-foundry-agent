# Overview of the AI Agent template

This app template is built on top of [Teams AI library](https://aka.ms/teams-ai-library).
It showcases how to build an AI agent in Teams capable of chatting with users and helping users accomplish a specific task using natural language right in the Teams conversations, such as managing tasks.

## Azure AI Foundry Integration

This template integrates with **Azure AI Foundry** to leverage enterprise-grade AI capabilities while interfacing the agent in Teams:

### Benefits of Azure AI Foundry
- **Model Variety**: Access to multiple model providers in one platform
- **Integration with third-part tools**: Integration with MCP and A2A
- **Enterprise Security**: Role-based access control and data governance
- **Scalability**: Auto-scaling based on demand
- **Cost Management**: Usage tracking and budget controls

- **Model Management**: Deploy and manage various AI models including GPT-4, Llama, and custom models
- **Content Safety**: Built-in content filtering and safety guardrails
- **Prompt Flow**: Visual prompt engineering and testing capabilities
- **Model Monitoring**: Track usage, performance, and costs
- **Fine-tuning**: Customize models with your organization's data
- **Multi-modal Support**: Handle text, images, and other content types

## Get started with the template

> **Prerequisites**
>
> To run the template in your local dev machine, you will need:
>
> - [Python](https://www.python.org/), version 3.8 to 3.11.
> - [Python extension](https://code.visualstudio.com/docs/languages/python), version v2024.0.1 or higher.
> - [Microsoft 365 Agents Toolkit Visual Studio Code Extension](https://aka.ms/teams-toolkit) latest version or [Microsoft 365 Agents Toolkit CLI](https://aka.ms/teams-toolkit-cli).
> - A Project in [Azure AI Foundry]
> - A [Microsoft 365 account for development](https://docs.microsoft.com/microsoftteams/platform/toolkit/accounts).

### Configuration For Local Debugging

1. Open the command box and enter `Python: Create Environment` to create and activate your desired virtual environment. Remember to select `src/requirements.txt` as dependencies to install when creating the virtual environment.

**Configure Environment Variables**

Create or update your environment files with the appropriate configuration:

1. In file *.env*, fill in your Azure AI Foundry project and agent details
FOUNDRY_AGENT_ID=your_ai_foundry_agent_id
FOUNDRY_PROJECT_ENDPOINT=your_ai_foundry_project_endpoint
FOUNDRY_PROJECT_KEY=your_ai_foundry_project_key

Note: Leave all other environment variables in *.env* empty for local testing. 

### Conversation with agent
1. Select the Microsoft 365 Agents Toolkit icon on the left in the VS Code toolbar.
1. In the Account section, sign in with your [Microsoft 365 account](https://docs.microsoft.com/microsoftteams/platform/toolkit/accounts) if you haven't already.
1. Press F5 to start debugging which launches your app in Teams using a web browser. Select `Debug in Teams (Edge)` or `Debug in Teams (Chrome)`.
1. When Teams launches in the browser, select the Add button in the dialog to install your app to Teams.
1. You will receive a welcome message from the agent, or send any message to get a response.

**Congratulations**! You are running an application that can now interact with users in Teams:

> For local debugging using Microsoft 365 Agents Toolkit CLI, you need to do some extra steps described in [Set up your Microsoft 365 Agents Toolkit CLI for local debugging](https://aka.ms/teamsfx-cli-debugging).

![ai agent](https://github.com/OfficeDev/TeamsFx/assets/109947924/775a0fde-f2ba-4198-a94d-a43c598d6e9b)

### Configuration For PROD Deployment

## Step1: Configure Environment Variables

For production deployment, configure the following variables in your `.env` file:

Bot Framework Registration (Required)
APP_ID=your_microsoft_app_id_from_azure_bot_registration
APP_PASSWORD=your_microsoft_app_password_from_azure_bot_registration
APP_TYPE=MultiTenant
APP_TENANTID=  # Leave empty for multi-tenant apps

Bot Configuration (Required)
BOT_ID=your_microsoft_app_id_from_azure_bot_registration  # Same as APP_ID
BOT_PASSWORD=your_microsoft_app_password_from_azure_bot_registration  # Same as APP_PASSWORD
```

Azure AI Foundry Configuration:
```env
Azure AI Foundry Agent Configuration (Required)
FOUNDRY_AGENT_ID=your_ai_foundry_agent_id
FOUNDRY_PROJECT_ENDPOINT=https://your-project-name.cognitiveservices.azure.com/
FOUNDRY_PROJECT_KEY=your_ai_foundry_project_api_key
```
**Teams Framework Configuration:**
```env
# Teams Notification Store (Optional)
TEAMSFX_NOTIFICATION_STORE_FILENAME=notification.db

Step2: Azure App Service Deployment

**Prerequisites for Deployment:**
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) installed
- Azure subscription with appropriate permissions
- Resource group for your application

**2.1 Create Azure Resources**

```bash
# Login to Azure
az login

# Set your subscription (if you have multiple)
az account set --subscription "your-subscription-id"

# Create resource group
az group create --name rg-teams-ai-agent-prod --location eastus

# Create App Service plan (Production tier)
az appservice plan create \
  --name asp-teams-ai-agent-prod \
  --resource-group rg-teams-ai-agent-prod \
  --sku P1V2 \
  --is-linux

# Create Web App with Python runtime
az webapp create \
  --name your-unique-app-name \
  --resource-group rg-teams-ai-agent-prod \
  --plan asp-teams-ai-agent-prod \
  --runtime "PYTHON:3.9"
```

**2.2 Configure Azure App Service Settings**

```bash
# Configure startup command for the Python app
az webapp config set \
  --name your-unique-app-name \
  --resource-group rg-teams-ai-agent-prod \
  --startup-file "python -m aiohttp.web -H 0.0.0.0 -P 8000 src.app:init"

# Set environment variables in App Service
az webapp config appsettings set \
  --name your-unique-app-name \
  --resource-group rg-teams-ai-agent-prod \
  --settings \
  APP_ID="your_app_id" \
  APP_PASSWORD="your_app_password" \
  APP_TYPE="MultiTenant" \
  APP_TENANTID="" \
  BOT_ID="your_app_id" \
  BOT_PASSWORD="your_app_password" \
  FOUNDRY_AGENT_ID="your_foundry_agent_id" \
  FOUNDRY_PROJECT_ENDPOINT="your_foundry_endpoint" \
  FOUNDRY_PROJECT_KEY="your_foundry_key" \
  TEAMSFX_NOTIFICATION_STORE_FILENAME="notification.db" \
  WEBSITES_PORT="8000"
```

**2.3 Deploy Application Code**

**ZIP Deployment**
```bash
# Navigate to your project root
cd c:\Users\mumann\AgentsToolkitProjects\test2

# Create deployment package (exclude development files)
zip -r deployment.zip . \
  -x "*.git*" "*__pycache__*" "*.env.local*" "*node_modules*" "*.vscode*" "build/*"

# Deploy to App Service
az webapp deployment source config-zip \
  --name your-unique-app-name \
  --resource-group rg-teams-ai-agent-prod \
  --src deployment.zip
```

## Step 3: Connect Azure Bot Service to App Service

## Step 4: Upload Teams App Manifest

**4.1 Update Teams App Manifest for Production**

Update `appPackage/manifest.json` with production values:

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.16/MicrosoftTeams.schema.json",
  "manifestVersion": "1.16",
  "version": "1.0.0",
  "id": "your_production_app_id",
  "packageName": "com.yourcompany.teamsaiagent",
  "developer": {
    "name": "Your Company",
    "websiteUrl": "https://your-company.com",
    "privacyUrl": "https://your-company.com/privacy",
    "termsOfUseUrl": "https://your-company.com/terms"
  },
  "icons": {
    "color": "color.png",
    "outline": "outline.png"
  },
  "name": {
    "short": "AI Agent",
    "full": "Your Company AI Agent"
  },
  "description": {
    "short": "AI-powered assistant for Teams",
    "full": "An intelligent AI agent that helps with tasks and answers questions in Microsoft Teams"
  },
  "accentColor": "#FFFFFF",
  "bots": [
    {
      "botId": "your_production_app_id",
      "scopes": [
        "personal",
        "team",
        "groupchat"
      ],
      "supportsFiles": false,
      "isNotificationOnly": false,
      "commandLists": [
        {
          "scopes": [
            "personal",
            "team",
            "groupchat"
          ],
          "commands": [
            {
              "title": "Help",
              "description": "Get help with available commands"
            }
          ]
        }
      ]
    }
  ],
  "permissions": [
    "identity",
    "messageTeamMembers"
  ],
  "validDomains": [
    "your-unique-app-name.azurewebsites.net"
  ]
}
```

**4.2 Create Teams App Package**

```bash
# Navigate to appPackage directory
cd appPackage

# Create Teams app package
zip -r ../teams-app-prod.zip manifest.json color.png outline.png

# Verify package contents
unzip -l ../teams-app-prod.zip
```

**4.3 Upload to Teams Admin Center (Organization-wide)**

1. **Access Teams Admin Center**:
   - Navigate to [Microsoft Teams Admin Center](https://admin.teams.microsoft.com)
   - Sign in with admin credentials

2. **Upload Custom App**:
   - Go to **Teams apps** → **Manage apps**
   - Click **Upload new app** → **Upload**
   - Select your `teams-app-prod.zip` file
   - Wait for validation to complete

3. **Configure App Permissions**:
   - Find your uploaded app in the list
   - Click on the app name
   - Go to **Permissions** tab
   - Review and approve required permissions

4. **Set App Availability**:
   - Go to **Setup policies** or **Permission policies**
   - Create or modify policy to include your app
   - Assign policy to appropriate users/groups



## What's included in the template

| Folder       | Contents                                            |
| - | - |
| `.vscode`    | VSCode files for debugging                          |
| `appPackage` | Templates for the application manifest        |
| `env`        | Environment files                                   |
| `infra`      | Templates for provisioning Azure resources          |
| `src`        | The source code for the application                 |

The following files can be customized and demonstrate an example implementation to get you started.

| File                                 | Contents                                           |
| - | - |
|`src/app.py`| Hosts an aiohttp api server and exports an app module.|
|`src/bot.py`| Handles business logics for the AI Agent.|
|`src/config.py`| Defines the environment variables.|
|`src/state.py`| Defines the app state of AI Agent.|

The following are Microsoft 365 Agents Toolkit specific project files. You can [visit a complete guide on Github](https://github.com/OfficeDev/TeamsFx/wiki/Teams-Toolkit-Visual-Studio-Code-v5-Guide#overview) to understand how Microsoft 365 Agents Toolkit works.

| File                                 | Contents                                           |
| - | - |
|`m365agents.yml`|This is the main Microsoft 365 Agents Toolkit project file. The project file defines two primary things:  Properties and configuration Stage definitions. |
|`m365agents.local.yml`|This overrides `m365agents.yml` with actions that enable local execution and debugging.|
|`m365agents.playground.yml`|This overrides `m365agents.yml` with actions that enable local execution and debugging in Microsoft 365 Agents Playground.|

## Extend the template

You can follow [Build an AI Agent in Teams](https://aka.ms/teamsfx-ai-agent) to extend the AI Agent template with more AI capabilities, like:
- [Add functions](https://aka.ms/teamsfx-ai-agent#add-functions-build-new)

## Additional information and references

- [Microsoft 365 Agents Toolkit Documentations](https://docs.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals)
- [Microsoft 365 Agents Toolkit CLI](https://aka.ms/teamsfx-toolkit-cli)
- [Microsoft 365 Agents Toolkit Samples](https://github.com/OfficeDev/TeamsFx-Samples)

## Known issue
- If you use `Debug in Microsoft 365 Agents Playground` to local debug, you might get an error `InternalServiceError: connect ECONNREFUSED 127.0.0.1:3978` in Microsoft 365 Agents Playground console log or error message `Error: Cannot connect to your app,
please make sure your app is running or restart your app` in log panel of Microsoft 365 Agents Playground web page. You can wait for Python launch console ready and then refresh the front end web page.
- When you use `Launch Remote in Teams` to remote debug after deployment, you might loose interaction with your agent. This is because the remote service needs to restart. Please wait for several minutes to retry it.