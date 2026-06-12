# GloSSH

**Permissively-licensed secure transport for ESP-IDF — TinySSH all the way down.**

GloSSH is a de-wolfed secure-transport stack for ESP32 / ESP-IDF. The embedded
SSL/SSH options are awkward: wolfSSL/wolfSSH are GPLv2-or-commercial, and even
the permissive default (mbedTLS) is Apache-2.0. GloSSH is **MIT / public-domain**
end to end — no GPL, no commercial license, no Apache NOTICE chain.

It was extracted from the [drawbridge](https://github.com/EC-SH/drawbridge)
edge-PBX project to replace its `wolfSSH` / `wolfSSL` dependency.

## Architecture

Both the SSH server and the TLS client are built on **TinySSH's tinynacl
crypto** (X25519, ChaCha20-Poly1305, SHA-256, Ed25519, sntrup761 PQ) — modern,
constant-time, public-domain, and small. No libtomcrypt, no X.509, no RSA.

| Component | Role | License | Status |
|-----------|------|---------|--------|
| `components/tinyssh`  | Minimal SSH server | Public domain | ESP-IDF port, working |
| `components/glo_mtls` | Outbound mutual-TLS 1.3 client (raw-key Ed25519, RFC 7250) | MIT | **crypto layer done + self-tested**; record/handshake next |
| `main/`               | Example: Wi-Fi bring-up + glo_mtls self-test + TinySSH server | MIT | example |

## glo_mtls

A deliberately narrow TLS 1.3 client — client-only, version-pinned, AEAD-only,
`TLS_CHACHA20_POLY1305_SHA256` + X25519 — that authenticates peers with **raw
Ed25519 public keys (RFC 7250)** rather than X.509 certificates. That fits the
drawbridge model exactly (outbound-only mTLS to an operator-owned media anchor:
you control both ends, so no CA/PKI is needed).

**Implemented now** (built on TinySSH primitives, with RFC known-answer tests):

| Layer | Spec | Verified by |
|-------|------|-------------|
| X25519 ECDH | RFC 7748 | §5.2 KAT + ECDH agreement |
| HMAC / HKDF / HKDF-Expand-Label | RFC 5869 / 8446 | RFC 5869 TC1 KAT |
| IETF ChaCha20 | RFC 8439 §2.4 | (via AEAD KAT) |
| AEAD ChaCha20-Poly1305 | RFC 8439 §2.8 | §2.8.2 KAT + seal/open + tamper |
| Ed25519 raw-key auth | RFC 8032 | sign/open + tamper |

The self-test runs at boot and prints per-check `PASS`/`FAIL`. See
[`glo_mtls_crypto.h`](components/glo_mtls/include/glo_mtls_crypto.h).

> One small wrinkle handled: TinySSH ships the DJB 64-bit-nonce ChaCha20, but
> TLS needs the IETF 96-bit-nonce / 32-bit-counter construction — so glo_mtls
> supplies its own IETF ChaCha20 and reuses TinySSH's Poly1305/X25519/SHA-256.

**Roadmap:** TLS 1.3 record layer + handshake state machine
(`glo_mtls_connect/read/write/close`).

## Build (example)

```sh
idf.py set-target esp32s3
# edit main/main.c: set EXAMPLE_STA_SSID / EXAMPLE_STA_PASSWORD / EXAMPLE_AUTHORIZED_KEY
idf.py build flash monitor
```

On boot the example runs the glo_mtls crypto self-test, then brings up Wi-Fi
and the TinySSH server. **All credentials in `main.c` are placeholders** —
replace them before flashing; never commit real ones.

## Licensing

GloSSH's original work (port glue, `glo_mtls`, build system) is **MIT** — see
[LICENSE](LICENSE). TinySSH is vendored under its own public-domain terms.
Nothing here is copyleft. See [NOTICE](NOTICE) and
[THIRD-PARTY-LICENSES.md](THIRD-PARTY-LICENSES.md).

TinySSH was **vendored flat** (copied source, not a submodule) from a working
ESP-IDF port, so it may carry local compatibility edits. To pull upstream
security patches, diff against the upstream listed in THIRD-PARTY-LICENSES.md.
