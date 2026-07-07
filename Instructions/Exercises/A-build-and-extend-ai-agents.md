---
lab:
    title: 'Build and extend AI agents'
    description: 'Create a grounded AI agent, then extend it with tools using remote MCP servers, custom functions, and a client app.'
    level: 300
    concepts: 'agent creation and grounding, tools, Model Context Protocol (MCP)'
    duration: 35
    islab: true
    status: 'draft'
---

<!--
PILOT NOTE (remove before publishing):
This is a pilot of the new lab template (Core + Optional tasks) applied to
"Lab A" = a consolidation of the current exercises 01, 02, and 03.
Starter code lives in Labfiles/A-build-and-extend-ai-agents/ (duplicated from the source
labs so it can evolve independently), organized as: portal-agent/ (Task 3),
custom-functions/ (Task 4), and mcp/ (Tasks 2 and 5).
Remaining follow-up tracked in the design spec: author a shared "Getting started" setup
page and link it instead of repeating setup here.
-->

# Build and extend AI agents

An agent becomes genuinely useful when it can *do* things — look up live information,
call your business logic, and act on a user's behalf. In this exercise you'll build a
grounded agent and then give it capabilities using **tools**.

You'll start with the **Core** tasks that get you to a working, tool-using agent as
quickly as possible. From there, a set of **Optional** tasks lets you go deeper into the
areas that interest you most.

> **Note**: Some of the technologies used in this exercise are in preview or in active
> development. You may experience some unexpected behavior, warnings, or errors.

## What you'll learn

By completing the **Core** tasks of this exercise, you'll be able to:

- **Create and ground an agent** in the Microsoft Foundry portal so it answers from your
  own data rather than guessing.
- **Extend an agent with a tool** by connecting it to a remote **Model Context Protocol
  (MCP)** server, and handle tool-approval requests in code.

The **Optional** tasks let you additionally:

- Call your agent from a **client application**.
- Give an agent **custom function tools** that run your own Python logic.
- Build and connect your **own MCP server**.

## Lab at a glance

Complete the **Core** tasks first (about **35 minutes**) — they're self-contained and end
with a working, tool-using agent. Then expand any **Optional** tasks that interest you.
The full lab, including all optional tasks, takes about **1 hour 50 minutes**. Use the
buttons below to auto-expand a set of tasks that match the time you have.

| Section | Task | Difficulty | Time |
| --- | --- | --- | --- |
| **Core** | Task 1 – Create and ground an agent in the portal | ★☆☆ | ~15 min |
| **Core** | Task 2 – Connect the agent to a remote MCP server in code | ★★☆ | ~20 min |
| *Optional* | Task 3 – Call your agent from a client app | ★★☆ | ~20 min |
| *Optional* | Task 4 – Add custom function tools | ★★☆ | ~25 min |
| *Optional* | Task 5 – Build your own MCP server + client | ★★★ | ~30 min |

<!-- Lab-length picker. Works on the rendered GitHub Pages site; on GitHub.com's raw
     markdown view the buttons are inert, but every task can still be expanded manually.
     To change which tasks each preset opens, edit the data-tier on the task <details>. -->
<div class="lab-length-picker" role="group" aria-label="Choose your lab length">
  <span class="lab-length-label">Choose your lab length:</span>
  <button type="button" class="lab-btn is-active" data-tier="1">Core only · ~35 min</button>
  <button type="button" class="lab-btn" data-tier="2">Core + recommended · ~1h 20m</button>
  <button type="button" class="lab-btn" data-tier="3">Everything · ~1h 50m</button>
</div>

<style>
.lab-length-picker { display:flex; flex-wrap:wrap; gap:.5rem; align-items:center; margin:1rem 0; }
.lab-length-label { font-weight:600; }
.lab-btn { cursor:pointer; padding:.4rem .8rem; border:1px solid #8888; border-radius:6px; background:#f3f3f3; color:#111; font:inherit; }
.lab-btn:hover { background:#e6e6e6; }
.lab-btn.is-active { background:#0969da; color:#fff; border-color:#0969da; }
details.opt-task.is-included { border-left:3px solid #0969da; padding-left:.6rem; }
details.opt-task.is-included > summary { font-weight:600; }
</style>

<script>
(function () {
  function applyTier(tier) {
    document.querySelectorAll('.lab-btn').forEach(function (b) {
      b.classList.toggle('is-active', Number(b.dataset.tier) === tier);
    });
    document.querySelectorAll('details.opt-task').forEach(function (d) {
      var included = tier >= Number(d.dataset.tier);
      d.open = included;
      d.classList.toggle('is-included', included);
    });
  }
  document.querySelectorAll('.lab-btn').forEach(function (b) {
    b.addEventListener('click', function () { applyTier(Number(b.dataset.tier)); });
  });
  applyTier(1);
})();
</script>

## Prerequisites

Before starting this exercise, ensure you have:

- An [Azure subscription](https://azure.microsoft.com/free/) with sufficient permissions and quota to provision Azure AI resources
- [Visual Studio Code](https://code.visualstudio.com/) installed on your local machine
- [Python 3.13](https://www.python.org/downloads/) or later installed
- [Git](https://git-scm.com/downloads) installed on your local machine
- Basic familiarity with Python

> \* Python 3.14 is available, but some dependencies are not yet compiled for that release. The lab has been successfully tested with Python 3.13.12.

## Set up your environment

> **Note**: These setup steps are common to every lab in this path. Once the shared
> *Getting started* page exists, this section will link to it instead of repeating it.

### Create a Microsoft Foundry project

Microsoft Foundry uses projects to organize models, resources, data, and other assets.

1. In a web browser, open the [Foundry portal](https://ai.azure.com) at `https://ai.azure.com` and sign in using your Azure credentials. Close any tips or quick start panes, and if necessary use the **Foundry** logo at the top left to navigate to the home page.

    > **Important**: For this lab, you're using the **New** Foundry experience.

1. In the top banner, select **Start building**.

1. When prompted, create a **new** project and enter a valid name (for example, `agents-lab-project`).

1. Expand **Advanced options** and specify:
    - **Microsoft Foundry resource**: *A valid name for your Foundry resource*
    - **Region**: *Select one available near you*\*
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Select or create a resource group*

    > \* Some Azure AI resources are constrained by regional model quotas. If you hit a quota limit later, you may need to create another resource in a different region.

1. Select **Create** and wait for your project to be created. When prompted, continue through the welcome dialog and select **Create agent**.

1. Set the **Agent name** to `it-support-agent` and create the agent. The playground opens with a deployed model already selected for you.

Keep this browser tab open — you'll use it in Task 1.

---

# Core

The next two tasks are the required core of this lab. Task 1 builds a grounded agent in
the portal; Task 2 extends an agent with a tool in code.

## Task 1 — Create and ground an agent (portal)

Grounding gives your agent trusted source material so it answers accurately instead of
guessing.

1. In the agent playground, set the **Instructions** to:

    ```prompt
    You are an IT Support Agent for Contoso Corporation.
    You help employees with technical issues and IT policy questions.

    Guidelines:
    - Always be professional and helpful
    - Use the IT policy documentation to answer questions accurately
    - If you don't know the answer, admit it and suggest contacting IT support directly
    ```

1. Download the sample IT policy document. Open a new browser tab and navigate to:

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/portal-agent/IT_Policy.txt
    ```

    Save the file to your local machine.

1. Back in the playground, in the **Tools** section, select **Add**, and add **File search**.

1. To the right of **Add**, select **Upload files**, browse to the `IT_Policy.txt` file you downloaded, and select **Attach**. Wait for the file to be indexed.

1. **Save** the agent.

### Test your grounded agent

1. In the chat pane, enter:

    ```
    What's the policy for password resets?
    ```

    The agent should reference the IT policy document in its answer.

1. Try a second question to confirm it's using the grounding data:

    ```
    How do I request new software?
    ```

> ✅ **Checkpoint**: Your agent answers IT questions using the uploaded policy document.
> You've created and grounded an agent entirely in the portal.

## Task 2 — Extend an agent with a remote MCP server (code)

The **Model Context Protocol (MCP)** lets an agent discover and call tools hosted by a
server. In this task you'll connect an agent to the **Microsoft Learn Docs** remote MCP
server so it can pull trusted, up-to-date documentation on demand.

### Get the starter code

1. In VS Code, open the Command Palette (**Ctrl+Shift+P**), run **Git: Clone**, and enter:

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Open the cloned repo, then **File > Open Folder** and select `mslearn-ai-agents/Labfiles/A-build-and-extend-ai-agents/mcp`.

1. Expand the **Python** folder, right-click **requirements.txt**, and choose **Open in Integrated Terminal**. Then create a virtual environment and install packages:

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. Open the **.env** file and set `PROJECT_ENDPOINT` to your project endpoint and `MODEL_DEPLOYMENT_NAME` to your model deployment name. Save the file.

    > **Tip**: In the Foundry Toolkit VS Code extension, right-click your project deployment and select **Copy Project Endpoint** to get the endpoint URL.

### Connect the agent to the MCP server

Open **agent.py** and add code at each commented placeholder.

> **Tip**: As you add code, keep the indentation aligned with the comments.

1. **Add references**:

    ```python
    # Add references
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.projects.models import PromptAgentDefinition, MCPTool
    from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
    ```

1. **Connect to the agents client**:

    ```python
    # Connect to the agents client
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

1. **Initialize agent MCP tool** — this points the agent at the Microsoft Learn Docs MCP server:

    ```python
    # Initialize agent MCP tool
    mcp_tool = MCPTool(
        server_label="api-specs",
        server_url="https://learn.microsoft.com/api/mcp",
        require_approval="always",
    )
    ```

1. **Create a new agent with the MCP tool**:

    ```python
    # Create a new agent with the MCP tool
    agent = project_client.agents.create_version(
        agent_name="MyAgent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="You are a helpful agent that can use MCP tools to assist users. Use the available MCP tools to answer questions and perform tasks.",
            tools=[mcp_tool],
        ),
    )
    print(f"Agent created (id: {agent.id}, name: {agent.name}, version: {agent.version})")
    ```

1. **Create a conversation thread**:

    ```python
    # Create a conversation thread
    conversation = openai_client.conversations.create()
    print(f"Created conversation (id: {conversation.id})")
    ```

1. **Send initial request that will trigger the MCP tool**:

    ```python
    # Send initial request that will trigger the MCP tool
    response = openai_client.responses.create(
        conversation=conversation.id,
        input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    )
    ```

1. **Process any MCP approval requests** — because the tool requires approval, the agent
    pauses and asks permission before each call. This loop auto-approves each request:

    ```python
    # Process any MCP approval requests that were generated
    while True:
        input_list: ResponseInputParam = []
        for item in response.output:
            if item.type == "mcp_approval_request":
                if item.server_label == "api-specs" and item.id:
                    input_list.append(
                        McpApprovalResponse(
                            type="mcp_approval_response",
                            approve=True,
                            approval_request_id=item.id,
                        )
                    )

        # No more approvals needed -> the agent has produced its final response
        if not input_list:
            break

        response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )

    print(f"\nAgent response: {response.output_text}")
    ```

1. **Clean up the agent version** so you don't leave test agents behind:

    ```python
    # Clean up resources by deleting the agent version
    project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
    print("Agent deleted")
    ```

1. Save the file (**Ctrl+S**).

### Run and test

1. In the terminal, sign in and run the app:

    ```
    az login
    ```

    ```
    python agent.py
    ```

1. Watch the agent create itself, call the MCP tool (approved automatically by your loop), and answer using live documentation. You should see output similar to:

    ```
    Agent created (id: MyAgent:2, name: MyAgent, version: 2)
    Created conversation (id: conv_...)

    Agent response: Here are Azure CLI commands to create an Azure Container App with a managed identity:
    ...
    Agent deleted
    ```

1. Try changing the `input` string to ask about a different Azure service, and run again.

> ✅ **Checkpoint**: You've built a grounded agent *and* extended an agent with an
> external tool via a remote MCP server, including approval handling. That's the Core of
> this lab — everything below is optional.

When you're finished, enter `deactivate` to exit the virtual environment.

---

# Optional

These tasks are independent — expand any that interest you, in any order. Each begins with
a **Try it first** prompt; expand **Show a solution** when you want the full walkthrough.

<details markdown="1" class="opt-task" data-tier="2">
<summary><strong>Task 3 — Call your agent from a client app</strong> &middot; ★★☆ &middot; ~20 min</summary>

**Goal**: Interact with the grounded portal agent from Python instead of the playground,
including handling files the agent produces (for example, charts from code interpreter).

**Concept reinforced**: consuming an agent programmatically with the Foundry SDK — loading
an existing agent by name and driving it with the Responses API.

**Set up:**

1. In the portal, open your `it-support-agent`, add a **Code interpreter** tool, and upload
    a data file so there's something to analyze. Download and attach:

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/portal-agent/system_performance.csv
    ```

    Save the agent.

1. In VS Code, open the `Labfiles/A-build-and-extend-ai-agents/portal-agent/Python` folder.
    Create a virtual environment, install requirements, then open **.env** and set
    `PROJECT_ENDPOINT` and `AGENT_NAME` (`it-support-agent`):

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

> **Try it first**: The `agent_with_functions.py` file already contains a complete client.
> Before running it, predict: which SDK call loads your existing portal agent *by name*?
> How does the client tell the Responses API to use that agent? Where would a generated
> chart end up?

<details markdown="1">
<summary>Show a solution</summary>

The provided `agent_with_functions.py` already implements the client. The lines that
matter are:

1. **Load your portal agent by name** (using `AGENT_NAME` from **.env**):

    ```python
    agent = project_client.agents.get(agent_name=agent_name)
    ```

2. **Route each request to that agent** through the Responses API:

    ```python
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
    ```

3. **Saved files**: helper functions detect image outputs and `container_file_citation`
    annotations and write them to a local `agent_outputs/` folder.

Sign in and run it:

```
az login
python agent_with_functions.py
```

Ask for something that uses code interpreter:

```
Analyze the system performance data and create a chart of CPU usage over time.
```

You should see the analysis plus a line like
`[Agent generated a chart - saved to: agent_outputs\chart_1.png]`. Type `exit` to quit.

</details>

**Stretch**: display the agent's token usage after each response.

</details>

<details markdown="1" class="opt-task" data-tier="2">
<summary><strong>Task 4 — Add custom function tools</strong> &middot; ★★☆ &middot; ~25 min</summary>

**Goal**: Give an agent tools backed by your **own Python functions**, and process the
function calls it makes.

**Concept reinforced**: the function-calling loop — the agent decides *which* tool to
call and with *what* arguments; your code executes it and returns the result.

**Set up:**

1. Open the `Labfiles/A-build-and-extend-ai-agents/custom-functions/Python` folder, create
    a virtual environment, install requirements, and set `PROJECT_ENDPOINT` and
    `MODEL_DEPLOYMENT_NAME` in **.env**. Review **functions.py**, which contains an astronomy
    assistant's helper functions.

> **Try it first**: Look at `next_visible_event(location)` in **functions.py**. How would
> you describe its single `location` parameter to the model so it knows when and how to
> call it? Write the JSON schema before revealing the solution.

<details markdown="1">
<summary>Show a solution</summary>

Work through the comments in **agent.py**. Add references and connect to the project (the
same pattern as Task 2), then:

1. **Define the three function tools.** Each schema tells the model how to call one of the
    Python functions — for example the event tool:

    ```python
    # Define the event function tool
    event_tool = FunctionTool(
        name="next_visible_event",
        description="Get the next visible event in a given location.",
        parameters={
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "continent to find the next visible event in (e.g. 'north_america', 'south_america', 'australia')",
                },
            },
            "required": ["location"],
            "additionalProperties": False,
        },
        strict=True,
    )
    ```

    Define `cost_tool` (`calculate_observation_cost`) and `report_tool`
    (`generate_observation_report`) the same way, matching each function's parameters.

2. **Create the agent with all three tools:**

    ```python
    agent = project_client.agents.create_version(
        agent_name="astronomy-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""You are an astronomy observations assistant that helps users find
                information about astronomical events and calculate telescope rental costs.
                Use the available tools to assist users with their inquiries.""",
            tools=[event_tool, cost_tool, report_tool],
        ),
    )
    ```

3. **Run the tool-calling loop** — read each `function_call` from the response, run the
    matching Python function, return a `FunctionCallOutput`, then send the outputs back:

    ```python
    # Process function calls
    for item in response.output:
        if item.type == "function_call":
            result = None
            if item.name == "next_visible_event":
                result = next_visible_event(**json.loads(item.arguments))
            elif item.name == "calculate_observation_cost":
                result = calculate_observation_cost(**json.loads(item.arguments))
            elif item.name == "generate_observation_report":
                result = generate_observation_report(**json.loads(item.arguments))
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result,
                )
            )

    # Send outputs back and print the final answer
    if input_list:
        response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )
    print(f"AGENT: {response.output_text}")
    ```

Run `python agent.py` and try a prompt that needs **two** tools at once:

```
Find me the next event I can see from South America and give me the cost for 5 hours of premium telescope time at normal priority.
```

The agent calls both functions in one turn and combines the results, for example:

```
AGENT: The next event visible from South America is the Jupiter-Venus Conjunction on May 1st.
The cost for 5 hours of premium telescope time at normal priority is $1,875.
```

Enter `quit` to exit.

</details>

**Stretch**: add a fourth function tool of your own and update the instructions to mention it.

</details>

<details markdown="1" class="opt-task" data-tier="3">
<summary><strong>Task 5 — Build your own MCP server</strong> &middot; ★★★ &middot; ~30 min</summary>

**Goal**: Instead of connecting to someone else's MCP server, host your **own** tools and
connect an agent to them.

**Concept reinforced**: the MCP server/client split — a server *registers* tools; a client
*discovers and calls* them on the agent's behalf.

**Set up:**

1. Open the `Labfiles/A-build-and-extend-ai-agents/mcp/Python` folder (you set up its
    virtual environment and **.env** in Task 2). You'll edit **server.py** and **client.py**.

> **Try it first**: Wire up **server.py** and **client.py** using the comments in each file.
> As you go, consider: why must diagnostic output go to `stderr` (or be suppressed) rather
> than `stdout`? *(Hint: MCP speaks JSON-RPC over stdio, so anything printed to stdout is
> parsed as protocol messages — a stray banner corrupts the stream. That's why the server
> starts with `show_banner=False`.)*

<details markdown="1">
<summary>Show a solution</summary>

**In `server.py`** — create the server and expose the two provided functions as tools:

```python
# Add references
from fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP(name="Inventory")

@mcp.tool()
def get_inventory_levels() -> dict:
    ...  # returns the sample inventory dict already in the file

@mcp.tool()
def get_weekly_sales() -> dict:
    ...  # returns the sample sales dict already in the file

# Run the MCP server
mcp.run(show_banner=False)
```

**In `client.py`** — connect to the server, discover its tools, and register them with an agent:

1. Start the server over stdio and open a session, then list the available tools:

    ```python
    stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
    stdio, write = stdio_transport
    session = await exit_stack.enter_async_context(ClientSession(stdio, write))
    await session.initialize()
    tools = (await session.list_tools()).tools
    ```

2. Wrap each MCP tool as a callable and build `FunctionTool` definitions the agent can use:

    ```python
    def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            return await session.call_tool(tool_name, kwargs)
        tool_func.__name__ = tool_name
        return tool_func

    functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}

    mcp_function_tools = [
        FunctionTool(
            name=tool.name,
            description=tool.description,
            parameters={"type": "object", "properties": {}, "additionalProperties": False},
            strict=True,
        )
        for tool in tools
    ]
    ```

3. Create the agent with those tools, then run the tool-calling loop (same pattern as
    Task 4) — but invoke the wrapped **async** MCP function for each `function_call`:

    ```python
    agent = project_client.agents.create_version(
        agent_name="inventory-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""
            You are an inventory assistant. Here are some general guidelines:
            - Recommend restock if item inventory < 10 and weekly sales > 15
            - Recommend clearance if item inventory > 20 and weekly sales < 5
            """,
            tools=mcp_function_tools,
        ),
    )

    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)
            output = await functions_dict[item.name](**kwargs)
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=output.content[0].text,
                )
            )
    ```

Run the client and chat with the agent:

```
python client.py
```

```
Show me the current inventory levels for all products.
```

The agent calls your custom tools and answers from the returned data. Because the
conversation is stateful, follow-ups like *"Are there any products that should be
restocked?"* work too. Enter `quit` to exit.

</details>

**Stretch**: add a third tool (for example, `get_reorder_threshold`) and watch the agent
discover it without any other code changes.

</details>

## Summary

In this exercise you:

- Created and **grounded** an agent in the Foundry portal so it answers from your data.
- **Extended an agent with a tool** by connecting it to a remote MCP server and handling
  tool-approval requests in code.
- (Optionally) consumed an agent from a **client app**, added **custom function tools**,
  and built your **own MCP server**.

Together these show the two big levers for making agents useful: giving them the right
**knowledge** (grounding) and the right **capabilities** (tools).

## Clean up

If you're finished, delete the resources you created to avoid unnecessary Azure costs.

1. In the [Azure portal](https://portal.azure.com), navigate to the resource group that contains your Foundry resource.
1. On the toolbar, select **Delete resource group**, enter the resource group name, and confirm.

> The code you ran in Task 2 already deletes the agent version it creates. Portal
> agents are removed when you delete the resource group.
