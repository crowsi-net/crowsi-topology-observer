# crowsi-topology-observer

Acquire a topology view only after checking authorization and source authenticity.

## What you can do

- Validate the configured topology exchange.
- Return a bounded, verified observation.

## Current scope

The operator supplies registered endpoints and trust. Acquisition does not establish ownership or permission to alter the topology.

Package distribution is not activated by this documentation. Use the checked-in source and the declared dependency versions; published availability must be verified separately.

## Getting started

Install Rust 1.97 or newer and make the declared dependencies available. Use the configured private registry when a dependency is not distributed publicly. Run from this repository:

```sh
cargo test --locked
```

## Documentation and source

[Usage guide](docs/getting-started.md)

[Schemas](schemas) · [Implementation and public interfaces](src) · [Verification cases](tests) · [Contributing](CONTRIBUTING.md) · [Security reporting](SECURITY.md) · [License](LICENSE) · [Attribution notices](NOTICE)
