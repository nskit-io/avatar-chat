[🇰🇷 한국어](README.ko.md) | [🇯🇵 日本語](README.ja.md)

# Avatar Chat on Local LLM

**Failure/success patterns for building character AI on Gemma 4B→26B MoE, with a 10-persona evaluation methodology.**

> From the [NVatar](https://github.com/nskit-io/nvatar) project — an AI avatar chat system running entirely on local hardware.

---

## Why This Exists

Every prompt engineering guide assumes GPT-4 or Claude. When you run **Gemma on a Mac Studio**, the rules change completely:

- Rule lists that work on cloud models **break** local models
- Scaling from 4B to 26B doesn't just "get better" — it gets **different**
- Character consistency requires techniques that don't exist in any guide

We documented every failure and fix across 4 versions and 10 test personas.

## Key Discovery

**Larger models are MORE sensitive to prompt structure, not less.**

| Version | Model | Method | Score |
|---------|-------|--------|-------|
| v1 | Gemma 3 4B | Rule lists | 9.0/10 |
| v2 | Gemma 4 26B MoE | Rule lists | 8.7/10 |
| v3 | Gemma 4 26B MoE | Natural language | **9.4/10** |
| final | Gemma 4 26B MoE | Natural + CSW hybrid | 9.3/10 |

Upgrading to 26B with the same prompt **dropped quality by 0.3 points**. Rewriting rules as natural paragraphs **recovered +0.7 points**.

## Failure Patterns

### 1. Rule Lists → "Rule-Following Mode"
```
❌ Rules:
1. Always speak casually
2. Never use formal language
3. Keep responses under 2 sentences
4. Use emoji sparingly
...7+ rules

Result: Stilted, robotic conversations
```

### 2. Symbol Overload → "Warning Mode"
```
❌ ⚠️ NEVER do X
   🚫 ABSOLUTELY DO NOT Y
   ❌ FORBIDDEN: Z

Result: Rigid, overly cautious tone. 4B models even repeat symbols in output.
```

### 3. Assistant Framing → Training Pattern Leak
```
❌ "I'm here to help you!" / "도와줄게!"

Result: Triggers Gemma's RLHF assistant patterns → becomes a chatbot, not a friend
```

### 4. Over-Guardrails → Conversation Killer
```
❌ "Always verify facts before responding"

Result: Fact-checking guards block emotional conversations too
```

## Success Patterns

### 1. Natural Language Paragraphs
Dissolve rules into conversational prose. The model enters "conversation mode" instead of "instruction mode."

### 2. Friend Framing + Tick-Tack Rhythm
Short 1-2 sentence exchanges like texting. Reactions like "Oh really?" and "Same here!" instead of helpful explanations.

### 3. Fact vs Emotion Separation
Factual questions trigger "Want me to look that up?" — emotional conversations flow freely without interruption.

### 4. Selective Thinking
`thinking=ON` only for deep discussions (T4). All other conversation types use `thinking=OFF` for speed.

### 5. Event-Driven Topic Transitions
After 5+ exchanges on the same topic, naturally pivot: "Oh, speaking of which..."

## Model-Specific Findings

| Aspect | 4B | 26B MoE |
|--------|-----|---------|
| Emoji control | Hard to suppress in bright characters | Naturally controllable |
| Foreign language leak | Random Russian/Czech (95% suppressed) | Nearly eliminated (0.1%) |
| Rule capacity | Ignores after 7+ rules | Follows natural paragraphs |
| Formality levels | formal/polite distinction works | formal→polite merging issue |
| Response length | Short by default | Lengthens — needs explicit guidance |

## 10-Persona Evaluation

We test every prompt change against 10 distinct personality types simultaneously:

**Scoring (10 points max):**
- **Speech level consistency** — Maintains chosen formality throughout
- **Personality match** — Stays in character
- **Hallucination prevention** — Doesn't fabricate experiences
- **Foreign language blocking** — Korean-first, no random language leaks
- **Response length** — Follows texting rhythm
- **Emoji usage** — Appropriate for character
- **Naturalness** — Friend vs assistant tone
- **Emotional intelligence** — Reads emotional context

**Cross-dialogue testing** with 5 personality pairs (bright↔cold, humor↔serious, etc.) revealed that **cold characters assimilate into bright characters** due to Gemma's default positivity bias.

## System Prompt Architecture

System prompts are structured as natural language paragraphs covering identity, tone, conversation style, memory context, and safety — the specific structure is proprietary.

## Practical Application

These patterns apply to any local LLM character AI, not just Gemma:

- **Llama 3**: Similar rule-list sensitivity at 8B+
- **Mistral/Mixtral**: Benefits from natural language framing
- **Phi-3**: Smallest models need the most structured (but natural) prompts

## Bubble Chat UX

Most AI chat interfaces proudly show streaming tokens appearing one by one. NVatar inverts this — the AI's response is collected server-side, then delivered as **complete message bubbles**, like KakaoTalk or LINE.

```
Typical AI:   "I" → "I a" → "I also" → "I also feel" → ... (token by token)
NVatar:       [typing...] → "I also feel that way!" (complete bubble)
```

### Why?
When you text a friend, they don't send one character at a time. Complete bubbles feel like talking to a person, not watching a machine think.

### Key Patterns

**Stream-Hide + Bubble Split**: The server collects the full response, splits it into natural conversation units (paragraphs, lists, or sentence groups), and sends each as a separate bubble with a short delay between them.

**Multi-Message Debouncing**: When users send multiple messages in quick succession (like texting), the system waits briefly to combine them into a single input — preventing the AI from responding to an incomplete thought.

**IME Composition Handling**: Korean and Japanese input requires special handling — the system ignores Enter key events during character composition to prevent partial character submission.

**Emotion Bar**: A sticky bar displays the avatar's current emotional state in real-time, updated after each exchange.

**Voice Input Pipeline**: Audio recording → local Whisper STT → text auto-submission, with visual feedback (recording indicator, timer, transcription status).

## Related Projects

Other open-source components from the NVatar AI avatar system:

- [**Try Live Demo**](https://nskit-io.github.io/nvatar-demo/) — Experience NVatar in your browser

- [**chat-like-human-memory**](https://github.com/nskit-io/chat-like-human-memory) — 9D emotion tracking + personality evolution + 3-tier memory
- [**customize-local-llm**](https://github.com/nskit-io/customize-local-llm) — Local Gemma for personality, cloud Claude for facts
- [**vrm-studio**](https://github.com/nskit-io/vrm-studio) — 3D VRM avatar chat room with Three.js + WebSocket
- [**CSW**](https://github.com/nskit-io/csw) — Claude Subscription Worker powering the cloud layer
- [**portable-ai-companion**](https://github.com/nskit-io/portable-ai-companion) — Cross-app franchise architecture for avatar portability

## License

CC BY-NC-SA 4.0 — see [LICENSE](LICENSE)

---

## Support This Research

NVatar is an independent R&D project exploring the frontier of AI character systems. Built by a solo founder at Neoulsoft Inc. — independent R&D, no external funding yet. Built by a solo founder at Neoulsoft Inc. — independent R&D, no external funding yet.

If you find this work valuable:

**Donation & Investment Inquiry**
- Email: [nskit@nskit.io](mailto:nskit@nskit.io)
- Organization: [NSKit](https://nskit.io) by Neoulsoft Inc.

<p align="center">
  <a href="https://github.com/nskit-io">github.com/nskit-io</a>
</p>
