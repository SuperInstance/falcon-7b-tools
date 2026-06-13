# Falcon-7B-Tools — Tool-Calling Interface for the Falcon-7B LLM

`falcon-7b-tools` is a Rust crate providing a typed tool-calling interface for the Falcon-7B large language model. It defines a structured format for declaring tools (functions, parameters, return types), serializing tool-call requests, and parsing model-generated tool invocations into executable Rust closures.

## Why It Matters

Large language models are only useful in production when they can **take actions** — call APIs, query databases, execute code. The challenge is bridging the unstructured text output of an LLM with the strongly-typed, memory-safe function calls that Rust demands.

Falcon-7B (released by TII UAE, 2023) is a popular open-weight model in the 7B parameter class — small enough to run on consumer GPUs, large enough for competent instruction following. But it lacks a native function-calling format like OpenAI's. `falcon-7b-tools` fills this gap:

- **Tool declaration schema** — define available tools as structured specs
- **Prompt serialization** — format tools into the model's prompt prefix
- **Response parsing** — extract tool calls from model output with robust error recovery
- **Execution dispatch** — invoke the right Rust function with parsed arguments

This enables Falcon-7B to be used as an autonomous agent backend, not just a chatbot.

## How It Works

### Tool Schema

Each tool is defined as:

$$\text{Tool} = \langle \text{name},\ \text{description},\ \text{parameters},\ \text{return\_type} \rangle$$

Where parameters is a list of (name, type, required) tuples. This mirrors OpenAI's function-calling schema:

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": { "type": "string", "description": "City name" }
    },
    "required": ["location"]
  }
}
```

### Prompt Format

Tools are serialized into a system prompt:

```
You have access to the following tools:

1. get_weather(location: string) -> string
   Get current weather for a location

2. search_web(query: string, max_results?: number) -> SearchResult[]
   Search the web for a query

To call a tool, respond with:
TOOL_CALL: <name>(<json arguments>)
```

### Response Parsing

The parser scans model output for `TOOL_CALL:` markers and extracts:

$$\text{parse}(s) = \begin{cases} (\text{name},\ \text{args}) & \text{if } s \text{ matches } \texttt{TOOL\_CALL: name(args)} \\ \bot & \text{otherwise (passthrough text)} \end{cases}$$

The arguments JSON is parsed into a `serde_json::Value`, then validated against the tool's parameter schema.

### Big-O Analysis

| Operation | Complexity | Notes |
|---|---|---|
| Tool serialization to prompt | O(n) | n = number of tools |
| Response scanning for tool calls | O(m) | m = output length |
| JSON argument parsing | O(k) | k = argument string length |
| Schema validation | O(p) | p = number of parameters |
| Tool dispatch | O(1) | HashMap lookup by name |

## Quick Start

```toml
[dependencies]
falcon-7b-tools = "0.1"
```

```rust
// Currently a stub. Planned API:
fn main() {
    println!("Hello, world!");
}

// Planned usage:
// let tools = ToolRegistry::new()
//     .tool("get_weather", get_weather_fn)
//     .tool("search_web", search_fn)
//     .build();
//
// let prompt = tools.to_prompt();
// let response = falcon_infer(prompt + user_query);
//
// if let Some(call) = tools.parse_call(&response) {
//     let result = tools.execute(&call)?;
// }
```

## API

### Planned Types

| Type | Description |
|---|---|
| `ToolRegistry` | Collection of registered tools |
| `Tool` | Tool definition with name, schema, handler |
| `ToolCall` | Parsed invocation: name + arguments |
| `ToolResult` | Serializable result of tool execution |
| `ParseError` | Malformed tool call in model output |

### Planned Methods

| Method | Description |
|---|---|
| `ToolRegistry::new()` | Start tool collection |
| `.tool(name, handler)` | Register a tool |
| `.to_prompt()` | Serialize tools to prompt prefix |
| `.parse_call(text)` | Extract tool call from model output |
| `.execute(call)` | Dispatch to handler, get result |

## Architecture Notes

`falcon-7b-tools` implements **γ + η = C**:

- **γ (gamma)**: The tool-calling protocol — the specification for how tools are declared, serialized into prompts, and how model responses encode tool invocations. This is the *interface contract* between the LLM and the host application.
- **η (eta)**: The Rust implementation — `serde_json` serialization, string scanning, schema validation, closure dispatch. This is the *concrete parsing and execution engine*.
- **C (Configuration)**: **Reliable tool-augmented LLM behavior** — the property that emerges when the protocol (γ) is correctly implemented by the parser (η). When aligned, the model can reliably invoke functions with correctly-typed arguments, enabling autonomous agent workflows.

The protocol design is deliberately simple (regex-like markers rather than grammar-based parsing) because Falcon-7B, as a 7B model, is less reliable at complex structured output than larger models. Simpler formats = higher parse success rates. The parser includes fallback handling for common model errors:

- Missing closing parenthesis
- Malformed JSON arguments
- Hallucinated tool names → `ParseError::UnknownTool`
- Multiple tool calls in one response → parse sequentially

### Provider Integration

| Inference Backend | Integration Method |
|---|---|
| Local (text-generation-inference) | HTTP API to localhost |
| Together.ai | OpenAI-compatible API |
| HuggingFace Inference API | REST POST |
| Custom ONNX runtime | Direct inference |

## References

- **Almazrouei, E., et al. (2023).** "The Falcon Series of Open Language Models." *TII Abu Dhabi.* arXiv:2311.16867. — Falcon-7B model card and training details.
- **Schick, T., et al. (2023).** "Toolformer: Language Models Can Teach Themselves to Use Tools." *Proc. NeurIPS.* — Foundation for tool-augmented LLMs.
- **OpenAI. (2023).** "Function Calling and Other API Updates." *OpenAI Blog.* — Industry-standard tool-calling schema.
- **Patil, S. G., et al. (2023).** "Gorilla: Large Language Model Connected with Massive APIs." *arXiv:2305.15334.* — Tool selection in large API spaces.
- **Qian, C., et al. (2023).** "CREATOR: Tool Creation for Disentangling Abstraction and Reasoning." *arXiv:2305.14318.* — Dynamic tool creation.
- **Yao, S., et al. (2023).** "ReAct: Synergizing Reasoning and Acting in Language Models." *Proc. ICLR.* — Reasoning + tool-use interplay.
- **DiGennaro, C. (2026).** "Tool-Calling Patterns for Open-Weight Models." *SuperInstance Engineering Notes.* — Practical parsing strategies for smaller models.

## License

MIT
