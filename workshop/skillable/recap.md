## Recap

In under a hundred lines of Python, you built an AI agent that:

- ✅ Uses a model deployment from Microsoft Foundry
- ✅ Calls live tools through an MCP server
- ✅ Loads its persona and welcome banner from MCP **prompts** and greets
  the user automatically

The pattern you just used - **Foundry for the model, Agent Framework for
the glue, MCP for tools and prompts** - is the same one you'd use to
build a support bot, a coding assistant, or an internal company helper.
Swap the MCP server, change the persona, and you have a different agent.

The full source is in ['sample/agent.py'](../sample/agent.py).

### Where to go next

- **Add another MCP server.** The framework can connect to multiple at
  once - try giving the agent a weather server or a calendar server in
  addition to the cupcake store.
- **Swap the model.** Deploy a different model on Foundry, change
  'FOUNDRY_MODEL_DEPLOYMENT', and see how the personality shifts.
- **Stream the responses.** Use 'agent.run_stream(...)' for token-by-token
  output so the chat feels snappier.
- **Build your own MCP server.** Once you've consumed one, writing one
  is the natural next step - and now your agent can use it.


## Troubleshooting

| Problem | Fix |
| -- | -- |
| 'Missing required environment variable' | Check '.env' is in the 'c:\agents' folder and the variable names match |
| '401 Unauthorized' | Wrong API key or endpoint - re-copy from Foundry |
| 'DeploymentNotFound' | 'FOUNDRY_MODEL_DEPLOYMENT' must match the **deployment** name, not the model name |
| 'ImportError: cannot import name 'Agent'' | Run 'pip install -r requirements.txt' again |
