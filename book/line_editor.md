# Reedline. Nushell's line editor

Nushell's line editor [Reedline](https://github.com/nushell/reedline) is a
cross platform line reader designed to be modular and flexible. The engine is
in charge of controlling the command history, validations, completions, hints
and screen paint.

## Configuration

### Editing mode

Reedline allows you to edit text using two modes: vi and emacs. If not
specified, the default edit mode is emacs mode. In order to select your
favorite you need to modify your config file and write down your preferred
mode.

For example:
```bash
  let $config = {
    ...
    edit_mode: emacs
    ...
  }
```

Each edit mode comes with the usual keybinding for vi and emacs text editing.
However you can define additional keybindings to improve your editing
experience.

### Command history

As mentioned before, Reedline manages and stores all the commands that are
edited and sent to Nushell. To configure the max number of records that
Reedline should store you will need to adjust this value in your config file:

```bash
  let $config = {
    ...
    max_history_size: 1000
    ...
  }
```

### Customizing your prompt

Reedline prompt is also highly customizable. In order to construct your perfect
prompt, you could define the next environmental variables in your config file:

```bash
# Use nushell functions to define your right and left prompt
def create_left_prompt [] {
    let path_segment = ($nu.cwd)

    $path_segment
}

def create_right_prompt [] {
    let time_segment = ([
        (date now | date format '%m/%d/%Y %r')
    ] | str collect)

    $time_segment
}

let-env PROMPT_COMMAND = { create_left_prompt }
let-env PROMPT_COMMAND_RIGHT = { create_right_prompt }
```

> Note: You don't have to define the environmental variables using Nushell
> functions. You can use simple strings to define them.

You can also customize the prompt indicator for the line editor by modifying
the next env variables.

```bash
let-env PROMPT_INDICATOR = "〉"
let-env PROMPT_INDICATOR_VI_INSERT = ": "
let-env PROMPT_INDICATOR_VI_NORMAL = "〉"
let-env PROMPT_MULTILINE_INDICATOR = "::: "
```

> Note: The prompt indicators are environmental variables that represent the
> state of the prompt


## Keybindings

Reedline keybindings are powerful constructs that let you build chains of
events that can be triggered with a specific combination of keys.

For example, let's say that you would like to map the completion menu to the
`Ctrl + t` keybinding (default is `tab`). You can add the next entry to your
config file.

```bash
  let $config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: { send: menu name: completion_menu }
      }
    ]

    ...
  }
```

After loading this new `config.nu`, your new keybinding (`Ctrl + t`) will open
the completion command.

Each keybinding requires the next elements:
- name: Unique name for your keybinding for easy reference in `$config.keybindings`
- modifier: A key modifier for the keybinding. The options are:
  - none
  - control
  - alt
  - shift
  - control | alt
  - control | alt | shift
- keycode: This represent the key to be pressed
- mode: emacs, vi_insert, vi_normal (a single string or a list. e.g.
  [`vi_insert` `vi_normal`])
- event: The type of event that is going to be sent by the keybinding. The
  options are:
    - send
    - edit
    - until

> Note: All of the available modifiers, keycodes and events can be found with
> the command `keybindings list`

The event section of the keybinding entry is where the actions to be performed
are defined. In this field you can use either a record or a list of records.
Something like this

```bash
  ...
  event: { send: Enter }
  ...
```
or
```bash
  ...
  event: [
    { edit: Clear }
    { send: Enter }
  ]
  ...
```

The first keybinding example shown in this page follows the first case; a
single event is sent to the engine.

The next keybinding is an example of a series of events sent to the engine. It
first clears the prompt, inserts a string and then enters that value

```bash
  let $config = {
    ...

    keybindings: [
    {
      name: change_dir_with_fzf
      modifier: CONTROL
      keycode: Char_t
      mode: emacs
      event:[
          { edit: Clear }
          { edit: InsertString,
            value: "cd (ls | where type == dir | each { |it| $it.name} | str collect (char nl) | fzf | decode utf-8 | str trim)"

          }
          { send: Enter }
        ]
    }

    ...
  }
```

One disadvantage of the previous keybinding is the fact that the inserted text
will be processed by the validator and saved in the history, making the
keybinding a bit slow and populating the command history with the same command.
For that reason there is the `executehostcommand` type of event. The next
example does the same as the previous one in a simpler way, sending a single
event to the engine

```bash
  let $config = {
    ...

    keybindings: [
    {
      name: change_dir_with_fzf
      modifier: CONTROL
      keycode: Char_y
      mode: emacs
      event: {
        send: executehostcommand,
        cmd: "cd (ls | where type == dir | each { |it| $it.name} | str collect (char nl) | fzf | decode utf-8 | str trim)"
      }
    }
  ]

    ...
  }
```

Before we continue you must have noticed that the syntax changes for edits and
sends, and for that reason it is important to explain them a bit more. A `send`
is all the `Reedline` events that can be processed by the engine and an `edit`
are all the `EditCommands` that can be processed by the engine.

### Send type

To find all the available options for `send` you can use
```bash
keybindings list | where type == events
```
And the syntax for `send` events is the next one
```bash
    ...
      event: { send: <NAME OF EVENT FROM LIST> }
    ...
```
> Note: You can write the name of the events with capital letters. The
> keybinding parser is case insensitive

There are two exceptions to this rule: the `Menu` and `ExecuteHostCommand`.
Those two events require an extra field to be complete. The `Menu` needs the
name of the menu to be activated (completion_menu or history_menu)

```bash
    ...
      event: {
        send: menu
        name: completion_menu
      }
    ...
```
and the `ExecuteHostCommand` requires a valid command that will be sent to the
engine
```bash
    ...
      event: {
        send: executehostcommand
        cmd: "cd ~"
      }
    ...
```

It is worth mentioning that in the events list you will also see `Edit([])`,
`Multiple([])` and `UntilFound([])`. These options are not available for the
parser since they are constructed based on the keybinding definition. For
example, a `Multiple([])` event is built for you when defining a list of
records in the keybinding's event.  An `Edit([])` event is the same as the
`edit` type that was mentioned. And the `UntilFound([])` event is the same as
the `until` type mentioned before.

### Edit type

The `edit` type is the simplification of the `Edit([])` event. The `event` type
simplifies defining complex editing events for the keybindings. To list the
available options you can use the next command
```bash
keybindings list | where type == edits
```
The usual syntax for an `edit` is the next one
```bash
    ...
      event: { edit: <NAME OF EDIT FROM LIST> }
    ...
```
The syntax for the edits in the list that have a `()` changes a little bit.
Since those edits require an extra value to be fully defined. For example, if
we would like to insert a string where the prompt is located, then you will
have to use
```bash
    ...
      event: {
        edit: insertstring
        value: "MY NEW STRING"
      }
    ...
```
or say you want to move right until the first `S`
```bash
    ...
      event: {
        edit: moverightuntil
        value: "S"
      }
    ...
```

As you can see, these two types will allow you to construct any type of
keybinding that you require

### Until type

To complete this keybinding tour we need to discuss the `until` type for event.
As you have seen so far, you can send a single event or a list of events. And
as we have seen, when a list of events is sent, each and every one of them is
processed.

However, there may be cases when you want to assign different events to the
same keybinding. This is especially useful with Nushell menus. For example, say
you still want to activate your completion menu with `Ctrl + t` but you also
want to move to the next element in the menu once it is activated using the
same keybinding.

For these cases, we have the `until` keyword. The events listed inside the
until event will be processed one by one with the difference that as soon as
one is successful, the event processing is stopped.

The next keybinding represents this case.

```bash
  let $config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: {
          until: [
            { send: menu name: completion_menu }
            { send: menunext }
          ]
        }
      }
    ]

    ...
  }
```

The previous keybinding will first try to open a completion menu. If the menu
is not active, it will activate it and send a success signal. If the keybinding
is pressed again, since there is an active menu, then the next event it will
send is MenuNext, which means that it will move the selector to the next
element in the menu.

As you can see the `until` keyword allows us to define two events for the same
keybinding. At the moment of this writing, only the Menu events allow this type
of layering. The other non menu event types will always return a success value,
meaning that the `until` event will stop as soon as it reaches the command.

For example, the next keybinding will always send a `down` because that event
is always successful

```bash
  let $config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: {
          until: [
            { send: down }
            { send: menu name: completion_menu }
            { send: menunext }
          ]
        }
      }
    ]

    ...
  }
```


## Menus

Thanks to Reedline, Nushell has menus that can help you with your day to day
shell scripting.

### Completion menu

The completion menu is a context sensitive menu that will present suggestions
based on the status of the prompt. These suggestions can range from path
suggestions to command alternatives. While writing a command, you can activate
the menu to see available flags for an internal command. Also, if you have
defined your custom completions for external commands, these will appear in the
menu as well.

The completion menu by default is accessed by pressing `tab` and it can be configured by
modifying these values from the config object:

```bash
  let $config = {
    ...

    quick_completions: true    # set this to false to prevent auto-selecting completions when only one remains
    partial_completions: true  # set this to false to prevent partial filling of the prompt

    menu_config: {
      columns: 4         # Number of columns in the menu
      col_width: 20      # Optional value. If missing all the screen width is used to calculate column width
      col_padding: 2     # Number of characters between string and next column
      text_style: green  # Text color style for non selected text
      selected_text_style: green_reverse  # Color style for selected text
      marker: "| "       # Indicator that appears when the menu is active
    }

    ...
  }
```

By modifying these parameters you can customize the layout of your menu to your
liking.

### History menu

The history menu is a handy way to access the editor history. When activating
the menu (default `Ctrl+x`) the command history is presented in reverse
chronological order, making it extremely easy to select a previous command.

The history menu can be configured by modifying these values from the config object:

```bash
  let $config = {
    ...

    history_config: {
      page_size: 10      # Number of entries that will presented when activating the menu
      selector: "!"      # Character to indicate a quick selection in the menu
      text_style: green  # Text color style for non selected text
      selected_text_style: green_reverse  # Color style for selected text
      marker: "? "       # Indicator that appears when the menu is active
    }

    ...
  }

```

When the history menu is activated, it pulls `page_size` records from the
history and presents them in the menu. If there is space in the terminal, when
you press `Ctrl+x` again the menu will pull the same number of records and
append them to the current page. If it isn't possible to present all the pulled
records, the menu will create a new page. The pages can be navigated by
pressing `Ctrl+z` to go to previous page or `Ctrl+x` to go to next page.

#### Searching the history

To search in your history you can start typing key words for the command you
are looking for. Once the menu is activated, anything that you type will be
replaced by the selected command from your history. for example, say that you
have already typed this

```bash
let a = ()
```
you can place the cursor inside the `()` and activate the menu. You can filter
the history by typing key words and as soon as you select an entry, the typed
words will be replaced

```bash
let a = (ls | where size > 10MiB)
```

#### Menu quick selection

Another nice feature of the menu is the ability to quick select something from
it. Say you have activated your menu and it looks like this

```bash
>
0: ls | where size > 10MiB
1: ls | where size > 20MiB
2: ls | where size > 30MiB
3: ls | where size > 40MiB
```
Instead of pressing down to select the fourth entry, you can type `!3` and
press enter. This will insert the selected text in the prompt position, saving
you time scrolling down the menu.

History search and quick selection can be used together. You can activate the menu, do a quick
search, and then quick select using the quick selection character.

### Menu keybindings

In case you want to change the default way both menus are activated, you can
change that by defining new keybindings. For example, the next two keybindings
assign the completion and history menu to `Ctrl+t` and `Ctrl+y` respectively

```bash
  let $config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: [vi_insert vi_normal]
        event: {
          until: [
            { send: menu name: completion_menu }
            { send: menupagenext }
          ]
        }
      }
      {
        name: history_menu
        modifier: control
        keycode: char_y
        mode: [vi_insert vi_normal]
        event: {
          until: [
            { send: menu name: history_menu }
            { send: menupagenext }
          ]
        }
      }
    ]

    ...
  }
```
