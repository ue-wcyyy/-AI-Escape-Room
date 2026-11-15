# Legacy — a preserved bug

This folder documents the development process. **Nothing here is meant to be run.**
The two notebooks in the repository root are the deliverables and both work.

## `Version2_broken_client.ipynb`

Version 2 immediately before its last fix. The API client ends with

```python
return LLMReply(
    text="".join(part.get("text", "") for part in parts).strip(),
    provider="gemini",
    model=model,
)
```

but `LLMReply` is never defined — the class is missing from that cell.

Every model call therefore raises `NameError: name 'LLMReply' is not defined`, and the
narrator catches it on purpose:

```python
except Exception as exc:                      # network, quota, provider outage
    return Narration(text=outcome.facts, provider="fallback", fallback=True)
```

So the game does not crash. It keeps playing, and every turn quietly drops back to the
engine's factual sentence. The symptom is not a traceback in the chat, it is a
"Last turn" panel that always reads `Narrator — fallback`.

The fix was to define the dataclass above the client; the shipped notebook has it.

## The other bug, same week

Version 1's Gradio callback assumed the session state was already initialised:

```
AttributeError: 'NoneType' object has no attribute 'send'
```

`gr.State` starts at `None` and the `load` event fills it, but a message can arrive
first. Fixed by creating the game when `state` is still `None`. That file was not kept,
but the guard is in the shipped notebook and the incident is written up in
`docs/report.pdf` §5.

## Why keep either of them

Because the more interesting half of the project is not the code that works — it is
the two failures that only appeared once a real API key was configured, since without
one both notebooks take the offline branch and those lines never execute. A repository
that shows only the fixed state hides that.
