# Repo1

In repo1, `packages/a/package.json` specifies the peerDependency: `has-symbols: '*'`
However, at the moment, that package is not installed in the root directory or mentioned in the root `package.json`.
After calling `pnpm add -w has-symbols@1.0.0`, we will have `has-symbols@1.0.3` as the version for `packages/a` and `has-symbols@1.0.0` in the root dir

## Steps
`cd repo1 && pnpm add -w has-symbols@1.0.0`

## Expected
Expected the `has-symbol@1.0.0` package to be installed in the root workspace, and for `packages/a` to get the same version for `has-symbols` as the one found in the root workspace

## Actual
`packages/a` will get `has-symbols@1.0.3`, which is what `*` matches with no existing dep (now that default resolution is highest match). Only afterwards will `has-symbol@1.0.0` be installed in the root workspace, having no effect on `packages/a`

`pnpm-lock.yaml` content after:
```yaml
lockfileVersion: '6.0'

settings:
  autoInstallPeers: true
  excludeLinksFromLockfile: false

importers:

  .:
    dependencies:
      has-symbols:
        specifier: 1.0.0
        version: 1.0.0

  packages/a:
    dependencies:
      has-symbols:
        specifier: '*'
        version: 1.0.3

packages:

  /has-symbols@1.0.0:
    resolution: {integrity: sha512-QfcgWpH8qn5qhNMg3wfXf2FD/rSA4TwNiDDthKqXe7v6oBW0YKWcnfwMAApgWq9Lh+Yu+fQWVhHPohlD/S6uoQ==}
    engines: {node: '>= 0.4'}
    dev: false

  /has-symbols@1.0.3:
    resolution: {integrity: sha512-l3LCuF6MgDNwTDKkdYGEihYjt5pRPbEg46rtlmnSPlUbgmB8LOIrKJbYYFBSbnPaJexMKtiPO8hmeRjRz2Td+A==}
    engines: {node: '>= 0.4'}
    dev: false

```

Note that if we manually edit `pnpm-lock.yaml` and remove this part from the importers section:
```
  packages/a:
    dependencies:
      has-symbols:
        specifier: '*'
        version: 1.0.3
```
Then call `pnpm i`, the resolution will be fixed (Supposedly because this time there is already a match in the workspace root)


# Repo2

In repo2 we have a tricker situation. Everything is already working as intended with `has-symbols@1.0.0` being present in the root dir and in the package.
However. calling `pnpm add -w has-symbols@1.0.3` will only update the version of `has-symbol` that the workspace root uses, leaving `packages/a` still using `has-symbols@1.0.0`

## Steps
`cd repo2 && pnpm add -w has-symbols@1.0.3`

## Expected
Expected `has-symbol@1.0.3` to be installed in the root workspace and then updated in `packages/a` to version `1.0.3` also

## Actual
`has-symbol@1.0.3` is installed in the root workspace but `packages/a` remains unchanged

pnpm-lock before:

```yaml
lockfileVersion: '6.0'

settings:
  autoInstallPeers: true
  excludeLinksFromLockfile: false

importers:

  .:
    dependencies:
      has-symbols:
        specifier: 1.0.0
        version: 1.0.0

  packages/a:
    dependencies:
      has-symbols:
        specifier: '*'
        version: 1.0.0

packages:

  /has-symbols@1.0.0:
    resolution: {integrity: sha512-QfcgWpH8qn5qhNMg3wfXf2FD/rSA4TwNiDDthKqXe7v6oBW0YKWcnfwMAApgWq9Lh+Yu+fQWVhHPohlD/S6uoQ==}
    engines: {node: '>= 0.4'}
    dev: false

```

pnpm-lock after:
```yaml
lockfileVersion: '6.0'

settings:
  autoInstallPeers: true
  excludeLinksFromLockfile: false

importers:

  .:
    dependencies:
      has-symbols:
        specifier: 1.0.3
        version: 1.0.3

  packages/a:
    dependencies:
      has-symbols:
        specifier: '*'
        version: 1.0.0

packages:

  /has-symbols@1.0.0:
    resolution: {integrity: sha512-QfcgWpH8qn5qhNMg3wfXf2FD/rSA4TwNiDDthKqXe7v6oBW0YKWcnfwMAApgWq9Lh+Yu+fQWVhHPohlD/S6uoQ==}
    engines: {node: '>= 0.4'}
    dev: false

  /has-symbols@1.0.3:
    resolution: {integrity: sha512-l3LCuF6MgDNwTDKkdYGEihYjt5pRPbEg46rtlmnSPlUbgmB8LOIrKJbYYFBSbnPaJexMKtiPO8hmeRjRz2Td+A==}
    engines: {node: '>= 0.4'}
    dev: false
```
