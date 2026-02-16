## Universal Squirrel Project Template

A Squirrel project template for structured scripting.

## Requirements

- Python 3 (Optional, for using build script)

## File Structure

- `vkzlib.config.nut`: configuration file of vkzlib, also serve as project config
- `lib/`: Libraries
    - `vkzlib/`: My library, provides essentials for the project
        - `build.bat` / `build.sh`: Build vkzlib
        - `build.py`: Python script used to build
        - `setup.build.json`: Build configuration for module setup scripts
        - `vkzlib.build.json`: Build configuration for vkzlib
        - `module/`: Scripts related to module system
        - `setup/`: Module setup scripts
- `src/`: Project source codes

## File Naming Convention

- `.in.nut`: Script that requires a build to become a `.nut` with same name
- `.entry.nut`: Script designed to be run directly
- `.test.nut`: Script contains tests, can be run directly
- `.setup.nut`: Setup script that will be prepend to other scripts

## vkzlib Configuration

```squirrel
/**
 * @desc Configuration
 *
 * You can modify this variable in `vkzlib.config.nut`
 * e.g. ::VKZLIB_CONFIG.DEBUG = true
 */
::VKZLIB_CONFIG <- {

    /**
     * @desc Path to library storage
     * @type {string}
     */
    LIB_DIR = "lib/",

    /**
     * @desc Path to vkzlib
     *
     * Defaults to `LIB_DIR + "vkzlib/"`
     */
    VKZLIB_DIR = null,

    /**
     * @desc Main project directory
     * @type {string}
     *
     * Used for setting default `require` path
     */
    PROJECT_DIR = "",

    /**
     * @desc Debug flag
     * @type {bool}
     *
     * Enable this should print additional messages for debugging
     */
    DEBUG = false,

    /**
     * @desc A function runs after setup script executed
     * @type {() => void}
     */
    ON_LOADED = null,

}
```

## Build script

In this project template, only `.in.nut` in `vkzlib` requires a build. However, the products of build is already shipped with it. Hence **you are not required to build any file**. This section is kept here just for reference.

Run `build.py` with no argument

```shell
python lib/vkzlib/build.py
```

will print help message

```txt
Usage: python build.py <config_path>

Configuration is a json file with following properties:

    root: string
    Path to the root directory for scanning, e.g. ".", "src/"

    setup: string
    Path to setup script, e.g. "setup/minimal.setup.nut", "../lib/vkzlib/setup/full.setup.nut"

    exclude?: string[]
    A list of path to exclude from scanning, e.g. ["setup/", "src/exclude.in.nut"]

    target_suffix?: string
    suffix of input files to be processed
    Defaults to ".in.nut"

    output_suffix?: string
    suffix of output files
    Defaults to ".nut"
    
! All path are relative to the directory of build configuration file
```

## Guide

### Using Module System

I've written a simple module system similar to `Lua`'s. You can utilize it to organize your code.

Module is basically a table with the item exported. For example, if you want to export something, say, `add` in a file, you can use `export`:

```squirrel
// src/math.nut
dofile("lib/vkzlib/setup/full.setup.nut")

local function add(a, b) {
    return a + b
}

// Recommended way to export a module
return export({
    // Export local function `add`
    add = add,
    // Export same `add` with the name `add2`
    add2 = add,
})
```

For pure squirrel, you can just return a table. But, it won't work if you are writing VScript. In contrast, `export` works for both.

Import the module with `require` so that you can use the items it exported:

```squirrel
// src/main.entry.nut

// Import from project
local math = require("math.nut")

print("2 + 3 = " + math.add(2, 3)) // 5
print("4 + 2 = " + math.add2(4, 2)) // 6

// Import from vkzlib
local inspect = require.vkzlib("utils/inspect.nut").inspect

print(inspect({ x = 114, y = 5.14 }))
/*
 * {
 *    "x": 114,
 *    "y": 5.14,
 * }
 */
```

### Configure vkzlib

In order to make `require` relative to your project directory, create a file named `vkzlib.config.nut` in root with `PROJECT_DIR` set. For example:

```squirrel
// The trailing `/` is necessary
::VKZLIB_CONFIG.PROJECT_DIR = "src/"
```

For other properties, see [vkzlib Configuration](#vkzlib-configuration).

### Run with Squirrel Interpreter

Get a copy of Squirrel interpreter `sq`, make sure it is on `PATH`.

Open a shell and `cd` to this (root) directory.

You can run files with suffix `.entry.nut`.

To run `src/main.entry.nut`:

```shell
sq src/main.entry.nut
```

To run tests, for example, `src/plant/test/testPlant.entry.nut`:

```shell
sq src/plant/test/testPlant.entry.nut
```
