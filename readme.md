# ProgArg
[[العربية]](readme.ar.md)

## Overview

The **ProgArg** library provides a flexible and powerful solution for defining and parsing
command-line arguments and commands in Alusus-based applications. It supports hierarchical
command structures, options, and arguments, as well as built-in localization.

---

## Features

- **Command Definition**: Define commands with keywords, descriptions, options, and arguments.
- **Subcommands**: Support for hierarchical command structures.
- **Localization**: Built-in internationalization support using `Alusus/I18n`.
- **Help Display**: Automatically generates help messages.
- **Argument and Option Parsing**: Parses arguments and options.
- **Error Handling**: Handles errors related to incorrect or missing arguments.

---

## Adding to a Project

```
import "Apm";
Apm.importFile("Alusus/ProgArg");
```

Once the library is added to the project, its definitions will be available under the `ProgArg` module.

---

## Classes

### `CmdDef`

Defines a command and its associated metadata.

```
class CmdDef {
    def kwd: String;
    def description: String;
    def options: Map[String, String];
    def args: Map[String, String];
    def numRequiredArgs: Int = -1;
    def subCmds: Array[SrdRef[CmdDef]];
    def callback: closure(options: Map[String, String], args: Array[String]);
    def startCallback: closure(options: Map[String, String], args: Array[String]);
    def endCallback: closure(options: Map[String, String], args: Array[String]);
    
    handler this_type(): SrdRef[CmdDef];
}
```

#### Properties

- `kwd`: The keyword that identifies the command.
- `description`: A brief description of the command.
- `options`: A list of option names and their descriptions.
- `args`: A list of argument names and their descriptions.
- `numRequiredArgs`: The number of required arguments. If set to `-1` (default), all arguments
  are considered required.
- `subCmds`: An array of subcommands defined under this command.
- `callback`: A closure function invoked when the command is executed, assuming it has no subcommands.
- `startCallback`: A closure function invoked before processing subcommands.
- `endCallback`: A closure function invoked after processing subcommands.

---

## Global Variables

### `cmdDef`

A global variable that holds the main command definition from which parsing starts. The keyword for
this command is ignored, and its argument, option, and subcommand definitions are used instead.

---

## Functions

### `initialize`

```
function initialize();
function initialize(lang: CharsPtr);
```

Initializes the library and loads localized translations. The first version of this function uses
the system's current language. This function must be called at the beginning of the program before
using other functions.

### `printHelp`

```
function printHelp(argStartIndex: Int, localizeCommands: Bool);
function printHelp(cmdDef: SrdRef[CmdDef], tabs: Int, localizeCommands: Bool);
```

Prints the help message for a command and its subcommands. The first version infers the command to
explain from program inputs, while the second version accepts an explicit command definition.

**Parameters:**

- `argStartIndex`: The index of the first argument in the program's input (excluding the program name).
  Typically `1` for precompiled programs, but `2` for interpreted execution since in the case of
  interpretation the first argument in `Process.args` will be `alusus`.
- `cmdDef`: A reference to the command definition to explain.
- `tabs`: The number of leading spaces for indentation.
- `localizeCommands`: If `true`, keywords, argument names, and option names will be localized in the help output.

### `parse`

```
function parse(argStartIndex: Int);
function parse(cmdDef: SrdRef[CmdDef], argStartIndex: Int): Int;
```

Parses command-line arguments starting from the specified index and invokes the appropriate processing functions. The
first version uses the main command definition (`ProgArg.cmdDef`), while the second version accepts an explicit command
definition.

**Parameters:**

- `argStartIndex`: The index of the first argument in the program's input (excluding the program name). Typically `1`
  for precompiled programs, but `2` for interpreted execution since in the case of interpretation the first argument in
  `Process.args` will be `alusus`.
- `cmdDef`: The command definition to start parsing from. The first version defaults to the main command definition.

**Return Value:**

The second version returns the argument index at which parsing stopped. The first version assumes all arguments will be
consumed and exists the program with an error if any unknown arguments remain, i.e if the return value isn't at the end
of the `Process.args` array.

---

## Example

```
import "Apm";
Apm.importFile("Alusus/ProgArg");

ProgArg.initialize();
ProgArg.cmdDef = ProgArg.CmdDef().{
    args = Map[String, String]()
        .set(String("arg1"), String("This is description for argument 1."))
        .set(String("arg2"), String("This is description for argument 2."));
    options = Map[String, String]()
        .set(String("opt1"), String("This is description for option 1."))
        .set(String("opt2"), String("This is description for option 2."));
    callback = closure (options: Map[String, String], args: Array[String]) {
        // Process command options and arguments here.
    };
}
ProgArg.parse(2);
```

---

## Error Handling

- Error messages are printed to `stderr`, and a non-zero exit code is returned if incorrect arguments are passed or
  required arguments are missing.

---

## Localization

All texts can be localized using the `I18n` module. The `localizeCommands` parameter controls whether keywords, option
names, and argument names are localized when generating help messages with functions like `printHelp` and
`printCmdHelp`.

