# GitHub Copilot CLI Tool Mapping

Skills are written using Claude Code tool names. When a skill references one of these tools, use the Copilot CLI equivalent.

| Skill references | GitHub Copilot CLI equivalent |
|-----------------|-------------------------------|
| `Read` | Read files directly |
| `Write` | Create or overwrite files |
| `Edit` / `MultiEdit` | Edit files |
| `Bash` | Run shell commands |
| `Grep` / `Glob` | Search files or use shell search commands |
| `Skill` tool | Use the platform's native skills support when available, or read the `SKILL.md` file directly |
| `Task` tool (dispatch subagent) | Use a custom agent, `/agent`, or Copilot CLI subagent support when available |
| `TodoWrite` | Track progress in the plan file or working notes |
| `WebSearch` / `WebFetch` | Use native web capabilities if available; otherwise say the platform lacks them |

## Custom Agents

GitHub Copilot CLI supports custom agents. When a Superpowers workflow expects role separation, prefer:

- `agents/implementer.agent.md` for task execution
- `agents/spec-reviewer.agent.md` for scope checking
- `agents/code-quality-reviewer.agent.md` for quality review
- `agents/code-reviewer.agent.md` for final whole-change review

If the runtime cannot launch the agent automatically by name, read the profile file and continue in the same session with the same role mindset rather than silently skipping the review.

## Important Limitations

- Plugin-installed skills can be discovered without copying them into the repo
- Startup bootstrap and always-on context injection may still depend on project or user instructions
- In local verification on Copilot CLI 1.0.14, plugin-packaged agent profile files were present on disk but were **not** exposed through `--agent` as selectable custom agents
- Do not assume a plugin hook can enforce behavior unless you have verified that capability in the current Copilot CLI version

## Practical Fallback

When a workflow expects role-separated agents and Copilot CLI does not expose the plugin's agent profiles as invocable custom agents:

1. Read the matching agent profile file from the plugin directory
2. Announce that you are switching into that role
3. Continue in the same session with that role's instructions

This preserves the review mindset even when the runtime does not surface the plugin-packaged agents directly.
