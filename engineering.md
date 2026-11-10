# Suite engineering requirements

## Rust product CLIs (engineering:rust-product-clis)

All public-facing, released CLI tools that are part of the Winter product suite are implemented in Rust. This includes
the platform executable and launcher shim, Workspace CLI, and Fleet hub and runner CLIs; it does not constrain internal
development scripts or long-running services.
