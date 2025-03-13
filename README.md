# Multi-agent AI sample with Azure Cosmos DB

A sample personal shopping AI Chatbot that can help with product enquiries, making sales, and refunding orders by transferring to different agents for those tasks.

Features:
- **Multi-agent**: [OpenAI Swarm](https://github.com/openai/swarm) to orchestrate multi-agent interactions with [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview) API calls.
- **Transactional data management**: planet scale [Azure Cosmos DB database service](https://learn.microsoft.com/azure/cosmos-db/introduction) to store transactional user and product operational data.
- **Retrieval Augmented Generation (RAG)**: [vector search](https://learn.microsoft.com/azure/cosmos-db/nosql/vector-search) in Azure Cosmos DB with powerful [DiskANN index](https://www.microsoft.com/en-us/research/publication/diskann-fast-accurate-billion-point-nearest-neighbor-search-on-a-single-node/?msockid=091c323873cd6bd6392120ac72e46a98) to serve product enquiries from the same database.
- **Gradio UI**: [Gradio](https://www.gradio.app/) to provide a simple UI ChatBot for the end-user.

## Backend agent activity

Run the CLI interactive session to see the agent handoffs in action...

![Demo](./media/demo-cli.gif)

## Front-end AI chat bot

Run the AI chat bot for the end-user experience...

![Demo](./media/demo-chatbot.gif)

## Overview

The personal shopper example includes four main agents to handle various customer service requests:

1. **Triage Agent**: Determines the type of request and transfers to the appropriate agent.
2. **Product Agent**: Answers customer queries from the products container using [Retrieval Augmented Generation (RAG)](https://learn.microsoft.com/azure/cosmos-db/gen-ai/rag).
3. **Refund Agent**: Manages customer refunds, requiring both user ID and item ID to initiate a refund.
4. **Sales Agent**: Handles actions related to placing orders, requiring both user ID and product ID to complete a purchase.

## Prerequisites

- [Azure Cosmos DB account](https://learn.microsoft.com/azure/cosmos-db/create-cosmosdb-resources-portal) - ensure the [vector search](https://learn.microsoft.com/azure/cosmos-db/nosql/vector-search) feature is enabled and that you have created a database called "MultiAgentDemoDB".
- [Azure OpenAI API key](https://learn.microsoft.com/azure/ai-services/openai/overview) and endpoint.
- [Azure OpenAI Embedding Deployment ID](https://learn.microsoft.com/azure/ai-services/openai/overview) for the RAG model.
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) to authenticate to Azure Cosmos DB and Azure OpenAI with [Entra ID RBAC](https://learn.microsoft.com/entra/identity/role-based-access-control/).

## How to run locally

Clone the repository:

```shell
git clone https://github.com/AzureCosmosDB/multi-agent-swarm
cd multi-agent-swarm
```

Install dependencies:

```shell
pip install git+https://github.com/openai/swarm.git
pip install azure-cosmos==4.9.0
pip install gradio
pip install azure-identity
```

Ensure you have the following environment variables set:
```shell
AZURE_COSMOSDB_ENDPOINT=your_cosmosdb_account_uri
AZURE_OPENAI_ENDPOINT=your_azure_openai_endpoint
AZURE_OPENAI_EMBEDDINGDEPLOYMENTID=your_azure_openai_embeddingdeploymentid
```

Once you have installed dependencies, authenticate in Azure using Azure CLI:

```shell
az login
```

If your signed-in Azure user does not already have access to Azure Cosmos DB via RBAC, run one of the following to grant the `Cosmos DB Built-in Data Contributor` role (choose bash or Powershell depending on which you are running in):

```bash
# Bash (replace <YOUR_COSMOSDB_ACCOUNT_NAME> and <YOUR_RESOURCE_GROUP> with appropriate values)
az role assignment create --assignee $(az ad signed-in-user show --query id -o tsv) \
    --role "Cosmos DB Built-in Data Contributor" \
    --scope $(az cosmosdb show --name <YOUR_COSMOSDB_ACCOUNT_NAME> --resource-group <YOUR_RESOURCE_GROUP> --query id -o tsv)
```

```powershell
# PowerShell (replace <YOUR_COSMOSDB_ACCOUNT_NAME> and <YOUR_RESOURCE_GROUP> with appropriate values)
$assignee = az ad signed-in-user show --query id -o tsv
$scope = az cosmosdb show --name <YOUR_COSMOSDB_ACCOUNT_NAME> --resource-group <YOUR_RESOURCE_GROUP> --query id -o tsv

az role assignment create --assignee $assignee --role "Cosmos DB Built-in Data Contributor" --scope $scope
```

If your signed-in Azure user does not already have access to Azure OpenAI via RBAC, run one of the following to grant the `Cognitive Services User` role (choose bash or Powershell depending on which you are running in). 

```bash
# Bash (replace <YOUR_OPENAI_RESOURCE_NAME> and <YOUR_RESOURCE_GROUP> with appropriate values)
az role assignment create --assignee $(az ad signed-in-user show --query id -o tsv) \
    --role "Cognitive Services User" \
    --scope $(az cognitiveservices account show --name <YOUR_OPENAI_RESOURCE_NAME> --resource-group <YOUR_RESOURCE_GROUP> --query id -o tsv)
```

```powershell
# PowerShell (replace <YOUR_OPENAI_RESOURCE_NAME> and <YOUR_RESOURCE_GROUP> with appropriate values)
$assignee = az ad signed-in-user show --query id -o tsv
$scope = az cognitiveservices account show --name <YOUR_OPENAI_RESOURCE_NAME> --resource-group <YOUR_RESOURCE_GROUP> --query id -o tsv

az role assignment create --assignee $assignee --role "Cognitive Services User" --scope $scope
```

Run below and click on URL provided in output:

```shell
python src/app/ai_chat_bot.py
```

To see the agent handoffs, you can also run as an interactive Swarm CLI session using:

```shell
python src/app/multi_agent_service.py
```