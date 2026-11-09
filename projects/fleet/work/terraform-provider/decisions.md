# Terraform provider decisions

## Provider repository (decision:terraform-provider/provider-repository)

**Open.** Whether the provider lives in blizzard's own repository or in one of its own: the registry expects a
repository per provider, and the provider is written in Go where blizzard is not.

## Retiring a record with live work (decision:terraform-provider/retiring-live-records)

**Open.** What an apply does when it would retire a record that still carries live work: refuse, or retire it and let
the work already in flight finish. It blocks `product:destroy-retires`.

## Hub-level records (decision:terraform-provider/hub-level-records)

**Open.** Whether the provider reaches hub-level records, such as enrolled runners and the administrator's tenant list,
or stops at what a tenant owns. It blocks `product:record-resources`.

## Recommended configuration path (decision:terraform-provider/configuration-path)

**Open.** Whether `epic:live-config`'s declarative file and this provider both remain, or one becomes the recommended
path for configuration kept in a repository.
