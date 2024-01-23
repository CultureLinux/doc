# packages
## Apt
### update / upgrade
#### update 
refresh repository
```
# apt update 
```

#### upgrade
really upgrade packages
```
# apt upgrade
```
    
really upgrade packages and remove unused packages
```
# apt full-upgrade
```
    

### install package
    # apt install nmon

### remove package
```
# apt remove nmon
```
    
remove + purge 
```
# apt autoremove nmon
```
    

### package provided by
    # apt install apt-file
    # apt-file update
    # apt-file search --package-only netstat

## Nala
### install from repo
    # apt install nala
    # nala --version
    nala 0.12.0

### install from asset
    # wget https://gitlab.com/volian/nala/uploads/dd7631fdacb0c4837df18ed11b8bfe15/nala_0.14.0_all.deb
    # apt install ./nala_0.14.0_all.deb
    # nala --version
    nala 0.14.0

### fetch best mirror
    # nala fetch

### install package 
    # nala install nmon

### get history
    nala history

### rollback
    nala history undo ID