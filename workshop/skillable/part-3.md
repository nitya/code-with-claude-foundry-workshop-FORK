## Part 3 - Get the Endpoint and API Key

Now that you know the model works, you need three things to call it from
your own code: the **endpoint** (where to send requests), the **API key**
(proof you're allowed to), and the **deployment name** (which model
instance to use). All three live one click away.

1. In the playground - click the **Details** tab at the top.
2. On the **Details** tab, copy:
   - **Target URI** - the endpoint, e.g. "https://<your-resource>.services.ai.azure.com/anthropic"
   - **Key** - click the eye icon to reveal it, then the copy icon next to it
   - **Name** of the deployment (e.g. claude-sonnet-4-6) - shown under **Deployment info**

![Endpoint keys](../images/03-endpoint-keys.png)

> 🔐 Treat the API key like a password. Don't paste it into chats,
> screenshots, or commits. The '.env' file you'll edit next is already
> listed in '.gitignore' so it stays on your machine.

3. Open the existing ".env" file in the "sparkles-agent/" folder and replace
   the placeholder values with the ones you just copied:

   ```env
   FOUNDRY_ENDPOINT="https://<your-resource>.services.ai.azure.com/anthropic"
   FOUNDRY_API_KEY="<your-api-key>"
   FOUNDRY_MODEL_DEPLOYMENT="<your-deployment-name>"
   ```
