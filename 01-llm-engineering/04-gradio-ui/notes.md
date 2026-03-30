# Gradio UI — Building Web Apps for AI in Python

> **Track:** LLM Engineering
> **Level:** Beginner → Intermediate
> **Last updated:** 2026-03-26
> **Status:** Final

---

## 1. What is Gradio?

Gradio is a **Python library** that lets you build **web UIs** for your Python functions — especially AI and ML models. You write Python, and Gradio creates a web page with inputs, buttons, and outputs for you.

**In simple words:** you write the brain (Python logic), Gradio gives it a face (web page).

```
Your Python Function  +  Gradio  =  Web App with UI
```

### Key facts

- Made by **Hugging Face**
- Works with **any Python function**
- **No HTML, CSS, or JavaScript** needed
- Runs **locally** or can be shared with a **public link**
- Used heavily in AI/ML demos and tools

---

## 2. Why Use Gradio?

| If you want to... | Without Gradio | With Gradio |
|---|---|---|
| Show AI output to users | Build a Flask/React app (days) | 10 lines of Python (minutes) |
| Let someone test your model | Send them a script | Send them a link |
| Build a quick demo | HTML + CSS + JS + backend | Just Python |
| Create an internal tool | Full web app | One `.py` file |

### When Gradio is a good fit

- AI/ML model demos
- Internal tools for your team
- Hackathon and portfolio projects
- Prototyping before building a full frontend

### When Gradio is NOT the best fit

- Production apps with millions of users
- Apps that need pixel-perfect custom design
- Apps that need complex login/authentication
- Mobile-first apps

---

## 3. Installation and Setup

### Install

```bash
pip install gradio
```

### Check the version

```bash
python -c "import gradio; print(gradio.__version__)"
```

### Smallest working app

```python
import gradio as gr

def greet(name):
    return f"Hello, {name}!"

app = gr.Interface(fn=greet, inputs="text", outputs="text")
app.launch()
```

Run it with `python app.py` and open `http://127.0.0.1:7860` in your browser.

---

## 4. Two Ways to Build: Interface vs Blocks

Gradio gives you two ways to build. Think of them as **"easy mode"** and **"full control mode."**

### Option A: `gr.Interface` (simple)

Best for: **one function, one input, one output.**

```python
import gradio as gr

def add(a, b):
    return a + b

app = gr.Interface(
    fn=add,
    inputs=[gr.Number(label="A"), gr.Number(label="B")],
    outputs=gr.Number(label="Result"),
    title="Addition Calculator"
)
app.launch()
```

### Option B: `gr.Blocks` (full control)

Best for: **custom layouts, multiple sections, tabs, complex apps.**

```python
import gradio as gr

with gr.Blocks() as app:
    gr.Markdown("# My App")
    name = gr.Textbox(label="Name")
    output = gr.Textbox(label="Greeting")
    btn = gr.Button("Greet")
    btn.click(fn=lambda n: f"Hello, {n}!", inputs=name, outputs=output)

app.launch()
```

### Which one should you use?

| Scenario | Use |
|---|---|
| Quick demo with 1 function | `gr.Interface` |
| Custom layout with rows/columns | `gr.Blocks` |
| Multiple tabs or sections | `gr.Blocks` |
| Multiple buttons doing different things | `gr.Blocks` |
| Real project | `gr.Blocks` (almost always) |

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses `gr.Blocks` because it has a custom layout with rows, columns, tabs, and multiple output sections. See `app.py` → `build_ui()`.

---

## 5. Components (UI Building Blocks)

Components are the individual UI pieces — text boxes, buttons, dropdowns, etc. Think of them like LEGO pieces.

### Input components

| Component | What it looks like | Code |
|---|---|---|
| **Textbox** | Single or multi-line text input | `gr.Textbox(label="Name", lines=3)` |
| **Number** | Number input with arrows | `gr.Number(label="Age")` |
| **Slider** | Draggable range slider | `gr.Slider(minimum=0, maximum=100)` |
| **Dropdown** | Select from options | `gr.Dropdown(choices=["A", "B", "C"])` |
| **Checkbox** | True/False toggle | `gr.Checkbox(label="Agree?")` |
| **Radio** | Pick one from options | `gr.Radio(choices=["Yes", "No"])` |
| **File** | File upload | `gr.File(label="Upload CSV")` |
| **Image** | Image upload | `gr.Image(label="Upload Photo")` |

### Output components

| Component | What it shows | Code |
|---|---|---|
| **Textbox** | Plain text result | `gr.Textbox(label="Result", interactive=False)` |
| **Markdown** | Formatted text (bold, headers, links) | `gr.Markdown("**Hello**")` |
| **JSON** | Formatted JSON data | `gr.JSON(label="API Response")` |
| **Image** | Generated/processed image | `gr.Image(label="Output")` |
| **Dataframe** | Table of data | `gr.Dataframe()` |

### Key properties (work on most components)

```python
gr.Textbox(
    label="Your Prompt",
    placeholder="Type here...",
    lines=6,
    interactive=False,
    show_label=False,
    value="default text",
)
```

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses `gr.Textbox` for input, `gr.Textbox(interactive=False)` for read-only output, `gr.Markdown` for score display, and `gr.Button` for the analyze trigger.

---

## 6. Layout (Arranging Components)

Layout controls **where** components appear on the page. Three main tools:

### `gr.Row()` — side by side (horizontal)

```python
with gr.Row():
    gr.Textbox(label="Left")
    gr.Textbox(label="Right")
```

```
┌──────────┐ ┌──────────┐
│   Left   │ │   Right  │
└──────────┘ └──────────┘
```

### `gr.Column()` — stacked (vertical)

```python
with gr.Column():
    gr.Textbox(label="Top")
    gr.Textbox(label="Bottom")
```

```
┌──────────┐
│   Top    │
└──────────┘
┌──────────┐
│  Bottom  │
└──────────┘
```

### `gr.Tab()` — tabbed sections

```python
with gr.Tab("Tab 1"):
    gr.Textbox(label="Content 1")

with gr.Tab("Tab 2"):
    gr.Textbox(label="Content 2")
```

```
[ Tab 1 ] [ Tab 2 ]
┌──────────────────┐
│   Content 1      │
└──────────────────┘
```

### Nesting layouts

You can put rows inside columns, columns inside rows, etc.

```python
with gr.Row():
    with gr.Column(scale=1):
        gr.Textbox(label="Input")
        gr.Button("Submit")
    with gr.Column(scale=1):
        gr.Textbox(label="Output")
```

The `scale` parameter controls how much space each column takes. Equal scale = equal width.

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) nests a `Row` with two `Column`s — left column has input + button, right column has score + intent. Below that, two `Tab`s show "Improved Prompt" and "Before vs After."

### PromptForge layout structure

```
┌─ Row ──────────────────────────────────┐
│ ┌─ Column ──────┐ ┌─ Column ────────┐ │
│ │ Textbox(input) │ │ Markdown(score) │ │
│ │ Button         │ │ Textbox(intent) │ │
│ └────────────────┘ └─────────────────┘ │
└────────────────────────────────────────┘
┌─ Textbox (issues) ────────────────────┐
└────────────────────────────────────────┘
┌─ Tab: Improved Prompt ────────────────┐
│ Textbox (optimized version)           │
└────────────────────────────────────────┘
┌─ Tab: Before vs After ────────────────┐
│ ┌─ Column ──────┐ ┌─ Column ────────┐ │
│ │ Original      │ │ Improved        │ │
│ └────────────────┘ └─────────────────┘ │
└────────────────────────────────────────┘
```

---

## 7. Events (Making Things Interactive)

Events connect **user actions** (like clicking a button) to **Python functions**. This is the glue between UI and logic.

### Basic pattern

```python
button.click(
    fn=my_function,
    inputs=[input_box],
    outputs=[output_box]
)
```

### How it works step by step

```
User clicks button
       │
       ▼
Gradio reads value from inputs (e.g., text from a Textbox)
       │
       ▼
Gradio calls fn(input_value)
       │
       ▼
Function returns result
       │
       ▼
Gradio puts result into outputs (e.g., shows text in another Textbox)
```

### Multiple inputs and outputs

Your function can take multiple inputs and return multiple outputs:

```python
def process(name, age):
    greeting = f"Hello {name}"
    status = f"You are {age} years old"
    return greeting, status

btn.click(
    fn=process,
    inputs=[name_box, age_box],
    outputs=[greeting_box, status_box]
)
```

**Important:** the order of `inputs` must match the function parameters. The order of `outputs` must match the return values.

### Other events (not just clicks)

| Event | When it fires | Code |
|---|---|---|
| `.click()` | Button is clicked | `btn.click(fn=..., inputs=..., outputs=...)` |
| `.change()` | Value changes | `textbox.change(fn=..., inputs=..., outputs=...)` |
| `.submit()` | User presses Enter | `textbox.submit(fn=..., inputs=..., outputs=...)` |
| `.select()` | Item is selected | `dropdown.select(fn=..., inputs=..., outputs=...)` |

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) connects the "Analyze Prompt" button click to the `on_analyze` function. It takes 1 input (the prompt textbox) and returns 6 outputs (intent, issues, improved prompt, score, original display, improved display).

---

## 8. Themes (Styling Your App)

Themes change the overall look — colors, fonts, spacing. No CSS needed.

### Built-in themes

```python
gr.Blocks(theme=gr.themes.Default())      # standard look
gr.Blocks(theme=gr.themes.Soft())         # softer, rounded
gr.Blocks(theme=gr.themes.Glass())        # glassmorphism style
gr.Blocks(theme=gr.themes.Monochrome())   # black and white
```

### Customizing a theme

```python
gr.Blocks(
    theme=gr.themes.Soft(
        primary_hue="indigo",
        secondary_hue="blue",
        neutral_hue="slate",
    )
)
```

Available color hues: `red`, `orange`, `yellow`, `green`, `teal`, `cyan`, `blue`, `indigo`, `violet`, `purple`, `pink`, `slate`, `gray`, `zinc`, `stone`

### Dark mode

Gradio supports dark mode automatically. Users can toggle it from the settings icon. It works out of the box — you do not need to do anything.

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses `gr.themes.Soft(primary_hue="indigo")` for a clean, modern purple-accented look.

---

## 9. Launch Options

The `launch()` method starts your Gradio server. You can control how it starts:

```python
app.launch(
    server_name="0.0.0.0",
    server_port=8080,
    share=True,
    show_api=False,
    inbrowser=True,
)
```

### Common setups

| Scenario | Code |
|---|---|
| Local development | `app.launch()` |
| Show to a friend | `app.launch(share=True)` |
| Deploy on a server | `app.launch(server_name="0.0.0.0")` |
| Clean footer (hide API link) | `app.launch(show_api=False)` |

### What `share=True` does

Gradio creates a temporary public URL like `https://abc123.gradio.live`. Anyone with the link can use your app — even if it runs on your laptop. The link expires after 72 hours.

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses `app.launch(show_api=False)` to hide the API docs link and keep the UI clean.

---

## 10. State Management

Sometimes you need to pass data between function calls without showing it in the UI. That is what `gr.State` does.

### What is State?

State is an **invisible storage box**. It holds a value that can be passed between events, but the user never sees it on screen.

```python
counter = gr.State(value=0)

def increment(count):
    return count + 1, count + 1

btn.click(fn=increment, inputs=[counter], outputs=[display, counter])
```

### When to use State

| Situation | Use State? |
|---|---|
| Pass data between button clicks | Yes |
| Store user session data | Yes |
| Show data on screen | No — use a regular component |
| Store large files | No — use file paths instead |

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) creates `gr.State("")` to hold the improved prompt value for passing between the main view and the comparison tab.

---

## 11. Common Patterns

### Pattern 1: Loading state

Gradio automatically shows a **loading spinner** on output components while the function runs. You do not need to add anything — it is built in.

### Pattern 2: Error handling

Wrap your function logic in try/except and return error messages as strings:

```python
def process(text):
    if not text.strip():
        return "Please enter some text."
    try:
        result = call_ai(text)
        return result
    except Exception as e:
        return f"Error: {e}"
```

> **See it in action:** [PromptForge](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) checks for empty input, missing API key, and API errors — all return clean messages instead of crashing.

### Pattern 3: Multiple return values

When your function returns multiple values, Gradio maps them to outputs **in order**:

```python
def analyze(text):
    return "intent here", "issues here", "improved here"

btn.click(
    fn=analyze,
    inputs=[input_box],
    outputs=[intent_box, issues_box, improved_box]
)
```

### Pattern 4: Using Markdown for rich output

`gr.Markdown` supports bold, headers, lists, and links:

```python
output = gr.Markdown()

def show_result(data):
    return f"**Score: {data['score']}**\n\n{data['reason']}"
```

---

## 12. Deployment

### Option 1: Hugging Face Spaces (free, easiest)

1. Create an account on [huggingface.co](https://huggingface.co)
2. Create a new Space (select Gradio as the SDK)
3. Push your code to the Space's git repo
4. It deploys automatically

### Option 2: Any server with Python

```bash
pip install gradio
python app.py
```

Use `server_name="0.0.0.0"` to make it accessible on the network.

### Option 3: Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

## 13. Quick Reference Cheatsheet

### Components

```python
gr.Textbox(label, placeholder, lines, interactive, value, show_label)
gr.Button(value, variant)          # variant: "primary", "secondary", "stop"
gr.Markdown(value)
gr.Number(label, value)
gr.Slider(minimum, maximum, step, label)
gr.Dropdown(choices, label, value)
gr.Checkbox(label, value)
gr.Radio(choices, label, value)
gr.Image(label, type)
gr.File(label, type)
gr.JSON(label)
gr.Dataframe(headers, datatype)
gr.State(value)
```

### Layout

```python
with gr.Row():          # horizontal
with gr.Column(scale):  # vertical
with gr.Tab(label):     # tabbed section
with gr.Accordion(label, open):  # collapsible section
with gr.Group():        # visual grouping (no spacing between children)
```

### Events

```python
component.click(fn, inputs, outputs)
component.change(fn, inputs, outputs)
component.submit(fn, inputs, outputs)
component.select(fn, inputs, outputs)
```

### Themes

```python
gr.themes.Default()
gr.themes.Soft(primary_hue, secondary_hue, neutral_hue)
gr.themes.Glass()
gr.themes.Monochrome()
```

### Launch

```python
app.launch(
    server_name,    # "0.0.0.0" for network access
    server_port,    # default 7860
    share,          # True for public link
    show_api,       # False to hide API docs
    inbrowser,      # True to auto-open browser
)
```

---

## 14. Real Project: PromptForge AI

Everything in these notes comes together in a real project:

**[Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer)** — an AI tool that takes any prompt, finds its problems, and rewrites it into a better version.

### What it uses

| Concept | How PromptForge uses it |
|---|---|
| **System Prompt** | Defines behavior as a prompt engineering expert |
| **Few-Shot Prompting** | Provides example critiques for consistent output |
| **Basic RAG** | Injects prompt engineering best practices as context |
| **Structured Output** | Returns intent, issues, improved prompt, and score in a parseable format |
| **Gradio** | The entire web UI — `gr.Blocks`, rows, columns, tabs, themes, events |

### Tech stack

- **Python** — core language
- **OpenAI API** — LLM backbone (GPT-4o-mini by default)
- **Gradio** — web UI

### Architecture

```
User Input (prompt)
       │
       ▼
┌─────────────────┐
│   Gradio UI      │   UI layer — input, results, before/after tabs
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Pipeline      │   Orchestrates the full analysis flow
└────────┬────────┘
         │
    ┌────┴─────────────────┐
    │                      │
    ▼                      ▼
┌──────────┐    ┌─────────────────┐
│ Prompts  │    │  RAG Knowledge  │   Context & examples
│ (system  │    │  Base (best     │
│  + few-  │    │   practices)    │
│  shot)   │    │                 │
└────┬─────┘    └────────┬────────┘
     │                   │
     └─────────┬─────────┘
               │
               ▼
       ┌──────────────┐
       │  LLM Service │   OpenAI API call
       │  (OpenAI)    │
       └──────┬───────┘
              │
              ▼
       ┌──────────────┐
       │    Parser     │   Extracts structured sections
       └──────┬───────┘
              │
              ▼
       Structured Output
       (Intent, Issues, Improved Prompt, Score)
```

> **GitHub:** [github.com/hasnaat-iftikhar/prompt-critic-and-optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer)

---

## 15. One-Line Summary

**Gradio** turns any Python function into a **web app** in minutes — no frontend code needed — making it the fastest way to put a face on your AI models and tools.

---

*Notes paired with the [PromptForge AI](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) project — a prompt critic and optimizer built with Python, OpenAI API, and Gradio.*
