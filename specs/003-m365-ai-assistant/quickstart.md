# Quickstart: M365 AI Chapter Assistant

**Date**: 2026-03-18 | **Branch**: `003-m365-ai-assistant`

## Prerequisites

- .NET 8 SDK
- Node.js 20+ and npm
- An Azure subscription (for Azure OpenAI Service and hosting)
- Access to a Microsoft 365 Business Standard tenant for testing
- Azure CLI (`az`) installed

## 1. Clone and Setup

```bash
git clone <repo-url>
cd tu-modernization
git checkout 003-m365-ai-assistant
```

## 2. Entra ID App Registration

1. Go to [Azure Portal → Entra ID → App registrations](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps)
2. Click **New registration**:
   - Name: `TU Chapter AI Assistant`
   - Supported account types: **Accounts in any organizational directory (Multitenant)**
   - Redirect URI: `http://localhost:3000/auth/callback` (SPA platform)
3. Note the **Application (client) ID**
4. Under **Certificates & secrets**, create a client secret → note the value
5. Under **API permissions**, add delegated permissions:
   - `Files.Read.All`, `Sites.Read.All`, `Mail.Read`, `Mail.Send`
   - `Calendars.ReadWrite`, `Chat.Read`, `Group.Read.All`
6. Click **Grant admin consent** for your test tenant

## 3. Azure OpenAI Setup

1. Create an Azure OpenAI resource in the Azure Portal
2. Deploy a `gpt-4o` (or latest) model
3. Note the **endpoint URL** and **API key**

## 4. Backend Configuration

```bash
cd backend
cp appsettings.Development.example.json appsettings.Development.json
```

Edit `appsettings.Development.json`:

```json
{
  "AzureAd": {
    "ClientId": "<your-app-client-id>",
    "ClientSecret": "<your-client-secret>",
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "common"
  },
  "AzureOpenAI": {
    "Endpoint": "<your-azure-openai-endpoint>",
    "ApiKey": "<your-api-key>",
    "DeploymentName": "<your-model-deployment-name>"
  },
  "SessionStore": {
    "ConnectionString": "<cosmos-db-or-table-storage-connection-string>"
  }
}
```

## 5. Run Backend

```bash
cd backend
dotnet restore
dotnet run
```

Backend starts on `https://localhost:5001`.

## 6. Frontend Configuration

```bash
cd frontend
npm install
cp .env.example .env.local
```

Edit `.env.local`:

```
REACT_APP_API_URL=https://localhost:5001/api/v1
REACT_APP_CLIENT_ID=<your-app-client-id>
REACT_APP_REDIRECT_URI=http://localhost:3000/auth/callback
```

## 7. Run Frontend

```bash
npm start
```

Frontend starts on `http://localhost:3000`.

## 8. Test the Flow

1. Open `http://localhost:3000` in a browser
2. Click **Sign in** — authenticate with a test M365 account that is a member of the designated security group
3. Ask a question: "What documents do we have about conservation projects?"
4. Verify the response includes citations with links to SharePoint/OneDrive content
5. Try a write action: "Create a calendar event for a board meeting next Tuesday at 7pm"
6. Confirm the action preview and verify the event appears in the test account's calendar

## Running Tests

```bash
# Backend unit + integration tests
cd backend
dotnet test

# Frontend tests
cd frontend
npm test

# E2E tests (requires running backend + frontend)
cd frontend
npx playwright test
```
