# lint-staged-multi-pkg

Example repo to demostrate use of [`lint-staged`][lint-staged] in multi
package project with [`lerna`][lerna] and [`husky`][husky].

### pre-commit hook

`husky` is installed in the root `package.json` as recommended in
[husky docs][husky-docs].

The pre-commit hook is configured with the script
_**`lerna run --concurrency 1 --stream precommit --since HEAD`**_. This
executes the `precommit` script for each package (if it exists). Execution is
limited to only those packages with modified files, however the `--since HEAD`
option does not consider whether the file is staged. To further limit
`precommit` to only staged files please look at the discussion in
[sudo-suhas/lint-staged-multi-pkg#4][lint-staged-multi-pkg-issues-4].
Furthermore, concurrent execution is disabled because it can cause problems
during `git add` (see [okonet/lint-staged#225][lint-staged-issue-225]).

### `lint-staged` configuration

Starting with v5.0, `lint-staged` automatically resolves the git root
**without any** additional configuration. You configure `lint-staged` as you
normally would, if your project root and git root were the same directory.

Project structure:

```
.
|-- .git
|-- .gitattributes
|-- .gitignore
|-- lerna.json
|-- license
|-- package.json
|-- packages
|   |-- say-bye
|   |   |-- .eslintrc.yml
|   |   |-- .lintstagedrc.yml
|   |   |-- lib
|   |   |   `-- index.js
|   |   |-- package.json
|   |   `-- yarn.lock
|   `-- say-hi
|       |-- .eslintrc.yml
|       |-- .lintstagedrc.yml
|       |-- package.json
|       |-- src
|       |   `-- index.js
|       `-- yarn.lock
|-- readme.md
`-- yarn.lock
```

`lint-staged` config for package `say-hi`,
[`.lintstagedrc.yml`](packages/say-hi/.lintstagedrc.yml):

```yml
linters:
  src/*.js:
    - eslint --fix
    - git add
```

Note: `lint-staged` has not been upgraded to v8 as it requires Node >= 8.6.

### Output

Example output from the git hook execution:

```
$ git commit
husky > pre-commit (node v11.9.0)
lerna notice cli v3.13.1
lerna info Executing command in 2 packages: "yarn run precommit"
say-bye: yarn run v1.13.0
say-bye: $ lint-staged
say-bye: Running tasks for lib/*.js [started]
say-bye: eslint --fix [started]
say-bye: eslint --fix [completed]
say-bye: git add [started]
say-bye: git add [completed]
say-bye: Running tasks for lib/*.js [completed]
say-bye: Done in 1.05s.
say-hi: yarn run v1.13.0
say-hi: $ lint-staged
say-hi: Running tasks for src/*.js [started]
say-hi: eslint --fix [started]
say-hi: eslint --fix [completed]
say-hi: git add [started]
say-hi: git add [completed]
say-hi: Running tasks for src/*.js [completed]
say-hi: Done in 1.03s.
lerna success run Ran npm script 'precommit' in 2 packages in 2.5s:
lerna success - say-bye
lerna success - say-hi
```

### What if I don't want to or can't use `lerna`?

You might not want to use `lerna` either because you think it is overkill or if
you have a different monorepo setup than the one required by `lerna`. You can
still use `lint-staged`, though not as easily:

- [Install `lint-staged` at the package root](#install-lint-staged-at-the-package-root)
- [Manually call `precommit` in each package](#manually-call-precommit-in-each-package)

#### Install `lint-staged` at the package root

Install `lint-staged` and `husky` at the project root and configure all linters
at the root:

Project structure:

```
.
|-- .git
|-- .gitignore
|-- .lintstagedrc.yml 🢀 Our `lint-staged` config at the monorepo root.
|-- package.json
|-- prj-1
|   |-- .eslintrc.yml
|   |-- .gitignore
|   |-- package.json
|   |-- src
|   |   `-- index.js
|   `-- yarn.lock
`-- prj-2
    |-- .eslintrc.yml
    |-- .gitignore
    |-- package.json
    |-- src
    |   `-- index.js
    `-- yarn.lock
```

`.lintstagedrc.yml`:

```yml
linters:
  prj-1/*.js:
    - eslint --fix
    - some-other-cmd
    - git add
  prj-2/**/*.js:
    - eslint --fix
    - git add
```

_Note: All linters would have to be installed in the root package._

#### Manually call `precommit` in each package

Install `husky` at the project root and setup the pre-commit hook to execute
the `precommit` script in each package. [`npm-run-all`][npm-run-all] could also
be used to make things a bit easier:

```js
// package.json
{
  "...": "...",
  "scripts": {
    "precommit:prj-1": "cd prj-1 && npm run precommit",
    "precommit:prj-2": "cd prj-2 && npm run precommit",
    "precommit": "npm-run-all precommit:*"
  },
}
```

Project structure:

```
.
|-- .gitignore
|-- package.json
|-- prj-1
|   |-- .eslintrc.yml
|   |-- .gitignore
|   |-- .lintstagedrc.yml 🢀 Each project has it's own config.
|   |-- package.json
|   |-- src
|   |   `-- index.js
|   `-- yarn.lock
`-- prj-2
    |-- .eslintrc.yml
    |-- .gitignore
    |-- .lintstagedrc.yml
    |-- package.json
    |-- src
    |   `-- index.js
    `-- yarn.lock
```

## License

MIT © [Suhas Karanth][sudo-suhas]

[lint-staged]: https://github.com/okonet/lint-staged
[lerna]: https://github.com/lerna/lerna
[husky]: https://github.com/typicode/husky
[sudo-suhas]: https://github.com/sudo-suhas
[husky-docs]: https://github.com/typicode/husky/blob/v1.3.1/DOCS.md#multi-package-repository-monorepo
[lint-staged-issue-225]: https://github.com/okonet/lint-staged/issues/225
[lint-staged-multi-pkg-issues-4]: https://github.com/sudo-suhas/lint-staged-multi-pkg/issues/4
[npm-run-all]: https://github.com/mysticatea/npm-run-all
