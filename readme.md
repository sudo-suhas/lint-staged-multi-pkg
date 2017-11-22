# lint-staged-multi-pkg

Example repo to demostrate use of [`lint-staged`][lint-staged] in multi
package project with [`lerna`][lerna] and [`husky`][husky].

### pre-commit hook

`husky` is installed in the root `package.json` as recommended in
[husky docs][husky-docs]. Please note that the documentation is for a beta
version of husky but can be applied to the stable one as well.

The pre-commit hook is configured with the script
_**`lerna run --concurrency 1 --stream precommit`**_. This executes the
`precommit` script for each package(if it exists). Furthermore, concurrent
execution is disabled because it can cause problems during `git add`(see
[okonet/lint-staged#225][lint-staged-issue-225]).

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

### Output

Example output from the git hook execution:

```
λ git commit
husky > npm run -s precommit (node v8.9.1)

lerna info version 2.5.1
say-bye: > say-bye@1.0.0 precommit E:\Projects\experiments\lint-staged-multi-pkg\packages\say-bye
say-bye: > lint-staged
say-bye: Running tasks for lib/*.js [started]
say-bye: eslint --fix [started]
say-bye: eslint --fix [completed]
say-bye: git add [started]
say-bye: git add [completed]
say-bye: Running tasks for lib/*.js [completed]
say-bye:
say-hi: > say-hi@1.0.0 precommit E:\Projects\experiments\lint-staged-multi-pkg\packages\say-hi
say-hi: > lint-staged
say-hi: Running tasks for src/*.js [started]
say-hi: eslint --fix [started]
say-hi: eslint --fix [completed]
say-hi: git add [started]
say-hi: git add [completed]
say-hi: Running tasks for src/*.js [completed]
say-hi:
lerna success run Ran npm script 'precommit' in packages:
lerna success - say-bye
lerna success - say-hi
Done in 6.75s.
```

## License

MIT © [Suhas Karanth][sudo-suhas]

[lint-staged]: https://github.com/okonet/lint-staged
[lerna]: https://github.com/lerna/lerna
[husky]: https://github.com/typicode/husky
[sudo-suhas]: https://github.com/sudo-suhas
[husky-docs]: https://github.com/typicode/husky/blob/dev/docs.md#multi-package-repository-monorepo
[lint-staged-issue-225]: https://github.com/okonet/lint-staged/issues/225
