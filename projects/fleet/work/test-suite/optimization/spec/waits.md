# Waits that end when the thing has happened

Three fixed waits, in two files, each replaced by a wait on the event it stood for (`engineering:event-driven-waits`).

## `tests/unit/cli/test_cli.py`

`_serve_on` stands up a fake runner daemon on a Unix socket, and when its reply is `None` it holds the connection with
`time.sleep(LOCAL_CLIENT_TIMEOUT * 3)`, a daemon that accepts and never answers. That sleep runs on a daemon thread and
costs nothing. What costs is the client side: `test_an_ambiguous_liveness_answer_fails_closed[None-did not answer]`
waits out the production client's `LOCAL_CLIENT_TIMEOUT`, five seconds (`src/blizzard/runner/cli/daemon.py`), before the
liveness check gives up.

The test's subject is that silence fails closed, not that the timeout is five seconds. `daemon_holding` takes the client
timeout as a keyword, defaulting to `LOCAL_CLIENT_TIMEOUT`, and the silent case passes a quarter of a second. The value
of the production constant is asserted where it is defined, if anywhere, not by every test that meets it. The fake
daemon's hold becomes an `Event` the fixture sets on teardown, so the thread ends with the test rather than fifteen
seconds later.

## `tests/component/runner/harness/opencode/compatibility/test_diagnostic.py`

`test_timeout_kills_and_reaps_an_opencode_descendant` and
`test_successful_run_rejects_and_reaps_a_pipe_closing_descendant` each start a launcher whose descendant sleeps half a
second and then writes a marker, and the test sleeps `0.7` seconds before asserting the marker is absent. The absence is
a stand-in for the claim the tests make, that the descendant was reaped, and it costs 0.7 seconds each while proving it
only as long as the machine is fast enough to have written the marker by then.

Each descendant writes its pid to a file as its first act, and the test asserts the reaping directly: once `run`
returns, the recorded process is gone, checked with `os.kill(pid, 0)` raising `ProcessLookupError` and, where the
descendant is the test's own child, `os.waitpid` reporting it. The check polls with a bound of five seconds, as the
file's other bounds are, and returns the moment it holds. The marker and the half-second sleep go, since the descendant
no longer has to live long enough to be caught misbehaving.
