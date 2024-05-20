# History
This section provides an overview of commands and configurations related to the command history in a Unix/Linux shell.

## View
This subsection explains how to view and search through the command history.

### list
Lists all previously executed commands.
```sh
$ history
```

### search all
Explains how to search for specific commands in the history using `grep` and shortcuts.
```sh
$ history | grep sudo
$ !sudo:p 
$ !18:p
```

### search interactive
Describes how to use interactive search to find commands in the history.
```sh
Ctrl + R  
```

## Action
This subsection covers the actions that can be performed on the command history.

### execute
Shows how to re-execute specific commands from the history.
```sh
$ !sudo
$ !18
```

### clean history
Explains how to clear the command history.
```sh
$ history -cw
$ > ~/.bash_history
```

## config
This subsection details the possible configurations for the command history.

### Add timestamp
Indicates how to add a timestamp to each command in the history.
```sh
$ echo "export HISTTIMEFORMAT='%F, %T '" >> ~/.bashrc
$ source ~/.bashrc
```

### Increase history length (default 1000)
Explains how to increase the default command history length.
```sh
$ echo "HISTSIZE=10000" >> ~/.bashrc
$ echo "HISTFILESIZE=10000" >> ~/.bashrc    
$ source ~/.bashrc
```

### Direct write to history
Shows how to configure direct writing of commands to the history file.
```sh
$ echo "PROMPT_COMMAND='history -a'" >> ~/.bashrc   
$ source ~/.bashrc
```
