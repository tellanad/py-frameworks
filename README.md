# Python Frameworks — LLM API Practice

A Python learning project for calling large language models programmatically. The current notebook demonstrates an OpenAI API request, with API credentials loaded from a local `.env` file.

## Progress so far

- Set up project dependencies for OpenAI, Anthropic, Google Gemini, and OpenRouter.
- Created a notebook that finds `.env` in the current directory or a parent directory and loads the API key.
- Added an explicit environment reload with `override=True` to pick up changed credentials.
- Implemented `query_openai(prompt)` using Chat Completions with `gpt-4.1-mini`, system/user messages, and temperature `0.7`.
- Extracted the assistant's text and added error reporting for completion requests.
- Ran a sample prompt successfully; the notebook includes a saved response.

Only the OpenAI example is implemented so far. Calls to Anthropic, Gemini, and OpenRouter are not yet implemented.

## Project files

| File | Purpose |
| --- | --- |
| `notebooks/llm_programatically_call.ipynb` | Environment setup and working OpenAI example |
| `pyproject.toml` | Project metadata, Python requirement, and dependencies |
| `uv.lock` | Dependency lockfile for uv |
| `requirements.txt` | Separate, unpinned dependency list |
| `main.py` | Starter script that prints a greeting |
| `.env` | Local API credentials; excluded from Git |

## Setup

Python **3.13 or later** is required by `pyproject.toml`. Run the following from the project root using uv:

```powershell
uv sync
uv pip install ipykernel
```

Alternatively, create a virtual environment and install the project dependencies with pip:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install . ipykernel
```

The commands above use `pyproject.toml` for the project dependencies. `ipykernel` is installed separately for running the notebook in VS Code; it is not currently declared as a project dependency.

Create a `.env` file in the project root:

```dotenv
OPENAI_APIKEY=your_api_key_here
```

Use the exact variable name `OPENAI_APIKEY`: the notebook reads it explicitly and passes it to the OpenAI client. The key's project must have access to `gpt-4.1-mini`. Keep real credentials out of notebook cells and outputs.

## Run the notebook

1. Open `notebooks/llm_programatically_call.ipynb` in VS Code with Python and Jupyter support installed.
2. Select the project's `.venv` Python environment as the notebook kernel.
3. Run all cells from top to bottom.
4. Edit `test_prompt` and rerun the prompt and invocation cells to try another question.

The final invocation is:

```python
print(query_openai(test_prompt))
```

The function returns the assistant's response text, or an error string if the completion request fails. API requests use your OpenAI account and may incur usage charges. Running `main.py` only prints a greeting; it does not call a model.

## Troubleshooting encountered during development

| Symptom | Resolution |
| --- | --- |
| `NameError: OPENAI_APIKEY is not defined` | Load `.env` and assign `OPENAI_APIKEY` before calling the function. |
| Missing required `messages` and `model` arguments | Use `messages=`, not `message=`, then rerun the function-definition cell. |
| A changed `.env` key is not picked up | Run the cell containing `load_dotenv(env_path, override=True)` and the key assignment, or restart the kernel and run all cells. |
| `403` stating the project does not have access to `gpt-4.1-mini` | Check the API key's project and its model access. Reload credentials after changing the key. |
| `StopIteration` during environment setup | Create `.env` in the project root so the notebook's parent-directory search can find it. |

The exception handler currently covers the completion request. Errors during environment setup or client construction occur outside that handler.

## Next steps

- Add separate examples for Anthropic, Google Gemini, and OpenRouter.
- Improve validation and error messages for missing configuration.
- Align `requirements.txt` with the dependencies in `pyproject.toml`.
