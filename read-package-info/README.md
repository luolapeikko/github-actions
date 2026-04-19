# read-package-info

- Common package.json value parsing as env's
- package.json scripts as `HAVE_{KEY}_SCRIPT` (all uppercase)
- Current package's NPM version from NPM registry as env.
- Package manager based on lock files.

## Generated envs

```
env.PACKAGE_NAME (from package.json .name)
env.PACKAGE_VERSION (from package.json .version)
env.PACKAGE_DESCRIPTION (from package.json .description)

env.HAVE_{KEY}_SCRIPT "true" (from package.json .scripts keys[])

env.NPM_VERSION (from NPM registry)

env.PACKAGE_MANAGER (from based on lock files)
```

## Example usage

```yaml
# env defaults
env:
  PACKAGE_NAME: "unknown"
  PACKAGE_VERSION: "unknown"
  PACKAGE_MANAGER: "npm"
  NPM_VERSION: "unknown"
  HAVE_COVERAGE_SCRIPT: false
  HAVE_VALIDATE_SCRIPT: false
  HAVE_LINT_SCRIPT: false
  HAVE_UNIT_TEST_SCRIPT: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      - uses: luolapeikko/github-actions/read-package-info@v0

      - uses: pnpm/action-setup@9b5745cdf0a2e8c2620f0746130f809adb911c19 # v4
        if: env.PACKAGE_MANAGER == 'pnpm'

      - uses: actions/setup-node@v6
        with:
          node-version: "lts/*"
          registry-url: "https://registry.npmjs.org"

      - name: Install dependencies
        run: ${{ env.PACKAGE_MANAGER }} install

      - name: Run unit tests
        if: env.HAVE_UNIT_TEST_SCRIPT == 'true'
        run: ${{ env.PACKAGE_MANAGER }} test

      - name: Run validate
        if: env.HAVE_VALIDATE_SCRIPT == 'true'
        run: ${{ env.PACKAGE_MANAGER }} run validate

      - name: Run linter checks
        if: env.HAVE_LINT_SCRIPT == 'true'
        run: ${{ env.PACKAGE_MANAGER }} run lint

      - name: Run coverage
        if: env.HAVE_COVERAGE_SCRIPT == 'true'
        run: ${{ env.PACKAGE_MANAGER }} run coverage
```
