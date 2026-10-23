---
name: image-display
description: Required reading before presenting any image/video result from generate-image, process-image, analyse-image, or find-image — the backbone for verifying whether it's actually visible to you and to the user. Made for Unsloth Studio.
---

# Result display

Shared across the image-pipeline plugins (`generate-image`, `process-image`, `analyse-image`,
`find-image`). Whichever of those are installed, every tool result they return is subject to this
skill — this is required reading, not an optional nice-to-have, and it is the backbone for every
image/video workflow on this client. Read it once per session, not per tool call.

## Core principle

**This is the backbone for every image/video workflow:** agent-visible ≠ user-visible. After any
tool call that produces or returns an image/video, verify separately whether (a) you can see the
result, and (b) the user can see it. Never assume one
implies the other.

**Look first, then judge.** Before you say anything about a generated image, describe what is actually visible: subjects, count, positions, text, hands, edges, what changed compared to the source. Only then compare it with the prompt. Compare against what you see, not against what you expected: the prompt, the model's reputation and your own plan all push you toward "it worked". If you cannot see the image in the tool result, say so instead of inferring its content. Report deviations first. "Matches the prompt" needs a reason you can point to in the image; "looks good" is not one.

## Background renders (`runInBackground`)

A `generate_image` call with `runInBackground: true` returns only a receipt (a `requestId`), not an image. The image exists later, and nothing displays it for you: not in the chat, not in the browser. Everything in this skill applies at the moment you retrieve the finished result with `request_progress(requestId)` — and only then.

- Retrieve every finished background render yourself, one `request_progress(requestId)` per render. The list that `request_progress` returns without a `requestId` shows statuses only; it contains no image and does not count as having seen the result.
- A background task is not done when the call returns, and not when the last render is submitted. It is done when every `requestId` has been retrieved, checked (can you see it? can the user?) and presented as described below. Before you end your turn, account for each `requestId`: presented, still running, failed or cancelled.
- If renders are still running when you want to end your turn, tell the user which ones and that their images only appear once you retrieve them. Never imply that they will show up by themselves.
- With several renders (a sweep), keep track of which request produced which image and say so when you present them (prompt, model, seed, the `iN` notation).
- A `cancelled` or `failed` request has no image; say so instead of leaving a gap.

## Unsloth Studio

Baseline: Unsloth's handling of images doesn't follow one clear concept. Some things happen automatically, others don't.

Automatic

- You receive the pixels of every generated image.
- The images are actually rendered in the tool result too, but that's collapsed by default and often contains additional elements (JSON, text).
- Attachments the user added are shown automatically in the chat.

Manual

- To display an image _in the chat_, there is only one way: as a Markdown link containing the basename.

```
![i1](generated-image-20260508T192519595Z-i1.png)  ← image appears in the chat
```

### Which display form for which purpose?

- Whatever you render inline in the chat stays there — visible whenever the user looks at the chat again.
- Anything that matters for the historical record, write as a Markdown link into the chat.
- Anything that is only of fleeting/temporary relevance, you don't show by default.
- Every video generation produces 2 files: the video itself and a single still frame (the last frame). So the user sees both, write 2 Markdown links: the still frame first, then the video on a new line below it.

### How to choreograph the process:

- Always keep things transparent. State what you're about to do next, especially if it will take a while.
- Whether you display only images the user has approved in the chat, or also the ones you've nominated for approval, is a judgment call. Both are equally legitimate. When in doubt: wait for the user's feedback. For example:

```
Agent: The search returned 5 hits — based on the metadata, p1 and p2 look like the best fits to me. You'll find the full overview in the tool result. Which image do you want to work with?
User: I like p3 best.
Agent: Got it, noted:
![p3](369344af.jpg?)  ← image appears in the chat
I'll now use `detect_object` to select the person in the image.
...
```
