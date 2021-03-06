# unified-args [![Build Status][travis-badge]][travis] [![Coverage Status][codecov-badge]][codecov]

Interface for creating CLIs around [**unified**][unified] processors.
Wrapper around the [**engine**][engine] to configure it with command-line
arguments.  Should be `require`d and configured in an executable script,
on its own, as it handles the whole process.

## Installation

[npm][npm-install]:

```bash
npm install unified-args
```

## Usage

This example creates a CLI for [**remark**][remark], loading
`remark-` plugins, searching for markdown files, and loading
[configuration][config-file] and [ignore][ignore-file] files.

`cli` (executable: `chmod +x cli`):

```js
#!/usr/bin/env node
'use strict';

var start = require('unified-args');
var extensions = require('markdown-extensions');
var remark = require('remark');
var pack = require('remark/package.json');

var name = pack.name;

start({
  'processor': remark,
  'name': name,
  'description': pack.description,
  'version': pack.version,
  'pluginPrefix': name,
  'extensions': extensions,
  'packageField': name + 'Config',
  'rcName': '.' + name + 'rc',
  'ignoreName': '.' + name + 'ignore'
});
```

## Table of Contents

*   [API](#api)
    *   [start(configuration)](#startconfiguration)
*   [CLI](#cli)
    *   [--help](#--help)
    *   [--version](#--version)
    *   [--output \[path\]](#--output-path)
    *   [--rc-path &lt;path>](#--rc-path-path)
    *   [--ignore-path &lt;path>](#--ignore-path-path)
    *   [--setting &lt;settings>](#--setting-settings)
    *   [--use &lt;plugin>](#--use-plugin)
    *   [--preset &lt;presets>](#--preset-presets)
    *   [--ext &lt;extensions>](#--ext-extensions)
    *   [--watch](#--watch)
    *   [--tree](#--tree)
    *   [--tree-in](#--tree-in)
    *   [--tree-out](#--tree-out)
    *   [--quiet](#--quiet)
    *   [--silent](#--silent)
    *   [--frail](#--frail)
    *   [--file-path &lt;path>](#--file-path-path)
    *   [--stdout](#--stdout)
    *   [--color](#--color)
    *   [--config](#--config)
    *   [--ignore](#--ignore)
*   [Diagnostics](#diagnostics)
*   [Debugging](#debugging)
*   [License](#license)

## API

### `start(configuration)`

Create a CLI for a [**unified**][unified] processor.

###### `configuration`

All options are required.

*   `processor` ([`Processor`][unified-processor])
    — Processor to transform files
    (engine: [`processor`][engine-processor]).
*   `name` (`string`)
    — Name of executable.
*   `description` (`string`)
    — Description of executable.
*   `version` (`string`)
    — Version of executable.
*   `extensions` (`Array.<string>`)
    — Default file [extensions][ext] to include.
    (engine: [`extensions`][engine-extensions]).
*   `ignoreName` (`string`)
    — Name of [ignore files][ignore-file] to load.
    (engine: [`ignoreName`][engine-ignore-name]).
*   `rcName` (`string`)
    — Name of [configuration files][config-file] to load.
    (engine: [`rcName`][engine-rc-name]).
*   `packageField` (`string`)
    — Property at which [configuration][config-file] can be found in
    `package.json` files.
    (engine: [`packageField`][engine-package-field]).
*   `pluginPrefix` (`string`)
    — Prefix to use when searching for [plug-ins][use]
    (engine: [`pluginPrefix`][engine-plugin-prefix]);
*   `presetPrefix` (`string`)
    — Prefix to use when searching for presets from config files
    (engine: [`presetPrefix`][engine-preset-prefix]).

## CLI

CLIs created with **unified-args**, such as the [example][usage] above,
creates an interface similar to the below (run `cli --help` for accurate
information):

```txt
Usage: remark [options] [path | glob ...]

  Markdown processor powered by plugins

Options:

  -h  --help                output usage information
  -v  --version             output version number
  -o  --output [path]       specify output location
  -r  --rc-path <path>      specify configuration file
  -i  --ignore-path <path>  specify ignore file
  -s  --setting <settings>  specify settings
  -e  --ext <extensions>    specify extensions
  -u  --use <plugins>       use plugins
  -p  --preset <presets>    use presets
  -w  --watch               watch for changes and reprocess
  -q  --quiet               output only warnings and errors
  -S  --silent              output only errors
  -f  --frail               exit with 1 on warnings
  -t  --tree                specify input and output as syntax tree
      --file-path <path>    specify path to process as
      --tree-in             specify input as syntax tree
      --tree-out            output syntax tree
      --[no-]stdout         specify writing to stdout (on by default)
      --[no-]color          specify color in report (on by default)
      --[no-]config         search for configuration files (on by default)
      --[no-]ignore         search for ignore files (on by default)

Examples:

  # Process `input.md`
  $ remark input.md -o output.md

  # Pipe
  $ remark < input.md > output.md

  # Rewrite all applicable files
  $ remark . -o
```

All non-options are seen as input and can be:

*   Paths (`cli README.txt`) and [globs][glob] (`cli *.txt`) pointing
    to files to load.
*   Paths (`cli test`) and globs (`cli fixtures/{in,out}`) pointing to
    directories.  These are searched for files with known [extensions][ext]
    which are not ignored by patterns in [ignore files][ignore-file].
    The default behaviour is to exclude files in `node_modules`
    and hidden directories (those starting with a dot: `.`).

<!-- Options: -->

*   **Default**: none;
*   **Engine**: [`globs`][engine-globs].

### `--help`

```sh
cli --help
```

Output short usage information.

*   **Default**: off;
*   **Alias**: `-h`.

### `--version`

```sh
cli --version
```

Output version number.

*   **Default**: off;
*   **Alias**: `-v`.

### `--output [path]`

```sh
cli . --output
cli . --output doc
cli input.txt --output doc/output.text
```

Whether to write successfully processed files, and where to.  Can be
set from [configuration files][config-file].

*   If output is **not** given, files are not written to the file-system;
*   If output is given **without** `path`, input files are overwritten when
    successful;
*   If output is given with `path` and it points to an existing directory,
    files are written to that directory;
*   If output is given with `path`, the parent directory of that path
    exists, and one file is processed, the file is written to the given
    path.

<!-- Options: -->

*   **Default**: off;
*   **Alias**: `-o`;
*   **Engine**: [`output`][engine-output].

### `--rc-path <path>`

```sh
cli . --rc-path config.json
```

File-path to a JSON [configuration file][config-file] to load, regardless
of [`--config`][config].

> **Note**: Be careful not to pass a file which is also detected.
>
> **Note**: Do not pass `package.json` files or `.js` files.

*   **Default**: none;
*   **Alias**: `-r`;
*   **Engine**: [`rcPath`][engine-rc-path].

### `--ignore-path <path>`

```sh
cli . --ignore-path .gitignore
```

File-path to an [ignore file][ignore-file] to load, regardless of
[`--ignore`][ignore].

*   **Default**: none;
*   **Alias**: `-i`;
*   **Engine**: [`ignorePath`][engine-ignore-path].

### `--setting <settings>`

```sh
cli input.txt --setting alpha:true
cli input.txt --setting bravo:true --setting '"charlie": "delta"'
cli input.txt --setting echo-foxtrot:-2
cli input.txt --setting 'golf: false, hotel-india: ["juliet", 1]'
```

Configuration for the parser and compiler of the processor.  Can be set
from [configuration files][config-file].

The given settings are JSON, with two exceptions:

*   Surrounding braces must not be used: `"foo": 1, "bar": "baz"`
    is valid;
*   Keys do not need to be quoted: both `"foo": "bar"` and
    `foo: "bar"` are considered equal.

Keys are camel-cased: `foo-bar: true` and `fooBar: true` are treated
the same.

*   **Default**: none;
*   **Alias**: `-s`;
*   **Engine**: [`settings`][engine-settings].

### `--use <plugin>`

```sh
cli input.txt --use man
cli input.txt --use 'lint=alpha:"bravo"'
```

Plug-in to load by its name or path, optionally with options, and use
on every processed file.  Can be set from [configuration files][config-file].

To pass options, follow the plugin by an equals sign (`=`) and settings,
which have the same in syntax as [`--setting <settings>`][setting].

Prefers plug-ins prefixed with the [configured `pluginPrefix`][configured].

*   **Default**: none;
*   **Alias**: `-u`;
*   **Engine**: [`plugins`][engine-plugins].

### `--preset <presets>`

```sh
cli input.txt --preset lint-recommended
cli input.txt --preset 'minify=force:true'
```

Preset to load by its name or path, optionally with options.  Can be set
from [configuration files][config-file].

To pass options, follow the plugin by an equals sign (`=`) and settings,
which have the same in syntax as [`--setting <settings>`][setting].

Prefers presets prefixed with the [configured `presetPrefix`][configured].

*   **Default**: none;
*   **Alias**: `-p`;
*   **Engine**: [`presets`][engine-presets].

### `--ext <extensions>`

```sh
cli . --ext html
cli . --ext html,htm
```

Specify one or more extensions to include when searching for files.

If no extensions are given, uses the [configured `extensions`][configured].

*   **Default**: Configured [`extensions`][configured];
*   **Alias**: `-e`;
*   **Engine**: [`extensions`][engine-extensions].

### `--watch`

```sh
cli . -w
```

Process as normal, then watch found files and reprocess when they change.

The watch is stopped when `SIGINT` is received (usually done by pressing
`CTRL-C`).

If [`--output`][output] is given **without** `path` it is not honoured,
to prevent an infinite loop.  When the watch closes, a final process
runs including `--output`.

```sh
cli . -oqw
```

Yields:

```txt
Watching... (press CTRL+C to exit)
Note: Ignoring `--output` until exit.
```

*   **Default**: off;
*   **Alias**: `-w`.

### `--tree`

```sh
cli --tree < input.json > output.json
```

Treat input as a syntax tree in JSON and output the transformed syntax
tree.  This runs neither the [parsing nor the compilation
phase][description].

*   **Default**: off;
*   **Alias**: `-t`;
*   **Engine**: [`tree`][engine-tree].

### `--tree-in`

```sh
cli --tree-in < input.json > input.txt
```

Treat input as a syntax tree in JSON.  This does not run the [parsing
phase][description].

*   **Default**: off;
*   **Engine**: [`treeIn`][engine-tree-in].

### `--tree-out`

```sh
cli --tree-out < input.txt > output.json
```

Output the transformed syntax tree.  This does not run the [compilation
phase][description].

*   **Default**: off;
*   **Engine**: [`treeOut`][engine-tree-out].

### `--quiet`

```sh
cli input.txt --quiet
```

Ignore files without any messages in the report.  The default behaviour
is to show a success message.

*   **Default**: off;
*   **Alias**: `-q`;
*   **Engine**: [`quiet`][engine-quiet].

### `--silent`

```sh
cli input.txt --silent
```

Show only fatal errors in the report.  Turns [`--quiet`][quiet] on.

*   **Default**: off;
*   **Alias**: `-S`;
*   **Engine**: [`silent`][engine-silent].

### `--frail`

```sh
cli input.txt --frail
```

Exit with a status code of `1` if warnings or errors occur.  The default
behaviour is to exit with `1` on errors.

*   **Default**: off;
*   **Alias**: `-f`;
*   **Engine**: [`frail`][engine-frail].

### `--file-path <path>`

```sh
cli --file-path input.txt < input.txt > doc/output.txt
```

File path to process the given file on **stdin**(4) as, if any.

*   **Default**: none;
*   **Engine**: [`filePath`][engine-file-path].

### `--stdout`

```sh
cli input.txt --no-stdout
```

Whether to write a processed file to **stdout**(4).

*   **Default**: off if [`--output`][output] or [`--watch`][watch] are given,
    or if multiple files are processed;
*   **Engine**: [`out`][engine-out].

### `--color`

```sh
cli input.txt --no-color
```

Whether to output ANSI codes in the report.

*   **Default**: whether the terminal [supports colour][supports-color];
*   **Engine**: [`color`][engine-color].

### `--config`

```sh
cli input.txt --no-config
```

Whether to load [configuration files][config-file].

Searches for files with the [configured `rcName`][configured]
(`$rcName` and `$rcName.js`) and looks for the [configured
`packageField`][configured] in `package.json` files.

*   **Default**: on;
*   **Engine**: [`detectConfig`][engine-detect-config].

### `--ignore`

```sh
cli . --no-ignore
```

Whether to load [ignore files][ignore-file].

Searches for files with named [`ignoreName`][configured].

*   **Default**: on;
*   **Engine**: [`detectIgnore`][engine-detect-ignore].

## Diagnostics

CLIs created with **unified-args** exit with:

*   `1` on fatal errors;
*   `1` on warnings in [`--frail`][frail] mode;
*   `0` on success.

## Debugging

CLIs can be debugged by setting the [`DEBUG`][debug] environment
variable to `*`, such as `DEBUG="*" cli example.txt`.

## License

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[travis-badge]: https://img.shields.io/travis/wooorm/unified-args.svg

[travis]: https://travis-ci.org/wooorm/unified-args

[codecov-badge]: https://img.shields.io/codecov/c/github/wooorm/unified-args.svg

[codecov]: https://codecov.io/github/wooorm/unified-args

[npm-install]: https://docs.npmjs.com/cli/install

[license]: LICENSE

[author]: http://wooorm.com

[debug]: https://github.com/visionmedia/debug#debug

[glob]: https://github.com/isaacs/node-glob#glob-primer

[supports-color]: https://github.com/chalk/supports-color

[unified]: https://github.com/wooorm/unified

[engine]: https://github.com/wooorm/unified-engine

[unified-processor]: https://github.com/wooorm/unified#processor

[remark]: https://github.com/wooorm/remark

[config-file]: https://github.com/wooorm/unified-engine/blob/master/doc/configure.md

[ignore-file]: https://github.com/wooorm/unified-engine/blob/master/doc/ignore.md

[description]: https://github.com/wooorm/unified#description

[engine-processor]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsprocessor

[engine-globs]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsglobs

[engine-output]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsoutput

[engine-rc-path]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsrcpath

[engine-rc-name]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsrcname

[engine-package-field]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionspackagefield

[engine-ignore-name]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsignorename

[engine-ignore-path]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsignorepath

[engine-settings]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsettings

[engine-plugins]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsplugins

[engine-presets]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionspresets

[engine-plugin-prefix]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionspluginprefix

[engine-preset-prefix]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionspresetprefix

[engine-extensions]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsextensions

[engine-tree]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionstree

[engine-tree-in]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionstreein

[engine-tree-out]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionstreeout

[engine-quiet]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsquiet

[engine-silent]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsilent

[engine-frail]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsfrail

[engine-file-path]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsfilepath

[engine-out]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsout

[engine-color]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionscolor

[engine-detect-config]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsdetectconfig

[engine-detect-ignore]: https://github.com/wooorm/unified-engine/blob/master/doc/options.md#optionsdetectignore

[configured]: #startconfiguration

[usage]: #usage

[watch]: #--watch

[output]: #--output-path

[config]: #--config

[ext]: #--ext-extensions

[use]: #--use-plugin

[ignore]: #--ignore

[setting]: #--setting-settings

[quiet]: #--quiet

[frail]: #--frail
