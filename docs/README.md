# docs (SomeoneIsWorking fork)

This fork carries the RE notes of the [psx-crash](https://github.com/SomeoneIsWorking/psx-crash) port, which consumes
it as a submodule and reads guest addresses from its symbol map (`config/symbol_addrs.txt`). Paths such as `game/...`, `titles/...` and
`docs/issues/...` in these files are psx-crash paths. The decomp itself is upstream's; see the top-level README.

| file | what |
|---|---|
| `re-frontier.md` | the port's ordered RE evidence chain for this title (`re_frontier.py`) |
| `prior-art.md` | other work on this build, licences and what may be taken |
| `our-names.md` | the port's names where they differ from or add to this tree's symbol map, and the code sites it uses |
| `notes/` | behaviour notes on guest routines |
