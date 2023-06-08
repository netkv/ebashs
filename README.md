# EBASHS
## This is Ebashs, one component of the Bash/Bash operating system.

*… or even better an attempt to clone GNU Emacs in bash.*

> Note that the readme, may be currently outdated as I am changing the core functioning of Ebashs.

> Versions of Ebashs ending in WIP-x are and will be broken, you can check version via M-x about
or by reading the second line of Ebashs script.

![Ebashs logo](etc/Ebashs.png)

# FEATURES

- Syntax highlighting
- Custom keybindings
- Custom modes (so you can implement evil-mode as well)
- File picker
- Mouse support

# CONFIG
Ebashs is configured via variables defined at start, you can turn it into a config file and then source it.

## MISC. SETTINGS
- `mouse` *(0)* - enables mouse at launch
- `todonote` *(1)* - highlight TODO & NOTE
- `menuline` *(1)* - display menuline
- `tabchar` *('|   ')* - what should tab display as
- `refresh_timeout` *(1)* - how often to refresh input
- `file_prompt` *('Path: ')* - how often to refresh input
- `cmd_prompt` *(M-x)* - command line prompt
- `cancelhex` *('18 0' (C-x)*) - keybinding to exist debug input menu
- `default_mode` *(edit)* - what mode should be set on launch
- `help_message` *('F10 to open menu')* - top right help message
- `dired_message` *('Pick a file')* - same as above but for file picker

## KEYBINDINGS 

Keybinding arrays match with ***modes***. The array has to be associative and named
`keys_<mode>`. `keys_def` is reserved and used as reference for other arrays, it's
also reversed compared to other keybinding arrays.

There is also a optionable option array possible for all keybinding arrays.
Currently only defined option is `else` which defines what should happen if no key
is matched from the keybinding. These arrays have to be named `key_options_<mode>`.

## MODES

Modes determine used keybinding and other properties of buffer.

- `edit` - general editing mode                       
- `dired mode` - used in file picker                   
- `view` - readonly mode                              
- `command` - proof of concept vim like command mode    
- `menu` - mode of menus                            
- `debuginput` - for getting input codes *(M-x input)*
- `prefix` - for *(C-x)* prefix                           
- `prefix_help` - for C-h prefix                          
- `quit` - used for quit message                 
- `list_buffers` - used for buffer switcher               

## STYLE

Style of UI Elements is defined as escape code.

- `none` *(\e[m)* - Style reset
- `TODO` *(\e[0;97;45m)* - Highlighting of TODO
- `NOTE` *(\e[0;97;100m)* - Highlighting of NOTE
- `menuline` *(\e[0;37;40)* - Menuline
- `menuitem` *(\e[0;37;40)* - Items in menuline
- `link` *(\e[94m)* - Redirects
- `menuitemselected` *(\e[30;45m)* - Selected item inside a menu
- `infoline` *(\e[40;97m)* - Bottom statusline
- `number` *(\e[0;90m)* - Line count
- `numberempty` *(\e[0;90m)* - Lines that do not exist
- `numberselected` *(\e[0;91m)* - Currently selected line
- `tab` *(\e[0;90m)* - Tabs
- `quotevar` *(\e[0;36;48m)* - Quoted variables
- `comment` *(\e[3;37;48m)* - Comments
- `variable` *(\e[0;96m)* - Variables
- `option` *(\e[0;93m)* - Options (like read -r)
- `flow` *(\e[0;93;m)* - Control flow
- `bracket` *(\e[1;95m)* - Brackets
- `quote` *(\e[0;92m)* - String literals
- `dquote` *(\e[0;32m)* - Strings
- `set` *(\e[0;94;108m)* - Variable assignments
- `fn` *(\e[0;30;44m)* - Function definitions
- `keyword` *(\e[0;91m)* - Keywords

### SYNTAX

Defines which syntax functions should be used for which file types.

### MENUS

Each menu has to have helper function to set it up on request:

```bash
<menu>() { declare -ng menucon=<menu>; menu; }
```

The contents of menu are defined by an associative array.

### MENULINE

Defines items in menuline, content of keys defines which functions should be ran on invocation.

### COMMANDS

Defines commands that can be used in `M-x`.

## EXTENDING

Some useful variables for extending Ebashs;

- `buffer` - File data 
- `buffersyntax` - Multidimensional buffer for rendering 
- `bufferexpand` - Special characters filtered out
- `bufferdata` - Options of current buffer                  
- `charmap` - Definitions for bufferexpand               
- `mode`- Current mode                        
- `commands` - List of `M-x` commands 

# SYNTAX HIGHLIGHTING

Ebashs handles highlighting via checking 'syntax' array which consists of `***file type***=syntax-function`

## SYNTAX FUNCTIONS

Here is sample bash syntax function included with Ebashs:

```bash
    syntax+=(
        [bash]=syntax-bash
    )
    syntax-bash() {
        ((comment)) && set-style comment && return
        local word="$1"
        case "${word}" in
            '"$'*) set-style quotevar;;
             '#'*) set-style comment && comment=1;;
             '$'*) set-style variable;;
             '-'*) set-style option;;
            *'()') set-style fn;;
            '||'|'&&'|';') set-style flow;;
            '('|')'|'{'|'}'|'[['|']]'|'['|']') set-style bracket;;
            'function') set-style fn;;
            *"'"*) set-style quote;;
            *'"'*) set-style dquote;;
            *'='*) set-style set;;
            'echo'|'return'|'case'|'esac'|'for'|'while'|'do'|'done'|'if'|'elif'|\
            'else'|'printf'|'fi'|'continue'|'exit'|'bind'|'then'|'break'|'read'|'declare'|\
            'typeset'|'local'|'let'|'shopt'|'trap'|'set'|'eval'\
             ) set-style keyword;;
            *) set-style none;;
        esac
```

The comments are handled specially via comment variable which gets reseted at every newline. 

## EXAMPLES

### A simple function to jump to line 11 when `C-x M-e` is pressed:

```bash
    keys_prefix+=( # prefix is the mode for C-x
        [1b 65 0]='jump-to-11' # '1b 65 0' is the M-e in hex.
                               # You can use the M-x input to convert to hex. format.
    )
    jump-to-11() {
        [[ -n "${buffer[11]}" ]] && line=11 # If line 11 exists, set current line to 11.
        [[ "${bufferdata[mode]}" = 'prefix' ]] && quit-prefix
                                    # Since the key stroke contains C-x as prefix,
                                    # quit-prefix is is required as otherwise
                                    # it would stay in 'prefix' mode.
        redraw # Redraw whole buffer.
    }
```

### A function to write `'Hello world!'` at current cursor position when `M-x h i` is typed.

```bash
    commands+=(
        [hi]='hello-world' # Add command 'hi' invoking 'hello-world' function:
    )
    hello-world() {
        insert-word 'Hello-world!' # The function 'insert-word' handles insertion
                                   # of stuff, so no redraw or other magic is needed.
    }
```

### Create a menu containing previous functions `'jump-to-11'` & `'hello-world'`:

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

# CREDITS

Based on https://github.com/comfies/bed
