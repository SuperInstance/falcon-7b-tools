# Falcon-7B Tools

**Falcon-7B Tools** is a Rust scaffold for building tool-calling interfaces compatible with the Falcon-7B large language model architecture. It provides a structured registry of callable functions that an LLM agent can invoke to interact with external systems.

## Why It Matters

Modern LLM applications require **tool use** — the ability for a language model to call external functions (database queries, API requests, calculations) to ground its responses. Frameworks like OpenAI's function calling, LangChain tools, and Anthropic's tool use all share the same pattern: register functions with typed signatures, let the model choose which to call, and return results to the model. This crate brings that pattern to Rust, providing type-safe tool definitions and a dispatch system suitable for building autonomous agents on top of open-weight models like Falcon-7B.

## How It Works

The tool-calling pattern operates in three phases:

### 1. Tool Registration

Each tool exposes a name, a JSON-schema-compatible description, and a handler function. The registry maps tool names to closures, enabling the LLM to discover available capabilities:

```text
tools = [
  { name: "search", description: "Search the web", params: { query: String } },
  { name: "calculate", description: "Evaluate math", params: { expression: String } }
]
```

### 2. Model Dispatch

The LLM receives the system prompt with tool descriptions, generates a response that may include a **tool call** (name + arguments), and the runtime dispatches to the matching handler.

### 3. Result Injection

The handler's return value is fed back into the conversation context as a tool result, enabling multi-turn reasoning.

### Complexity

Tool dispatch is **O(1)** via hashmap lookup. Serialization overhead dominates at O(s) where s is the JSON payload size.

## Quick Start

```rust
fn main() {
    // Currently a scaffold — the crate provides the structure
    // for registering and dispatching tool calls.
    //
    // Planned API:
    //   let mut tools = ToolRegistry::new();
    //   tools.register("search", search_handler);
    //   tools.register("calculate", calc_handler);
    //   let result = tools.call("search", r#"{"query": "rust async"}"#);
    println!("Hello, world!");
}
```

## API

| Type / Function | Description |
|----------------|-------------|
| `main()` | Entry point (scaffold) |

> **Note:** This crate is currently a scaffold. The full tool registry API is under development.

## Architecture Notes

Part of the **SuperInstance** model-serving stack. Falcon-7B Tools integrates with the inference pipeline to provide agentic capabilities — the model generates tool calls, the runtime executes them, and results feed back into the context window. This realizes **γ + η = C**: γ (structured tool dispatch) and η (efficient execution) combine for correct agent behavior.

See [ARCHITECTURE.md](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md) for the full system design.

## References

1. Falcon-7B Model Card, Technology Innovation Institute (TII), 2023.
2. Schick et al. "Toolformer: Language Models Can Teach Themselves to Use Tools." *NeurIPS 2023*.
3. Patil, S. G., et al. "Gorilla: Large Language Model Connected with Massive APIs." *arXiv 2305.15334*, 2023.

## License

MIT
