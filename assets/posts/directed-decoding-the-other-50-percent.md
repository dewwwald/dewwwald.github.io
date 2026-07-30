# Directed Decoding: The Other 50 Percent

I typed "give me all items that have name gadget and price 50" into a chat box wired up to a local model, and it came back with a clean, correctly-shaped JSON query. I ran it against the database. It worked. That took about an hour, and it felt like the whole problem — natural language in, structured query out, demo over.

It wasn't. That hour got me maybe halfway. The rest of the project ([source on GitHub](https://github.com/dewwwald/article-artifact-directed-llm-output)) was spent on the half that doesn't show up in a demo: making sure the model's output isn't just *plausible*, but something a system can actually be trusted to run.

## Plausible isn't the bar

When you chat with an LLM, you are the validator. If it says something slightly wrong, you notice, because you're a human reading human language and you have context it doesn't. That's general inference, and "pretty good most of the time" is often an acceptable outcome, because a person is standing between the model and anything that matters.

That's not what I was building. I was building a query engine: natural language in, a structured filter out, and that filter gets executed against a database with nobody reading it first. The moment an LLM's output is consumed by code instead of a person, "sounds about right" stops being good enough, because nothing is left to notice when it isn't.

Push that further, into the context I actually cared about: an enterprise system, where the data behind the query is something like PII. Now a wrong answer doesn't look like a bad chatbot reply. It looks like a filter that silently returns — or silently omits — records it shouldn't, because the model invented a field, or nested a condition wrong, or mapped "cost" onto a field that doesn't exist. Nobody reads the query before it runs. The failure is structural, not conversational.

That's the actual problem directed decoding is for. Not "make the model smarter." Make its output something a system can trust the shape of — and then, separately, deal with the fact that shape is not the same thing as meaning.

## Building a grammar small enough to trust

Before any of that, I needed something to constrain the model *against*. So the project didn't start with the LLM at all — it started with a tiny, custom AST:

```
Node {
  operand: AND | OR
  action:  EQUALS | NOT_EQUALS | CONTAINS | NOT_CONTAINS
  path:    JSONPath-lite string, e.g. "$.items[]"
  value:   string
  then:    Node[]?
    (nested, applied per-element under a wildcard path)
}
```

An array of these nodes, each optionally nesting a `then` array of child nodes applied per-element when a path ends in a wildcard. Small enough to hold in my head, small enough to validate exhaustively, expressive enough to answer real questions against a real SQLite-backed dataset. I built the matcher and the persistence layer first, with no model anywhere near it, specifically so the grammar the LLM would eventually target was solid before I asked anything to generate it.

## Attempt one: hope disguised as validation

The first LLM integration was exactly the demo I described at the top. A plain-English description of the grammar as a system prompt:

```
Generate a JSON array of "Node" objects representing a search
AST. Respond with ONLY the JSON array: no prose, no code fences.

Each Node object has exactly these keys:
  "operand": "AND" | "OR"
    - combines this node with the previous one; always "AND"
      on the first node (never null, never omitted)
  "action": "EQUALS" | "NOT_EQUALS" | "CONTAINS" | "NOT_CONTAINS"
  "path": a JSONPath-like string, e.g. "$.name" or "$.items[]"
    - "[]" means "iterate every element"; use "then" to check
      fields on each element instead of chaining after "[]"
  "value": a string to compare against
  "then": optional; nested Node[] applied to each element when
    "path" ends in "[]"
```

Send the prompt, get a response, run it through `json.parseFromSlice`, and hope. When it worked, it looked exactly like real guided decoding — the output matched the grammar. When it didn't, the process crashed on a malformed field or just produced nonsense that happened to parse.

I was calling this "guided decoding" in my own head until I was asked, directly, whether it actually was. The honest answer was no. This was post-hoc validation: check a finished response and retry if it's wrong. Real constrained decoding steers generation at the token level — masking invalid next-tokens before they're ever produced — and a plain chat-completions call doesn't expose the logits you'd need to do that. I'd built a spell-checker, not a grammar. Worth having, but not what I thought I'd built, and worth stopping to correct rather than letting the description drift further from the reality.

## Attempt two: actually constraining the model

That correction sent me looking for whether the model I was running locally (Gemma, served through LM Studio) could really do token-level constraint. It could — LM Studio exposes `response_format: {type: "json_schema", ...}` on its OpenAI-compatible endpoint, and for MLX models it's backed by the [Outlines](https://github.com/dottxt-ai/outlines) library, which genuinely masks invalid tokens during sampling. I didn't take that on faith; I checked it against LM Studio's own documentation before wiring anything to it, because "an AI search summary said so" is exactly the kind of plausible-but-unverified claim this whole project was supposed to be skeptical of.

That meant writing a real JSON Schema — not the ad-hoc description I'd been hand-parsing against, but a document the *sampler* itself could enforce:

```json
{
  "type": "array",
  "items": { "$ref": "#/$defs/node" },
  "$defs": {
    "node": {
      "type": "object",
      "properties": {
        "operand": {
          "type": "string",
          "enum": ["AND", "OR"]
        },
        "action": {
          "type": "string",
          "enum": [
            "EQUALS", "NOT_EQUALS",
            "CONTAINS", "NOT_CONTAINS"
          ]
        },
        "path": { "type": "string" },
        "value": { "type": "string" },
        "then": {
          "anyOf": [
            { "type": "null" },
            {
              "type": "array",
              "items": { "$ref": "#/$defs/node" }
            }
          ]
        }
      },
      "required": [
        "operand", "action", "path", "value"
      ],
      "additionalProperties": false
    }
  }
}
```

Wired into the request body, this closed off an entire category of failure for free. Malformed JSON, a made-up key, an enum value outside `AND`/`OR` — structurally impossible now, not just checked-for-and-rejected after the fact.

By this point I'd also wrapped the whole pipeline in a small web UI — a chat pane next to a live diagram of Browser / Zig Server / LM Studio, with the request animated as it moves through each stage:

![Chat pane and animated architecture diagram showing a request travel from browser to Zig server to LM Studio](/assets/img/directed-decoding-demo.gif)

Not decoration — it's what made the next bug visible at all. Watching the AST get validated character by character, live, is what turned "the query failed" from a log line into an obvious, watchable moment of *where* it failed.

I kept the validator anyway. That turned out to be the right call, for a reason I didn't fully appreciate until the next bug.

## The 50% a schema can't buy

A few sessions later, live testing surfaced a query that failed: "items that cost 50." Schema-valid JSON came back every time. It was also useless — the model had nowhere to put "cost," because neither the AST's grammar doc nor the JSON Schema said one word about what the underlying *data* actually looked like. Both only describe the AST's own shape: operand, action, path, value, then. Nothing in either document says "and by the way, the field is called `price`."

That's the sentence that mattered most in this whole project: **a schema constrains shape, not intent.** `additionalProperties: false` stops the model from inventing a key that isn't one of the five defined ones; the `enum` on `operand` separately stops it from writing a value outside `"AND"`/`"OR"` once that key exists. Two different mechanisms, both enforcing closed, checkable sets — and neither one can stop the model from confidently mapping "cost" onto a field that doesn't exist, because "cost means price" isn't a shape problem, it's a meaning problem, and meaning isn't something a JSON Schema has any vocabulary for.

The fix was a hint describing the real data model, generated by reflecting over the actual `Item` struct so it can never quietly drift out of sync with the code:

```zig
fn dataSchemaHint(allocator: std.mem.Allocator) ![]u8 {
    const fields = std.meta.fields(Item);
    // ...
    try w.writeAll(
        "The data being queried is {\"items\": [...]}, " ++
        "where each item has exactly these fields " ++
        "(no others exist): "
    );
    inline for (fields, 0..) |field, i| {
        if (i > 0) try w.writeAll(", ");
        try w.print("\"{s}\"", .{field.name});
    }
    try w.writeAll(
        \\. Map the user's wording onto exactly these
        \\field names, never a synonym (e.g. "cost" or
        \\"amount" both mean the "price" field).
    );
    // ...
}
```

The first version of that fix caused its own regression. Its phrasing read enough like a literal path template that the model started emitting bare `"price"` instead of `"$.price"`. That crashed the path resolver — a different bug, caused by the very sentence meant to fix the first one. Prompt text is an interface, and it broke the same way any interface breaks: a change that fixes one caller's misunderstanding introduces a new one. I fixed the wording, and — more durably — made path-resolution failures retry through the same feedback loop as validator rejections, instead of crashing the request outright:

```zig
switch (try attemptOnce(
    allocator, io, bus, doc, prompt, schema.value
)) {
    .valid => |content| {
        // Schema-valid JSON can still be semantically
        // unusable — e.g. a path that doesn't start with
        // "$" or resolves against nothing. Treat that the
        // same as a validator rejection and retry, rather
        // than letting it crash the request.
        if (reportMatches(
            allocator, bus, rows, data, content
        )) |_| {
            return;
        } else |err| {
            reason = try std.fmt.allocPrint(
                allocator,
                "the query could not be run against " ++
                    "the data: {s}",
                .{@errorName(err)},
            );
            bad_content = content;
        }
    },
    .rejected => |r| {
        reason = r.reason;
        bad_content = r.content;
    },
}
```

That's the shape the "other 50%" actually takes in practice. Not a smarter model. A bounded retry loop that feeds the model the *specific, concrete reason* its last attempt failed, and asks it to correct exactly that. It's the same discipline you'd apply to any unreliable upstream dependency — applied here to a model instead of a flaky API.

## Trust, but verify — literally

Constrained decoding and the retry loop covered generation. Almost every other real bug in this project was caught the same way, and it wasn't by reading code and deciding it looked fine — it was by actually running it. A raw `curl` handshake and a Python `websocket-client` script surfaced a deadlock where the server's WebSocket handshake response wasn't being flushed. A real browser opening a page load and a WebSocket connection nearly simultaneously exposed a sequential accept-loop that could only ever service one of them. Live browser automation with the console open — not a glance at the rendered page — caught a frontend comparison against the wrong serialized shape of a Zig tagged union. Separately, it caught a stray unmatched parenthesis in a syntax-highlighting regex that silently broke the entire page script.

None of those were exotic bugs. All of them were invisible to a code read and obvious the moment something real touched the system. That's the same lesson I keep landing on with AI-assisted work generally: the plausible explanation for why something is behaving a certain way is not evidence, and neither is code that looks correct. Running it is evidence.

## The takeaway

Directed decoding is necessary and it is not sufficient, and the distance between those two words was most of this project. Grammar-constrained sampling gives you something real: a guarantee that what comes back is structurally valid, which for anything you intend to execute against a live system is not optional. It does not give you a guarantee that the query means what your system needs it to mean — that a well-formed filter is also a filter that's right for your data. That gap doesn't close itself, and it's exactly the gap an enterprise system built on sensitive data can't afford to wave away as a rounding error. Closing it took a real data-aware hint, a retry loop that treats semantic failure as seriously as syntactic failure, and a lot of actually running the thing instead of trusting that it probably worked. Directed decoding was never going to hand me that part for free. That was always going to be the other 50 percent — mine to build, not the sampler's.
