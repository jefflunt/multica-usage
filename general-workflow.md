# Simplicity First: Orchestrating Game Development with One Daemon and Five Global Squads

As developers, we are constantly looking for ways to reduce friction and minimize cognitive overhead. When I first adopted Multica for my game development projects—such as building an `asteroids` clone from scratch—I faced a common architectural question: Do I configure separate, specialized agent squads and run a separate daemon instance for every single repository? 

Managing thirty different agent configs and background processes for thirty different repositories is a recipe for configuration fatigue and context rot. Instead, I established a "Simplicity First" paradigm: running exactly **one local Multica daemon** coupled with **five global, role-based squads** across all of my codebases.

This article details the four engineering pillars of this workflow, illustrating how you can build a highly-scalable, tech-stack-agnostic development pipeline that relies on local-first simplicity, clear human-in-the-loop checkpoints, and robust codebase grounding.

---

## Pillar 1: Local Directory Mapping (Bypassing GitHub Overhead)

Many AI-assisted development tools require extensive cloud integration, pushing and pulling branch states through remote GitHub APIs. While remote integrations are powerful, they introduce latency, security considerations, and synchronization overhead. 

In my workflow, I bypass remote GitHub integrations entirely by leveraging local directory mapping.

### How to Attach Local Projects
In the Multica UI, you can directly bind a project to a local absolute directory on your filesystem. 
1. Create a new project in your Multica Workspace.
2. In the project settings, attach the absolute local path of your directory (e.g., `/Users/jefflunt/code/asteroids`).
3. Ensure your local Multica daemon is running. The daemon will automatically detect and bind to this local directory when executing tasks.

### The Power of Local-First Repositories
Because the local directory on your machine is already a git repository, you do not need an intermediary remote host to manage version control. The local daemon executes file modifications and commands directly inside your working directory. 

This model offers several major advantages:
- **Zero Remote Latency:** No waiting for webhook roundtrips or GitHub API limits.
- **Firewall & LAN Friendly:** Ideal for offline development, local-first environments, or corporate networks with strict egress rules.
- **Independent Commits:** You retain absolute control over when, how, and where changes are staged, committed, and pushed to your LAN git server or upstream remotes.

---

## Pillar 2: Global, Tech-Stack-Agnostic Squads

Instead of defining bespoke squads within each project, I utilize a central workspace directory (`jefflunt/build-multica`) containing a single set of five global, role-based squads:
1. **Design Squad** (Initiates brainstorming and grills the user)
2. **Build Squad** (Implements features and runs tests)
3. **Doc Squad** (Drafts and updates technical markdown documentation)
4. **Review Squad** (Performs static analysis and verification)
5. **Multica Squad** (Assists with overall workspace coordination)

### Decoupling Squads from Projects
By separating our squads from individual repositories, we achieve complete architectural decoupling. We do not have to maintain duplicated configurations across repositories. The same five squads seamlessly transition between a Go backend, a Next.js frontend, or a custom game engine.

### How Tech-Stack-Agnostic Agents Succeed
You might wonder: *How can a single, generic Build Squad build a Go server in the morning and a game engine feature in the afternoon without custom, per-language instructions?*

The secret lies in the natural capabilities of modern LLMs:
1. **Implicit Language Fluency:** Modern LLMs are inherently multi-lingual and deeply versed in software engineering standards across virtually all major languages and frameworks. They do not require hand-holding or verbose prompting just to write clean Go or TypeScript.
2. **Standardized Tooling and Discovery:** By configuring the squads with flexible, general-purpose filesystem and execution tools (such as file reads, writes, and bash execution), the agents can inspect local files (like `package.json`, `go.mod`, or `Cargo.toml`) to auto-discover dependencies, compilers, and test runners on their own.
3. **Community Blueprints:** This centralized squad blueprint is open-source and available in the [jefflunt/build-multica](https://github.com/jefflunt/build-multica) repository, serving as an easily extensible starting point for the community.

---

## Pillar 3: Issue Lifecycle State Machine (Context-Rich Threads)

The issue lifecycle in this workflow is governed by a human-in-the-loop state machine. Rather than spawning new issues for every small handoff, we keep the **same issue open** and reassign it dynamically across different agents and squads.

```mermaid
graph TD
    A[1-2 Sentence Intent] -->|Status: Todo| B(Design Squad)
    B -->|Grill Me Protocol / Clarification| C{Agreed Design & Step Breakdown?}
    C -->|No / Abandoned| D[Cancel Issue]
    C -->|Yes| E(Reassign Same Issue to Build Squad)
    E -->|Git Log / Agent Docs Context| F(Build Squad Implements Steps)
    F -->|Human Checkpoints| G[Human Reviews Code & Runs Tests]
    G -->|Iterate / Feedback| F
    G -->|Approved| H[Commit & Push to LAN Git Server / GitHub]
    H -->|Status: Done| I[Issue Completed]
```

### The 5-Step Lifecycle
1. **Intent:** The human starts with a brief, high-level intent (often just 1-2 sentences, such as *"Add a medium asteroid size to the game"*).
2. **Design & "Grill Me" Loop:** The issue is assigned to the `Design Squad`. The design agent initiates the **"Grill Me" protocol**, asking targeted, sharp questions to flush out edge cases, structural choices, and verify requirements. Once aligned, the Design Squad produces a high-level design document and a step-by-step implementation plan.
3. **Reassignment (The Handoff):** With a solid plan approved, the human reassigns the **same issue** to the `Build Squad`.
4. **Implementation:** The build agent reads the plan from the comment history or design folder, claims the task, modifies the code, and verifies the changes.
5. **Human Validation Checkpoint:** The build agent hands the issue back to the human. The human checks out the code, runs local tests, and performs the final sign-off.

### The "Load-Bearing" Comment Thread
Keeping the same issue active across the entire lifecycle is crucial. The issue comment thread becomes a **load-bearing documentation artifact**. Because all design questions, user answers, planning breakdowns, and tool outputs are recorded in a single continuous timeline, the incoming build agent has immediate access to the full conversational history. No context is lost in handoff.

---

## Pillar 4: Continuous Alignment & Grounding

A major concern with AI agents is drift—either the agent misunderstands the codebase patterns, or its context goes stale because the local repository shifted while the issue sat dormant. 

To achieve continuous alignment, we ground our agents using two authoritative local sources:

### 1. `agent_docs`
We maintain a lightweight `/agent_docs` directory in our repository following the [agent_docs framework](https://github.com/jefflunt/agent_docs). This acts as a progressive disclosure and alignment layer:
- `01_orientation/`: Quickly establishes high-level architecture and domain boundaries.
- `02_patterns/`: Teaches the agent local code styles, state management rules, and testing standards.
- `03_deep_dives/`: Holds detailed technical articles on complex internal subsystems.
- `04_plans/`: Keeps track of current design specifications and step-by-step task lists.

Before writing a single line of code, the agents read the `agent_docs` files to understand exactly what standards they must adhere to.

### 2. Git History (`git log` & `git status`)
Because our local Multica projects are bound to active git repositories, incoming agents are instructed to run `git log` and `git status` during their initial orientation phase. This allows them to:
- Instantly see what files have been recently modified.
- Understand what changes have occurred in neighboring branches or main since the issue was created.
- Re-orient themselves to the exact current state of the local filesystem, ensuring they never overwrite active local edits or work on top of stale code.

---

## Conclusion: Embodying "Simplicity First"

By combining local directory mapping, global role-based squads, load-bearing issue threads, and strict codebase grounding, we eliminate the complexity and configuration rot often associated with multi-agent orchestration. 

This model proves that you do not need complex cloud integrations or per-project agent configurations to build software efficiently. All you need is **one local daemon, five global squads, and a simple, disciplined workflow** to bring your ideas to life.

*To explore the global squad configurations used in this guide, check out the [jefflunt/build-multica](https://github.com/jefflunt/build-multica) repository.*
