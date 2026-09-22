# IT Support Agent

[This notebook](../notebooks/It_support_agent.ipynb) builds a small IT support agent that investigates server problems. You give it an issue, such as a slow payment server, and it can check the server's health, read its logs, and decide what to do next.

It uses `gpt-4.1-mini` to choose the next step. Python runs the requested tools and passes the results back so the model can continue investigating. The server data and actions are simulated, which makes this a useful way to practice the full workflow without connecting to real infrastructure.

## Getting started

Follow the [project setup instructions](../README.md#setup), then add your API key to a `.env` file in the project root:

```dotenv
OPENAI_APIKEY=your_api_key_here
```

Use `OPENAI_APIKEY` exactly as shown, since that's the name the notebook reads. It looks for `.env` in the working directory and its parent folders. Running the setup cells again reloads the values from that file.

Open the notebook, select the project's `.venv` kernel, and run the cells from top to bottom. The final cell starts with this incident:

```python
run_it_agent("The payment-server-01 is extremely slow and timing out.")
```

You'll see messages as the agent requests tools, followed by a `[FINAL RESPONSE]` explaining what it found and did.

One small detail: the earlier `Sending data to OpenAI API...` message only prints some sample JSON. The actual API calls happen in `run_it_agent`.

## What the tools do

The agent has four Python functions available:

| Tool | What it does |
| --- | --- |
| `get_server_health(server_id)` | Looks up the server's sample CPU usage, memory usage, and status. |
| `fetch_recent_logs(server_id, lines=8)` | Returns recent sample logs, using the last eight entries by default. |
| `restart_service(server_id)` | Prints a restart message and returns a simulated success result. |
| `escalate_to_engineer(summary)` | Returns a simulated escalation result. It doesn't yet save the summary or contact anyone. |

Each function returns its result as a JSON string. `tools_schema` tells the model what tools are available and what arguments they accept. `AVAILABLE_FUNCTIONS` lets Python find the function when the model requests it.

For an unknown server, the health tool returns `N/A` metrics and an `Unknown` status. The log tool returns generic fallback entries, so those logs don't establish that an unknown server is healthy.

## How an investigation works

Every call to `run_it_agent` starts a new conversation with the incident and the instructions for handling it.

The model reads that conversation and may ask for one or more tools. Python runs each requested function, adds the results to the conversation, and asks the model for the next step. This repeats until the model replies without requesting another tool. The notebook then prints the answer and ends the loop. The function itself returns `None`.

Each tool result carries the original `tool_call_id`, which connects the result to the request. All requested tools need a response before the next API call.

That is also why the tool-handling `if/else` belongs inside `while True`: the agent needs to process the results on each pass, and the final `break` needs to be inside the loop.

## Scenarios to try

The agent's instructions say to restart when CPU or memory usage is above 90%. If the logs show a critical dependency failure that a restart won't solve, it should escalate to an engineer. The model follows these instructions; there is no separate Python check enforcing the thresholds.

The notebook includes five sample servers:

| Server | What's happening | Expected action |
| --- | --- | --- |
| `payment-server-01` | CPU is at 98%, with a hung process and timeouts. | Restart the service. |
| `db-node-02` | CPU is at 12%, memory is at 60%, and the logs look healthy. | Report that the server looks healthy. |
| `auth-service-03` | Memory is at 95%, with out-of-memory errors and a memory leak. | Restart under the memory rule. The leak may need further investigation. |
| `search-index-09` | CPU and memory are low, but connections to the search dependency are failing. | Escalate to an engineer. |
| `frontend-node-04` | CPU is at 25%, memory is at 30%, and requests are succeeding. | Report that the server looks healthy. |

The final notebook cell runs only the payment-server example. To explore the others, add a cell after it and try:

```python
run_it_agent("Check whether db-node-02 is healthy.")
run_it_agent("The auth-service-03 keeps crashing. Investigate.")
run_it_agent("The search-index-09 cannot index documents.")
run_it_agent("Check the health of frontend-node-04.")
```

The order of tool calls and the wording of the answer can change between runs because the model chooses how to investigate.

## If something goes wrong

| Problem | What to try |
| --- | --- |
| `StopIteration` during setup | Check that `.env` exists in the working directory or a parent folder. |
| A missing-key `RuntimeError` | Check the spelling of `OPENAI_APIKEY`, then rerun the setup cells. |
| `SyntaxError: 'break' outside loop` | Check that the final `else` and its `break` are indented inside `while True`. |
| HTTP 400 about missing tool responses | Check that every tool call gets a result with the matching `tool_call_id` before the next request. |
| Your edit doesn't seem to take effect | Rerun the function-definition cell, or restart the kernel and run all cells. If a definition fails, an older version can remain in memory. |

## What's still unfinished

The notebook demonstrates the investigation loop, but a few parts are still placeholders. Restarting always reports success and doesn't change the saved metrics or logs. Escalating returns a message without creating a ticket or notifying an engineer. The restart tool description and escalation docstring also still contain TODOs.

The loop has no limit on the number of requests, and API or tool errors aren't caught. An unknown tool name is skipped, leaving a missing tool response in the conversation. Those are useful next improvements if you want to build on this example.
