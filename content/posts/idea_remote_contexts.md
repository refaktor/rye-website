# Idea: remote contexts

Rye is a small, embeddable language focused on data exchange and fluent expression, implemented in Go. You can learn more about its core ideas at [ryelang.org](https://ryelang.org). While developing Rye, exploring its unique features led to an interesting thought experiment about remote communication.

We usually interact with remote systems via APIs – REST endpoints, gRPC calls, etc. This works well, but it's often rigid. The server predefines exactly which functions you can call and what they do. What if we could flip that? Instead of just calling predefined functions, what if we could send *code* to be executed remotely in a controlled environment?

This is the core idea behind **Remote Contexts**.

## Rye's Building Blocks

Two features of Rye make this concept viable:

1.  **First-Class Contexts:** Rye allows you to create isolated environments ("contexts") that contain only specific functions and data types. You can build a context that *only* knows how to do math, or one that *only* has functions to interact with a specific database table. It's a powerful isolation mechanism.
2.  **No Special Forms:** In Rye, control flow constructs like `if`, `loop`, `try`, and even function definition (`fn`) are just regular functions. This means a context can be incredibly minimal – it doesn't even need to include `if` unless you explicitly add it!

## Remote Contexts Explained

Imagine a server hosting a specific Rye context – say, one designed for querying user profiles, but *only* allowing read operations.

Instead of the server exposing fixed functions like `getUser(id)` and `getPosts(userId)`, a client could send a small block of Rye code to be executed within that server's context:

```rye
// Client sends this code block:
{
    user: find-user id: 123 ;
    posts: find-posts user-id: user/id count: 5 ;
    // Maybe combine or format the result right here:
    { name: user/name posts: posts/*title }
}

The server receives this code, verifies it (more on security soon!), executes it within the pre-defined "user profile reader" context, and sends the result (the combined name and post titles map) back to the client.
Why is this Interesting?

    Flexibility & Composability: Clients aren't limited to predefined API calls. They can compose complex queries or operations on the fly, tailored to their exact needs, using the functions available in the remote context.
    Reduced Network Chattiness: Operations that might take multiple back-and-forth API calls could potentially be performed in a single remote evaluation, reducing latency.
    Powerful Interaction: Imagine connecting to a remote context via a REPL! You could interactively explore data, debug issues, or orchestrate tasks manually by executing Rye code directly on the remote system within its secure context.

## The Elephant in the Room: Security

Executing arbitrary remote code sounds dangerous, and it absolutely can be without rigorous safeguards. Making Remote Contexts viable requires multiple layers of security:

    Code Signing (Mandatory): Code sent for evaluation must be digitally signed (e.g., using Ed25519 keys, similar to SSH key pairs). The server verifies the signature against a list of trusted public keys before even looking at the code. This confirms who sent the code.
    Secure Contexts (Capability Control): The context definition is the core authorization mechanism. If a function isn't explicitly included in the context the code runs in, it cannot be called. Contexts must be carefully defined to grant least privilege – only the functions absolutely necessary for the intended purpose. This limits what the code can do.
    Resource Limiting: Evaluations must be strictly constrained. CPU time limits, memory allocation limits, and instruction count limits are essential to prevent denial-of-service attacks from resource-hungry code (malicious or accidental).
    Sandboxing: The Rye interpreter itself needs to enforce context boundaries, and ideally, the entire evaluation process should run within OS-level sandboxes (containers, microVMs, syscall filters) for an extra layer of containment.

Security isn't an afterthought here; it has to be woven into the very fabric of the system.
Connecting Systems

This idea could extend to cross-language communication. A simpler core of Rye (Rye0) could potentially be implemented in other languages (Rust, Python, JavaScript), allowing systems built on different platforms to communicate by evaluating Rye code in shared contexts.

For the network protocol, something like WebSockets (WSS) seems like a good fit. It provides a persistent, low-latency connection ideal for REPL interaction and can be easily used by both native command-line clients and web-based applications.
Just an Idea?

Remote Contexts offer a glimpse into a more dynamic, flexible way for programs – and people – to interact with remote systems. It trades the perceived simplicity of fixed RPC endpoints for greater power, but demands a strong focus on security.

It's an idea under exploration within the Rye ecosystem. What are your thoughts? Does the flexibility outweigh the inherent risks and complexity? Let us know!

Find out more about Rye at ryelang.org.
