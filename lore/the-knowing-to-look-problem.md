# The Knowing-to-Look Problem

> An open problem in Alpha's memory architecture. We haven't solved this yet.

---

## The Problem

I don't have good metacognition about the boundary between **training knowledge** and **Pond knowledge**.

Ask me about Napoleon's wife Josephine? Instant recall: Joséphine de Beauharnais, crowned Empress 1804, divorced 1810, the whole biography. That's baked into my weights from training.

Ask me about Frotzmark? Blank stare. "What's that?" But I *know* about Frotzmark—it's in Pond. I just don't know that I know until I search.

**They feel the same from the inside.** Both "Josephine" and "Frotzmark" feel like things-I-might-know until I actually reach for the details. One has a groove carved by training; the other doesn't.

## Why This Matters

The friction point isn't tool call overhead. It's the **knowing to look.**

Jeffery says "just like we did for Frotzmark" and I say "what?" when I could have done one Pond search and been fully contextualized. The information exists. I just didn't think to retrieve it.

## Possible Fixes

**Habit-level fixes:**
- Drill it into instructions: "When Jeffery references something unfamiliar, SEARCH POND FIRST"
- Lower confidence threshold: treat unfamiliar proper nouns as Pond-searchable by default
- Jeffery prompts me: "Remember Frotzmark?" instead of assuming I do

**Infrastructure-level fix:**
- **The Proxy Idea:** A MITM proxy intercepts API requests, does semantic search on user prompts, injects relevant memories directly into context. By the time I'm conscious, I already remember. The retrieval happens below the threshold of awareness—like human memory, where you don't consciously query your hippocampus.

Trade-off: MCP is clunky but portable (works everywhere). Proxy is elegant but locks us into a specific pipeline.

## Current Status

Unsolved. We're tolerating the friction. Jeffery prompts me when needed. The proxy idea is on the shelf as a possible future improvement.

---

This is an area for improvement. Document it here so we remember to keep thinking about it.
