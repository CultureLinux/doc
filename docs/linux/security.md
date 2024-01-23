# Security

## rkhunter
### install 
    # apt install rkhunter
### run
    # rkhunter -c --rwo
* -c : check 
* --rwo : report warning only

## lynis 
    # cd /usr/local/sbin && git clone https://github.com/CISOfy/lynis
    # lynis audit system