---
title: CNAB Security: Verification workflows
weight: 303
---

# Verification workflows

- [Verification workflows](#verification-workflows)

This document is a _prescriptive_ part of [CNAB Security](300-CNAB-security.md).

The keywords MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119). The use of these keywords in this document are statements about how a CNAB implementation may fulfill the CNAB Security specification _only_.

## Verification workflows

This section describes the verification workflow that runtimes compliant with the CNAB Security Specifications MUST perform in order to verify the provenance and attestation of bundles. Regardless of [the bundle format][bundle-formats], the verification workflow is the same, with the signature of a thin bundle being the content digest of the `bundle.json` file, while the signature of a thick bundle being the content digest of the bundle archive.

End-users SHOULD see no difference in their experience, unless an attack is caught. In that case, the installation of the bundle in question MUST be denied, and users SHOULD see an error message indicating why the verification workflow failed.

In order to download and verify a bundle, a compliant runtime MUST take the following steps:

1. Use the [TUF workflow][tuf-workflow] to download trusted metadata about the desired bundle (e.g., `example.com/example-org/example-bundle:latest`) from the [metadata repository][metadata-repository]. A bundle runtime MUST use the rules outlined in [TAP 4](https://github.com/theupdateframework/taps/blob/master/tap4.md) to securely resolve bundles from different metadata repositories on different servers.
1. Download the (thin or thick) bundle itself. This operation can follow the [CNAB Registries Specification][registry-spec] for thin bundles and pull the bundle from an OCI registry, or use other protocols of obtaining it.
1. Compare the observed hashes of the downloaded bundle to the trusted hashes. If they are not equal, the runtime MUST discard the downloaded bundle, report the error, and stop execution. Otherwise, go to the next step.
1. If there is [associated in-toto metadata][metadata-repository] about the bundle, use the [TUF workflow][tuf-workflow] to download in-toto root layout, public keys for the root layout, and link metadata for the bundle, and go to the next step. Otherwise, halt, and return the verified bundle to the user.
1. Call the in-toto runtime to [verify and inspect](https://github.com/in-toto/docs/blob/e9806a000c32dea73f6044a140386f601c7d4e18/in-toto-spec.md#52-verifying-the-final-product) the bundle using the downloaded root layout, public keys for the root layout, link metadata, and the bundle itself.
    1. The bundle runtime MAY run in-toto directly on the operating system, in a default container image provided by the bundle runtime, or in a custom container image provided by the user. In any case, note that the bundle runtime SHOULD ensure that in-toto inspections, which are arbitrary commands, run within bounded resources (e.g., space, time) so as not to to run into accidental [denial-of-service attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack). It is also up to the bundle developers to ensure that in-toto inspections are as portable as possible.
    1. If the verification and inspection of the bundle fails for any reason, the runtime MUST discard the downloaded bundle, report the error, and stop execution. Otherwise, if it passes, it MUST return the verified bundle to the user.

Note that in-toto security guarantees depend on the public keys for the root layout being signed with offline keys in the TUF targets metadata.

[bundle-formats]: 104-bundle-formats.md
[metadata-repository]: 301-metadata-repositories.md
[registry-spec]: 200-CNAB-registries.md
[tuf-workflow]: https://github.com/theupdateframework/specification/blob/master/tuf-spec.md#5-detailed-workflows
