# Agent design notes: scope, data and human review

Things I check before and while building a Copilot Studio / Agent Builder agent for internal users. Most agent problems I've seen were not about prompts but about unclear scope and unclear data.

## 1. Scope in one sentence

Write down what the agent does and for whom, e.g. *"Helps analysts create a first version of a Power BI report from a description of what they need."* If it doesn't fit in one sentence, it's probably two agents.

Then list what it should **not** do. For a reporting agent: no publishing to shared workspaces, no changes to the semantic model, no answers about numbers it can't trace back to a source.

## 2. Knowledge sources

- Prefer a small set of curated sources (a SharePoint library with the standards, templates and examples) over "everything in the site".
- Each source should have an owner and a review date. Stale guidance is worse than none, because the agent repeats it confidently.
- Test with questions whose answers you know are in the sources, and with questions whose answers are *not* there. The second set matters more.

## 3. Native vs. extra integration

Before building anything custom, check what the platform already does. For each capability:

| Capability | Native in Copilot? | Needs a flow / connector? | Needs an external execution layer? |
|---|---|---|---|
| answer from documents | yes | | |
| look up a record in a list | | Power Automate / connector | |
| generate a file (e.g. report definition) | | | often yes |

Anything in the last column is where effort, risk and support cost go up. It's also where users end up depending on developer tools, which is usually what the agent was supposed to avoid.

## 4. Human in the loop

- The agent drafts, a person approves. Especially for anything that ends up in front of management or regulators.
- Show sources with every answer that contains a number or a rule.
- Log questions the agent couldn't answer. That list is the best input for improving the knowledge sources.

## 5. Rollout

- Start with a few users who know the process well and will report wrong answers.
- Collect use cases in one place (what the user tried, what happened, what they expected).
- A short guide for colleagues who want to build their own agent saves a lot of repeated questions.
