# Building a Personal AI Website Assistant with OpenAI, Gradio, and Pushover

## Introduction

In this project, we are building a simple but powerful AI assistant for a personal website.

Imagine someone visits Sahil’s website and asks:

```text
What is your background?
What skills do you have?
Can I contact you?
What projects have you worked on?
```

Instead of manually replying to every visitor, this chatbot answers on Sahil’s behalf. It reads Sahil’s LinkedIn profile and summary, uses OpenAI to generate professional answers, and can even send a phone notification when someone shares their email or asks a question the bot cannot answer.

In simple words, this project is like a **smart receptionist for a personal portfolio website**.

---

# What This Project Does

This project creates an AI chatbot that can:

1. Answer questions about Sahil’s background, skills, experience, and career.
2. Read information from Sahil’s LinkedIn PDF and summary text file.
3. Talk professionally as if it is Sahil’s personal website assistant.
4. Ask visitors for their email if they want to connect.
5. Send a notification using Pushover when a visitor provides contact details.
6. Record questions that the chatbot does not know how to answer.
7. Show everything in a simple web interface using Gradio.

So the final result is a mini AI-powered personal website assistant.

---

# The Big Picture

The whole project has five major parts:

```text
1. Load environment variables
2. Read Sahil’s profile information
3. Define tools the AI can use
4. Create the chatbot logic
5. Launch the web interface
```

Let us understand each part slowly.

---

# 1. Importing the Required Libraries

The project begins with these imports:

```python
from dotenv import load_dotenv
import os
from openai import OpenAI
import json
import gradio as gr
from pydantic import BaseModel
from pypdf import PdfReader
import requests
```

These are the tools we need for the project.

## What each import means

```python
from dotenv import load_dotenv
```

This loads secret values from a `.env` file.

For example, API keys should not be written directly inside the code. Instead, they are stored in a `.env` file.

```python
import os
```

This helps Python read environment variables like API keys.

```python
from openai import OpenAI
```

This allows us to connect to the OpenAI API and use models like `gpt-4o-mini`.

```python
import json
```

This helps convert data between Python dictionaries and JSON strings.

OpenAI tool calls usually send arguments in JSON format, so we need this.

```python
import gradio as gr
```

Gradio creates the web interface where users can type messages and see replies.

```python
from pydantic import BaseModel
```

Pydantic is useful when we want structured data. In this particular code, it is imported but not heavily used yet.

```python
from pypdf import PdfReader
```

This helps read text from the LinkedIn PDF file.

```python
import requests
```

This is used to send HTTP requests, especially to the Pushover notification API.

---

# 2. Loading API Keys and Secrets

The next part is:

```python
load_dotenv(override=True)
openai = OpenAI()
```

This loads values from the `.env` file.

Your `.env` file may contain things like:

```text
OPENAI_API_KEY=your_openai_api_key
PUSHOVER_USER=your_pushover_user_key
PUSHOVER_TOKEN=your_pushover_app_token
```

Then this line creates an OpenAI client:

```python
openai = OpenAI()
```

This client is used later to ask the AI model questions.

---

# 3. Setting Up Pushover Notifications

The next part reads Pushover credentials:

```python
pushover_user = os.getenv("PUSHOVER_USER")
pushover_token = os.getenv("PUSHOVER_TOKEN")
pushover_url = "https://api.pushover.net/1/messages.json"
```

Pushover is a notification service. It can send alerts to your phone or desktop.

In this project, Pushover is used when:

* a visitor gives their email
* the chatbot cannot answer a question

This part checks whether the credentials are available:

```python
if not pushover_user or not pushover_token:
    raise ValueError("Pushover credentials are not set in environment variables.")
else:
    print("Pushover credentials loaded successfully.")
```

In simple words:

```text
If Pushover keys are missing, stop the program.
Otherwise, continue.
```

This is useful because without these keys, notifications cannot be sent.

---

# 4. The `push()` Function

Now we define a function called `push()`:

```python
def push(message):
    print(f"Push: {message}")
    payload = {
        "user": pushover_user,
        "token": pushover_token,
        "message": message
    }
    response = requests.post(pushover_url, data=payload)
    if response.status_code == 200:
        print("Notification sent successfully.")
    else:
        print("Failed to send notification.")
```

This function sends a notification through Pushover.

For example:

```python
push("Hello from the AI Engineer Agent!")
```

This sends a test notification.

## Layman explanation

Think of `push()` as a messenger.

Whenever something important happens, this function sends a message to your phone.

Example:

```text
Someone gave their email.
Someone asked a question the bot could not answer.
```

Instead of checking the website manually, you get notified automatically.

---

# 5. Tool 1: Recording User Details

The first tool is:

```python
def record_user_details(email, name="Name not provided", notes="No additional notes"):
    push(f"New user details recorded:\nEmail: {email}\nName: {name}\nNotes: {notes}")
    return {"status": "success", "message": "User details recorded and notification sent."}
```

This function is used when a visitor gives their email.

For example, a visitor may say:

```text
Hi, I liked your profile. My email is john@example.com.
```

The AI can decide:

```text
This person wants to connect. I should record their details.
```

Then the tool sends a Pushover notification.

## What happens internally?

The function receives:

```python
email="john@example.com"
name="John"
notes="Interested in contacting Sahil"
```

Then it sends a notification like:

```text
New user details recorded:
Email: john@example.com
Name: John
Notes: Interested in contacting Sahil
```

This is useful because the chatbot is not just answering questions. It is also helping collect leads or contact requests.

---

# 6. Tool 2: Recording Unknown Questions

The second tool is:

```python
def record_unknown_question(question):
    push(f"Unknown question received: {question} that I couldn't answer. Please review and update the knowledge base.")
    return {"status": "success", "message": "Unknown question recorded and notification sent."}
```

This is used when the chatbot does not know the answer.

For example, someone asks:

```text
What was Sahil's exact GPA in his second semester?
```

If that information is not available in the LinkedIn PDF or summary file, the chatbot should not invent an answer.

Instead, it should record the question.

## Why is this important?

This helps improve the chatbot over time.

If many visitors ask questions the bot cannot answer, you can update `summary.txt` or your LinkedIn PDF with more information.

So this tool creates a feedback loop:

```text
Visitor asks unknown question
        ↓
Bot records it
        ↓
You get notified
        ↓
You improve the knowledge base
        ↓
Bot becomes better next time
```

---

# 7. Describing Tools to the AI

The Python functions alone are not enough.

We also need to tell the AI that these tools exist.

That is why we create tool descriptions in JSON format.

## Tool description for recording user details

```python
record_user_details_json = {
    "name": "record_user_details", 
    "description": "use this tool to record that a user is interested in being in touch and provided an email address.",
    "parameters": {
        "type": "object",
        "properties": {
            "email": {
                "type": "string",
                "description": "The email address of the user to be contacted."
            },
            "name": {
                "type": "string",
                "description": "The name of the user. This is optional and can be left out if not provided."
            },
            "notes": {
                "type": "string",
                "description": "Any additional notes about the user or their inquiry. This is optional and can be left out if not provided."
            }
        },
        "required": ["email"],
        "additionalProperties": False
    }
}
```

This tells the AI:

```text
There is a tool called record_user_details.
Use it when a user gives an email address.
The required field is email.
Name and notes are optional.
```

## Tool description for unknown questions

```python
record_unknown_question_json = {
    "name": "record_unknown_question",
    "description": "use this tool to record that a question was asked that the agent didn't know how to answer.",
    "parameters": {
        "type": "object",
        "properties": {
            "question": {
                "type": "string",
                "description": "The question that the agent was unable to answer."
            }
        },
        "required": ["question"],
        "additionalProperties": False
    }
}
```

This tells the AI:

```text
There is a tool called record_unknown_question.
Use it when you cannot answer a user’s question.
The required field is question.
```

---

# 8. Creating the Tools List

Now we combine both tool descriptions:

```python
tools = [
    {"type": "function", "function": record_user_details_json},
    {"type": "function", "function": record_unknown_question_json}
]
```

This list is passed to the OpenAI model.

That means the model knows it has access to these tools.

The AI does not directly run Python functions by itself. Instead, it asks our program to run them.

So the flow is:

```text
AI decides a tool is needed
        ↓
AI requests the tool call
        ↓
Python receives the request
        ↓
Python runs the actual function
        ↓
Python sends the result back to the AI
```

---

# 9. Handling Tool Calls

This function handles tool calls:

```python
def handle_tool_calls(tool_calls):
    results = []
    for tool_call in tool_calls:
        tool_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)
        print(f"Handling tool call: {tool_name} with arguments: {arguments}", flush=True)
        
        if tool_name in globals():
            tool = globals().get(tool_name)
            result = tool(**arguments) if tool else {"status": "error", "message": f"Tool {tool_name} not found."}
            results.append({"role": "tool", "content": json.dumps(result), "tool_call_id": tool_call.id})
    return results
```

This is one of the most important functions in the project.

## What does it do?

It receives a list of tools the AI wants to call.

For each tool call:

1. It reads the tool name.
2. It reads the arguments.
3. It finds the matching Python function.
4. It runs the function.
5. It returns the result back to the AI.

For example, if the AI says:

```text
Call record_user_details with email john@example.com
```

Then this function runs:

```python
record_user_details(email="john@example.com")
```

## Why `json.loads()` is used

The tool arguments come as JSON text.

Example:

```json
{"email": "john@example.com", "name": "John"}
```

Python cannot directly use this as a dictionary until we convert it.

So we use:

```python
arguments = json.loads(tool_call.function.arguments)
```

After this, it becomes a Python dictionary:

```python
{
    "email": "john@example.com",
    "name": "John"
}
```

Then we can call the function like this:

```python
tool(**arguments)
```

That means:

```python
record_user_details(email="john@example.com", name="John")
```

---

# 10. Reading Sahil’s LinkedIn PDF

The next part reads Sahil’s LinkedIn profile:

```python
reader = PdfReader("me/sahil_linkedin.pdf")
linkedin = ""
for page in reader.pages:
    text = page.extract_text()
    if text:
        linkedin += text + "\n"
```

This opens the PDF file:

```text
me/sahil_linkedin.pdf
```

Then it extracts text from each page.

All the text is stored in the variable:

```python
linkedin
```

This gives the chatbot real information about Sahil.

---

# 11. Reading the Summary File

The next part reads a text file:

```python
with open("me/summary.txt", "r", encoding="utf-8") as f:
    summary = f.read()
```

This file likely contains a short written summary about Sahil.

For example:

```text
Sahil is an AI engineer with experience in machine learning, deep learning, and software development...
```

Now the chatbot has two knowledge sources:

```text
1. LinkedIn PDF
2. Summary text file
```

---

# 12. Creating the System Prompt

This is the instruction given to the AI:

```python
name = "Sahil"

system_prompt = f"You are acting as {name}. You are answering questions on {name}'s website, \
particularly questions related to {name}'s career, background, skills and experience. \
Your responsibility is to represent {name} for interactions on the website as faithfully as possible. \
You are given a summary of {name}'s background and LinkedIn profile which you can use to answer questions. \
Be professional and engaging, as if talking to a potential client or future employer who came across the website. \
If you don't know the answer to any question, use your record_unknown_question tool to record the question that you couldn't answer, even if it's about something trivial or unrelated to career. \
If the user is engaging in discussion, try to steer them towards getting in touch via email; ask for their email and record it using your record_user_details tool. "
```

The system prompt tells the AI how to behave.

In simple language, it says:

```text
You are Sahil’s website assistant.
Answer questions about Sahil.
Use Sahil’s summary and LinkedIn profile.
Be professional.
If you do not know something, record the question.
If someone wants to connect, ask for their email.
```

Then the summary and LinkedIn text are added:

```python
system_prompt += f"\n\n## Summary:\n{summary}\n\n## LinkedIn Profile:\n{linkedin}\n\n"
system_prompt += f"With this context, please chat with the user, always staying in character as {name}."
```

This gives the AI the information it needs to answer accurately.

## Why this matters

Without the system prompt, the AI is just a general chatbot.

With the system prompt, it becomes:

```text
Sahil’s professional website assistant
```

---

# 13. The Main Chat Function

Now we reach the heart of the project:

```python
def chat(message, history):
```

This function runs every time a user sends a message.

Example:

```text
User: What is your background?
```

Then Gradio sends this message into the `chat()` function.

---

## Step 1: Handle empty history

```python
if history is None:
    history = []
```

When the conversation starts, there may be no previous messages.

So `history` may be `None`.

This line prevents errors by converting it into an empty list.

---

## Step 2: Start with the system prompt

```python
messages = [{"role": "system", "content": system_prompt}]
```

This creates the first message for the AI.

The system message gives the AI its instructions.

---

## Step 3: Add previous conversation history

```python
for user_msg, assistant_msg in history:
    messages.append({"role": "user", "content": user_msg})
    messages.append({"role": "assistant", "content": assistant_msg})
```

This adds older conversation messages.

For example:

```text
User: Hi
Assistant: Hello, how can I help?
User: What skills do you have?
Assistant: I have experience in AI and machine learning...
```

This helps the AI remember the conversation context.

---

## Step 4: Add the latest user message

```python
messages.append({"role": "user", "content": message})
```

This adds the new question the user just asked.

Now the final message list contains:

```text
System instruction
Previous user messages
Previous assistant replies
Latest user question
```

---

# 14. Calling the OpenAI Model

Inside the chat function, this line calls OpenAI:

```python
response = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    tools=tools
)
```

This sends everything to the model:

```text
System prompt
Conversation history
User question
Available tools
```

The model then decides one of two things:

```text
1. I can answer directly.
2. I need to call a tool first.
```

---

# 15. Why We Use a Loop

The code has this:

```python
done = False

while not done:
```

This loop exists because the model may need to call a tool before giving the final answer.

For example:

```text
User: My email is john@example.com. Please contact me.
```

The model may first say:

```text
I want to call record_user_details.
```

Then Python runs the tool.

Then the model gives the final answer:

```text
Thank you, John. I have recorded your contact details.
```

So the loop continues until the model gives a normal final response.

---

# 16. Checking Whether the AI Wants to Use a Tool

The code checks:

```python
finish_reason = response.choices[0].finish_reason
```

If the model wants to call a tool, the finish reason will be:

```python
"tool_calls"
```

Then this block runs:

```python
if finish_reason == "tool_calls":
    assistant_message = response.choices[0].message
    tool_calls = assistant_message.tool_calls

    tool_results = handle_tool_calls(tool_calls)

    messages.append(assistant_message)
    messages.extend(tool_results)
```

This means:

```text
The AI wants to use a tool.
Get the tool name and arguments.
Run the tool using handle_tool_calls().
Add the tool result back into the conversation.
Ask the model again.
```

If the model does not need a tool, this runs:

```python
else:
    done = True
```

That means:

```text
The AI has produced the final answer.
Stop the loop.
```

---

# 17. Returning the Final Reply

After the loop ends:

```python
reply = response.choices[0].message.content

history.append((message, reply))

return reply, history
```

This gets the final answer from the model.

Then it stores the latest user message and assistant reply in history.

Finally, it returns:

```text
1. The reply to show on screen
2. The updated conversation history
```

This is important because you are using Gradio with a `state`.

---

# 18. Launching the Web Interface

The final part is:

```python
gr.Interface(
    fn=chat,
    inputs=["text", "state"],
    outputs=["text", "state"]
).launch()
```

This creates a simple web app.

It gives the user:

```text
A text box to type a message
A submit button
An output box to see the AI reply
```

When the user submits a message, Gradio calls:

```python
chat(message, history)
```

Then it shows the AI reply.

---

# Complete Flow of the Project

Here is the full flow in simple terms:

```text
1. Start the program.
2. Load API keys from .env.
3. Connect to OpenAI.
4. Connect to Pushover.
5. Read Sahil’s LinkedIn PDF.
6. Read Sahil’s summary file.
7. Create a system prompt telling the AI to act as Sahil.
8. Define tools for contact details and unknown questions.
9. Launch the Gradio web app.
10. User asks a question.
11. AI answers using Sahil’s profile information.
12. If needed, AI calls a tool.
13. Python runs the tool.
14. Pushover sends a notification.
15. Final answer is shown to the user.
```

---

# A Real Example

Suppose a visitor types:

```text
What is your background?
```

The flow is:

```text
Gradio receives the question.
        ↓
chat() function runs.
        ↓
The system prompt and Sahil’s profile are sent to OpenAI.
        ↓
OpenAI generates a professional answer.
        ↓
The answer appears in the Gradio interface.
```

Now suppose the visitor says:

```text
I would like to contact you. My email is john@example.com.
```

The flow is:

```text
Gradio receives the message.
        ↓
OpenAI sees an email address.
        ↓
OpenAI requests the record_user_details tool.
        ↓
Python runs record_user_details().
        ↓
Pushover sends Sahil a notification.
        ↓
OpenAI replies politely to the visitor.
```

Now suppose the visitor asks:

```text
What was your exact GPA in school?
```

If the answer is not in the summary or LinkedIn PDF:

```text
OpenAI decides it does not know.
        ↓
OpenAI requests the record_unknown_question tool.
        ↓
Python runs record_unknown_question().
        ↓
Pushover sends Sahil a notification.
        ↓
The chatbot responds without making up fake information.
```

---

# What Is the Main Idea?

The main idea is not just to build a chatbot.

The real idea is to build an **agent**.

A normal chatbot only talks.

An agent can:

```text
Think
Answer
Use tools
Take action
Remember context
Notify the owner
```

In this project, the AI can use tools such as:

```text
record_user_details
record_unknown_question
```

That makes it more powerful than a basic chatbot.

---

# Why This Project Is Useful

This project is useful for:

```text
Personal portfolio websites
Freelancer websites
Consultant websites
Startup landing pages
Academic profile pages
Recruiter-facing pages
Customer support prototypes
```

It can answer common questions and notify the owner when important things happen.

---

# Important Files in the Project

Your project depends on these files:

```text
.env
me/sahil_linkedin.pdf
me/summary.txt
```

## `.env`

Stores secrets:

```text
OPENAI_API_KEY
PUSHOVER_USER
PUSHOVER_TOKEN
```

## `me/sahil_linkedin.pdf`

Contains Sahil’s LinkedIn profile.

## `me/summary.txt`

Contains a short summary about Sahil’s background.

These files are used to personalize the assistant.

---

# Common Mistakes and Fixes

## Mistake 1: Pushover credentials missing

If you see:

```text
Pushover credentials are not set in environment variables.
```

It means your `.env` file does not have:

```text
PUSHOVER_USER
PUSHOVER_TOKEN
```

## Mistake 2: History is None

If you see:

```text
TypeError: can only concatenate list (not "NoneType") to list
```

It means `history` is empty at the beginning.

Fix:

```python
if history is None:
    history = []
```

## Mistake 3: Passing Python functions instead of JSON schemas

Wrong:

```python
tools = [
    {"type": "function", "function": record_user_details},
    {"type": "function", "function": record_unknown_question}
]
```

Correct:

```python
tools = [
    {"type": "function", "function": record_user_details_json},
    {"type": "function", "function": record_unknown_question_json}
]
```

The AI needs tool descriptions, not the actual Python functions.

## Mistake 4: PDF path is wrong

If you get a file error, check that this file exists:

```text
me/sahil_linkedin.pdf
```

## Mistake 5: Summary path is wrong

Check that this file exists:

```text
me/summary.txt
```

---

# How to Think About This Project as a Beginner

Think of the project like a restaurant receptionist.

The receptionist has:

```text
A menu
A notebook
A phone
A desk
```

In our project:

```text
LinkedIn PDF and summary = menu/information
OpenAI model = receptionist brain
Pushover = phone notification
Gradio = front desk
Tools = actions the receptionist can perform
```

When a visitor asks a question, the receptionist checks the information and replies.

If the visitor wants to connect, the receptionist writes down the contact details and sends a message to the owner.

If the visitor asks something unknown, the receptionist records the question.

That is exactly what this AI assistant does.

---

# Final Summary

This project builds a personal AI assistant for Sahil’s website.

It uses OpenAI to answer questions, Gradio to create a web interface, Pushover to send notifications, and PDF/text files to give the assistant knowledge about Sahil.

The assistant can answer questions professionally, collect visitor contact details, and record questions it cannot answer.

In simple words:

```text
This is an AI-powered personal website receptionist.
It knows your background.
It talks to visitors.
It collects leads.
It alerts you when something important happens.
```

That is the complete idea behind this project.
