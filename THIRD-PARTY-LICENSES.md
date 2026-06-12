# Third-Party Licenses

GloSSH is MIT-licensed (see [LICENSE](LICENSE)). It vendors one third-party
source tree, under public-domain terms. The full upstream license text is
retained inside the vendored tree.

| Component | License | Upstream | In-tree license file |
|-----------|---------|----------|----------------------|
| TinySSH | Public domain (CC0 fallback) | https://github.com/janmojzis/tinyssh | `components/tinyssh/tinyssh/LICENCE` |

TinySSH supplies all of GloSSH's cryptography (SSH server *and* the glo_mtls
TLS 1.3 client): X25519, ChaCha20, Poly1305, SHA-256/512, Ed25519, sntrup761.
None of it is copyleft — no GPL, LGPL, or commercial-license obligation.

GloSSH's own MIT code adds an IETF ChaCha20 (RFC 8439) in
`components/glo_mtls/src/glo_chacha20.c`.

## Provenance note

TinySSH was vendored flat (copied source, no submodule) from a working
ESP-IDF port, so it may contain local ESP-IDF compatibility edits relative to
pristine upstream. To resync with upstream security patches, diff against the
upstream repository above.
