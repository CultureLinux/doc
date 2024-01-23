# History 

## View
### list 
    $ history
### search all
    $ history | grep sudo
    $ !sudo:p 
    $ !18:p
### search interactive
    Ctrl + R  

## Action
### execute 
    $ !sudo
    $ !18

### clean history
    $ history -cw
    $ > ~/.bash_history

## config
### Add timestamp
    $ echo "export HISTTIMEFORMAT='%F, %T '" >> ~/.bashrc
    $ source ~/.bashrc
### Increase history length (default 1000)
    $ echo "HISTSIZE=10000" >> ~/.bashrc
    $ echo "HISTFILESIZE=10000" >> ~/.bashrc    
    $ source ~/.bashrc
### Direct write to history
    $ echo "PROMPT_COMMAND='history -a'" >> ~/.bashrc   
    $ source ~/.bashrc
