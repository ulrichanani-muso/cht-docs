---
title: "Updating Dependencies"
linkTitle: "Dependencies"
weight: 8
description: >
  Process for updating dependencies
---

Every minor release we update dependencies to get the latest fixes and improvements. We do this early in the release cycle so that we have some more time to find regressions and issues. This is done on all folders with a package.json, including:

- cht-core
  - / (root)
  - /admin
  - /api
  - /sentinel
  - /shared-libs/*
  - /webapp
- cht-conf

## Steps

1. Checkout and pull the latest default branch - get the latest code
2. `git checkout -b "<issue>-update-dendencies"` - make a branch

Then for each folder go through these steps.

1. `npm ci` - update your local node_modules to match expected
2. `npm outdated` - report on any dependencies which aren't at the latest
3. `npm install --save[-dev] package@version` - install the latest version (be careful and read the release notes if the new version is a major change from the current)
4. `npm dedupe` - remove duplicated dependencies
5. `npm audit fix` - automatically fix any nested dependencies with vulnerabilities
6. `npm audit` - get a report on any remaining vulnerabilities and manually scan it to see if there's anything else you can do

## Problems

- Don't update `bootstrap` to 4+ as it has many breaking changes. One day we will either raise an issue to upgrade it or migrate off it, but that is outside the scope of this change.
- Don't update `bootstrap-daterangepicker`.
- Don't update `select2` as the latest patch always seems to fail.
- Don't update `jquery` to 3.6.0+ as the `select2` search input looses focus on click event, this is an [open issue](https://github.com/select2/select2/issues/5993) in their repository.
- CHT-Core's webapp is using Enketo and jQuery library, at the same time Enketo internally uses a specific version of jQuery. Make sure webapp installs the same jQuery version than the one Enketo uses internally: `3.2.x`.

  Do this by checking the jquery entry in `./webapp/package.json` matches `./webapp/node_modules/enketo-core/package.json`:

  ```
  grep '"jquery"' ./webapp/package.json
  "jquery": "3.2.x",
  grep '"jquery"' ./webapp/node_modules/enketo-core/package.json
  "jquery": "3.2.x",
  ```

- Make sure the version of `api/enketo-xslt` is the same as `webapp/enketo-core/enketo-transformer/enketo-xslt`.
- Don't update `lodash/core` since 4.17.21 is failing, this is an [open issue](https://github.com/lodash/lodash/issues/4904) in their repository.
- Don't update `helmet` since it is not supporting NodeJS 8 anymore. Its minimum is NodeJS 10+.
- If you have trouble upgrading any other dependency and you think it'll be challenging to fix it then raise a new issue to upgrade just that dependency. Don't hold up all the other upgrades you've made.

## Troubleshooting

### npm errno -17

If `npm ci` errors with "errno -17" in shared-libs you may need to manually remove the nested dependencies from the package-lock.json. This needs move investigation to work out why this is happening.

### select2 is not a function

If you get `TypeError: "$(...).select2 is not a function"` then either:
1. You bumped select2. For some reason this breaks it.
2. You have multiple jquery libraries and select2 is getting attached to one but not the other. Make sure the jquery versions in enketo-core and webapp match and you've `run dedupe` to remove the enketo-core copy.
