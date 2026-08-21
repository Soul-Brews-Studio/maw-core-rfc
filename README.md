# maw-core-rfc

Public design record for a small native maw kernel, its privileged tmux substrate, the separate
serve/God gateway, and external workflow plugins.

- [RFC-0001: Minimal maw Native Kernel](docs/RFC-0001-minimal-maw-kernel.md)
- [RFC-0002: Full maw tmux Substrate](docs/RFC-0002-maw-tmux-substrate.md)
- [RFC-0003: maw Serve/God Gateway](docs/RFC-0003-maw-serve-gateway.md)
- [Discussion and review](https://github.com/Soul-Brews-Studio/maw-core-rfc/issues/1)

## Detailed native command contracts

- [`maw wake`](docs/contracts/wake.md)
- [`maw attach` / `maw a`](docs/contracts/attach-a.md)
- [`maw work`](docs/contracts/work.md)
- [`maw hey`](docs/contracts/hey.md)
- [`maw peek`](docs/contracts/peek.md)

## Source inventories

- [Serve fixed-route and policy ledger](docs/inventories/serve-route-policy.md)
- [Serve WebSocket, fallback, and runtime-plugin ledger](docs/inventories/serve-ws-plugin-routes.md)
