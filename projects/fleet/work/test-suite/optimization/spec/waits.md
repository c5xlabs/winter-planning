# Waits on events

Paths are relative to the `blizzard` repo root.

## The silent daemon

```python
# src/blizzard/runner/cli/daemon.py — engineering:event-driven-waits,
# decision:test-suite/optimization/client-timeout-keyword
class Door:
    def uds_client(self, sock: Path, *, timeout: float = LOCAL_CLIENT_TIMEOUT) -> Client: ...


def daemon_holding(sock: Path, door: Door, *, timeout: float = LOCAL_CLIENT_TIMEOUT) -> str | None:
    """Unchanged behaviour: a socket that does not answer its health probe within ``timeout`` is still held."""
```

```python
# tests/unit/cli/test_cli.py
def _serve_on(sock_path: Path, reply: bytes | None, release: threading.Event) -> socket.socket:
    """``reply is None`` accepts the connection and never answers, until ``release`` is set."""
    # the silent branch: release.wait() replaces time.sleep(LOCAL_CLIENT_TIMEOUT * 3), which outlived the test by 15 s


# the fixture sets `release` on teardown, so the thread ends with the test
# test_an_ambiguous_liveness_answer_fails_closed[None-did not answer]: today it waits out the 5.0 s LOCAL_CLIENT_TIMEOUT
holding = daemon_holding(HostLayout(root).legacy_socket_path, _DOOR, timeout=0.25)
assert holding is not None and "did not answer" in holding  # still fails closed
```

## Reaping a descendant

```python
# tests/component/runner/harness/opencode/compatibility/test_diagnostic.py — engineering:event-driven-waits,
# decision:test-suite/optimization/reaping-asserted-by-pid
# test_timeout_kills_and_reaps_an_opencode_descendant, test_successful_run_rejects_and_reaps_a_pipe_closing_descendant.
# The launcher's descendant writes its pid to `pid_file` as its first act. Its 0.5 s sleep and marker go, and so does the
# test's time.sleep(0.7) before asserting the marker absent.

result = SubprocessOpenCodeProcess(children=CHILDREN).run(
    [sys.executable, str(launcher), str(pid_file)], cwd=tmp_path, env={}, timeout=...
)  # each test keeps its own timeout and its existing assertions on `result`
pid = int(pid_file.read_text())


def _gone(pid: int, *, bound: float = 5.0) -> bool:
    """Polls until os.kill(pid, 0) raises ProcessLookupError, returning the moment it does; False after `bound`.
    Where the descendant is the test's own child, os.waitpid reporting it counts as gone."""


assert _gone(pid)  # no fixed sleep; 5 s matches the file's other bounds
```

## The rule

`blizzard-context`'s `bzh:service-waits-on-the-system` (`verification/blizzard/service-tests/time.md`) names every tier
(`engineering:waits-rule`):

```markdown
A test in any tier waits on an event, a state, or a process it can observe. A bounded poll is its upper limit, never its
duration.
```
