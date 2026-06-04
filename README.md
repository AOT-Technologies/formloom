# Formloom

[![CI](https://github.com/formloom/formloom/actions/workflows/ci.yml/badge.svg)](https://github.com/formloom/formloom/actions/workflows/ci.yml)
[![npm @formloom/schema](https://img.shields.io/npm/v/@formloom/schema.svg?label=%40formloom%2Fschema)](https://www.npmjs.com/package/@formloom/schema)
[![npm @formloom/llm](https://img.shields.io/npm/v/@formloom/llm.svg?label=%40formloom%2Fllm)](https://www.npmjs.com/package/@formloom/llm)
[![npm @formloom/react](https://img.shields.io/npm/v/@formloom/react.svg?label=%40formloom%2Freact)](https://www.npmjs.com/package/@formloom/react)
[![npm @formloom/zod](https://img.shields.io/npm/v/@formloom/zod.svg?label=%40formloom%2Fzod)](https://www.npmjs.com/package/@formloom/zod)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Some moments are better served by a dropdown than a dialogue.**

Formloom lets your LLM hand the user a real form mid-conversation, then pick the chat back up once the answers are in.

<p align="center">
  <img src="./docs/assets/demo.svg" alt="Formloom demo: the same user request ('set up a weekly 1:1 with Sarah next Tuesday 2pm') handled by a plain LLM chat on the left and by a Formloom-generated form on the right. The chat loops through email validation, time zone clarification, duration, and recurrence length with a typing indicator before each reply. The form on the right fills every field in one pass, pops a conditional End after field when Weekly is picked, and submits a scheduled meeting in about 8 seconds while the chat is still going. Animated 20-second loop." width="820">
</p>

---

## What is Formloom?

**Formloom is a toolkit for letting an LLM render a real form in the middle of a conversation.** When the model decides a moment is better handled by a UI than by more back-and-forth, it emits a small JSON schema. Formloom validates it, your frontend renders it with _your own_ components, the user fills it out in one pass, and the validated, structured result flows straight back into the conversation.

It works with OpenAI, Anthropic, Gemini, Mistral, and Ollama, ships as four small packages you adopt as you need them, and renders nothing on its own — your design system stays yours.

New here? Read on for the feel of it. In a hurry? Jump to [Requirements](#requirements) and the [Quickstart](#quickstart), or browse the [examples](#examples-in-this-repo).

## Chat vs Form

Your user wants to book a consultation. In an ordinary chat, that turns into an interview:

```text
You  ›  I'd like to book a consultation.
Bot  ›  Sure! What's your full name?
You  ›  Alice Chen
Bot  ›  And your email?
You  ›  alice@example.com
Bot  ›  What date works?
You  ›  Friday?
Bot  ›  This Friday or next?
You  ›  …
```

Nothing here is _broken_ — the model is polite, the user is patient. But every turn is a tiny writing exercise, a sentence composed for a machine that could have handed over a button. Chat treats every input as prose: wonderful for open-ended conversation, the wrong mode for picking a date or choosing one of three options.

With Formloom, the model decides this is a form moment and hands over an interface the user already knows how to use:

```text
You  ›  I'd like to book a consultation.
Bot  ›  Sure. Grab a few details below.

        ┌────────────────────────────────────────┐
        │                                        │
        │  Book a consultation                   │
        │                                        │
        │  Name    [ Alice Chen           ]      │
        │  Email   [ alice@example.com    ]      │
        │  Date    [ Fri, Apr 24     ▾    ]      │
        │  Time    ○ Morning                     │
        │          ● Afternoon                   │
        │                                        │
        │          [   Submit   ]                │
        │                                        │
        └────────────────────────────────────────┘

You  ›  (fills it once, submits)
Bot  ›  Booked for Fri, Apr 24 at 2:00 PM.
        Confirmation sent to alice@example.com.
```

A tap on a date picker instead of typing "Friday afternoon." A radio instead of a sentence. The user sees the whole task at once, fixes a mistake from earlier without scrolling, and submits with a single click. They're not being interviewed — they're getting something done.

## Why it matters

- **Familiar beats novel.** Users have tapped date pickers, radios, and dropdowns since the 90s. Let them use muscle memory instead of composing sentences.
- **The whole task, at a glance.** A form shows everything up front. Chat drips it out one question at a time, so the user can't skim, can't plan, can't see how close they are to done.
- **Fix mistakes where you made them.** Edit field one after filling field five. In chat, that's a scroll-up, a re-explain, and a hope that the model understands.
- **Your design system stays yours.** Formloom renders nothing on its own. It hands you the schema and state; your components render the pixels.

## Which package do you need?

Formloom is four small packages you adopt à la carte. `@formloom/schema` is the shared foundation; the other three sit on top of it for the LLM, the UI, and server-side validation.

| Package                                | What it does                                                                                                                  | When you reach for it                                                                                                 |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| [`@formloom/schema`](packages/schema/) | The shared schema types, validator, `showIf` engine, safe-regex handling, and capability profiles. Zero runtime dependencies. | Always — every other package builds on it.                                                                            |
| [`@formloom/llm`](packages/llm/)       | System prompt, tool definitions for five providers, response parser, `formatSubmission`, and the capability factory.          | To get the model to emit a form and to read its output back safely.                                                   |
| [`@formloom/react`](packages/react/)   | Headless hooks (`useFormloom`, `useFormloomWizard`) for state, visibility, validation, and submission.                        | To render the form in React with your own components. Peer dependency: React 18 or 19.                                |
| [`@formloom/zod`](packages/zod/)       | Zod and Standard Schema adapters.                                                                                             | To validate the submission on your server against the _same_ schema the client used. Peer dependency: Zod 3.23+ or 4. |

Most React apps install:

```bash
npm install @formloom/schema @formloom/llm @formloom/react
```

Add `@formloom/zod` when you want server-side validation that can't drift from the client.

## Requirements

**To use Formloom in your app**

- **Node.js 20+** — built and tested against Node 20 and 22.
- **Any package manager** — npm, pnpm, or yarn. The published packages are standard.
- **React 18 or 19** — only if you use `@formloom/react`.
- **Zod 3.23+ or 4** — only if you use `@formloom/zod`.
- **An LLM provider account + API key** — only for live model calls. The libraries themselves need no key.

**To run the examples or develop this repo**

- **Node.js 20+**.
- **pnpm 9.11.0** — pinned via the `packageManager` field, so Corepack picks it up automatically (run `corepack enable` once). npm and yarn won't work with the workspace.
- The [fullstack example](examples/fullstack/) also needs an **OpenAI API key**.

## Quickstart

Install the core packages:

```bash
npm install @formloom/schema @formloom/llm @formloom/react
```

Then wire the round-trip — ask the model for a form, render it, and send the result back.

### 1. Ask the model for a form

```ts
import {
  FORMLOOM_SYSTEM_PROMPT,
  FORMLOOM_TOOL_OPENAI,
  parseFormloomResponse,
} from "@formloom/llm";

const response = await openai.chat.completions.create({
  model: "your-model", // any model that supports tool calling
  messages: [
    {
      role: "system",
      content: `You are a helpful assistant.\n\n${FORMLOOM_SYSTEM_PROMPT}`,
    },
    { role: "user", content: "I want to book an appointment" },
  ],
  tools: [FORMLOOM_TOOL_OPENAI],
});

const toolCall = response.choices[0].message.tool_calls?.[0];
const parsed = parseFormloomResponse(toolCall?.function.arguments);
```

`FORMLOOM_SYSTEM_PROMPT` teaches the model the vocabulary. `FORMLOOM_TOOL_OPENAI` fixes the return shape. `parseFormloomResponse` validates everything before it ever touches your UI.

### 2. Render it, your way

```tsx
import { useFormloom } from "@formloom/react";

function MyForm({ schema, onDone }) {
  const form = useFormloom({ schema, onSubmit: onDone });

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        void form.handleSubmit();
      }}
    >
      {form.visibleFields.map(({ field, state, onChange, onBlur }) => (
        <div key={field.id}>
          <label>{field.label}</label>
          {field.type === "text" && (
            <input
              value={(state.value as string | null) ?? ""}
              onChange={(e) => onChange(e.target.value)}
              onBlur={onBlur}
            />
          )}
          {state.touched && state.error !== null && <p>{state.error}</p>}
        </div>
      ))}

      <button type="submit" disabled={form.isSubmitting || !form.isValid}>
        {schema.submitLabel ?? "Submit"}
      </button>
    </form>
  );
}
```

`useFormloom` owns state, visibility, validation, and submission. `visibleFields` already respects `showIf`, so hidden fields never render and never submit. You own every pixel.

### 3. Send the result back

```ts
import { formatSubmission } from "@formloom/llm";

const nextMessage = formatSubmission(submittedData, {
  provider: "openai",
  toolCallId: toolCall.id,
});
```

`formatSubmission` wraps the submitted data in the shape the provider expects. Append it to the conversation. The model continues, this time with clean structured values in hand.

## Integration modes

Pick whichever fits your flow:

| Mode                  | Best for                                                        | What you reach for                                                                          |
| --------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Tool calling**      | Chat flows where the model chooses when a form is needed        | `FORMLOOM_SYSTEM_PROMPT`, provider tool export, `parseFormloomResponse`, `formatSubmission` |
| **`response_format`** | Deterministic flows where the model must always return a schema | `FORMLOOM_RESPONSE_FORMAT_OPENAI`, `parseFormloomResponse`                                  |
| **Text fallback**     | Models without tool calling                                     | `FORMLOOM_TEXT_PROMPT`, `parseFormloomResponse`                                             |

Provider tool exports shipped today:

`FORMLOOM_TOOL_OPENAI` • `FORMLOOM_TOOL_ANTHROPIC` • `FORMLOOM_TOOL_GEMINI` • `FORMLOOM_TOOL_MISTRAL` • `FORMLOOM_TOOL_OLLAMA`

## Features you get

- Schema validation before a single pixel renders
- Conditional fields via `showIf` with `equals`, `in`, `notEmpty`, composed via `allOf` / `anyOf` / `not`
- Section grouping for longer forms
- Multi-step wizards with `useFormloomWizard` — one section per step, validation-gated `next()`
- Option descriptions (two-line radio/select) and opt-in "Other…" freeform input on radio/select
- `readOnly` / `disabled` view modes for recap and review surfaces
- File uploads, inline or delegated to your upload handler
- Async field validators in React (debounced, abortable, flushed on submit)
- Live-sync hook `onValueChange` to stream partial answers to an LLM while the user types
- Custom widget variants via `hints.variant` — host-defined, opaque, declaration-mergeable
- Capability profiles — one declaration narrows the system prompt, tool JSON Schema, and validator per surface
- ReDoS-safe regex handling for LLM-authored patterns
- Zod and Standard Schema adapters for server-side parity

## Examples in this repo

Three runnable examples, simplest first. Each has its own README with the full walkthrough — what it demonstrates, how it works, and the schemas it ships.

| Example                                    | Best for                                                                                                                        | API key? | Run                                                 |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | -------- | --------------------------------------------------- |
| [`basic-react`](examples/basic-react/)     | Seeing forms render from schemas with **no LLM at all** — every field type, validation, sections, and the wizard                | No       | `pnpm --filter @formloom/example-basic-react dev`   |
| [`provider-free`](examples/provider-free/) | Trying **every LLM integration path offline** (tool call, `response_format`, plain text) plus capability profiles — no network  | No       | `pnpm --filter @formloom/example-provider-free dev` |
| [`fullstack`](examples/fullstack/)         | A **real end-to-end chat app** on OpenAI: the model generates a form and reads the submission back, with server-side validation | Yes      | `pnpm --filter @formloom/example-fullstack dev`     |

Clone the repo, then run any of them from its root:

```bash
git clone https://github.com/formloom/formloom.git
cd formloom

pnpm install   # once
pnpm build     # build the packages the examples import
pnpm --filter @formloom/example-basic-react dev
```

The dev server opens at `http://localhost:5173`. For the fullstack example, add your key first:

```bash
cp examples/fullstack/.env.example examples/fullstack/.env   # then set OPENAI_API_KEY inside
```

## Development

This repo is a `pnpm` + Turborepo workspace (Node 20+, pnpm 9.11.0 — see [Requirements](#requirements)).

```bash
pnpm install
pnpm build
pnpm test
pnpm lint
pnpm typecheck
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for setup, PR flow, and changesets.

For security issues, see [SECURITY.md](SECURITY.md). Please don't open a public vulnerability report.

## License

[MIT](LICENSE)
