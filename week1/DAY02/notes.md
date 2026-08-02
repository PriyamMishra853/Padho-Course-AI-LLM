# DAY02 Notes — System role and Temperature

## Overview
These notes summarize what I learned on DAY02 about the three chat roles (system, user, assistant) and the "temperature" hyperparameter that controls how creative or random an AI model's outputs are.

## Roles
- system: Sets high-level behavior, constraints, and persona for the model. System messages are not visible to end users and are used to define the assistant's role, safety rules, or style (e.g., "You are a helpful, concise assistant who avoids profanity").
- user: The person asking questions or giving instructions. Prompts from the user drive the content of the assistant's responses.
- assistant: The AI model that replies. It follows the system instructions and responds to user prompts.

## Temperature (creativity/randomness)
- Temperature is a sampling hyperparameter that controls randomness in generated text.
- Typical range used here: 0 to 2.
  - 0.0: Nearly deterministic/greedy. Gives the safest, most predictable answer.
  - ~0.5-1.0: Balanced — mixes coherence with some creativity.
  - ~1.0-2.0: Increasingly creative and varied; higher chance of surprising or unusual outputs.
- Use low temperature for factual, precise, or safety-sensitive tasks. Use higher temperature for brainstorming, creative writing, or idea generation.

## Examples (same prompt across different temperatures)
Prompt: "Tell someone 'I love you'"
- Temperature 0.0 (deterministic): Short, direct, and safe.
  - Example assistant output to girlfriend: "I love you. You mean a lot to me."
  - Example assistant output to an office colleague (professional context): "I value our professional relationship."
- Temperature 1.0 (balanced): Warmer and slightly more expressive.
  - Girlfriend: "I love you — you make my days brighter and I'm grateful for you."
  - Office colleague: "I appreciate working with you and value our collaboration."
- Temperature 2.0 (creative/open): Poetic, unexpected, or playful.
  - Girlfriend: "My heart sings when I think of you; I love you more than words can hold."
  - Office colleague: May produce a humorous or overly personal reply — be cautious in professional contexts.

Prompt: "Suggest names for a new clothes company"
- Temperature 0.0: Safe, conventional names (e.g., "Classic Threads", "Urban Wear").
- Temperature 1.0: More distinctive, brandable names (e.g., "Everloom Apparel", "MetroStitch").
- Temperature 2.0: Bold, unusual, or playful names (e.g., "VelvetEcho", "Stitch & Stardust").

## Practical tips
- Set system messages to define constraints (tone, style, forbidden topics) and role expectations before the user prompt.
- Choose temperature based on the task:
  - 0–0.3: precise & reproducible answers (code, facts, policies).
  - 0.4–1.0: helpful general tasks and grounded creativity.
  - 1.0–2.0: ideation, creative writing, playful outputs.
- For multi-turn conversations, maintain system constraints to keep consistent behavior.

## Summary
- The system role shapes behavior and enforces constraints. The user provides the intent. The assistant generates responses guided by the system and user.
- Temperature (0–2) controls how deterministic vs. creative the assistant is. Adjust it based on whether you need safe/factual answers or creative brainstorming.

---
Notes created based on DAY02 materials: system messages, roles (user/assistant/system), and examples demonstrating temperature effects.
