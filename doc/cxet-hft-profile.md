# CXET HFT transport profile

`WOLFSSL_CXET_HFT` is an opt-in profile for the CXET Linux x86-64 transport
build. It preserves the normal wolfSSL certificate, SNI, hostname-verification,
record, cipher and socket-I/O behavior.

The profile changes only the bounded read-to-write mailbox used after
`wolfSSL_write_dup()`:

- fatal read errors and close notifications use acquire/release atomics;
- TLS 1.3 `KeyUpdate(update_requested)` uses a one-byte atomic mailbox;
- the existing mutex remains the owner of cold duplicate lifetime/refcount
  destruction.

The application must quiesce both duplicate sides before either side is freed.
The profile does not make concurrent destruction safe.

The CMake profile is fail-closed. It supports Linux x86-64 with GCC or Clang,
enables `WOLFSSL_WRITEDUP`, and rejects `SINGLE_THREADED`, DTLS, QUIC and TLS
1.3 post-handshake authentication.

Configure with:

```sh
cmake -S . -B build-cxet \
  -DWOLFSSL_CXET_HFT=yes \
  -DWOLFSSL_DTLS=no \
  -DWOLFSSL_DTLS13=no \
  -DWOLFSSL_POSTAUTH=no \
  -DWOLFSSL_QUIC=no
```

The profile build exports `wolfSSL_CXET_hft_profile_version()` and defines
`WOLFSSL_CXET_HFT` in generated `wolfssl/options.h`. CXET's build/staging gates
require both markers and reject a profile-off or stale shared library.

This profile is a latency candidate until the CXET end-to-end transport gates
have been run. The source change alone is not evidence of an improvement.
