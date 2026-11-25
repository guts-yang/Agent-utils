# Prompt Migration Map

The project-specific prompt files remain available as compatibility templates:

1. [`01-project-init.md`](../../../../../prompts/research-reproduction/01-project-init.md)
   maps to Preflight / Init.
2. [`02-idea-discovery.md`](../../../../../prompts/research-reproduction/02-idea-discovery.md)
   maps to Idea discovery.
3. [`03-experiment-bridge.md`](../../../../../prompts/research-reproduction/03-experiment-bridge.md)
   maps to Experiment bridge.
4. [`04-review-and-claim.md`](../../../../../prompts/research-reproduction/04-review-and-claim.md)
   maps to Review / audit / claim.

Use `/unlearning-reproduction` as the canonical entry point for new runs. The
prompt files are useful when a user wants to run one phase manually or inspect
the project-specific wording. They must not introduce a different phase order,
stop condition, or evidence gate.

The canonical cross-project rules live in:

- [`SKILL.md`](../SKILL.md): orchestration and routing;
- [`project-config.md`](project-config.md): paths, repositories, baselines, and
  environment variable names;
- [`handoff-contract.md`](handoff-contract.md): state and artifact contract.
