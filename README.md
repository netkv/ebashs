This is Ebashs, one component of the Bash/Bash operating system.
… or even better an attempt to clone GNU Emacs in bash.

Note that the readme, may be currently outdated as I am changing the core functioning of Ebashs.

Versions of Ebashs ending in WIP-x are and will be broken, you can check version via M-x about or by reading the second line of Ebashs script.

# EBASHS

## DESCRIPTION

An attempt to clone GNU Emacs but in bash.

## FEATURES

syntax highlighting
custom keybindings
custom modes -- so you can implement the evil too
file picker
mouse support

## CONFIG

Ebashs is configured via variables defined at start, you can separate it into file and then source it.

## OPTIONS ARRAY


|       NAME        |      DEFAULT      |              DESCRIPTION              |
|-------------------|-------------------|---------------------------------------|
|mouse              | 0                 |enable mouse at launch                 |
|todonote           | 1                 |highlight 'TODO:' & 'NOTE:'            |
|menuline           | 1                 |display menuline                       |
|tabchar            |`|   `             |what should tab display as             |
|file_prompt        |'Path: '           |file setting prompt                    |
|cmd_prompt         |'M-x '             |command line prompt                    |
|cancelhex          |'18 0' (C-x)       |keybinding to exit debug input menu    |
|default_mode       | edit              |what mode should be set on launch      |
|help_message       |'F10 to open menu' |top right help message                 |
|dired_message      |'Pick a file'      |same as above but for dired buffer     |


## KEYBINDING ARRAYS

Keybinding arrays match with modes. The array has to be associative and nammed `keys_<mode>`. `keys_def` is reserved and used as reference for other arrays, it's also reversed compared to other keybinding arrays.

Keybindings are defined via hexadecimal syntax suffixed with ' 0'. Function `@kbd field` is equivalent to `${keys_def[field]}` but nicer. Function kbd provides Emacsy keybinding syntax.

There is also a optionable option array possible for all keybinding arrays. Currently only defined option is [else] which defines what should happen if no key is matched from the keybinding. These arrays have to be named `key_options_<mode>`.

## MODES

Modes determine used keybinding and other properties of buffer.


|       NAME        |                DESCRIPTION                |
|-------------------|-------------------------------------------|
|edit               |general editing mode                       |
|dired              |mode used in file picker                   |
|view               |read-only mode                             |
|menu               |mode of menus                              |
|debuginput         |for getting input codes (M-x input)        |
|prefix             |for C-x prefix                             |
|prefix_help        |for C-h prefix                             |
|quit               |for quit confirmation                      |
|list_buffers       |used in buffer switcher                    |


## STYLE

Style of stuff is defined as escape code. See extensions/gruvboxdark for example of custom theme.


|              NAME              |      DEFAULT      |              DESCRIPTION              |
|--------------------------------|-------------------|---------------------------------------|
|default                         | \e[m              |Default face                           |
|TODO                            | \e[0;97;45m       |Highlighting of 'TODO: '               |
|NOTE                            | \e[0;97;100m      |Highlighting of 'NOTE: '               |
|menu                            | \e[0;37;40        |Menuline                               |
|menu-enabled-face               | \e[0;37;40        |Items of menuline                      |
|selected                        | \e[30;45m         |Selected item                          |
|link                            | \e[94m            |Redirects                              |
|menu-selected-face              | \e[30;45m         |Selected item of menu                  |
|mode-line                       | \e[40;97m         |Bottom statusline                      |
|line-number                     | \e[0;90m          |Line count                             |
|line-number-empty               | \e[0;90m          |Lines that do not exist                |
|line-number-current-line        | \e[0;91m          |Currently selected line                |
|tab-face                        | \e[0;90m          |Tabs                                   |
|minibuffer-prompt               | \e[m              |Bottom commandline                     |
|                                |                   |                                       |
|font-lock-variable-string-face  | \e[0;36;48m       |Quoted variables                       |
|font-lock-comment-face          | \e[3;37;48m       |Comments                               |
|font-lock-variable-name-face    | \e[0;96m          |Variables                              |
|font-lock-argument-face         | \e[0;93m          |Options                                |
|font-lock-flow-face             | \e[0;93m          |Control flow                           |
|font-lock-pipe-face             | \e[1;94m          |Pipes                                  |
|font-lock-bracket-face          | \e[1;95m          |Brackets                               |
|font-lock-constant-face         | \e[0;92m          |Constants                              |
|font-lock-string-face           | \e[0;32m          |Strings                                |
|font-lock-assign-face           | \e[0;94;108m      |Variable assignments                   |
|font-lock-function-name-face    | \e[0;30;44m       |Function definitions                   |
|font-lock-declare-face          | \e[0;91m          |Keywords                               |

Ebashs also includes the ansi-color-* faces, see M-x list-faces-display for full list.

## SYNTAX

Defines which syntax functions should be used for which file types.

## MENULINE

Defines items in menuline, content of keys defines which functions should be ran on invocation.

## MENUS

Each menu has to have helper function to set it up on request:
```bash
        <menu>() { declare -ng menucon=<menu>; menu; }
```
The contents of menu are defined by an associative  array.

## COMMANDS

Defines commands that can be used in `M-x`.

## EXTENDING

Some of useful variables and for extending Ebashs


|       NAME        |                DESCRIPTION                |
|-------------------|-------------------------------------------|
|buffer             |File data                                  |
|buffersyntax       |Multidimensional buffer for rendering      |
|bufferexpand       |Special characters filtered out            |
|bufferdata         |Options of current buffer                  |
|charmap            |Definitions for bufferexpand               |
|mode               |Current mode                               |
|commands           |List of M-x commands                       |

## SYNTAX HIGHLIGHTING

Ebashs handles highlighting via checking 'syntax' array which consists of `[file type]=syntax-function`

## SYNTAX FUNCTIONS

Here is sample bash syntax function included with Ebashs:

```bash
    syntax+=(
        [bash]=syntax-bash
    )
    function syntax-bash {
        ((comment)) && set-face font-lock-comment-face && return
        case "" in
             *'=()'|'declare'|'local'|'typeset') set-face font-lock-declare-face;;
            '"$'*) set-face font-lock-variable-string-face;;
            '#'*) set-face font-lock-comment-face && comment=1;;
            '$'*) set-face font-lock-variable-name-face;;
            '-'*) set-face font-lock-argument-face;;
            *'()') set-face font-lock-function-name-face;;
            '||'|'&&'|';'|'&') set-face font-lock-flow-face;;
            '>'|'<'|'|'|'>>'|'<<'|'<<<') set-face font-lock-pipe-face;;
            '('|')'|'{'|'}'|'[['|']]'|'['|']') set-face font-lock-bracket-face;;
            'function') set-face font-lock-function-name;;
            *"'"*) set-face font-lock-constant-face;;
            *'"'*) set-face font-lock-string-face;;
            *'='*) set-face font-lock-assign-face;;
            'echo'|'return'|'case'|'esac'|'for'|'while'|'do'|'done'|'if'|'elif'|
                'else'|'printf'|'fi'|'continue'|'exit'|'bind'|'then'|'break'|'read'|
                'let'|'shopt'|'trap'|'set'|'eval'
                ) set-face font-lock-keyword-face;;
            *) set-face default;;
        esac
    }
```

The comments are handled specially via comment variable which gets reseted at every newline.

## EXAMPLES

A simple function to jump to line 11 when `C-x M-e` is pressed:

```bash
    keys_prefix+=( # prefix is the mode for C-x
        [1b 65 0]='jump-to-11' # '1b 65 0' is the M-e in hex.
                               # You can use the M-x input to convert to hex. format.
    )
    jump-to-11() {
        [[ -n "" ]] && line=11 # If line 11 exists, set current line to 11.
        [[ "" = 'prefix' ]] && quit-prefix
                                    # Since the key stroke contains C-x as prefix,
                                    # quit-prefix is is required as otherwise
                                    # it would stay in 'prefix' mode.
        redraw # Redraw whole buffer.
    }
```

A function to write `Hello world!` at current cursor position when `M-x hi` is typed

```bash
    commands+=(
        [hi]='hello-world' # Add command 'hi' invoking 'hello-world' function:
    )
    hello-world() {
        insert-word 'Hello-world!' # The function 'insert-word' handles insertion
                                   # of stuff, so no redraw or other magic is needed.
    }
```

Create a menu containing previous functions `jump-to-11` & `hello-world`:

```bash
    example-menu-function() { declare -ng menucon='example_menu'; menu; }
    # A function with which the menu will be invoked.

    declare -A example_menu
    example_menu=(
        [Jump to 11  ]='jump-to-11'  # The names of items in menu should have
        [Hello world!]='hello-world' # same width to display correctly.
    )

    # Add the example_menu into default menuline
    menulineedit+=(
        [Example]='example-menu-function'
    )
```

## ETC

Logo and it's krita file is in etc/

This readme is generated from doc/README.bml via makedoc script.

## CREDITS

Based on https://github.com/comfies/bed

Early versions of Ebashs/Bano can be found at
https://github.com/aeknt/bashbox/blob/master/bin/nano
https://github.com/aeknt/bashbox/blob/master/bin/bano
