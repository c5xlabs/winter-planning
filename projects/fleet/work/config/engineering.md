# Config engineering

Binds `initiative:config`.

## Promoted constants declare their lifetime (engineering:promoted-constant-lifetime)

`epic:config/sweep` and `epic:config/runtime` land in either order, and neither waits for the other. A constant the
sweep promotes declares its lifetime when it is promoted, using the lifetime `epic:config/runtime` has already agreed
for it where one is agreed ([runtime/spec/settings.md](./runtime/spec/settings.md) §Lifetimes for constants the sweep
will promote).

- If the runtime epic's change for the constant's daemon has landed, a promoted `pass` or `session` constant is an entry
  from the start: seeded from the file and changed through its daemon's API.
- If it has not, the promoted constant arrives as a file key with its lifetime declared and is read at boot until that
  change lands, which makes it an entry like every other key of its lifetime.
