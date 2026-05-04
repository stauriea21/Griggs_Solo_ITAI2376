# 🧹 CleanFlow — Smart Home Cleaning Assistant

> An AI agent that turns your messy home situation into a realistic, prioritized cleaning plan you can actually follow.

-----

## Solo Project — Sharise Griggs

**Course:** ITAI 2376 – Deep Learning in Artificial Intelligence  
**Semester:** Spring 2026  
**Submission:** `Griggs_Solo_ITAI2376`

-----

## The Problem

Keeping a home clean sounds simple, but for students, working adults, and parents it genuinely is not. Standard to-do lists dump everything on you at once without any sense of priority or time. If you have 30 minutes before dinner and your app tells you to clean the bathroom, mop the floors, and do laundry — that is not useful. That is stress with bullet points.

**CleanFlow** solves this. You describe your situation in plain language — what needs cleaning, how much time you have — and CleanFlow builds a realistic, time-aware plan you can actually follow.

**Target User:** Students, working adults, and parents who want to stay on top of their home without spending mental energy figuring out where to start.

-----

## Option Choice

**Option A: Single AI Agent** — same as the Midterm blueprint. CleanFlow handles one focused job well: understand the user’s situation, reason about priorities, and produce a plan. A single-agent architecture keeps the system reliable, debuggable, and well-scoped for this use case.

-----

## Architecture Overview

CleanFlow uses a three-layer pipeline:

1. **BERT Semantic Layer** — A `sentence-transformers` BERT model embeds the user’s natural language input and computes cosine similarity against a task knowledge base. This is real semantic understanding, not keyword matching. “My kitchen looks terrible” and “dishes are piling up” both correctly resolve to kitchen tasks because BERT understands context across the full sentence via self-attention.
1. **Sequence Logic Layer** — Inspired by RNN sequence modeling from the course, this layer applies a three-pass ordering algorithm: (1) sort by urgency, (2) apply room sequence priority so high-sanitation rooms go first, (3) apply a sliding-window filter to fit tasks within the user’s time budget. Task dependencies (like sweeping before mopping) carry forward like hidden state in a recurrent network.
1. **LangChain ReAct Agent** — The LLM orchestration layer. The agent uses a ReAct (Reasoning + Acting) loop: it reasons about what tool to call, calls the task analysis tool, then the plan formatting tool, and synthesizes a warm, readable final response. Conversation memory persists across the session.

```
INPUT: User natural language description
         ↓
[BERT Semantic Extractor] → cosine similarity → matched tasks (JSON)
         ↓
[RNN-Inspired Sequence Ordering] → urgency + room + time → ordered task list
         ↓
[LangChain ReAct Agent] → tool calls → reasoning → final plan
         ↓
OUTPUT: Personalized prioritized cleaning checklist
```

![CleanFlow Architecture](docs/architecture.png)

-----

## Frameworks and Tools

|Component                     |Tool / Library                                          |
|------------------------------|--------------------------------------------------------|
|Agent Framework               |LangChain                                               |
|LLM                           |OpenAI GPT-4o-mini                                      |
|Deep Learning (Transformer)   |`sentence-transformers` (BERT-based: `all-MiniLM-L6-v2`)|
|Deep Learning (Sequence Logic)|Custom RNN-inspired ordering engine                     |
|Memory                        |`ConversationBufferMemory` (LangChain)                  |
|Reasoning Pattern             |ReAct (Reason + Act)                                    |
|Environment                   |Python 3.10, Jupyter Notebook                           |

-----

## Installation

**Requirements:** Python 3.10+

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/Griggs_Solo_ITAI2376.git
cd Griggs_Solo_ITAI2376
```

### Step 2 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Set up environment variables

```bash
cp .env.example .env
```

Open `.env` and add your OpenAI API key:

```
OPENAI_API_KEY=your_actual_key_here
```

### Step 4 — Launch the notebook

```bash
jupyter notebook agent.ipynb
```

Or open in **Google Colab** by uploading `agent.ipynb` — Cell 1 installs all dependencies automatically.

-----

## How to Run

Open `agent.ipynb` and run cells **in order from top to bottom**.

- **Cells 1–7:** Setup, model loading, tool registration, agent initialization
- **Cell 8:** Scenario 1 — Kitchen mess + laundry, 45 minutes
- **Cell 9:** Scenario 2 — Pre-guest full apartment clean, 2 hours
- **Cell 10:** Scenario 3 — Vague input, clarification fallback
- **Cell 11:** Interactive mode — type your own cleaning situation

-----

## Example Usage

### Input 1

```
My kitchen is a disaster, there are dishes everywhere and the counters are gross.
I also have laundry piling up. I only have about 45 minutes before I need to leave.
```

**Output:**

```
🧹 YOUR CLEANFLOW PLAN (45 minutes available)
==================================================
Tasks identified: 5 | Estimated total time: 38 min

Step 1. 🔴 [Kitchen] Do Laundry (Start) — ~5 min (~5 min total)
Step 2. 🔴 [Kitchen] Wash Dishes — ~10 min (~15 min total)
Step 3. 🟡 [Kitchen] Wipe Counters — ~8 min (~23 min total)
Step 4. 🟡 [Laundry Room] Fold Laundry — ~15 min (~38 min total)

✅ You can finish everything in your 45-minute window. Let's go!
💡 Tip: Start the laundry first — it runs while you clean everything else!
```

-----

### Input 2

```
Guests are coming over in 2 hours and my whole apartment needs to be cleaned.
The bathroom is bad, the living room has clutter everywhere, and the floors
haven't been swept or vacuumed in a week.
```

**Output:** Full multi-room plan prioritizing bathroom sanitation, then living room declutter and vacuuming, fitted within the 2-hour window.

-----

### Input 3

```
My place is kind of a mess. I have maybe 30 minutes.
```

**Output:** CleanFlow asks a follow-up clarifying question (“Which rooms are bothering you most?”) because the input is too vague for high-confidence task extraction — this is the confidence fallback from the blueprint.

-----

## Known Limitations

- **No fine-tuned classifier:** The BERT semantic matching uses a pre-trained general embedding model (`all-MiniLM-L6-v2`) rather than a fine-tuned cleaning-specific classifier. A fine-tuned model would improve accuracy for unusual phrasings.
- **Time estimates are averages:** Cleaning times vary by person and space size. The estimates are based on averages and may not match every user’s reality. A future version would learn from session feedback.
- **Memory is session-only:** `ConversationBufferMemory` does not persist between notebook restarts. Long-term user preference learning would require a database backend.
- **English only:** The agent currently handles English input only.
- **No real calendar integration:** The reminder/calendar tool described in the blueprint is noted as a future enhancement and is not implemented in this version.

-----

## Demo Video

📹 [Watch the CleanFlow Demo](demo/CleanFlow_Demo.mp4)

*(Also available in the `/demo` folder of this repository)*

-----

## Repository Structure

```
Griggs_Solo_ITAI2376/
├── agent.ipynb          ← Main agent notebook (run this)
├── README.md            ← This file
├── REFLECTION.md        ← Project reflection
├── requirements.txt     ← Python dependencies
├── .env.example         ← Environment variable template
├── .gitignore           ← Files excluded from Git
├── data/
│   └── task_dataset.json   ← Cleaning task knowledge base
├── docs/
│   └── architecture.png    ← Architecture diagram
└── demo/
    └── CleanFlow_Demo.mp4  ← Demo video
```

-----

*CleanFlow | ITAI 2376 Final Project | Sharise Griggs | Spring 2026*
