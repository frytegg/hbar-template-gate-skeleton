# hbar-template-gate-skeleton

A minimal, Hardhat-only Scaffold-HBAR template that exists to be scaffolded. GitHub Actions scaffolds this repository through the published `create-scaffold-hbar` CLI, from GitHub, the way any community template is scaffolded, and checks the project that comes out: strict lint, types, production build, and a boot with no env file.
It is not a finished template and ships no product of its own: the app is the blank Scaffold-HBAR starter with Hardhat selected, minus its default keys, dead scripts and Foundry leftovers. It stands in for a template that is not public yet, whose `package.json` files, lockfile, `template.json`, configuration files and gate scripts (`tools/gate/`) are copied here when they change.

## Scaffold it

```bash
npx create-scaffold-hbar@latest my-app --template frytegg/hbar-template-gate-skeleton -s hardhat
```

The same through npm's `create` command; the `--` is required, since without it the options go to npm, not to the CLI:

```bash
npm "create" scaffold-hbar@latest my-app -- --template frytegg/hbar-template-gate-skeleton -s hardhat
```

Both commands install the dependencies and make the first commit. To scaffold for npm, not the default package manager, end either one with `--package-manager "npm"`.

| Needed | Why |
| --- | --- |
| Node.js 20.18.3 or later | `engines` in `package.json` |
| git, with `user.name` and `user.email` set | the CLI commits the scaffold, and stops before creating anything when git has no identity |
| the default package manager on your `PATH`, any release from 1.0 | the CLI checks for it before scaffolding; the project then runs the release pinned in `package.json` (`packageManager`) |

Foundry (`forge`) is not needed. Name the project in lowercase, as a single path segment: with `--yes` the CLI replaces a name it rejects, one with a capital letter for instance, by `my-hedera-dapp` and still exits 0.

## What CI runs here

| Workflow | When | What |
| --- | --- | --- |
| `gate-skeleton.yml` | every push to `main`, nightly, by hand | the scaffold path against this repository. A baseline leg (Node 20.18.3, `-s hardhat`, `--yes`, the default package manager, Hedera Skills off, `CI=true`), then one leg per switch changed against it and one per pair of switches that interact: no `-s`, `--package-manager "npm"`, the same with `npm@12`, Hedera Skills on, `CI` unset, Node 22 and the current LTS, flags instead of `--yes`. Each of these legs fails the run when it fails; the one without `-s` tests that `template.json` alone selects Hardhat. Two exploratory legs, `--ci` instead of `--yes` and no directory argument, never fail it. Then a secret scan of the tree and of every commit, which fails the run too, and three recorded runs of the scaffold command as the bounty brief prints it, with `npm@10`, `npm@11` and `npm@12` |
| `hosts-control.yml` | nightly, by hand | the same gate script against the hosts' own blank template, never blocking: a leg red in both workflows points at the base, a leg red in `gate-skeleton.yml` only points at this template |
| `lint.yaml` | every push and pull request to `main` | this checkout installed from its lockfile, then `lint:strict`, `typecheck`, `check:tools`, `probe:routes:check` and ShellCheck on the gate scripts |

No workflow uses a secret or a token. Each gate leg prints the runner's unauthenticated GitHub quota first: the CLI reads `template.json` from the GitHub API without a token and, going by its source, falls back to its own defaults when that read is refused, so a failed leg next to an exhausted quota says nothing about the template. `tools/gate/README.md` describes every step, and how to run one leg on your machine.

## Develop and check

```bash
yarn dev                        # development server on http://localhost:3000
yarn lint:strict                # ESLint and Prettier on both packages, no warning allowed
yarn typecheck                  # both packages; compiles the contracts first
yarn build                      # production build of the frontend
yarn test                       # Hardhat tests on a fork of Hedera testnet (network needed)
yarn check:all                  # lint:strict, typecheck, the tools' tests, the docs checks (registry needed)
yarn gate:local                 # one gate leg against the committed HEAD; well over 1 GB of disk
```

`AGENTS.md` lists every command, the checks to run before a change is done and the invariants a change has to keep; Claude Code reads it through `CLAUDE.md`.

## Environment variables

Nothing needs to be set: the app, the build and the tests run with no env file. Each variable goes in the file named below; a scaffolded project also gets a root `.env.example` listing them, but no package reads env files at the root.

| Variable | File | Required | Default | Read by |
| --- | --- | --- | --- | --- |
| `NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID` | `packages/nextjs/.env.local` | no | empty: WalletConnect is off, browser-injected and burner wallets are offered | `packages/nextjs/scaffold.config.ts` |
| `HEDERA_RPC_TESTNET_URL` | `packages/nextjs/.env.local` | no | `https://testnet.hashio.io/api` | the `/api/hedera/rpc` relay, on the server |
| `HEDERA_RPC_MAINNET_URL` | `packages/nextjs/.env.local` | no | `https://mainnet.hashio.io/api` | the same relay, for mainnet |
| `HEDERA_MIRROR_TESTNET_URL` | `packages/nextjs/.env.local` | no | `https://testnet.mirrornode.hedera.com` | the `/api/hedera/account` route, on the server |
| `HEDERA_MIRROR_MAINNET_URL` | `packages/nextjs/.env.local` | no | `https://mainnet.mirrornode.hedera.com` | the same route, for mainnet |
| `HEDERA_RPC_URL` | `packages/hardhat/.env` | no | `https://testnet.hashio.io/api` | the in-process Hardhat network, which forks it (`hardhat:chain`, `test`) |
| `DEPLOYER_PRIVATE_KEY_ENCRYPTED` | `packages/hardhat/.env` | for a live deploy | none; written by `hardhat:account:generate` or `hardhat:account:import` | the deploy script, which asks for its password |
| `__RUNTIME_DEPLOYER_PRIVATE_KEY` | never a file: the shell, for one command | no | none | `packages/hardhat/hardhat.config.ts`, the only key live networks sign with; the deploy script sets it from the encrypted key |

There is no fallback key: the upstream configuration fell back to Hardhat's well-known account #0, which is a funded account on Hedera testnet.

## What this skeleton does not prove

- Anything about the final tree of the template it stands in for: that template has its own code, routes and checks. This repository checks the scaffold path and the manifest mechanics, nothing more.
- The interactive path, prompts answered by hand: a runner has no TTY.
- Browser console errors: no workflow here loads the pages in a browser; `lint.yaml` runs the route probe's own tests only.
- Hedera mainnet: nothing here has been deployed to it or tested against it.
- This code is experimental and has not been audited.

## Licence and provenance

MIT, see `LICENCE`; the BuidlGuidl (Scaffold-ETH 2) and hedera-dev (Scaffold-HBAR) notices are kept above this repository's own.
The starter was scaffolded with `create-scaffold-hbar` 0.4.0 from the blank-template branch of hedera-dev/scaffold-hbar at commit 88c8837, with Hardhat selected and the skills install off. The `@x402/*` build guard in `packages/nextjs/next.config.ts` comes from that repository's main branch at commit 5eb46ef.

<!-- checks:allow
paths: template.json
-->
