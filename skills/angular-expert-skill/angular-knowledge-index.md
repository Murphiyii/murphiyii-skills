# Angular Knowledge Index

Use this index to locate the specific documentation file for a given topic.
When you need detailed information on a topic, use the `view_file` tool to read the corresponding file path.

**Base Path:** `~/.claude/skills/angular-expert-skill/KNOWLEDGE/`

| Topic                       | File                              | Description                                                                                       |
| :-------------------------- | :-------------------------------- | :------------------------------------------------------------------------------------------------ |
| **Core Concepts**           | `angular-core.md`                 | Signals, Change Detection (OnPush), Lifecycle Hooks, Dependency Injection, Standalone Components. |
| **Templates & UI**          | `angular-templates.md`            | Control Flow (@if, @for), Data Binding, Directives, Pipes, View Encapsulation.                    |
| **Forms & Routing**         | `angular-forms-routing.md`        | Reactive Forms, Validation, Router Setup, Lazy Loading, Guards, Resolvers.                        |
| **HTTP & RxJS**             | `angular-http-rxjs.md`            | HttpClient, Interceptors, RxJS Interop (toSignal, toObservable), Resources.                       |
| **Testing**                 | `angular-testing.md`              | Unit Testing (TestBed), Component Testing, Service Testing, Spies, Mocks.                         |
| **Animations & Harnesses**  | `angular-animations-harnesses.md` | CSS Animations (animate.enter/leave), Component Harnesses for testing.                            |
| **Security & i18n**         | `angular-security-i18n.md`        | XSRF/CSRF Protection, XSSI, Internationalization, Localization.                                   |
| **Migration & Performance** | `angular-zoneless-migration.md`   | Zoneless Change Detection, Removing Zone.js, View Transitions, Performance Optimization.          |

## Usage Instruction
If a user asks about "Signals", you should:
1. Look up "Signals" in this index -> Found in **Core Concepts**.
2. Construct the path: `~/.claude/skills/angular-expert-skill/KNOWLEDGE/angular-core.md`.
3. Use the `Read` tool on that path.
4. Use the content to answer the user.
