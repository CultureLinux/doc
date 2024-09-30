# Sudoers
## user to otheruser (no password)
### sudoers.d file
```
vi /etc/sudoers.d/10-gitlab-runner
```
```
gitlab-runner ALL=(otheruser) NOPASSWD: /full/path/to/executable
```
```
vi /etc/sudoers.d/10-gitlab-runner
```
### check
```
su - otheruser
sudo -l
```