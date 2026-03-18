# Harness Requirements

Medina is trained to operate inside a **stateful agentic harness** — a runtime that injects tool schemas, executes tool calls, and feeds results back into the conversation. Without this harness, the model will emit tool call XML as plain text output.

This document specifies the minimum harness implementation required for correct Medina behavior.

---

## Why a Harness Is Required

Medina's fine-tuning data consists almost entirely of multi-step agentic trajectories. The model has learned:

1. Receive a system message containing available tool definitions (XML schema)
2. Reason about which tool to call and with what arguments
3. Emit a `<function_calls>` block
4. Expect a `<function_results>` block injected by the harness
5. Continue reasoning and potentially call more tools
6. Emit a final response once the task is complete

**If no tool schemas are injected in step 1**, the model may still emit tool calls — the training distribution causes this. The fix is always harness-side: provide the correct context, not prompt engineering.

---

## Session Bootstrap

Every agentic session must begin with a system message in this structure:

```xml
<system>
{BASE_SYSTEM_PROMPT}

<tools>
{TOOL_DEFINITIONS}
</tools>
</system>
```

Where `{BASE_SYSTEM_PROMPT}` is the canonical prompt from [system-prompt.md](system-prompt.md) and `{TOOL_DEFINITIONS}` is a list of available tools (see Tool Schema Format below).

If no tools are available for a session (pure conversation mode), inject an empty tools block or omit it entirely and note in the system prompt that no tools are available:

```
You have no tools available in this session. Respond conversationally.
```

---

## Tool Schema Format

Medina uses the OpenClaw XML tool definition format:

```xml
<tool_description>
  <tool_name>send_message</tool_name>
  <description>Send a message to a contact via the configured channel.</description>
  <parameters>
    <parameter>
      <name>to</name>
      <type>string</type>
      <description>Recipient phone number or contact identifier</description>
      <required>true</required>
    </parameter>
    <parameter>
      <name>message</name>
      <type>string</type>
      <description>Message body text</description>
      <required>true</required>
    </parameter>
    <parameter>
      <name>channel</name>
      <type>string</type>
      <description>Delivery channel: sms, whatsapp, imessage, discord</description>
      <required>false</required>
    </parameter>
  </parameters>
</tool_description>
```

Multiple tools are listed sequentially inside the `<tools>` block.

---

## Tool Call Format (Model Output)

When Medina decides to invoke a tool, it emits:

```xml
<function_calls>
  <invoke>
    <tool_name>send_message</tool_name>
    <parameters>
      <to>+16195201221</to>
      <message>Meeting confirmed for 3pm tomorrow.</message>
      <channel>imessage</channel>
    </parameters>
  </invoke>
</function_calls>
```

The harness must:
1. Detect this block in the streamed output
2. Stop streaming to the user
3. Parse the tool name and parameters
4. Execute the tool call
5. Format the result (see below) and inject it as an assistant-turn continuation

---

## Tool Result Injection

After executing a tool call, inject the result as a continuation of the assistant's message:

```xml
<function_results>
  <result>
    <tool_name>send_message</tool_name>
    <stdout>Message delivered successfully. Message ID: msg_abc123</stdout>
  </result>
</function_results>
```

For errors:

```xml
<function_results>
  <result>
    <tool_name>send_message</tool_name>
    <stderr>Error: Contact not found for +16195201221</stderr>
  </result>
</function_results>
```

After injection, resume model generation. The model will either call another tool or produce a final response.

---

## Multi-Step Loop

The full execution loop:

```
WHILE model output contains <function_calls>:
    parse tool name + parameters
    execute tool
    append <function_results> to context
    resume generation

WHEN model output has no <function_calls>:
    deliver final response to user
```

Implement a maximum iteration guard (recommended: 10 tool calls per turn) to prevent infinite loops.

---

## OpenAI-Compatible API Integration

When serving Medina via Ollama or mlx_lm's OpenAI-compatible endpoint, the harness must manage context manually since these endpoints use the standard `messages` array format.

Map the session state to the messages array:

```json
[
  {"role": "system", "content": "<system prompt + tool definitions>"},
  {"role": "user", "content": "Schedule a meeting with Alex tomorrow at 3pm"},
  {"role": "assistant", "content": "<function_calls>...</function_calls>\n<function_results>...</function_results>\n\nDone — meeting scheduled for tomorrow at 3pm."}
]
```

The tool call and its result are both part of the **assistant** turn, not separate turns.

---

## OpenClaw Native Integration

When using Medina directly inside OpenClaw, set the provider to `ollama` or `mlx-local` and configure the agent session with:

```json
{
  "model": "medina-qwen3-14b-openclaw",
  "provider": "ollama",
  "systemPrompt": "<contents of system-prompt.md>",
  "toolCallFormat": "openclaw-xml",
  "maxToolIterations": 10
}
```

OpenClaw's native harness handles tool schema injection, call detection, execution, and result injection automatically when `toolCallFormat` is set to `openclaw-xml`.

---

## Minimal Harness (Python Reference)

```python
import ollama
import re

SYSTEM_PROMPT = open("configs/system-prompt.txt").read()
TOOL_DEFINITIONS = "<tools>...</tools>"  # your tool XML here

def run_tool(name: str, params: dict) -> str:
    # implement your tool dispatch here
    raise NotImplementedError(f"Tool not implemented: {name}")

def parse_tool_call(text: str):
    match = re.search(
        r"<function_calls>\s*<invoke>\s*<tool_name>(.+?)</tool_name>"
        r"\s*<parameters>(.*?)</parameters>\s*</invoke>\s*</function_calls>",
        text, re.DOTALL
    )
    if not match:
        return None, None
    tool_name = match.group(1).strip()
    params_xml = match.group(2)
    params = dict(re.findall(r"<(\w+)>(.*?)</\1>", params_xml, re.DOTALL))
    return tool_name, params

def agent_turn(user_message: str, history: list) -> str:
    messages = [
        {"role": "system", "content": f"{SYSTEM_PROMPT}\n{TOOL_DEFINITIONS}"},
        *history,
        {"role": "user", "content": user_message},
    ]

    for _ in range(10):  # max iterations
        response = ollama.chat(
            model="medina-qwen3-14b-openclaw",
            messages=messages
        )
        content = response["message"]["content"]

        tool_name, params = parse_tool_call(content)
        if tool_name is None:
            return content  # final response

        result = run_tool(tool_name, params)
        result_xml = (
            f"<function_results><result>"
            f"<tool_name>{tool_name}</tool_name>"
            f"<stdout>{result}</stdout>"
            f"</result></function_results>"
        )
        # Append tool call + result as single assistant turn, then continue
        messages.append({
            "role": "assistant",
            "content": content + "\n" + result_xml
        })

    return "Max tool iterations reached."
```
