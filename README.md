# `pkg:` for package root do not resolve import-only files

This is a minimal reproduction that `pkg:` URLs do not resolve import-only files
when used with `@import` and the imported package has `exports` defined pointing to specific files.

The repository uses npm workspaces to simulate two external packages.
- The `library` package provides:
    - a module as `dist/index.scss`
    - a corresponding import-only file as `dist/index.import.scss`
    - an explicit `exports` in its `package.json` asking Sass to resolve `pkg:library` as the package's `dist/index.scss` file.

The `no-exports` package also provides a module and corresponding import only file, except it doesn't have an `exports` field in its `package.json`.

After installing the libraries with `npm install`, use `npx sass --pkg-importer=node index.scss` to compile the `index.scss` file that `@import` both libraries (with `npx sass):
- the CSS for the `library` package comes from the `index.scss` file
- the CSS for the `no-exports` package comes from teh `index.import.scss` file

This is unexpected as both files are loaded using `@import` and a corresponding import-only file is present.

This seems due to the NodePackageImporter only [accounting for import-only files when there's no `exports` field](https://github.com/sass/dart-sass/blob/5fd18c75e31a855476059fb6fb0c6aa829292739/lib/src/importer/node_package.dart#L141-L155). Is it intentional?