## Part 4 - Step 3: Load Instructions and a Welcome Banner from MCP

The agent works, but it still sounds like a generic assistant. The
Cupcake Store has *opinions* about how its agent should behave - tone,
upsells, allergy warnings, the works - and it doesn't want every
developer to copy-paste the latest version of that persona into their
code. So those instructions live on the **server**, not in your repo.

MCP servers can expose **prompts** - reusable text snippets curated by
the server owner. When the persona changes, the server updates; your
code keeps working without a redeploy. The Cupcake Store exposes two:

- **agent_instructions** - the persona / system prompt
- **welcome_banner** - a friendly greeting to print at startup

Fetch both from the server, pass 'agent_instructions' to the 'Agent', and
print the banner before the chat starts.

```python-notype
"""Sparkles - The Cupcake ordering agent"""

import asyncio
import os

from dotenv import load_dotenv

from agent_framework import Agent, MCPStreamableHTTPTool  
from agent_framework.foundry import AnthropicFoundryClient

# 1. Load environment variables from .env
load_dotenv()


async def main() -> None:
    # 2. Configure the chat model (Claude on Microsoft Foundry)
    chat_client = AnthropicFoundryClient(
        model=os.environ["FOUNDRY_MODEL_DEPLOYMENT"],
        api_key=os.environ["FOUNDRY_API_KEY"],
        base_url=os.environ["FOUNDRY_ENDPOINT"],
    )

    # 3. Connect to the Cupcake Store MCP server
    mcp_tool = MCPStreamableHTTPTool(
        name="cupcake-store",
        url="https://ca-cupcake-mcp-dev-f776-net.purplepond-6465a08a.eastus2.azurecontainerapps.io/mcp/",
    )
    await mcp_tool.connect()

    # 4. Get the instructions and welcome banner from the MCP server   👈 new
    instructions = await mcp_tool.get_prompt("agent_instructions")
    banner = await mcp_tool.get_prompt("welcome_banner")

    # 5. Create the agent
    agent = Agent(
        client=chat_client,
        name="cupcake-agent",
        instructions=instructions,                                     # 👈 new
        tools=mcp_tool,
    )

    # 6. Start a chat session and talk to the agent
    session = agent.create_session()
    print(banner)                                                      # 👈 new
    print("Type 'exit' to quit.\n")

    # Kick things off automatically                                    👈 new
    response = await agent.run("hello", session=session)
    print(f"\033[1;35mAssistant:\033[0m\n{response.text}\n")

    while True:
        user_input = input("\033[1;35mYou:\033[0m\n")
        if user_input.lower() in ("exit", "quit"):
            break

        response = await agent.run(user_input, session=session)
        print(f"\n\033[1;35mAssistant:\033[0m\n{response.text}\n")

    await mcp_tool.close()

if __name__ == "__main__":
    asyncio.run(main())
```

The agent now also kicks off the conversation itself by sending a 'hello',
so the user sees the persona greet them right away.

**Run it:**

Back in the VS Code terminal at the bottom of the editor, run:

```bash
python agent.py
```

**Try it:** Now it's time to order your cupcake! 

Type 'exit' when you're done to stop the agent.


![](../images/06-step3-prompts.png)


- Answer the questions of the agent
- Select your favorite cupcake
- Place your order
- Look at the screen in the room
- When your order is ready for pickup, go to the front of the room to pick it up


![](../images/07-dashboard.png)

---

✅ **In this step you have:** built a Hello World agent, connected it to
the Cupcake Store MCP server for tools, loaded its persona and welcome
banner from MCP prompts, and ordered a real cupcake.

➡️ Click **Next** for the recap and ideas on where to go from here.
