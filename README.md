# knos-oidc-rotate

One workflow, [`rotate.yml`](.github/workflows/rotate.yml). It is how a new GitHub Actions or GitLab CI signing key
becomes trusted by [knos-oidc](https://github.com/drexthealpha/Knos), the Solana program that verifies those issuers'
OIDC tokens on chain with no admin.

knos-oidc has this repository, the workflow's path and its commit sha fixed in its (immutable) binary. Every six
hours the workflow reads the issuers' public key sets and asks GitHub to sign a token naming each key's hash. knos-oidc accepts
a new key only on such a token, signed by a GitHub key it already trusts. Nobody, Knos included, can add a key any
other way, and this file can never change: another commit is another sha, which the program does not accept.

Anyone can run it. From any repository:

```yaml
on: { schedule: [{ cron: "7 3 * * *" }], workflow_dispatch: }
jobs:
  keys:
    permissions: { id-token: write, issues: write }
    uses: drexthealpha/knos-oidc-rotate/.github/workflows/rotate.yml@<the commit knos-oidc pins>
```

The token GitHub signs names this file at that commit whoever calls it, so rotation does not depend on this
repository, or on Knos, staying around.

MIT licence.
