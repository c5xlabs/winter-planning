# Live preview decisions

## Reaching a live workspace (decision:preview/reachability)

The preview path is a tunnel the runner opens and the hub multiplexes, chosen over an operator-supplied private network
or a published runner, because it is the only one of the three that keeps the hub from reaching into a runner.
