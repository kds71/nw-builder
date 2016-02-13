# nw-builder
Builder for node-webkit projects written in node.

## Usage:

    node /path/to/nw-builder ACTIONS [OPTIONS]

    ACTION = make|run|make run

    OPTIONS = OPTION[ OPTION]*

    OPTION = 
      -v|--version VERSION_CHANGE|VERSION
      -s|--silent
      -l|--log-file /path/to/logfile

    VERSION = [0-9]+.[0-9]+.[0-9]+
    VERSION_CHANGE = [major|minor|build|none] /* default = none */

## Building project - make

Create a directory, enter it and initialize project by running:

    node /path/to/nw-builder make

This command will create directory tree and configuration files for a new
project. Edit file config.json:

    {
        "path": {
            "source": "",
            "target": "",
            "node-webkit": ""
        },
        "exclude": "",
        "data": "",
        "application": {
            "name": "",
            "version": "0.0.0"
        }
    }

Enter name of the application. Name has to be a valid filename in targeted OS.

Set path to:

1. source - directory with your node-webkit project. Source code of a project
should be stored in this directory. It is recommended to use path relative to
the directory with config.json.

2. target - directory to put built application.

3. node-webkit - directory with node-webkit binaries. It is recommended to
use an absolute path.

For this directory tree:

    + project
      + data
      + src
      + bin
      - config.json
      - run.json

path values should be set as follow:

    "source": "src"
    "target": "bin"

## Versioning

Value of "version" property should not be changed manually.
It is better to change version with `--version` option for `make`. Value of
this option should be `major`, `minor`, `build` or `none`. If `--value`
option is omitted, it will be assigned a default value (`none`). For example

    node /path/to/nw-builder make -v minor

will change v1.2.4 to v1.3.0.

## Accessing config in application

Contents of `application` hash from `config.json can be accessed by:

    var config = require('nw-builder').config;

so you can use it as a storage for additional application parameters.
Another accessible properties from `config.json` are extracted by:

    var version = require('nw-builder').version,
        name = require('nw-builder').name;

## Running application

Created application will be stored in `%TARGET%/%VERSION%` subdirectory.

You can run latest version with:

    node /path/to/nw-builder run

If you want to run earlier version use `--version` option for run. For example

    node /path/to/nw-builder run -v 1.2.4

will run v1.2.4.

## make run

    node /path/to/nw-builder make run

will build application using version from config.json without changing it and
run it. Note that `--version` option will be used by `make`, so calling:
   
    node /path/to/nw-builder make run -v 1.2.4

is not valid. Using `--version` variant for `make` is allowed:

    node /path/to/nw-builder make -v minor

will change v1.2.4 to v1.3.0 and run v1.3.0.

## Execution script

There is another file `run.json` created by first `make`:

    {
        "sequence": [
            {
                "name": "pack",
                "cmd": [
                    { "type": "cd", "target": "%SOURCE%" },
                    { "type": "shell", "cmd": "7z", "arg": [ "a", "%TARGET%/app.zip", "*", "-r", %EXCLUDE%" ]},
                    { "type": "cp", "source": "%DATA%", "target": "%TARGET%", "flag": "not-empty" }
                    { "type": "cd", "target": "%TARGET%" },
                    { "type": "cp", "source": "%NWJS%", "target": "%TARGET%" },
                    { "type": "merge", "source": "%NWJSBIN% app.zip", "target": "%APPNAME%", "remove-source": true },
                ]
            }
        ]
    }

Note that `shell` command is system specific. Generated example is for Windows
with 7z installed.

You can add other sequence blocks before and/or after default pack. It is not
recommended to modify `pack` sequence, unless it requires small adjustements
for different OS or configuration.

### %EXCLUDE%

Exclude property specifies files that should not be included in node-webkit
binary. Example exclude string for Windows with 7z:

    -xr!*.swp -xr!%.gitignore

You can replace it in config.json hash:

    "exclude": "new exclude string"

If you don't want to exclude anything, leave this string empty.

Note that value of `exclude` must be valid for your OS or configuration.

### %DATA%

    "data": "list of directories"

The %DATA% property stores a name of additional directory (or directories) 
that will not be included in application package, but copied to target
directory separately. These directories may be used to store data that has
to be loaded or saved in runtime. You can specify list of directories
using `data` property in `config.json`. Names of directories in the list
are separated by a space. If name of a directory contains space character,
escape it with `\`.

Data directories should not be subdirectories of directory with source code.
However, if you have to put them inside source code directory, make sure to
add them to `exclude` property.

### Available commands

Files/directories in lists must be separated by a space. If
any filename contains space character, escape it with `\`.

1. `cp` - copy list of files/directories (including subdirectories)

    { "type": "cp", "source": "list of files", "target": "target directory" }

2. `mv` - move list of files/directories (including subdirectories)

    { "type": "mv", "source": "list of files", "target": "target directory" }

3. `cd` - changes working directory

    { "type": "cd", "target": "target directory" }

4. `md` - creates a directory

    { "type": "md", "target": "target directory" }

4. `rm` - removes all files/directories specified in the list

    { "type": "rm", "target": "list of files" }

5. `merge` - combines list of files into one file.

    { "type": "merge", "source": "list of files", "target": "target file" }

If `merge` is used with `remove-source` flag, all files specified in the list
will be removed after merging.

6. `shell` - executes specified command using current shell

    { "type": "shell", "cmd": "command to execute", "arg": [arguments] }

### Available %words%

1. %EXCLUDE% - explained above;
2. %DATA% - explained above;
3. %SOURCE% - path to the directory with project source code, value is
taken from `config.json`;
4. %TARGET% - path to the directory with built application, value is generated
by joining value from `config.json` with version number being built;
5. %NWJS% - path to node-webkit binaries directory, value is taken from
`config.json`;
6. %NWJSBIN% - name of node-webkit executable, value is generated
automatically;
7. %APPNAME% - application executable name, value is generated by adjusting
value of `name` property from `config.json` for current OS.

## nw-builder module

You can access `nw-builder` module in your application:

    var Builder = require('nw-builder');

or by accessing its methods and properties directly:

    var config = require('nw-builder').config;

### Properties

1. `config` (JSON) - contains content of `application` hash from `config.json`;
2. `version` (STRING) - contains content of `version` property from
`config.json';
3. `name` (STRING) - contains content of `name` property from `config.json';
4. `appname` (STRING) - name of application executable file;
5. `cwd` (STRING) - path to directory containing application.

### Methods

