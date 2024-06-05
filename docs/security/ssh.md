# ssh daemon
## PKI 
### generate 
    ssh-keygen -t ed25519
### list 
    ls -l ~/.ssh/
    total 40
    -rw-------. 1 clinux clinux   508 Dec 13 14:21 authorized_keys      # list of authorized public keys
    -rw-r--r--. 1 clinux clinux    49 May 16 13:58 config               # configuration (ex ssh jump)
    -rw-------. 1 clinux clinux   411 Oct 20  2023 id_ed25519           # Keep this private 
    -rw-r--r--. 1 clinux clinux   101 Oct 20  2023 id_ed25519.pub       # used to authenticate 