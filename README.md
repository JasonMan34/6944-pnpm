# Repo1

In repo1, `packages/a/package.json` specifies the peerDependency: `has-symbols: '*'`
However, at the moment, that package is not installed in the root directory or mentioned in the root `package.json`.
After calling `pnpm add -w has-symbols@1.0.0`, we will have `has-symbols@1.0.3` as the version for `packages/a` and 

## Steps
`cd repo1 && pnpm add -w has-symbols@1.0.0`

## Expected
Expected the `has-symbol@1.0.0` package will be installed in the root workspace, then `packages/a` will resolve `has-symbols` to the version found in the root workspace

## Actual
`packages/a` will receive a `has-symbols@1.0.3`, which is what `*` matches (now that default resolution is highest match). Only afterwards will `has-symbol@1.0.0` be installed in the root workspace, having no effect on `packages/a`

Note that if we manually edit `pnpm-lock.yaml` and remove this part from the importers section:
```
  packages/a:
    dependencies:
      has-symbols:
        specifier: '*'
        version: 1.0.3
```
Then call `pnpm i`, the resolution will be fixed (Because this time there is already a match in the workspace root)


# Repo2

In repo2 we have a tricker situation. Everything is already working as intended with `has-symbols@1.0.0` being present in the root dir and in the package.
However. calling `pnpm add -w has-symbols@1.0.3` will only update the version of `has-symbol` that the workspace root uses, leaving `packages/a` still using `has-symbols@1.0.0`

## Steps
`cd repo2 && pnpm add -w has-symbols@1.0.3`

## Expected
Expected `has-symbol@1.0.3` to be installed in the root workspace and then updated in `packages/a` to version `1.0.3` also

## Actual
`has-symbol@1.0.3` is installed in the root workspace but `packages/a` remains unchanged