## About this fork

Ozobot forked the [pyodide](https://github.com/pyodide/pyodide) repository to allow customized builds of the [pyodide npm package](https://www.npmjs.com/package/pyodide).

The customization only happens in the CI configuration and in the pyodide's [package.json](./src/js/package.json) file.

- Pyodide's [package.json](./src/js/package.json) was updated to reference correctly the @ozobot namespace and fork.
- Additional step that automatically patches the pyodide's `package.json` to contain all the files that are part of the build was added. In the [upstream pyodide repo](https://github.com/pyodide/pyodide), the content of the `package.json` file is hardcoded and the `files` section doesn't reflect what modules were actually built. Therefore, we needed to do adjustments in our fork, see the step *Fixup package.json files to include built packages* in the main workflow file.
- Action that builds and deploys the pyodide npm package on tag push. The tag has to be prefixed `ozobot/` for the workflow to run.
- The macOS CI jobs (the `macos-15` build and the Safari tests) are temporarily disabled - they are broken upstream and we only publish the Linux build.
- The SciPy test job is allowed to fail - its remaining failures come from upstream, not from our build.

The change to pyodide upstream (`main` branch) are maintained in the `master` branch of this repo. This is the suggested flow to build updated version of `pyodide`:

1. Pull the branches and tags from upstream repo: `git fetch --tags https://github.com/pyodide/pyodide`
2. Rebase the `master` branch on the particular pyodide version to be built: `git rebase --onto 314.0.7 0.27.3 master` (replace `314.0.7` with the pyodide version you want to deploy - preferably a release version - replace `0.27.3` the version `master` branch is currently based on)
3. Resolve conflicts during rebase (if any).
4. Force-push the `master` branch (`git push --force origin master`), the CI will build the version with the Ozobot specific patches applied.
5. If the build passes OK, tag the current version, prefix it with `ozobot/`: `git tag --annotate --sign ozobot/0.25.1`
6. Push the tag to deploy the version to [Ozobot's Github npm registry](https://github.com/ozobot/pyodide/pkgs/npm/pyodide): `git push origin ozobot/0.25.1`
7. Bump the version of the `pyodide` npm package in projects that use it: `web$ npm install --save @ozobot/pyodide@latest`


<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/_static/img/pyodide-logo-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="docs/_static/img/pyodide-logo-light.svg">
  <img height="200" src="docs/_static/img/pyodide-logo-dark.svg">
</picture>
</div>

[![NPM Latest Release](https://img.shields.io/npm/v/pyodide)](https://www.npmjs.com/package/pyodide)
[![PyPI Latest Release](https://img.shields.io/pypi/v/pyodide-py.svg)](https://pypi.org/project/pyodide-py/)
[![Build Status](https://circleci.com/gh/pyodide/pyodide.png)](https://circleci.com/gh/pyodide/pyodide)
[![Documentation Status](https://readthedocs.org/projects/pyodide/badge/?version=stable)](https://pyodide.readthedocs.io/?badge=stable)

Pyodide is a Python distribution for the browser and Node.js based on WebAssembly.

## What is Pyodide?

Pyodide is a port of CPython to WebAssembly/[Emscripten](https://emscripten.org/).

Pyodide makes it possible to install and run Python packages in the browser with
[micropip](https://micropip.pyodide.org/). Any pure Python package with a wheel
available on PyPi is supported. Many packages with C, C++, and Rust extensions
have also been ported for use with Pyodide. These include many general-purpose
packages such as regex, PyYAML, and cryptography, and scientific Python packages
including NumPy, pandas, SciPy, Matplotlib, and scikit-learn.

Pyodide comes with a robust Javascript ⟺ Python foreign function interface so
that you can freely mix these two languages in your code with minimal friction.
This includes full support for error handling, async/await, and much more.

When used inside a browser, Python has full access to the Web APIs.

## Try Pyodide (no installation needed)

Try Pyodide in a
[REPL](https://pyodide.org/en/stable/console.html) directly in
your browser. For further information, see the
[documentation](https://pyodide.org/en/stable/).

## Getting Started

- If you wish to use a hosted distribution of Pyodide: see the [Getting
  Started](https://pyodide.org/en/stable/usage/quickstart.html) documentation.
- If you wish to host Pyodide yourself, you can download Pyodide from the [releases
  page](https://github.com/pyodide/pyodide/releases/) and serve it with a web server.
- If you wish to use Pyodide with a bundler, see [the documentation on Working with
  Bundlers](https://pyodide.org/en/stable/usage/working-with-bundlers.html)
- If you are a Python package maintainer, see [the documentation on building and testing Python
  packages](https://pyodide.org/en/stable/development/building-and-testing-packages.html).
- If you want to add a package to the Pyodide distribution, [see the documentation on adding
  a package to the Pyodide distribution](https://pyodide.org/en/stable/development/new-packages.html)
- If you wish to experiment or contribute back to the Pyodide runtime, see the documentation on
  [building Pyodide from source](https://pyodide.org/en/stable/development/building-from-sources.html)

## The Components of the Pyodide Project

The Pyodide project consists of the following components:
1. A build of CPython with a few patches (https://github.com/pyodide/pyodide/tree/main/cpython/)
2. A JS/Python foreign function interface
   (https://github.com/pyodide/pyodide/tree/main/src/core and
   https://github.com/pyodide/pyodide/tree/main/src/py)
3. JavaScript code for creating and managing Pyodide interpreters
   (https://github.com/pyodide/pyodide/tree/main/src/js)
4. A choice of Emscripten platform, which consists of a version + ABI-sensitive
   flags + static libraries to link. This is described
   [here](https://pyodide.org/en/stable/development/abi.html) and implemented in
   (https://github.com/pyodide/pyodide/tree/main/Makefile.envs)
5. A toolchain for [cross compiling](https://github.com/pyodide/pyodide-build)
   [testing](https://github.com/pyodide/pytest-pyodide) and
   [installing](https://github.com/pyodide/micropip) packages for Pyodide.

## History

Pyodide was created in 2018 by [Michael Droettboom](https://github.com/mdboom)
at Mozilla as part of the [Iodide
project](https://github.com/iodide-project/iodide). Iodide is an experimental
web-based notebook environment for literate scientific computing and
communication.

Iodide is no longer maintained. If you want to use Pyodide in an interactive
client-side notebook, see [Pyodide notebook
environments](https://pyodide.org/en/stable/project/related-projects.html#notebook-environments-ides-repls).

## Contributing

Please view the [contributing
guide](https://pyodide.org/en/stable/development/contributing.html) for tips
on filing issues, making changes, and submitting pull requests. Pyodide is an
independent and community-driven open-source project. The decision-making
process is outlined in the [Project
governance](https://pyodide.org/en/stable/project/governance.html).

## Communication

- Blog: [blog.pyodide.org](https://blog.pyodide.org/)
- Mailing list: [mail.python.org/mailman3/lists/pyodide.python.org/](https://mail.python.org/mailman3/lists/pyodide.python.org/)
- Twitter: [twitter.com/pyodide](https://twitter.com/pyodide)
- Stack Overflow: [stackoverflow.com/questions/tagged/pyodide](https://stackoverflow.com/questions/tagged/pyodide)
- Discord: [Pyodide Discord](https://dsc.gg/pyodide)

## Sponsors

For a full list of current and historical sponsors, please see the [Funding](https://pyodide.org/en/stable/project/about.html#funding) section of our About page.

Pyodide also has a large number of small donors. If you’re interested in supporting Pyodide, check out our [OpenCollective](https://opencollective.com/pyodide) and [GitHub Sponsors](https://github.com/sponsors/pyodide) pages.

### Special thanks

- [BrowserStack](https://www.browserstack.com/): This project is tested with BrowserStack

## License

Pyodide uses the [Mozilla Public License Version
2.0](https://choosealicense.com/licenses/mpl-2.0/).
