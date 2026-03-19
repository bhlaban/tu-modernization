# Quickstart: AI Chapter Assistant

**Date**: 2026-03-18
**Feature**: [spec.md](spec.md)

## Prerequisites

- .NET 8 SDK
- Node.js 20+ and npm
- PostgreSQL 16+ with pgvector extension
- An Azure OpenAI or OpenAI API key

## 1. Clone and checkout

```bash
git clone <repo-url>
cd tu-modernization
git checkout 001-ai-chapter-assistant
```

## 2. Database setup

```bash
# Start PostgreSQL (if using Docker)
docker run -d --name tu-postgres \
  -e POSTGRES_USER=tu_admin \
  -e POSTGRES_PASSWORD=dev_password \
  -e POSTGRES_DB=tu_assistant \
  -p 5432:5432 \
  pgvector/pgvector:pg16

# Or enable pgvector on an existing PostgreSQL instance:
# CREATE EXTENSION IF NOT EXISTS vector;
```

## 3. Backend setup

```bash
cd backend

# Restore dependencies
dotnet restore

# Configure environment (copy and edit)
cp src/TUAssistant.Api/appsettings.Development.example.json \
   src/TUAssistant.Api/appsettings.Development.json

# Edit appsettings.Development.json:
# - ConnectionStrings:DefaultConnection = "Host=localhost;Database=tu_assistant;Username=tu_admin;Password=dev_password"
# - AzureOpenAI:Endpoint = "https://your-instance.openai.azure.com/"
# - AzureOpenAI:ApiKey = "your-key"
# - AzureOpenAI:DeploymentName = "gpt-35-turbo"
# - AzureOpenAI:EmbeddingDeploymentName = "text-embedding-3-small"

# Run database migrations
dotnet ef database update --project src/TUAssistant.Api

# Run backend
dotnet run --project src/TUAssistant.Api
# API available at https://localhost:5001
```

## 4. Frontend setup

```bash
cd frontend

# Install dependencies
npm install

# Configure environment
cp .env.example .env.local

# Edit .env.local:
# REACT_APP_API_URL=https://localhost:5001/api/v1

# Run frontend
npm start
# App available at http://localhost:3000
```

## 5. Seed initial data

```bash
# Create an admin user (from backend directory)
dotnet run --project src/TUAssistant.Api -- seed-admin \
  --email admin@tu.org \
  --password "AdminP@ss1" \
  --name "TU Admin"

# Upload a sample document via the API
curl -X POST https://localhost:5001/api/v1/admin/documents \
  -H "Authorization: Bearer <admin-jwt-token>" \
  -F "file=@sample-docs/event-planning-guide.pdf" \
  -F "title=Event Planning Guide"
```

## 6. Run tests

```bash
# Backend tests
cd backend
dotnet test

# Frontend tests
cd frontend
npm test
```

## 7. Verify

1. Open http://localhost:3000
2. Register a new account or login with the admin account
3. Start a new conversation
4. Ask: "How do I plan a stream cleanup event?"
5. Verify the assistant responds with relevant content and source citations
6. Click thumbs-up/thumbs-down to test feedback

## Troubleshooting

| Problem | Solution |
|---------|----------|
| pgvector not found | Ensure the Docker image is `pgvector/pgvector:pg16` or run `CREATE EXTENSION vector;` manually |
| AI service timeout | Verify Azure OpenAI endpoint and API key in appsettings |
| CORS errors | Ensure `AllowedOrigins` in appsettings includes `http://localhost:3000` |
| Migration fails | Ensure PostgreSQL is running and connection string is correct |
