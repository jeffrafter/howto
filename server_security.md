# Server Security

This is very incomplete and covers some of the basics.

## UFW and ip_tables

## NTP
http://ubuntuforums.org/showthread.php?t=862620


    driftfile /var/lib/ntp/ntp.drift
    server 0.pool.ntp.org
    server 1.pool.ntp.org
    server 2.pool.ntp.org
    server 3.pool.ntp.org
    server 127.127.1.0
    fudge 127.127.1.0 stratum 10

    # Could use north america specific servers instead
    server 0.north-america.pool.ntp.org
    server 1.north-america.pool.ntp.org
    server 2.north-america.pool.ntp.org
    server 3.north-america.pool.ntp.org

## fail2Ban
See note on rsyslog

## OSSEC

## logstash

## sudo

## ssh


    # Disable password access
    sed -E -i "s/.*PasswordAuthentication.*/PasswordAuthentication no/" /etc/ssh/sshd_config
    sed -E -i "s/.*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication no/" /etc/ssh/sshd_config
    restart ssh

## Jump servers