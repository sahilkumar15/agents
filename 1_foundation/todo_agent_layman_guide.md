# Layman Guide: Building a Tool-Using AI Todo Agent

## 1. What Is This Project?

This project is a small **AI agent** that can solve a problem by first making a todo list, then completing each step one by one.

It is not just a normal chatbot.

A normal chatbot usually works like this:

```text
User asks question
        ↓
AI gives answer
```

But this project works more like this:

```text
User gives a problem
        ↓
AI creates a plan
        ↓
AI adds the plan to a todo list
        ↓
AI completes each todo step
        ↓
AI gives the final answer
```

So the main idea is:

> We are teaching the AI to solve problems in an organized way by using tools.

This is why it is called an **agentic AI pattern**. The AI is not only generating text. It is also using Python functions as tools to manage its work.

---

## 2. Why Do We Need This Project?

When we ask an AI a complex question, it may directly jump to the answer. Sometimes that answer is correct, but sometimes it misses steps.

For example:

```text
A train leaves Boston at 2:00 pm traveling 60 mph.
Another train leaves New York at 3:00 pm traveling 80 mph toward Boston.
When do they meet?
```

A normal chatbot might answer quickly.

But this project makes the AI behave more carefully:

```text
1. Understand the problem.
2. Identify missing information.
3. Create a todo list.
4. Complete each step.
5. Return the final answer.
```

This is useful because many real-world tasks need planning.

Examples:

```text
Solving math word problems
Planning research steps
Debugging code
Writing project plans
Breaking large tasks into smaller tasks
Tracking what has been completed
```

---

## 3. What Is the Main Application?

This project demonstrates how to build an AI agent that can use tools.

In this example, the tools are simple todo tools:

```text
create_todos
mark_complete
```

But in real applications, tools could be anything:

```text
Send email
Search database
Read PDF
Create calendar event
Call an API
Generate report
Update spreadsheet
Fetch weather
Run code
Create tickets
Save user details
```

So this project is a beginner-friendly example of a much bigger idea:

> AI becomes more useful when it can use tools, not just talk.

---

## 4. Layman Analogy

Imagine you ask a student to solve a problem.

A weak student may directly say the answer.

A better student does this:

```text
First, I will make a plan.
Then I will solve step 1.
Then I will solve step 2.
Then I will check the final answer.
```

Your AI agent is doing the same thing.

The todo list is like the student’s notebook.

The tools are like actions the student can perform.

The final answer is shown only after the work is done.

---

## 5. Main Parts of This Project

The project has these main parts:

```text
1. Imports
2. Environment setup
3. Rich display function
4. Todo storage
5. Todo tools
6. Tool JSON schemas
7. Tool handler
8. AI loop
9. System message
10. User problem
11. Final execution
```

---

## 6. Imports

The project starts with:

```python
from rich.console import Console
from dotenv import load_dotenv
import os
from openai import OpenAI
import json
import gradio as gr
from pydantic import BaseModel
from pypdf import PdfReader
import requests
```

### What each import does

`rich.console.Console` prints beautiful formatted text in the terminal or notebook. In this project, it helps show completed todos in green and strikethrough.

`dotenv` loads secret keys from a `.env` file.

`os` helps access environment variables.

`OpenAI` allows your Python code to call an OpenAI model.

`json` converts tool arguments from JSON text into Python dictionaries.

`gradio`, `pydantic`, `pypdf`, and `requests` may come from other related labs. For this specific todo-agent project, the most important imports are:

```python
from rich.console import Console
from dotenv import load_dotenv
from openai import OpenAI
import json
```

---

## 7. Loading Environment Variables

```python
load_dotenv(override=True)
openai = OpenAI()
```

This means:

```text
Load secrets from .env
Create an OpenAI client
```

The OpenAI client is what lets your Python program talk to the OpenAI model.

---

## 8. The `show()` Function

A better version is:

```python
console = Console()

def show(text):
    if text is None:
        return
    console.print(text)
```

### Why do we need `show()`?

Because we want Rich formatting to actually appear.

For example, this string:

```python
"[green][strike]Buy groceries[/strike][/green]"
```

will only look beautiful if Rich prints it.

If Python simply returns it, you may see the raw text:

```text
[green][strike]Buy groceries[/strike][/green]
```

But if Rich prints it, it appears as green strikethrough text.

---

## 9. Todo Storage

The project uses two lists:

```python
todos = []
completed = []
```

### What is `todos`?

This stores the task names.

Example:

```python
todos = ["Buy groceries", "Finish extra lab", "Eat banana"]
```

### What is `completed`?

This stores whether each task is done.

Example:

```python
completed = [False, False, False]
```

This means:

```text
Buy groceries       not done
Finish extra lab    not done
Eat banana          not done
```

If the first task is completed:

```python
completed = [True, False, False]
```

That means:

```text
Buy groceries       done
Finish extra lab    not done
Eat banana          not done
```

So the two lists work together.

---

## 10. Getting the Todo Report

```python
def get_todo_report() -> str:
    result = ""
    for index, todo in enumerate(todos):
        if completed[index]:
            result += f"Todo #{index + 1}: [green][strike]{todo}[/strike][/green]\n"
        else:
            result += f"Todo #{index + 1}: {todo}\n"
    show(result)
    return result
```

This function creates a readable todo list.

If a task is completed, it adds Rich markup:

```python
[green][strike]Buy groceries[/strike][/green]
```

That means:

```text
show it in green and crossed out
```

---

## 11. Creating Todos

The tool function should be:

```python
def create_todos(descriptions: list[str]) -> str:
    todos.extend(descriptions)
    completed.extend([False] * len(descriptions))
    return get_todo_report()
```

This function adds new todos.

Example:

```python
create_todos(["Buy groceries", "Finish extra lab", "Eat banana"])
```

After this:

```python
todos = ["Buy groceries", "Finish extra lab", "Eat banana"]
completed = [False, False, False]
```

### Important bug you faced

Your original function had:

```python
def create_todos(description: list[str]) -> str:
```

But your JSON schema used:

```python
"descriptions"
```

So OpenAI called:

```python
create_todos(descriptions=[...])
```

But your Python function expected:

```python
description
```

That mismatch caused:

```text
TypeError: create_todos() got an unexpected keyword argument 'descriptions'
```

The fix:

```python
def create_todos(descriptions: list[str]) -> str:
```

The function parameter name and JSON schema name must match.

---

## 12. Marking a Todo Complete

```python
def mark_complete(index: int, completion_notes: str) -> str:
    if not 1 <= index <= len(todos):
        return "No todo at this index."

    completed[index - 1] = True

    show(f"[bold green]Completed Todo #{index}:[/bold green] {completion_notes}")

    return get_todo_report()
```

This function marks one todo as completed.

### Why `index - 1`?

Humans count like this:

```text
Todo #1
Todo #2
Todo #3
```

But Python lists start at zero:

```text
todos[0]
todos[1]
todos[2]
```

So if the user says todo number 1, Python needs index 0.

That is why we write:

```python
completed[index - 1] = True
```

---

## 13. Tool JSON Schemas

The AI cannot automatically understand your Python functions. You need to describe the tools to the AI.

That is what the JSON schema does.

### Create Todos Tool Schema

```python
create_todos_json = {
    "name": "create_todos",
    "description": "Add new todos from a list of descriptions and return the full todo list.",
    "parameters": {
        "type": "object",
        "properties": {
            "descriptions": {
                "type": "array",
                "items": {"type": "string"},
                "description": "A list of todo descriptions to add."
            }
        },
        "required": ["descriptions"],
        "additionalProperties": False
    }
}
```

This tells the AI:

```text
There is a tool called create_todos.
It needs a list called descriptions.
Each item in that list is a string.
```

### Mark Complete Tool Schema

```python
mark_complete_json = {
    "name": "mark_complete",
    "description": "Mark complete the todo at the given position, starting from 1, and return the full todo list.",
    "parameters": {
        "type": "object",
        "properties": {
            "index": {
                "type": "integer",
                "description": "The 1-based index of the todo to mark as complete."
            },
            "completion_notes": {
                "type": "string",
                "description": "Notes about how the todo was completed. Rich markup is allowed."
            }
        },
        "required": ["index", "completion_notes"],
        "additionalProperties": False
    }
}
```

This tells the AI:

```text
There is a tool called mark_complete.
It needs an index and completion notes.
```

---

## 14. Creating the Tool List

```python
tools = [
    {"type": "function", "function": create_todos_json},
    {"type": "function", "function": mark_complete_json}
]
```

This gives the model a menu of tools.

Think of it like telling the AI:

```text
You are allowed to use these tools:
1. create_todos
2. mark_complete
```

The AI does not directly execute the Python code. It only requests a tool call. Then your Python program executes the function.

---

## 15. Why We Use `available_tools`

A safe version is:

```python
available_tools = {
    "create_todos": create_todos,
    "mark_complete": mark_complete
}
```

This maps tool names to actual Python functions.

This is safer than using `globals()` because it only allows approved tools.

---

## 16. Handling Tool Calls

```python
def handle_tool_calls(tool_calls):
    results = []

    for tool_call in tool_calls:
        tool_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)

        print(f"Handling tool call: {tool_name} with arguments: {arguments}", flush=True)

        tool = available_tools.get(tool_name)

        if tool is None:
            result = {
                "status": "error",
                "message": f"Tool {tool_name} not found."
            }
        else:
            result = tool(**arguments)

        results.append({
            "role": "tool",
            "content": json.dumps(result),
            "tool_call_id": tool_call.id
        })

    return results
```

This function is the bridge between the AI and Python.

The AI may say:

```text
I want to call create_todos.
```

The code receives:

```python
tool_name = "create_todos"
arguments = {"descriptions": ["Step 1", "Step 2"]}
```

Then Python runs:

```python
create_todos(descriptions=["Step 1", "Step 2"])
```

Then the result is sent back to the AI.

---

## 17. Why `tool(**arguments)` Is Important

Suppose:

```python
arguments = {
    "index": 1,
    "completion_notes": "Estimated the distance."
}
```

Then:

```python
tool(**arguments)
```

means:

```python
mark_complete(index=1, completion_notes="Estimated the distance.")
```

The `**` unpacks a dictionary into function arguments.

---

## 18. The AI Loop

```python
def loop(messages):
    done = False

    while not done:
        response = openai.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=tools
        )

        finish_reason = response.choices[0].finish_reason

        if finish_reason == "tool_calls":
            assistant_message = response.choices[0].message
            tool_calls = assistant_message.tool_calls

            results = handle_tool_calls(tool_calls)

            messages.append(assistant_message)
            messages.extend(results)

        else:
            done = True

    final_answer = response.choices[0].message.content
    show(final_answer)

    return final_answer
```

This is the main engine of the agent.

### Why do we need a loop?

Because the model may need several rounds:

```text
Round 1: AI creates todos
Round 2: AI marks todo 1 complete
Round 3: AI marks todo 2 complete
Round 4: AI marks todo 3 complete
Round 5: AI gives final answer
```

The loop keeps running until the AI stops asking for tools and gives a final answer.

---

## 19. Understanding `finish_reason`

```python
finish_reason = response.choices[0].finish_reason
```

If:

```python
finish_reason == "tool_calls"
```

it means:

```text
The AI wants to use a tool.
```

If not, it usually means:

```text
The AI gave a final answer.
```

---

## 20. The System Message

```python
system_message = """
You are given a problem to solve by using your todo tools to plan a list of steps, then carrying out each step in turn.

You must:
1. Create a todo list for solving the problem.
2. Complete each todo one by one using the tools.
3. Reply with the final solution.

If any quantity is not provided in the question, include a step to make a reasonable estimate.

Provide your solution in Rich console markup without code blocks.
Do not ask the user questions or clarification.
Respond only with the answer after using your tools.
"""
```

This is the instruction for the AI.

It tells the AI:

```text
Do not directly answer.
First create a plan.
Then complete each step.
Then answer.
```

This is what makes it agentic.

---

## 21. The User Problem

```python
user_message = """
A train leaves Boston at 2:00 pm traveling 60 mph.
Another train leaves New York at 3:00 pm traveling 80 mph toward Boston.
When do they meet?
"""
```

This is the problem we want the AI agent to solve.

The problem is missing one important piece:

```text
Distance between Boston and New York
```

The system message tells the AI:

```text
If a quantity is missing, make a reasonable estimate.
```

So the AI should include a todo like:

```text
Estimate the distance between Boston and New York.
```

---

## 22. Running the Agent

```python
messages = [
    {"role": "system", "content": system_message},
    {"role": "user", "content": user_message}
]

todos, completed = [], []

loop(messages)
```

This creates the conversation and runs the agent.

---

## 23. Expected Internal Flow

A good agent flow should look like this:

```text
1. Create todos:
   - Estimate Boston-New York distance.
   - Calculate first train distance before second train starts.
   - Calculate remaining distance.
   - Calculate closing speed.
   - Calculate meeting time.

2. Mark todo 1 complete.
3. Mark todo 2 complete.
4. Mark todo 3 complete.
5. Mark todo 4 complete.
6. Mark todo 5 complete.
7. Give final answer.
```

---

## 24. Expected Math Solution

The distance from Boston to New York is not given, so we estimate it as about:

```text
215 miles
```

The first train leaves Boston at 2:00 pm traveling 60 mph.

By 3:00 pm, it has traveled:

```text
60 miles
```

Remaining distance between trains at 3:00 pm:

```text
215 - 60 = 155 miles
```

At 3:00 pm, the New York train starts moving toward Boston at 80 mph.

The trains are moving toward each other, so their combined speed is:

```text
60 + 80 = 140 mph
```

Time to meet after 3:00 pm:

```text
155 / 140 = 1.107 hours
```

That is about:

```text
1 hour and 6 minutes
```

So they meet around:

```text
4:06 pm
```

---

## 25. What Error Did You Face?

You faced this error:

```text
TypeError: create_todos() got an unexpected keyword argument 'descriptions'
```

### Why did it happen?

Because your schema said:

```python
"descriptions"
```

but your Python function used:

```python
description
```

These names must match exactly.

### Wrong version

```python
def create_todos(description: list[str]) -> str:
```

### Correct version

```python
def create_todos(descriptions: list[str]) -> str:
```

---

## 26. Why Rich Formatting Looked Wrong

You saw something like:

```text
'Todo #1: [green][strike]Buy groceries[/strike][/green]\nTodo #2: Finish extra lab\n'
```

This is not actually wrong.

It is the raw string representation.

Rich markup only renders when printed by Rich:

```python
console.print(text)
```

So this:

```python
[green][strike]Buy groceries[/strike][/green]
```

means:

```text
When Rich prints it, show Buy groceries in green and strikethrough.
```

---

## 27. Corrected Full Code

```python
from rich.console import Console
from dotenv import load_dotenv
from openai import OpenAI
import json

load_dotenv(override=True)
openai = OpenAI()

console = Console()

todos = []
completed = []


def show(text):
    if text is None:
        return
    console.print(text)


def get_todo_report() -> str:
    result = ""

    for index, todo in enumerate(todos):
        if completed[index]:
            result += f"Todo #{index + 1}: [green][strike]{todo}[/strike][/green]\n"
        else:
            result += f"Todo #{index + 1}: {todo}\n"

    show(result)
    return result


def create_todos(descriptions: list[str]) -> str:
    todos.extend(descriptions)
    completed.extend([False] * len(descriptions))
    return get_todo_report()


def mark_complete(index: int, completion_notes: str) -> str:
    if not 1 <= index <= len(todos):
        return "No todo at this index."

    completed[index - 1] = True

    show(f"[bold green]Completed Todo #{index}:[/bold green] {completion_notes}")

    return get_todo_report()


create_todos_json = {
    "name": "create_todos",
    "description": "Add new todos from a list of descriptions and return the full todo list.",
    "parameters": {
        "type": "object",
        "properties": {
            "descriptions": {
                "type": "array",
                "items": {"type": "string"},
                "description": "A list of todo descriptions to add."
            }
        },
        "required": ["descriptions"],
        "additionalProperties": False
    }
}


mark_complete_json = {
    "name": "mark_complete",
    "description": "Mark complete the todo at the given position, starting from 1, and return the full todo list.",
    "parameters": {
        "type": "object",
        "properties": {
            "index": {
                "type": "integer",
                "description": "The 1-based index of the todo to mark as complete."
            },
            "completion_notes": {
                "type": "string",
                "description": "Notes about how the todo was completed. Rich markup is allowed."
            }
        },
        "required": ["index", "completion_notes"],
        "additionalProperties": False
    }
}


tools = [
    {"type": "function", "function": create_todos_json},
    {"type": "function", "function": mark_complete_json}
]


available_tools = {
    "create_todos": create_todos,
    "mark_complete": mark_complete
}


def handle_tool_calls(tool_calls):
    results = []

    for tool_call in tool_calls:
        tool_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)

        print(f"Handling tool call: {tool_name} with arguments: {arguments}", flush=True)

        tool = available_tools.get(tool_name)

        if tool is None:
            result = {
                "status": "error",
                "message": f"Tool {tool_name} not found."
            }
        else:
            result = tool(**arguments)

        results.append({
            "role": "tool",
            "content": json.dumps(result),
            "tool_call_id": tool_call.id
        })

    return results


def loop(messages):
    done = False

    while not done:
        response = openai.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=tools
        )

        finish_reason = response.choices[0].finish_reason

        if finish_reason == "tool_calls":
            assistant_message = response.choices[0].message
            tool_calls = assistant_message.tool_calls

            results = handle_tool_calls(tool_calls)

            messages.append(assistant_message)
            messages.extend(results)

        else:
            done = True

    final_answer = response.choices[0].message.content
    show(final_answer)

    return final_answer


system_message = """
You are given a problem to solve by using your todo tools to plan a list of steps, then carrying out each step in turn.

You must:
1. Create a todo list for solving the problem.
2. Complete each todo one by one using the tools.
3. Reply with the final solution.

If any quantity is not provided in the question, include a step to make a reasonable estimate.

Provide your solution in Rich console markup without code blocks.
Do not ask the user questions or clarification.
Respond only with the answer after using your tools.
"""


user_message = """
A train leaves Boston at 2:00 pm traveling 60 mph.
Another train leaves New York at 3:00 pm traveling 80 mph toward Boston.
When do they meet?
"""


messages = [
    {"role": "system", "content": system_message},
    {"role": "user", "content": user_message}
]


todos, completed = [], []

loop(messages)
```

---

## 28. Where Can This Project Be Used?

This todo-agent pattern can be used anywhere a task needs multiple steps.

### Math problem solving

The agent can create steps and solve each step one by one.

### Research planning

Example:

```text
Find papers about AI agents and summarize them.
```

The agent can create todos:

```text
Search papers
Read abstracts
Compare methods
Write summary
```

### Code debugging

Example:

```text
Fix my Python error.
```

The agent can create todos:

```text
Read error
Identify cause
Suggest fix
Test fixed code
```

### Personal productivity

Example:

```text
Plan my study schedule for tomorrow.
```

The agent can create todos and mark them complete.

### Customer support

The agent can create a checklist before giving a solution.

### Business workflows

Example:

```text
Prepare onboarding checklist for a new employee.
```

The agent can generate and track tasks.

---

## 29. What Is the Bigger Lesson?

The bigger lesson is:

> AI agents are built by combining language models with tools.

The language model is good at:

```text
Understanding language
Planning
Reasoning
Writing explanations
Choosing tools
```

Python tools are good at:

```text
Storing data
Running calculations
Calling APIs
Sending notifications
Updating files
Performing exact actions
```

Together, they become powerful.

---

## 30. Simple Final Summary

This project builds a small AI agent that solves a problem by using a todo list.

It works like this:

```text
User gives a problem
        ↓
AI creates a todo plan
        ↓
Python stores the todos
        ↓
AI marks each todo complete
        ↓
Python updates the todo list
        ↓
AI gives final answer
```

The project teaches you the foundation of tool-using AI agents.

Even though the current tools are simple, the same pattern can be used for much bigger applications like:

```text
AI personal assistants
Research assistants
Coding agents
Customer support bots
Workflow automation tools
Business process agents
```

In simple words:

> This project teaches how to make AI move from “just answering” to “planning and doing.”
