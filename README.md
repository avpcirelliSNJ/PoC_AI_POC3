


##  AI Agents & Process Automation

In this phase, you are building a **digital coworker**. An agent doesn't just talk—it _acts_. It perceives a goal, reasons through the steps, and uses "tools" (scripts, APIs, or searches) to complete a task.

## Phase 1: The "Zero-Cost" Lab Setup
Since we are working with limited means, we will use **Ollama** (to run the AI locally for free) and **CrewAI** (the framework to coordinate the agents).

### Install the "Brain" (Ollama)
First, you need the engine. Download Ollama from [ollama.com](https://ollama.com). Once installed, open your terminal and run: `ollama run llama3`

### Prepare the Python Environment
Create a dedicated folder for PoC 3 and set up a virtual environment to keep things clean:

*Bash*
```
mkdir poc3-ai-agents
cd poc3-ai-agents
python -m venv venv
```
**Activate it:**
-   **Windows:** `venv\Scripts\activate`
-   **Mac/Linux:** `source venv/bin/activate`

### The Strategy
We are bypassing the "heavy" frameworks like CrewAI (which are essentially just complex layers of Python code anyway).
Instead of letting a framework hide the logic, i'm are going to write the **Agent Logic** by myself using pure Python. This allows us to delve more deeply into how an agent works "behind the scenes."
In CrewAI, an agent is a "Class" with a `backstory`. In our "Lean" version, an agent is just a **Python function** that sends a specific "System Prompt" to your Docker-based Ollama.

Here is how we implement the **logic** without the heavy libraries:

1.  **The Role (System Prompt):** We tell the LLM "You are a [Role]."
2.  **The Memory (List):** We keep a Python `list` of the conversation so the agent remembers what it just said.
3.  **The Task (User Prompt):** We give it a specific goal.
4.  **The Handoff (The Logic):** We take the output of Agent A and pass it as the input to Agent B.

This instead is the  Infrastructure & Connectivity schema 

-   **Architecture:** Implemented a decoupled "Client-Server" model.
-   **Backend:** Deployed the **Ollama LLM engine within a Docker container** to ensure environment isolation and portability.
-   **Frontend Logic:** Developed a Python-based orchestration layer that interfaces with the containerized engine via **REST API mapping (Port 11434)**.
-   **Result:** A scalable, modular AI system that runs entirely on local hardware with zero cloud costs.

## Simple Agent Orchestrator: Deconstructing the Logic

Here is how the "Brain" of your script works:

-   **The Bridge (Client):** `ollama.Client(host=...)` establishes the API connection. It tells Python exactly which "door" (port 11434) to knock on to find the Docker container.
-   **The Agent Factory (`run_agent`):** This function is a **Template**. Instead of writing a new script for every agent, you created a reusable factory.
    -   It takes a **Role** (Persona) and a **Goal** (Objective).
    -   It wraps them in a **System Prompt**, which stays hidden from the user but dictates the AI's behavior.
-   **The Sequential Handoff:** This is the most critical part.
    -   `Agent 1` generates a string of text.
    -   `Agent 2` receives that string as its **Input**.
    -   This mimics a "relay race" where data is the baton being passed.

## Advanced Agentic Patterns
Let’s look at the "Big Three" patterns used in professional AI engineering. You can implement all of these using the "Lean" Python method we started.

### The Logic: What is "Shared Memory"?
In a custom Python orchestrator, Shared Memory is typically a **Dictionary** or a **JSON object**.
Instead of passing a single string from Agent A to Agent B, you pass a **`State` object**. Every agent reads from the `State`, performs its task, and _writes back_ to the `State`.

### Pattern A: The "Critic-Loop" with Refinement (Self-Correction)
File: *Orchestrator_Pattern01.py*
Instead of Agent A → Agent B, you create a loop.
We will build a **Self-Correcting Research Pipeline**. This uses the **Critic-Loop** pattern
-   **Agent 1 (Researcher)** writes a draft.
-   **Agent 2 (Auditor)** reviews it.
    -   If the Auditor finds critical errors, it sets `status = "REVISE"`.
    -   If the draft is solid, it sets `status = "APPROVED"`.
-   **The Controller (Python)** checks the status. If it's "REVISE," it sends the Auditor's notes back to the Researcher for a second attempt.

**Results**:
-   **Dynamic Control Flow:** "Implemented a non-linear logic gate that prevents the workflow from proceeding until specific quality benchmarks (defined by the Auditor Agent) are met."
-   **Stateful Iteration:** "Used Shared Memory to track 'Iteration Counts,' preventing infinite loops while ensuring the Researcher Agent adapts its output based on previous 'Failure' metadata." 
-   **Convergence:** "Demonstrated how multi-agent collaboration reduces hallucination by forcing the LLM to critique its own logic in a structured loop."


### Pattern B: The "Router" (Decision Making)
The **Router Pattern** is a sophisticated architectural move. Instead of a fixed sequence, the system acts like a **Dispatcher**. It analyzes the user's intent and dynamically "routes" the task to the most qualified specialist agent.
This is highly efficient for "limited means" because you only activate the specific "brain power" (LLM prompt) needed for the task, saving processing time and keeping the context window clean.

In a Router pattern, the first step is always **Classification**.
1.  **The Router (The Brain):** Receives the user input and matches it against a list of available "Expert Agents."
2.  **The Switch (The Logic):** A Python `if/elif` or `match` statement takes the Router's decision and triggers the correct function.
3.  **The Specialist (The Worker):** Only the chosen agent executes the task.
We will create a system that can handle **Technical Support**, **Creative Writing**, or **General Inquiry** by routing the request to different personas.
 ## "Dynamic Intent Routing"

**Results**
This pattern looks incredible on a CV or GitHub because it moves away from "static bots" toward "intelligent systems."
-   **Intelligent Classification:** "Developed a primary classification layer that utilizes LLM-based intent recognition to route queries to domain-specific specialist agents."
-   **Modular Extensibility:** "Designed a 'plug-and-play' architecture where new specialist agents can be added to the switchboard without disrupting existing logic."
-   **Resource Efficiency:** "Minimized token usage and processing overhead by ensuring only the relevant agent persona is activated for a given task."

### Pattern C: The "Tool-User" (RAG/Actions)
The agent isn't allowed to just "guess." It must use a tool (like a Python search script or a local text file) to find facts before answering.

Until now, your agents have only used their internal training data. A **Tool-User agent** can interact with the outside world: it can read a local file, check the weather, query a database, or perform a calculation.
Since you are using **Ollama/Docker** with limited means, we will implement a "Manual Tool Loop." This is exactly how sophisticated frameworks like LangChain work under the hood.

In this pattern, the Agent doesn't just answer. It follows a loop:
1.  **Thought:** "I need to know the price of Bitcoin, but I don't know it."
2.  **Action:** "I will call the `get_crypto_price` tool."
3.  **Observation:** The Python script runs a real function and gives the result back to the AI.
4.  **Final Answer:** "The price is $90,000."

We will give your agent a **Tool** that allows it to "read" a specific text file on your computer. This is the foundation of **RAG (Retrieval Augmented Generation)**.

Let's create a sample file first:
Create a file named `knowledge.txt` and write: _"The secret code for PoC 3 is: ALPHA-99."_

### Results
This pattern  demonstrates that you can bridge the gap between **Static LLMs** and **Dynamic Data**.
-   **Tool Augmentation:** "Engineered a ReAct-based (Reasoning + Acting) loop allowing LLMs to interact with local file systems via Python-defined functions."
-   **Constraint Management:** "Designed custom parsing logic to interpret AI 'Action' requests, ensuring secure execution of local operations before feeding observations back into the context window."
-   **RAG Foundation:** "Demonstrated a prototype for Retrieval Augmented Generation (RAG), where the agent autonomously decides when to query external data sources to ensure factual accuracy."
