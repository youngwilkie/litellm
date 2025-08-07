# Deploy LiteLLM on Render with Supabase Database

This guide walks you through deploying LiteLLM proxy on Render using Supabase as your PostgreSQL database.

## Prerequisites

- A [Render](https://render.com) account
- A [Supabase](https://supabase.com) account
- Your LLM API keys (OpenAI, Anthropic, etc.)

## Step 1: Set up Supabase Database

### 1.1 Create Supabase Project
1. Go to [supabase.com](https://supabase.com) and sign in
2. Click "New Project"
3. Choose your organization and set:
   - **Name**: `litellm-database` (or your preferred name)
   - **Database Password**: Generate a strong password and save it
   - **Region**: Choose closest to your users
4. Click "Create new project" and wait for setup to complete

### 1.2 Get Database Connection String
1. In your Supabase project dashboard, go to **Settings** → **Database**
2. Scroll down to **Connection string** section
3. Select **URI** tab
4. Copy the connection string (it looks like):
   ```
   postgresql://postgres.abcdefghijklmnop:[YOUR-PASSWORD]@aws-0-us-east-1.pooler.supabase.com:6543/postgres
   ```
5. Replace `[YOUR-PASSWORD]` with the password you set during project creation

### 1.3 Configure Database (Optional)
The LiteLLM proxy will automatically create the required tables on first run via Prisma migrations. No manual SQL setup is needed.

## Step 2: Deploy to Render

### 2.1 Deploy Using the Deploy Button
1. Click this button to deploy directly:
   
   [![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/BerriAI/litellm)

### 2.2 Alternative: Manual Render Setup
If the deploy button doesn't work, follow these steps:

1. **Fork this repository** (optional but recommended for customization)

2. **Create a new Web Service on Render**:
   - Go to [Render Dashboard](https://dashboard.render.com/)
   - Click "New" → "Web Service"
   - Connect your GitHub account and select this repository

3. **Configure the service**:
   - **Name**: `litellm-proxy`
   - **Runtime**: `Docker`
   - **Dockerfile Path**: Leave empty (will use root Dockerfile)
   - **Build Command**: Leave empty
   - **Start Command**: Leave empty

## Step 3: Configure Environment Variables

In your Render service dashboard, go to **Environment** and add these variables:

### Required Variables
```bash
# Database Configuration
DATABASE_URL=postgresql://postgres.abcdefghijklmnop:[YOUR-PASSWORD]@aws-0-us-east-1.pooler.supabase.com:6543/postgres

# Master Key (must start with 'sk-')
LITELLM_MASTER_KEY=sk-your-secret-master-key-here

# Enable database model storage
STORE_MODEL_IN_DB=True

# Port (Render requires this)
PORT=4000
```

### LLM Provider API Keys
Add your LLM provider keys as needed:
```bash
# OpenAI
OPENAI_API_KEY=sk-your-openai-key

# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Azure OpenAI
AZURE_API_KEY=your-azure-key
AZURE_API_BASE=https://your-resource.openai.azure.com/

# Add other providers as needed...
```

### Optional Variables
```bash
# Enable detailed debugging (useful for initial setup)
LITELLM_LOG=DEBUG

# Custom configuration (if using config.yaml)
# LITELLM_CONFIG_PATH=/app/config.yaml
```

## Step 4: Deploy and Test

### 4.1 Deploy
1. Click **Manual Deploy** or push changes to trigger auto-deploy
2. Monitor the deploy logs for any errors
3. Wait for the service to show as "Live"

### 4.2 Test Your Deployment
Once deployed, your LiteLLM proxy will be available at your Render URL (e.g., `https://your-service-name.onrender.com`).

**Test with curl**:
```bash
# Replace YOUR_RENDER_URL with your actual Render service URL
curl https://YOUR_RENDER_URL.onrender.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-your-secret-master-key-here" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "Hello, world!"}]
  }'
```

**Check health**:
```bash
curl https://YOUR_RENDER_URL.onrender.com/health/liveliness
```

## Step 5: Configure Models (Optional)

### 5.1 Using the Admin UI
1. Visit `https://YOUR_RENDER_URL.onrender.com/ui`
2. Log in with your `LITELLM_MASTER_KEY`
3. Add models, create API keys, set budgets, etc.

### 5.2 Using Configuration File
If you prefer configuration files, create a `config.yaml`:

```yaml
model_list:
  - model_name: gpt-4
    litellm_params:
      model: gpt-4
      api_key: os.environ/OPENAI_API_KEY
  - model_name: claude-3-sonnet
    litellm_params:
      model: anthropic/claude-3-sonnet-20240229
      api_key: os.environ/ANTHROPIC_API_KEY

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

Then update your Render environment variables:
```bash
LITELLM_CONFIG_PATH=/app/config.yaml
```

## Troubleshooting

### Common Issues

1. **Database Connection Errors**
   - Verify your `DATABASE_URL` is correct
   - Ensure Supabase project is running
   - Check if you replaced `[YOUR-PASSWORD]` with actual password

2. **Master Key Issues**
   - `LITELLM_MASTER_KEY` must start with `sk-`
   - Use a strong, random key (e.g., `sk-` + 32 random characters)

3. **Service Won't Start**
   - Check Render logs for specific errors
   - Ensure all required environment variables are set
   - Try enabling `LITELLM_LOG=DEBUG` for more detailed logs

4. **API Requests Failing**
   - Verify your LLM provider API keys are correct
   - Check if the model name is supported
   - Ensure you're using the correct authorization header

### Getting Help

- Check the [LiteLLM documentation](https://docs.litellm.ai/)
- Review [Render deployment docs](https://render.com/docs)
- Open an issue on the [LiteLLM GitHub repo](https://github.com/BerriAI/litellm)

## Cost Considerations

- **Render**: Free tier available, paid plans start at $7/month
- **Supabase**: Free tier includes 500MB database, paid plans start at $25/month
- **LLM APIs**: Pay per usage based on your provider's pricing

## Security Best Practices

1. **Use strong passwords** for your Supabase database
2. **Keep your master key secret** - don't commit it to version control
3. **Rotate API keys regularly**
4. **Enable Supabase Row Level Security** if storing sensitive data
5. **Use Render's environment variable encryption** for sensitive values

## Next Steps

- Set up monitoring and alerting
- Configure load balancing for high availability
- Implement custom authentication and authorization
- Set up logging and observability with your preferred tools

Your LiteLLM proxy is now running on Render with Supabase! 🚀
