# REFLECTION.md — CleanFlow

**Project:** CleanFlow AI Agent  
**Course:** ITAI 2376 – Deep Learning in Artificial Intelligence  
**Student:** Sharise Griggs | Solo Project | Spring 2026

-----

## What Worked Well

The architecture I planned in the blueprint translated cleanly into the actual implementation. The decision to use `sentence-transformers` with a BERT backbone for semantic task extraction turned out to be the right call. It handles the kind of informal, varied language real users actually type — “my kitchen looks terrible” and “dishes are everywhere” both correctly map to kitchen cleaning tasks even though they do not contain the exact words from the knowledge base. That is the BERT self-attention mechanism doing what it is supposed to do, and seeing it work in practice was satisfying.

The three-pass sequence ordering engine also worked better than I expected. Breaking the ordering logic into three distinct passes — urgency sort, room priority, then time-budget filtering — kept the code clean and easy to debug. Each pass has one job, which made it easy to test in isolation before connecting everything together.

The LangChain ReAct loop tied everything together well. The `verbose=True` setting during development let me watch the agent reason through each step, which was genuinely useful for understanding how it was deciding what to call.

-----

## What Did Not Work and How I Handled It

The biggest frustration was the first attempt at integrating the BERT extractor with the LangChain tool system. The tool needed to accept a plain string as input, but my initial implementation was passing a dictionary, which caused parsing errors in the ReAct loop. I fixed this by serializing the task data as JSON strings between tool calls and parsing them explicitly inside each tool function.

I also ran into an issue with the cosine similarity threshold. Setting it too high meant that vague inputs returned zero tasks and always triggered the clarification fallback even for inputs that were specific enough. Setting it too low let in irrelevant matches. I landed on 0.25 after testing with about fifteen different sample inputs, which gave a good balance between sensitivity and precision.

-----

## The Biggest Technical Challenge

The hardest part of this project was getting the end-to-end pipeline to work cleanly — the BERT layer talking to the sequence ordering layer, which feeds into the LangChain tool, which feeds into the agent executor, which uses memory across turns. Each piece worked individually, but when I connected them, data format mismatches caused the agent to fail silently or produce garbled outputs.

My solution was the staged integration strategy I described in the blueprint: build and test each component separately, then add one connection at a time. I verified the BERT extractor output, then verified the sequence ordering output, then wrote the tool wrappers and tested those independently, and only then brought the full agent executor online. This approach meant that when something broke, I knew exactly which connection was the problem.

-----

## Path Change from Midterm

I stayed with Option A: Single AI Agent, exactly as planned in the blueprint. No switch needed. The single-agent architecture was the right call — CleanFlow handles one focused job and does not need the coordination overhead of multiple agents.

The one difference from the blueprint is that the calendar/reminder integration tool is not implemented in this version. I scoped it as a future enhancement rather than rushing a partial implementation, because I wanted the core functionality to be solid rather than having three mediocre tools.

-----

## What I Would Build Next

If I had another semester, the first thing I would add is a fine-tuned text classifier trained specifically on cleaning-related language. The current BERT model is a general-purpose embedding model — it works well, but a model trained on a curated dataset of household task descriptions would be significantly more accurate for edge cases and unusual phrasings.

Second, I would build persistent user memory using a database backend so CleanFlow actually learns over time. If a user always starts with the kitchen, or always has more time on Saturdays, the agent should remember that and adjust future plans without being asked.

Third, I would add the calendar integration tool from the blueprint — connecting to Google Calendar so users can schedule recurring cleaning sessions directly from the plan output. That was always meant to be part of the full vision, and it would close the loop between planning and actually doing.

Finally, I would explore whether a small fine-tuned RNN or LSTM could handle the sequence ordering task more dynamically instead of using a hand-crafted rule system. The current three-pass ordering works, but a model trained on sequences of cleaning tasks might discover ordering patterns that are not captured in the manually defined dependency rules.

-----

*CleanFlow | ITAI 2376 | Sharise Griggs | Spring 2026*
