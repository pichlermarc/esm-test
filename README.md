# Reproducer

### Summary: 

`mocha` does not run tests when used with `import-in-the-middle` as `require.main === module` evaluates to `false`.

### Description: 

When running mocha from `./node_modules/bin/mocha`, [it spawns a new subprocess new `node` process](https://github.com/mochajs/mocha/blob/37deed262d4bc0788d32c66636495d10038ad398/bin/mocha.js#L105C1-L105C1), which runs [`./node_modules/mocha/lib/cli/cli.js`](https://github.com/mochajs/mocha/blob/37deed262d4bc0788d32c66636495d10038ad398/lib/cli/cli.js). When spawning the process it applys loaders as defined in `.mocharc.json` under the `node-option` key. When specifying the `import-in-the-middle` loader, the resulting command then looks like: `node --loader import-in-the-middle/hook.mjs node_modules/mocha/lib/cli/cli.js`.

However, [`./node_modules/mocha/lib/cli/cli.js` checks if `require.main === module`](https://github.com/mochajs/mocha/blob/37deed262d4bc0788d32c66636495d10038ad398/lib/cli/cli.js#L87-L89), which is a common way to test if a (commonjs) module is being loaded from the command line. However, when using `--loader import-in-the-middle/hook.mjs` this evaluates to `false`, and the process exits with `O`, even though none of the tests did run.

The example here includes a single test file `./tests/simple.test.mjs`, which should always fail:

- `npm install`
- `npm run test` 
  - this script does not use `import-in-the-middle`, test fails as expected,
- `npm run test:iitm` 
  - this script uses `import-in-the-middle/hook.mjs` as a loader, and does not fail

## Details:

- OS: Ubuntu 22.04
- Node.js v18.18.1
- other versions, see `package.json`