# State Template

Create `$state_dir/STATE.md` on the first tick. Update it whenever a sub-agent is launched, stopped, or reports. Keep the tick log short; prune entries older than a day.

```markdown
# PR steward state: <owner>/<repo>

## Sub-agents
| PR | Job | Sub-agent ID | Status / next step |
|---|---|---|---|

## Last clean review head per PR
#<N> <sha> · #<N> <sha>

## User rules and decisions
- <rule given mid-loop, with the date>
- <decision the steward made and told the user, e.g. which PR renumbers a colliding spec section>

## Pairwise conflicts between open PRs
- #<A> vs #<B>: <file, and how whichever merges second resolves it>

## Follow-ups filed
- #<issue> from #<PR>: <title>

## Open questions to the user
- <question>; do not act until answered

## Tick log
- <time>: <what changed>
```
