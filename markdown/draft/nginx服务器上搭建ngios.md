
chmod +x /usr/local/nagios/libexec/check_memory.pl  && chown nagios:nagios /usr/local/nagios/libexec/check_memory.pl && ll /usr/local/nagios/libexec/|grep mem && /usr/local/nagios/libexec/check_memory.pl -f -w 10 -c 5



vi /usr/local/nagios/etc/nrpe.cfg


command[check_free_mem]=/usr/local/nagios/libexec/check_memory.pl -f -w 10 -c 5





service xinetd restart


define service{
      use                            generic-service
      host_name                      train
      service_description            Memory Usage
      check_command                  check_nrpe!check_free_mem
      register                        1
      }


service nagios reload

scp /usr/local/nagios/libexec/check_memory.pl root@172.31.16.18:/usr/local/nagios/libexec/