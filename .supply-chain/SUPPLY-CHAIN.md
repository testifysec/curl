# TestifySec Certified curl — Supply-Chain Provenance

This repository is an **independent, TestifySec-controlled mirror** of
[`curl/curl`](https://github.com/curl/curl), maintained as a certified internal
source of truth. Source integrity and build provenance are enforced with
[cilock](https://cilock.dev) attestations and a signed verification policy.

## Certified snapshot

| Field | Value |
|---|---|
| Upstream | `github.com/curl/curl` |
| Pinned release | `curl-8_20_0` (curl 8.20.0) |
| Pinned commit | `a05f34973e6c4bb629d018f7cb51487be1c904d8` |
| Mirror | `github.com/testifysec/curl` (`master` = pristine upstream mirror) |
| License | curl license (MIT-derived) — see `COPYING`, preserved unchanged |

## Evidence produced (keyless Fulcio + RFC 3161 TSA → Archivista)

Generated with **cilock v3.0.0** (sha256 `fc4af7bac12b0caac9642283da63e30982c14a0d96879de72e0a27756c9ddd9c`,
itself integrity-validated against the cilock.dev manifest + sidecar before use).

| Attestation | Step | gitoid (sha256) |
|---|---|---|
| Source provenance | `source-git` | `8a162d0285df4ac20011718bab1196219fa54d95dedcc8f4c589a127e7c55cbd` |
| SBOM (SPDX-2.3, 42 pkgs) | `sbom-source` | `a3099204335ce22e8c06fa214cd52a423a07ed8aab4a8edf8db716eab98e9be5` |

- Archivista: `https://platform.testifysec.com/archivista`
- SBOM file: `.supply-chain/curl-8.20.0.sbom.spdx.json`

## Policy

`.supply-chain/testifysec-curl.policy.signed.json` — DSSE-signed Witness policy
(verify with `.supply-chain/testifysec-curl-policy.pub`). It enforces two steps,
`source-git` and `build`, each requiring a keyless Fulcio identity that chains to
the **TestifySec Platform Fulcio CA** (`platform-fulcio` root) AND whose cert
extensions pin:

- `Issuer = https://token.actions.githubusercontent.com`
- `SourceRepositoryURI = https://github.com/testifysec/curl`
- `BuildConfigURI = …/.github/workflows/cilock-attest.yml@*`
- `RunnerEnvironment = github-hosted`

`build` declares `artifactsFrom: [source-git]`, enforcing that the built binary's
inputs are the certified source tree (artifact continuity). Policy expires 2027-06-08.

## How to verify locally

```bash
cilock verify ./src/curl \
  --policy .supply-chain/testifysec-curl.policy.signed.json \
  --publickey .supply-chain/testifysec-curl-policy.pub \
  --enable-archivista
# exit 0 = certified; gate CI/deploys on the exit code, never on grep.
```

## Ongoing CI

`.github/workflows/cilock-attest.yml` re-attests source + build on every push to
`supply-chain` and on release tags, signing with the workflow's OIDC identity
(SLSA build-track L2+), then **gates the build** on the policy above. The repo's
Actions identity is federated to the TestifySec tenant via
`cilock trust github testifysec/curl`, so no long-lived signing secrets exist.

## Framework mapping

- **SSDF (NIST SP 800-218):** PS.1/PS.2/PS.3 (protect & provenance source/releases),
  PO.3 + PW.6 (build with integrity + provenance), RV.1–RV.3 (verify before use).
- **SLSA v1.0:** Source track (controlled, tamper-evident mirror); Build track L2→L3
  (signed provenance from a hosted builder identity, policy-gated verification).
