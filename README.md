# knos-oidc-rotate

Two workflows that the [Knos](https://github.com/drexthealpha/Knos) programs on Solana name by commit sha. A commit
sha fixes a file's content, so what these workflows do cannot be changed without the programs refusing the result.

## [`rotate.yml`](.github/workflows/rotate.yml): issuer keys

knos-oidc verifies GitHub Actions and GitLab CI tokens on chain. It has to learn the keys those issuers sign with.
This workflow reads both issuers' public key sets on a GitHub-hosted runner and asks GitHub to sign a token naming
each key's hash. A key reaches the program only on such a token, signed by a GitHub key it already trusts.

- The first deployment (immutable) accepts the token from any repository that calls this file at the pinned commit.
- The second deployment also requires GitHub's signature to say the run happened in a repository of this account
  (`repository_id` and `repository_owner_id`), then holds the key for a day in public view, needs the guardian's
  approval, and lets every key expire unless this workflow names it again. The guardian can remove a key and can
  never add one.

## [`claim.yml`](.github/workflows/claim.yml): where an account is paid

A GitHub account names the Solana address its Knos payments go to by running this file from a repository named
`knos-claim` that it owns. knos-pay accepts the token only from this file at the pinned commit, from a run started
by hand (`workflow_dispatch`) by the owner of the repository it ran in. Nobody else can get GitHub to sign that for
your account.

```yaml
# .github/workflows/claim.yml in <you>/knos-claim
on:
  workflow_dispatch:
    inputs: { address: { description: Solana address, required: true, type: string } }
permissions: {}
jobs:
  claim:
    permissions: { id-token: write, issues: write }
    uses: drexthealpha/knos-oidc-rotate/.github/workflows/claim.yml@<the commit knos-pay pins>
    with: { address: "${{ inputs.address }}" }
```

Neither workflow takes a secret. Both post the tokens in public: each names one key hash or one address and nothing
else, and a relayer who carries it only pays the fee.
