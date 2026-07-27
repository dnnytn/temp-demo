# SOPS + age POC

Proof of concept for encrypting secrets at rest with [SOPS](https://github.com/getsops/sops)
using [age](https://github.com/FiloSottile/age) as the encryption backend, instead of PGP or a
cloud KMS.

## Layout

```
.sops.yaml            # creation rules: which files get encrypted with which age recipient
examples/secrets.yaml # encrypted example (YAML)
examples/secrets.json # encrypted example (JSON)
examples/secrets.env  # encrypted example (dotenv)
keys/                 # local-only, gitignored: holds the age private key for this POC
```

Every file under `examples/` is already encrypted with sops. Only the values are ciphertext;
keys stay in plaintext so the structure is still diffable in git.

For a full explanation of what SOPS and age are, the problem they solve, why the
ciphertext is safe to commit, and a step-by-step encrypt/decrypt walkthrough, see
[`docs/sops-age-walkthrough.html`](docs/sops-age-walkthrough.html).

## Prerequisites

- [age](https://github.com/FiloSottile/age) (`age`, `age-keygen`)
- [sops](https://github.com/getsops/sops) v3.9+

## Public key used in this POC

```
age1d99kgmr6fd76fzck9pz0xv6vtfzzksemsl4ztz62pmtvwska4dfqlmlgfg
```

This is what `.sops.yaml` encrypts to. The matching private key is **not committed** — it
was generated locally for this demo and is gitignored (`keys/age-key.txt`). To actually
decrypt the example files, get the private key from whoever set up this POC, or generate
your own keypair and re-encrypt the examples against it (steps below).

## Using your own keypair

```bash
age-keygen -o keys/age-key.txt
# note the "Public key: age1..." line it prints, then update .sops.yaml with it
sops updatekeys examples/secrets.yaml examples/secrets.json examples/secrets.env
```

## Common commands

Set this once per shell so sops knows where to find the private key:

```bash
export SOPS_AGE_KEY_FILE=$(pwd)/keys/age-key.txt
```

Decrypt to stdout:

```bash
sops -d examples/secrets.yaml
```

Edit in place (opens decrypted content in `$EDITOR`, re-encrypts on save):

```bash
sops examples/secrets.yaml
```

Encrypt a new file in place (rules come from `.sops.yaml` based on path):

```bash
sops -e -i examples/new-secrets.yaml
```

Rotate to a new data key without changing recipients:

```bash
sops -r -i examples/secrets.yaml
```

## Notes

- The values in `examples/` are fake/demo credentials, not real secrets.
- `.sops.yaml` scopes creation rules to `examples/**` — add more `path_regex` entries as the
  POC grows to cover other directories or formats.
