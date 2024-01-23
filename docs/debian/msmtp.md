# msmtp
##  install
    # apt install msmtp msmtp-mta
## Config    
    # cat << EOF > /etc/msmtprc

    defaults
    auth           on
    tls            on
    tls_starttls   on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt
    logfile        /var/log/msmtp

    account        gmail
    auth           plain
    host           smtp.gmail.com
    port           587
    from           sender@gmail.com
    user           sender@gmail.com
    password       XXXXXXXXXXXXXXXXXXXXXXXX

    account default : gmail
    EOF
    # chown root:msmtp /etc/msmtprc
    # chmod 640 /etc/msmtprc
## Test
    $ echo "sample message" | mail -s "subject" email@address