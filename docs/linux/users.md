# Users

## regular 

### get id 
    $ id
    uid=1000(cyklodev) gid=1000(cyklodev) groups=1000(cyklodev)

### change password
    $ passwd
    Changing password for user cyklodev.
    Current password:
    New password:
    Retype new password:
    passwd: all authentication tokens updated successfully.

### get password age
    $ chage -l cyklodev
    Last password change                                    : Sep 30, 2023
    Password expires                                        : never
    Password inactive                                       : never
    Account expires                                         : never
    Minimum number of days between password change          : 0
    Maximum number of days between password change          : 99999
    Number of days of warning before password expires       : 7

## root 

### change age password 
    # chage -I -1 -m 1 -M 90 -E -1 -d $(date '+%F') cyklodev
* -I : day before account is locked
* -m : minimal duration before password can be changed
* -M : maximum duration of valid password
* -E : date of password expiration (days from timestamp init)
* -d : date of password changing

### add user to sudoers
    # usermod –aG wheel $USER

### add user to group1 and group2
    # usermod –aG group1,group2 $USER

### remove user to group1 and group2
    # usermod –aG group1,group2 $USER
    