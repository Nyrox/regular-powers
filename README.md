
# Regular Powers

superpowers, but adapted to people who wish to keep their brain enabled while clanking.

## Differences to superpowers

- Agents are no longer instructed to use skills unprompted and for everything. Invoke regular-powers only when you tell the agent to.
- A dedicated QA phase is added after finishing a implementation to do manual verification and code review. The agent performs bugfixes and suggested improvements as fixup commits and squashes them down before finishing the branch.
- No implementation plans. The design doc ends in a high-level numbered task list; subagent-driven development dispatches each implementer with just the design doc and a task number, and implementers own all implementation decisions.
- Design docs are no longer commited to every repo. Only when the superpowers directory exists, otherwise the agent writes them to a scratch directory.
- All skills have been simplified and slimmed down.

## Compatibility

The skills refer to themselves as superpowers and use the `docs/superpowers` path convention to be fully compatible with other team members using regular superpowers.