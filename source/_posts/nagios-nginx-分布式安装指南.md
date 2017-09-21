title: nagios+nginx+分布式安装指南
author: 卫振家
tags:
  - nagios 服务器监控
categories: []
date: 2017-09-21 17:58:00
---
# nagios配置监控

##监控主机安装

- 安装nginx支持

  1. 安装perl

     ```shell
     yum install perl
     ```

  2. 安装FCGI模块

     ```shell
     wget http://search.cpan.org/CPAN/authors/id/F/FL/FLORA/FCGI-0.73.tar.gz
     tar xvzf FCGI-0.73.tar.gz 
     cd FCGI-0.73
     perl Makefile.PL
     make
     make install
     cd .. 

     #如果出现Can't locate ExtUtils/MakeMaker.pm in @INC，先执行以下任务
      yum install perl-ExtUtils-Embed -y
     ```

  3. 安装FCGI-ProcManager模块

     ```shell
     wget http://search.cpan.org/CPAN/authors/id/G/GB/GBJK/FCGI-ProcManager-0.19.tar.gz
     tar xvzf FCGI-ProcManager-0.19.tar.gz 
     cd FCGI-ProcManager-0.19
     perl Makefile.PL 
     make
     make install
     cd ..
     ```

  4. 安装IO和IO::ALL模块

     ```shell
     wget http://search.cpan.org/CPAN/authors/id/G/GB/GBARR/IO-1.25.tar.gz
     tar zxvf IO-1.25.tar.gz
     cd IO-1.25
     perl Makefile.PL
     make
     make install
     cd ..

     wget https://cpan.metacpan.org/authors/id/F/FR/FREW/IO-All-0.87.tar.gz
     tar zxvf IO-All-0.87.tar.gz
     cd IO-All-0.87
     perl Makefile.PL
     make
     make install
     cd .. 
     ```

  5. 创建perl-fcgi脚本并给予权限

     fastcgi自启动服务脚本：

     ```perl
     文件路径：/etc/rc.d/init.d/perl-fastcgi
      
     #!/bin/sh
     #
     # nginx – this script starts and stops the nginx daemon
     #
     # chkconfig: - 85 15
     # description: Nginx is an HTTP(S) server, HTTP(S) reverse \
     # proxy and IMAP/POP3 proxy server
     # processname: nginx
     # config: /opt/nginx/conf/nginx.conf
     # pidfile: /opt/nginx/logs/nginx.pid
      
     # Source function library.
     . /etc/rc.d/init.d/functions
      
     # Source networking configuration.
     . /etc/sysconfig/network
      
     # Check that networking is up.
     [ "$NETWORKING" = "no" ] && exit 0
      
     perlfastcgi="/usr/bin/fastcgi-wrapper.pl"
     prog=$(basename perl)
      
     lockfile=/var/lock/subsys/perl-fastcgi
      
     start() {
         [ -x $perlfastcgi ] || exit 5
         echo -n $"Starting $prog: "
         daemon $perlfastcgi
         retval=$?
         echo
         [ $retval -eq 0 ] && touch $lockfile
         return $retval
     }
      
     stop() {
         echo -n $"Stopping $prog: "
         killproc $prog -QUIT
         retval=$?
         echo
         [ $retval -eq 0 ] && rm -f $lockfile
         return $retval
     }
      
     restart() {
         stop
         start
     }
      
     reload() {
         echo -n $”Reloading $prog: ”
         killproc $nginx -HUP
         RETVAL=$?
         echo
     }
      
     force_reload() {
         restart
     }
     rh_status() {
         status $prog
     }
      
     rh_status_q() {
         rh_status >/dev/null 2>&1
     }
      
     case "$1" in
         start)
             rh_status_q && exit 0
             $1
             ;;
         stop)
             rh_status_q || exit 0
             $1
             ;;
         restart)
             $1
             ;;
         reload)
             rh_status_q || exit 7
             $1
             ;;
         force-reload)
             force_reload
             ;;
         status)
             rh_status
             ;;
         condrestart|try-restart)
             rh_status_q || exit 0
             ;;
         *)
             echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
             exit 2
         esac
     ```

     ​

     fastcgi监听脚本

     ```perl
     文件路径：/usr/bin/fastcgi-wrapper.pl

     #!/usr/bin/perl
      
     use FCGI;
     use Socket;
     use POSIX qw(setsid);
      
     require 'syscall.ph';
      
     &daemonize;
      
     #this keeps the program alive or something after exec'ing perl scripts
     END() { } BEGIN() { }
     *CORE::GLOBAL::exit = sub { die "fakeexit\nrc=".shift()."\n"; };
     eval q{exit};
     if ($@) {
         exit unless $@ =~ /^fakeexit/;
     };
      
     &main;
      
     sub daemonize() {
         chdir '/'                 or die "Can't chdir to /: $!";
         defined(my $pid = fork)   or die "Can't fork: $!";
         exit if $pid;
         setsid                    or die "Can't start a new session: $!";
         umask 0;
     }
      
     sub main {
             $socket = FCGI::OpenSocket( "127.0.0.1:8999", 10 ); #use IP sockets
             $request = FCGI::Request( \*STDIN, \*STDOUT, \*STDERR, \%req_params, $socket );
             if ($request) { request_loop()};
                 FCGI::CloseSocket( $socket );
     }
      
     sub request_loop {
             while( $request->Accept() >= 0 ) {
      
                #processing any STDIN input from WebServer (for CGI-POST actions)
                $stdin_passthrough ='';
                $req_len = 0 + $req_params{'CONTENT_LENGTH'};
                if (($req_params{'REQUEST_METHOD'} eq 'POST') && ($req_len != 0) ){
                     my $bytes_read = 0;
                     while ($bytes_read < $req_len) {
                             my $data = '';
                             my $bytes = read(STDIN, $data, ($req_len - $bytes_read));
                             last if ($bytes == 0 || !defined($bytes));
                             $stdin_passthrough .= $data;
                             $bytes_read += $bytes;
                     }
                 }
      
                 #running the cgi app
                 if ( (-x $req_params{SCRIPT_FILENAME}) &&  #can I execute this?
                      (-s $req_params{SCRIPT_FILENAME}) &&  #Is this file empty?
                      (-r $req_params{SCRIPT_FILENAME})     #can I read this file?
                 ){
             pipe(CHILD_RD, PARENT_WR);
             my $pid = open(KID_TO_READ, "-|");
             unless(defined($pid)) {
                 print("Content-type: text/plain\r\n\r\n");
                             print "Error: CGI app returned no output - ";
                             print "Executing $req_params{SCRIPT_FILENAME} failed !\n";
                 next;
             }
             if ($pid > 0) {
                 close(CHILD_RD);
                 print PARENT_WR $stdin_passthrough;
                 close(PARENT_WR);
      
                 while(my $s = <KID_TO_READ>) { print $s; }
                 close KID_TO_READ;
                 waitpid($pid, 0);
             } else {
                         foreach $key ( keys %req_params){
                            $ENV{$key} = $req_params{$key};
                         }
                         # cd to the script's local directory
                         if ($req_params{SCRIPT_FILENAME} =~ /^(.*)\/[^\/]+$/) {
                                 chdir $1;
                         }
      
                 close(PARENT_WR);
                 close(STDIN);
                 #fcntl(CHILD_RD, F_DUPFD, 0);
                 syscall(&SYS_dup2, fileno(CHILD_RD), 0);
                 #open(STDIN, "<&CHILD_RD");
                 exec($req_params{SCRIPT_FILENAME});
                 die("exec failed");
             }
                 }
                 else {
                     print("Content-type: text/plain\r\n\r\n");
                     print "Error: No such CGI app - $req_params{SCRIPT_FILENAME} may not ";
                     print "exist or is not executable by this process.\n";
                 }
      
             }
     }
     ```

     权限分配

     ```shell
     chmod a+x /usr/bin/fastcgi-wrapper.pl
     chmod a+x /etc/rc.d/init.d/perl-fastcgi
     ```

  6. 配置nginx 虚拟服务器

     ```nginx
     #基本版本配置 /etc/nginx/conf.d/nagios.conf
     server {
      
             listen       80;
             server_name  localhost;
             #access_log  /data/logs/nginx/test.ttlsa.com.access.log  main;
      
             index index.html index.php index.html;
             root	/usr/local/nagios/share;
      
             location / 
             {
      
             }
      
             location ~ \.pl$ 
             {
                 include       fastcgi_params;
                 fastcgi_pass  127.0.0.1:8999;
                 #fastcgi_pass  unix:/var/run/ttlsa.com.perl.sock;
                 fastcgi_index index.pl;
             }
     }
     ```


     ##如果想把tcp/ip方式改为socket方式，可以修改fastcgi-wrapper.pl.
     ## $socket = FCGI::OpenSocket( "127.0.0.1:8999", 10 ); #use IP sockets
     ## 改为
     ## $socket = FCGI::OpenSocket( "/var/run/ttlsa.com.perl.sock", 10 ); #use IP sockets
     ```

  7. 启动nginx与fastcgi

     ```shell
     #重新加载nginx配置
     nginx -s reload
     #启动Nginx
     /etc/init.d/perl-fastcgi start
     ```

  8. fastcgi 测试

     ```perl
     文件路径/data/site/test.ttlsa.com/test.pl
     #!/usr/bin/perl
      
     print "Content-type:text/html\n\n";
     print <<EndOfHTML;
     <html><head><title>Perl Environment Variables</title></head>
     <body>
     <h1>Perl Environment Variables</h1>
     EndOfHTML
      
     foreach $key (sort(keys %ENV)) {
         print "$key = $ENV{$key}<br>\n";
     }
      
     print "</body></html>";
     ```

  9. 访问测试

     ```http
     http://http:test.ttlsa.com/test.pl,出现内容表示OK.
     ```

  10. 访问验证

     [在线htpasswd生成器](http://htpasswd.net)

     进入生成网站，进行密码生成

     ```html 
     user nagios
     password 123456

     crypt 		nagios:33QvMQsjxkn3w  *nix 校验 选这个就好了
     apache md5  nagios:$apr1$AVs0w61Q$JamRT9wWtri/lE3iLWxkd/   windows校验
     ```

     ```shell
     #复制加密后的密钥对，存入文件/usr/local/nagios/etc/nagiospasswd 保存即可
     crypt 		nagios:33QvMQsjxkn3w
     ```

     配置nginx 权限访问

     ```nginx
     ##完整版本配置 /etc/nginx/conf.d/nagios.conf
     server {
           listen	80;
           server_name	localhost;
           index	index.html index.htm index.php;
           root	/usr/local/nagios/share;
           auth_basic	"Nagios Access";
           auth_basic_user_file /usr/local/nagios/etc/nagiospasswd;     

           location ~ .*\.php?$ {
           #fastcgi_pass		unix:/tmp/php-cgi.sock;
           fastcgi_pass		127.0.0.1:9000;
           fastcgi_index  	index.php;
           include	       	fastcgi.conf;
           }
     ```


            location ~ .*\.(cgi|pl)?$ {
            gzip			off;
            root			/usr/local/nagios/sbin;
            rewrite			^/nagios/cgibin/(.*)\.cgi /$1.cgi break;
            fastcgi_pass		127.0.0.1:8999;
            fastcgi_param		SCRIPT_FILENAME	/usr/local/nagios/sbin$fastcgi_script_name;
            fastcgi_index		index.cgi;
            fastcgi_read_timeout	60;
            fastcgi_param		REMOTE_USER	$remote_user;
            include			fastcgi.conf;
            auth_basic		"Nagios Access";
            auth_basic_user_file	/usr/local/nagios/etc/nagiospasswd;
            }
    
            location /nagios {
            alias 			/usr/local/nagios/share;
            auth_basic  		"Nagios Access";
            auth_basic_user_file 	/usr/local/nagios/etc/nagiospasswd;
            }
      }
      ```
    
      重启加载nginx配置
    
      ```shell
      nginx -s reload
      ```
    
      ​

- 安装nagios

  1. 依赖安装

     ```shell
     #检测依赖安装状态
     rpm -q gcc glibc glibc-common gd gd-devel xinetd openssl-devel
     #安装依赖
     yum install -y gcc glibc glibc-common gd gd-devel xinetd openssl-devel
     ```


  2. 创建监控用户，用户组，指定安装目录，权限处理

     ```shell
     # 新增用户
     useradd -s /sbin/nologin nagios
     # 创建安装目录
     mkdir /usr/local/nagios
     # 修改目录归属用户
     chown -R nagios.nagios /usr/local/nagios
     # 查看nagios 目录的权限
     ll -d /usr/local/nagios/
     ```


  3. 下载安装nagios

     ```shell
     wget https://downloads.sourceforge.net/project/nagios/nagios-4.x/nagios-4.3.4/nagios-4.3.4.tar.gz
     tar zxvf nagios-4.3.4.tar.gz
     cd nagios-4.3.4

     ./configure --prefix=/usr/local/nagios
     make all
     make install
     make install-init
     make install-commandmode
     make install-config

     #检查安装
     chkconfig --add nagios
     chkconfig --level 35 nagios on
     chkconfig --list nagios

     ls /usr/local/nagios
     ....result....
     bin  etc  include  libexec  sbin  share  var
     ```

  4. 下载安装nagios 插件 nagios-plugin

     ```shell
     wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
     cd nagios-plugins-1.4.16
     ./configure --prefix=/usr/local/nagios
     make && make install
     cd ..
     ```

- 配置nagios

  1. 配置文件

     | **文件名或目录名**             | **用途**                                   |
     | ----------------------- | ---------------------------------------- |
     | cgi.cfg                 | 控制CGI访问的配置文件                             |
     | nagios.cfg              | Nagios 主配置文件                             |
     | resource.cfg            | 变量定义文件，又称为资源文件，在些文件中定义变量，以便由其他配置文件引用，如$USER1$ |
     | objects                 | objects 是一个目录，在此目录下有很多配置文件模板，用于定义Nagios 对象 |
     | objects/commands.cfg    | 命令定义配置文件，其中定义的命令可以被其他配置文件引用              |
     | objects/contacts.cfg    | 定义联系人和联系人组的配置文件                          |
     | objects/localhost.cfg   | 定义监控本地主机的配置文件                            |
     | objects/printer.cfg     | 定义监控打印机的一个配置文件模板，默认没有启用此文件               |
     | objects/switch.cfg      | 定义监控路由器的一个配置文件模板，默认没有启用此文件               |
     | objects/templates.cfg   | 定义主机和服务的一个模板配置文件，可以在其他配置文件中引用            |
     | objects/timeperiods.cfg | 定义Nagios 监控时间段的配置文件                      |
     | objects/windows.cfg     | 监控Windows 主机的一个配置文件模板，默认没有启用此文件          |

  2. 配置文件之间的关系

     在nagios的配置过程中涉及到的几个定义有：主机、主机组，服务、服务组，联系人、联系人组，监控时间，监控命令等，从这些定义可以看出，nagios各个配置文件之间是互为关联，彼此引用的。

     成功配置出一台nagios监控系统，必须要弄清楚每个配置文件之间依赖与被依赖的关系，最重要的有四点：

     - 定义监控哪些主机、主机组、服务和服务组； 
     - 定义这个监控要用什么命令实现； 
     - 定义监控的时间段； 
     - 定义主机或服务出现问题时要通知的联系人和联系人组。

  3. 配置Nagios

      为了能更清楚的说明问题，同时也为了维护方便，建议将nagios各个定义对象创建独立的配置文件：

     -  创建hosts.cfg文件来定义主机和主机组
     -  创建services.cfg文件来定义服务
     -  用默认的contacts.cfg文件来定义联系人和联系人组
     -  用默认的commands.cfg文件来定义命令
     -  用默认的timeperiods.cfg来定义监控时间段
     -  用默认的templates.cfg文件作为资源引用文件

  4. 配置文件详解

     templates.cfg

     ```cfg
     define contact{
             name                            generic-contact    ; 联系人名称
             service_notification_period     24x7               ; 当服务出现异常时，发送通知的时间段，这个时间段"24x7"在timeperiods.cfg文件中定义
             host_notification_period        24x7               ; 当主机出现异常时，发送通知的时间段，这个时间段"24x7"在timeperiods.cfg文件中定义
             service_notification_options    w,u,c,r            ; 这个定义的是“通知可以被发出的情况”。w即warn，表示警告状态，u即unknown，表示不明状态;
                                                                ; c即criticle，表示紧急状态，r即recover，表示恢复状态;
                                                                ; 也就是在服务出现警告状态、未知状态、紧急状态和重新恢复状态时都发送通知给使用者。
             host_notification_options       d,u,r                   ; 定义主机在什么状态下需要发送通知给使用者，d即down，表示宕机状态;
                                                                     ; u即unreachable，表示不可到达状态，r即recovery，表示重新恢复状态。
             service_notification_commands   notify-service-by-email ; 服务故障时，发送通知的方式，可以是邮件和短信，这里发送的方式是邮件;
                                                                     ; 其中“notify-service-by-email”在commands.cfg文件中定义。
             host_notification_commands      notify-host-by-email    ; 主机故障时，发送通知的方式，可以是邮件和短信，这里发送的方式是邮件;
                                                                     ; 其中“notify-host-by-email”在commands.cfg文件中定义。 
             register                        0                    ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL CONTACT, JUST A TEMPLATE!
             }
     define host{
             name                            generic-host    ; 主机名称，这里的主机名，并不是直接对应到真正机器的主机名;
                                                             ; 乃是对应到在主机配置文件里所设定的主机名。
             notifications_enabled           1               ; Host notifications are enabled
             event_handler_enabled           1               ; Host event handler is enabled
             flap_detection_enabled          1               ; Flap detection is enabled
             failure_prediction_enabled      1               ; Failure prediction is enabled
             process_perf_data               1               ; 其值可以为0或1，其作用为是否启用Nagios的数据输出功能;
                                                             ; 如果将此项赋值为1，那么Nagios就会将收集的数据写入某个文件中，以备提取。
             retain_status_information       1               ; Retain status information across program restarts
             retain_nonstatus_information    1               ; Retain non-status information across program restarts
             notification_period             24x7            ; 指定“发送通知”的时间段，也就是可以在什么时候发送通知给使用者。
             register                        0               ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
             }
     define host{
             name                            linux-server    ; 主机名称
             use                             generic-host    ; use表示引用，也就是将主机generic-host的所有属性引用到linux-server中来;
                                                             ; 在nagios配置中，很多情况下会用到引用。
             check_period                    24x7            ; 这里的check_period告诉nagios检查主机的时间段
             check_interval                  5               ; nagios对主机的检查时间间隔，这里是5分钟。
             retry_interval                  1               ; 重试检查时间间隔，单位是分钟。
             max_check_attempts              10              ; nagios对主机的最大检查次数，也就是nagios在检查发现某主机异常时，并不马上判断为异常状况;
                                                             ; 而是多试几次，因为有可能只是一时网络太拥挤，或是一些其他原因，让主机受到了一点影响;
                                                             ; 这里的10就是最多试10次的意思。
             check_command                   check-host-alive ; 指定检查主机状态的命令，其中“check-host-alive”在commands.cfg文件中定义。
             notification_period             24x7            ; 主机故障时，发送通知的时间范围，其中“workhours”在timeperiods.cfg中进行了定义;
                                                             ; 下面会陆续讲到。
             notification_interval           10              ; 在主机出现异常后，故障一直没有解决，nagios再次对使用者发出通知的时间。单位是分钟;
                                                             ; 如果你觉得，所有的事件只需要一次通知就够了，可以把这里的选项设为0
             notification_options            d,u,r           ; 定义主机在什么状态下可以发送通知给使用者，d即down，表示宕机状态;
                                                             ; u即unreachable，表示不可到达状态;
                                                             ; r即recovery，表示重新恢复状态。
             contact_groups                  ts              ; 指定联系人组，这个“admins”在contacts.cfg文件中定义。
             register                        0               ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
             }
     define host{
             name                    windows-server  ; The name of this host template
             use                     generic-host    ; Inherit default values from the generic-host template
             check_period            24x7            ; By default, Windows servers are monitored round the clock
             check_interval          5               ; Actively check the server every 5 minutes
             retry_interval          1               ; Schedule host check retries at 1 minute intervals
             max_check_attempts      10              ; Check each server 10 times (max)
             check_command           check-host-alive        ; Default command to check if servers are "alive"
             notification_period     24x7            ; Send notification out at any time - day or night
             notification_interval   10              ; Resend notifications every 30 minutes
             notification_options    d,r             ; Only send notifications for specific host states
             contact_groups          ts              ; Notifications get sent to the admins by default
             hostgroups              windows-servers ; Host groups that Windows servers should be a member of
             register                0               ; DONT REGISTER THIS - ITS JUST A TEMPLATE
             }
     define service{
             name                            generic-service         ; 定义一个服务名称
             active_checks_enabled           1                       ; Active service checks are enabled
             passive_checks_enabled          1                       ; Passive service checks are enabled/accepted
             parallelize_check               1                       ; Active service checks should be parallelized;
                                                                     ; (disabling this can lead to major performance problems)
             obsess_over_service             1                       ; We should obsess over this service (if necessary)
             check_freshness                 0                       ; Default is to NOT check service 'freshness'
             notifications_enabled           1                       ; Service notifications are enabled
             event_handler_enabled           1                       ; Service event handler is enabled
             flap_detection_enabled          1                       ; Flap detection is enabled
             failure_prediction_enabled      1                       ; Failure prediction is enabled
             process_perf_data               1                       ; Process performance data
             retain_status_information       1                       ; Retain status information across program restarts
             retain_nonstatus_information    1                       ; Retain non-status information across program restarts
             is_volatile                     0                       ; The service is not volatile
             check_period                    24x7             ; 这里的check_period告诉nagios检查服务的时间段。
             max_check_attempts              3                ; nagios对服务的最大检查次数。
             normal_check_interval           5                ; 此选项是用来设置服务检查时间间隔，也就是说，nagios这一次检查和下一次检查之间所隔的时间;
                                                              ; 这里是5分钟。
             retry_check_interval            2                ; 重试检查时间间隔，单位是分钟。
             contact_groups                  ts           ; 指定联系人组
             notification_options            w,u,c,r          ; 这个定义的是“通知可以被发出的情况”。w即warn，表示警告状态;
                                                              ; u即unknown，表示不明状态;
                                                              ; c即criticle，表示紧急状态，r即recover，表示恢复状态;
                                                              ; 也就是在服务出现警告状态、未知状态、紧急状态和重新恢复后都发送通知给使用者。
             notification_interval           10               ; Re-notify about service problems every hour
             notification_period             24x7             ; 指定“发送通知”的时间段，也就是可以在什么时候发送通知给使用者。
             register                        0                ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL SERVICE, JUST A TEMPLATE!
             }
     define service{
             name                            local-service           ; The name of this service template
             use                             generic-service         ; Inherit default values from the generic-service definition
             max_check_attempts              4             ; Re-check the service up to 4 times in order to determine its final (hard) state
             normal_check_interval           5             ; Check the service every 5 minutes under normal conditions
             retry_check_interval            1             ; Re-check the service every minute until a hard state can be determined
             register                        0             ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL SERVICE, JUST A TEMPLATE!
             }
     ```

     resource.cfg

     ```cfg
     $USER1$=/usr/local/nagios/libexec
     ```

     commands.cfg

     ```cfg
     #notify-host-by-email命令的定义 
     define command{
             command_name    notify-host-by-email             #命令名称，即定义了一个主机异常时发送邮件的命令。
             command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$                                     #命令具体的执行方式。
             }
     #notify-service-by-email命令的定义 
     define command{
             command_name    notify-service-by-email          #命令名称，即定义了一个服务异常时发送邮件的命令
             command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
             }
     #check-host-alive命令的定义
     define command{
             command_name    check-host-alive                 #命令名称，用来检测主机状态。
             command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w 3000.0,80% -c 5000.0,100% -p 5             
                             # 这里的变量$USER1$在resource.cfg文件中进行定义，即$USER1$=/usr/local/nagios/libexec;
                             # 那么check_ping的完整路径为/usr/local/nagios/libexec/check_ping;
                             # “-w 3000.0,80%”中“-w”说明后面的一对值对应的是“WARNING”状态，“80%”是其临界值。
                             # “-c 5000.0,100%”中“-c”说明后面的一对值对应的是“CRITICAL”，“100%”是其临界值。
                             # “-p 1”说明每次探测发送一个包。
             }
     define command{
             command_name    check_local_disk
             command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$            #$ARG1$是指在调用这个命令的时候，命令后面的第一个参数。
             }
     define command{
             command_name    check_local_load
             command_line    $USER1$/check_load -w $ARG1$ -c $ARG2$
             }
     define command{
             command_name    check_local_procs
             command_line    $USER1$/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$
             }
     define command{
             command_name    check_local_users
             command_line    $USER1$/check_users -w $ARG1$ -c $ARG2$
             }
     define command{
             command_name    check_local_swap
             command_line    $USER1$/check_swap -w $ARG1$ -c $ARG2$
             }
     define command{
             command_name    check_ftp
             command_line    $USER1$/check_ftp -H $HOSTADDRESS$ $ARG1$
             }
     define command{
             command_name    check_http
             command_line    $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
             }
     define command{
             command_name    check_ssh
             command_line    $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
             }
     define command{
             command_name    check_ping
             command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w $ARG1$ -c $ARG2$ -p 5
             }
     define command{
             command_name    check_nt
             command_line    $USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -v $ARG1$ $ARG2$
             }
     ```

     localhost.cfg

     ```cfg
     define host{
             use                     linux-server            ; Name of host template to use
                                                             ; This host definition will inherit all variables that are defined
                                                             ; in (or inherited by) the linux-server host template definition.
             host_name               Nagios-Server
             alias                   Nagios-Server
             address                 127.0.0.1
             }
     define hostgroup{
             hostgroup_name  linux-servers ; The name of the hostgroup
             alias           Linux Servers ; Long name of the group
             members         Nagios-Server ; Comma separated list of hosts that belong to this group
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             PING
             check_command                   check_ping!100.0,20%!500.0,60%
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             Root Partition
             check_command                   check_local_disk!20%!10%!/
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             Current Users
             check_command                   check_local_users!20!50
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             Total Processes
             check_command                   check_local_procs!250!400!RSZDT
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             Current Load
             check_command                   check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             Swap Usage
             check_command                   check_local_swap!20!10
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             SSH
             check_command                   check_ssh
             notifications_enabled           0
             }
     define service{
             use                             local-service         ; Name of service template to use
             host_name                       Nagios-Server
             service_description             HTTP
             check_command                   check_http
             notifications_enabled           0
             }
     ```

     windows.cfg

     ```cfg
     define host{
             use             windows-server  ; Inherit default values from a template
             host_name       Nagios-Windows  ; The name we're giving to this host
             alias           My Windows Server       ; A longer name associated with the host
             address         192.168.1.113   ; IP address of the host
             }
     define hostgroup{
             hostgroup_name  windows-servers ; The name of the hostgroup
             alias           Windows Servers ; Long name of the group
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     NSClient++ Version
             check_command           check_nt!CLIENTVERSION
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     Uptime
             check_command           check_nt!UPTIME
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     CPU Load
             check_command           check_nt!CPULOAD!-l 5,80,90
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     Memory Usage
             check_command           check_nt!MEMUSE!-w 80 -c 90
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     C:\ Drive Space
             check_command           check_nt!USEDDISKSPACE!-l c -w 80 -c 90
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     W3SVC
             check_command           check_nt!SERVICESTATE!-d SHOWALL -l W3SVC
             }
     define service{
             use                     generic-service
             host_name               Nagios-Windows
             service_description     Explorer
             check_command           check_nt!PROCSTATE!-d SHOWALL -l Explorer.exe
             }
     ```

      cgi.cfg

     ```cfg
     default_user_name=david
     authorized_for_system_information=nagiosadmin,david  
     authorized_for_configuration_information=nagiosadmin,david  
     authorized_for_system_commands=david
     authorized_for_all_services=nagiosadmin,david  
     authorized_for_all_hosts=nagiosadmin,david
     authorized_for_all_service_commands=nagiosadmin,david  
     authorized_for_all_host_commands=nagiosadmin,david
     ```

     nagios.cfg

     ```cfg
     log_file=/usr/local/nagios/var/nagios.log                  # 定义nagios日志文件的路径
     cfg_file=/usr/local/nagios/etc/objects/commands.cfg        # “cfg_file”变量用来引用对象配置文件，如果有更多的对象配置文件，在这里依次添加即可。
     cfg_file=/usr/local/nagios/etc/objects/contacts.cfg
     cfg_file=/usr/local/nagios/etc/objects/hosts.cfg
     cfg_file=/usr/local/nagios/etc/objects/services.cfg
     cfg_file=/usr/local/nagios/etc/objects/timeperiods.cfg
     cfg_file=/usr/local/nagios/etc/objects/templates.cfg
     cfg_file=/usr/local/nagios/etc/objects/localhost.cfg       # 本机配置文件
     cfg_file=/usr/local/nagios/etc/objects/windows.cfg         # windows 主机配置文件
     object_cache_file=/usr/local/nagios/var/objects.cache      # 该变量用于指定一个“所有对象配置文件”的副本文件，或者叫对象缓冲文件
     precached_object_file=/usr/local/nagios/var/objects.precache
     resource_file=/usr/local/nagios/etc/resource.cfg           # 该变量用于指定nagios资源文件的路径，可以在nagios.cfg中定义多个资源文件。
     status_file=/usr/local/nagios/var/status.dat               # 该变量用于定义一个状态文件，此文件用于保存nagios的当前状态、注释和宕机信息等。
     status_update_interval=10                                  # 该变量用于定义状态文件（即status.dat）的更新时间间隔，单位是秒，最小更新间隔是1秒。
     nagios_user=nagios                                         # 该变量指定了Nagios进程使用哪个用户运行。
     nagios_group=nagios                                        # 该变量用于指定Nagios使用哪个用户组运行。
     check_external_commands=1                                  # 该变量用于设置是否允许nagios在web监控界面运行cgi命令;
                                                                # 也就是是否允许nagios在web界面下执行重启nagios、停止主机/服务检查等操作;
                                                                # “1”为运行，“0”为不允许。
     command_check_interval=10s                                 # 该变量用于设置nagios对外部命令检测的时间间隔，如果指定了一个数字加一个"s"(如10s);
                                                                # 那么外部检测命令的间隔是这个数值以秒为单位的时间间隔;
                                                                # 如果没有用"s"，那么外部检测命令的间隔是以这个数值的“时间单位”的时间间隔。
     interval_length=60                                         # 该变量指定了nagios的时间单位，默认值是60秒，也就是1分钟;
                                                                # 即在nagios配置中所有的时间单位都是分钟。
     ```

     ​

     timeperiods.cfg

     ```cfg
     #下面是定义一个名为24x7的时间段，即监控所有时间段  
     define timeperiod{  
             timeperiod_name 24x7       #时间段的名称,这个地方不要有空格
             alias           24 Hours A Day, 7 Days A Week  
             sunday          00:00-24:00  
             monday          00:00-24:00  
             tuesday         00:00-24:00  
             wednesday       00:00-24:00  
             thursday        00:00-24:00  
             friday          00:00-24:00  
             saturday        00:00-24:00  
             }  
     #下面是定义一个名为workhours的时间段，即工作时间段。  
     define timeperiod{  
             timeperiod_name workhours   
             alias           Normal Work Hours  
             monday          09:00-17:00  
             tuesday         09:00-17:00  
             wednesday       09:00-17:00  
             thursday        09:00-17:00  
             friday          09:00-17:00  
             }
     ```

     ​

     host.cfg

     ```cfg
     define host{   
             use                     linux-server          #引用主机linux-server的属性信息，linux-server主机在templates.cfg文件中进行了定义。
             host_name               Nagios-Linux          #主机名
             alias                   Nagios-Linux          #主机别名
             address                 192.168.1.111         #被监控的主机地址，这个地址可以是ip，也可以是域名。
             }   
     #定义一个主机组   
     define hostgroup{      
             hostgroup_name          bsmart-servers        #主机组名称，可以随意指定。
             alias                   bsmart servers        #主机组别名
             members                 Nagios-Linux          #主机组成员，其中“Nagios-Linux”就是上面定义的主机。     
             }
     ```

      services.cfg

     ```cfg
     define service{  
             use                     local-service          #引用local-service服务的属性值，local-service在templates.cfg文件中进行了定义。
             host_name               Nagios-Linux           #指定要监控哪个主机上的服务，“Nagios-Server”在hosts.cfg文件中进行了定义。
             service_description     check-host-alive       #对监控服务内容的描述，以供维护人员参考。
             check_command           check-host-alive       #指定检查的命令。
             }
     ```

      contacts.cfg

     ```cfg
     define contact{
             contact_name                    David             #联系人的名称,这个地方不要有空格
             use                             generic-contact   #引用generic-contact的属性信息，其中“generic-contact”在templates.cfg文件中进行定义
             alias                           Nagios Admin
             email                           david.tang@bsmart.cn
             }

     define contactgroup{
             contactgroup_name       ts                              #联系人组的名称,同样不能空格
             alias                   Technical Support               #联系人组描述
             members                 David                           #联系人组成员，其中“david”就是上面定义的联系人，如果有多个联系人则以逗号相隔
             }
     ```

  5. 验证配置

     ```shell
     /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
     ```

- 新增从机监控

  1. 安装nrpe

     ```shell
     tar zxvf nrpe-2.13.tar.gz
     cd nrpe-2.13
     ./configure

     make all

     # 安装check_nrpe 
     make install-plugin
     ```

  2. 测试连接

     ```shell
     /usr/local/nagios/libexec/check_nrpe -H 192.192.192.192 被监控机IP
     ....result....
     NRPE v2.13
     ```

  3. 在commands.cfg中增加对check_nrpe的定义

     ```shell
     vi /usr/local/nagios/etc/objects/commands.cfg

     # 'check_nrpe' command definition
     define command{
             command_name    check_nrpe         # 定义命令名称为check_nrpe,在services.cfg中要使用这个名称.
             command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$       #这是定义实际运行的插件程序.
                             # 这个命令行的书写要完全按照check_nrpe这个命令的用法,不知道用法的就用check_nrpe –h查看.
             }
     ```

  4. 定义监控 services.cfg 

     ```shell
     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Current Load
             check_command           check_nrpe!check_load
             }

     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Check Disk sda1
             check_command           check_nrpe!check_sda1
             }

     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Total Processes
             check_command           check_nrpe!check_total_procs
             }

     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Current Users
             check_command           check_nrpe!check_users
             }

     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Check Zombie Procs
             check_command           check_nrpe!check_zombie_procs
             }
     ```

  5. 添加未定义监控

     现在我们要监控swap 分区，如果空闲空间小于20%则为警告状态 -> warning；如果小于10%则为严重状态 -> critical。我们可以查得需要使用check_swap插件，完整的命令行应该是下面这样。

     ```shell
     /usr/local/nagios/libexec/check_swap -w 20% -c 10%
     ```


     ```

     在**被监控机**（Nagios-Linux）上增加check_swap 命令的定义

     ```shell
     vi /usr/local/nagios/etc/nrpe.cfg
    
     # 增加下面这一行
    
     command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
    
     # 监控 http
     command[check_http]=/usr/local/nagios/libexec/check_http -I 127.0.0.1
     ```
    
     在 **监控机**（Nagios-Server）上增加这个check_swap 监控项目
    
     ```shell
     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     Check Swap
             check_command           check_nrpe!check_swap
             }
             
     define service{
             use                     local-service
             host_name               Nagios-Linux
             service_description     HTTP
             check_command           check_nrpe!check_http
             }
     ```
    
      **重启nagios服务**
    
     ​

- Nagios启动与停止

  1.  启动Nagios

  ```shell
  # /etc/init.d/nagios start
  or
  # service nagios start
  or
  /usr/local/nagios/bin/nagios -d /usr/local/nagios/etc/nagios.cfg
  ```

  2. 重启Nagios

  ```shell
  # /etc/init.d/nagios reload
  or
  # /etc/init.d/nagios restart
  or
  # service nagios restart

  #  通过web监控页重启nagios
  Process Info -> Restart the Nagios process

  # 手工方式平滑重启
  ps aux|grep nagios
  or
  ps -ef|grep nagios

  kill -HUP <nagios_pid>
  ```

  3. 停止Nagios

  ```shell
  # /etc/init.d/nagios stop
  or
  # service nagios stop

  # 通过web监控页停止nagios
  Process Info -> Shutdown the Nagios process

  # 手工方式停止Nagios
  ps aux|grep nagios
  or
  ps -ef|grep nagios

  kill <nagios_pid>
  ```

  ​

##被监控机安装依赖

- 依赖安装

  ```shell
  #检测依赖安装状态
  rpm -q gcc glibc glibc-common gd gd-devel xinetd openssl-devel
  #安装依赖
  yum install -y gcc glibc glibc-common gd gd-devel xinetd openssl-devel
  ```

- 创建监控用户，用户组，指定安装目录，权限处理

  ```shell
  # 新增用户
  useradd -s /sbin/nologin nagios
  # 创建安装目录
  mkdir /usr/local/nagios
  # 修改目录归属用户
  chown -R nagios.nagios /usr/local/nagios
  # 查看nagios 目录的权限
  ll -d /usr/local/nagios/
  ```

- 下载必备软件

  ```shell
  # 下载nagios-plugin
  wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
  # 下载nrpe
  wget http://prdownloads.sourceforge.net/sourceforge/nagios/nrpe-2.13.tar.gz
  ```

- 安装nagios-plugin

  ```shell
  # 编译安装
  tar zxvf nagios-plugins-2.2.1.tar.gz
  cd nagios-plugins-1.4.16
  ./configure --prefix=/usr/local/nagios
  make && make install
  cd ..

  # 修改权限
  chown nagios.nagios /usr/local/nagios
  chown -R nagios.nagios /usr/local/nagios/libexec
  ```

- 安装NRPE

  ```shell
  tar zxvf nrpe-2.13.tar.gz
  cd nrpe-2.13
  ./configure

  make all

  # 安装check_nrpe 
  make install-plugin

  # 安装deamon
  make install-daemon

  # 安装配置文件
  make install-daemon-config

  # 安装xinted 脚本
  make install-xinetd
  ```

- 配置NRPE

  ```shell
  vim /etc/xinetd.d/nrpe

  .....before......
         only_from       = 127.0.0.1
  .....after.......
         only_from       = 127.0.0.1 监控主机ip 120.120.120.120
  ```

- 配置网络属性

  ```shell
  vim /etc/services

  ......before......
  matahari        49000/tcp               # Matahari Broker
  ......after......
  matahari        49000/tcp               # Matahari Broker
  # nagios
  nrpe            5666/tcp                # nrpe
  ```

- 启动，并查看nrpe是否启动成功

  ```shell
  # 启动
  service xinetd restart
  # 查看启动
  netstat -an|grep 5666
  ....result....
  tcp6       0      0 :::5666                 :::*                    LISTEN 

  # 测试运行
  /usr/local/nagios/libexec/check_nrpe -H localhost
  ....result....
  NRPE v2.13

  # check_nrpe 用法
  # 注意：-c 后面接的监控命令必须是nrpe.cfg 文件中定义的。也就是NRPE daemon只运行nrpe.cfg中所定义的命令。
  check_nrpe –H 被监控的主机 -c 要执行的监控命令
  ```

- 配置nrpe监控命令

  ```shell
  cd /usr/local/nagios/etc
  cat nrpe.cfg |grep -v "^#"|grep -v "^$"
  ....result....
  log_facility=daemon
  pid_file=/var/run/nrpe.pid
  server_port=5666
  nrpe_user=nagios
  nrpe_group=nagios
  allowed_hosts=127.0.0.1
   
  dont_blame_nrpe=0
  debug=0
  command_timeout=60
  connection_timeout=300
  command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
  command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
  command[check_sda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1
  command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
  command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200 
  ....result....

  #新增 command
  #代理 内容
  ```

##分布式处理

- 主Nagios中心服务部署

  1. 安装nsca

     ```shell
     wget http://download.chekiang.info/nagios/nsca-2.7.2.tar.gz
     tar zxvf nsca-2.9.1.tar.gz
     cd nsca-2.7.2
     ./configure
     make all
     ```


  2. 修改nsca的配置文件

     ```shell
     vi /usr/local/nagios/etc/nsca.cfg

     password=123456
     ```

  3. 修改nagios的配置文件

     ```shell
     vi /usr/local/nagios/etc/nagios.cfg

     check_external_commands=1 # 配置nagios检查扩展命令
     accept_passive_service_checks=1 # 配置接受被动服务检测的结果
     accept_passive_host_checks=1 #配置接受被动主机检测的结果
     ```

  4. 检查没问题后重启nagios和启动nsca 

- 分布服务器部署

  1. 安装nsca-send

     ```shell
     wget http://download.chekiang.info/nagios/nsca-2.7.2.tar.gz
     tar zxvf nsca-2.9.1.tar.gz
     cd nsca-2.7.2
     ./configure 
     make all

     # 以上步骤检查正确执行以后会在src目录下生成两个程序 
     # nsca send_nsca（主程序），sample-config中会有nsca.cfg与send_nsca.cfg（配置文件）。
     ```


  2. 修改配置文件

     ```shell
     cp src/send_nsca /usr/local/nagios/bin/
     cp sample-config/send_nsca.cfg /usr/local/nagios/etc/
     chown nagios.nagios /usr/local/nagios/bin/send_nsca
     chown nagios.nagios /usr/local/nagios/etc/send_nsca.cfg
     # 修改send_nsca.cfg配置，改下密码。

     vi /usr/local/nagios/etc/send_nsca.cfg
     # password=123456
     # 这里的密码需要和主的一样。
     ```

  3. 修改nagios配置文件

     ```shell
     enable_notifications=0 //阻止它直接送出任何通知信息 
     obsess_over_services=1 // 配置为强迫型服务(obsess over services)类型 
     ocsp_command=submit_service_check_result //定义一个强迫型服务处理(ocsp)命令 
     obsess_over_hosts=1 // 配置为强迫型服务(obsess over host)类型 
     ochp_command=submit_host_check_result //定义一个强迫型主机处理(ochp)命令
     ```

  4. 添加ocsp命令

     ```shell
     # vi command.cfg
     define command{
         command_name submit_service_check_result
         command_line $USER1$/eventhandlers/submit_service_check_result $HOSTNAME$ '$SERVICEDESC$' $SERVICESTATE$ '$SERVICEOUTPUT$ | $SERVICEPERFDATA$ [$SERVICECHECKCOMMAND$]'
     }
     define command{
         command_name submit_host_check_result
         command_line $USER1$/eventhandlers/submit_host_check_result $HOSTNAME$ $HOSTSTATE$ '$HOSTOUTPUT$'
     }
     ```

  5. 添加命令脚本

     ```shell
     mkdir /usr/local/nagios/libexec/eventhandlers
     cd /usr/local/nagios/libexec/eventhandlers
     wget https://raw.githubusercontent.com/zhangnq/nagios/master/setup/submit_host_check_result
     wget https://raw.githubusercontent.com/zhangnq/nagios/master/setup/submit_service_check_result
     chmod +x /usr/local/nagios/libexec/eventhandlers/submit_host_check_result
     chmod +x /usr/local/nagios/libexec/eventhandlers/submit_service_check_result
     # 修改submit_host_check_result和submit_service_check_result两个脚本中的中心nagios监控主机ip或域名，即修改www.chekiang.info为你自己的地址。
     ```

  6. 检查没问题后有启动nagios。

- 添加监控主机和服务

  1. 注意事项

     - 中心服务器和分布服务器都需要添加监控的主机和服务。
     - 中心服务器主机定义的host_name值需要和分布服务器主机定义的host_name值一致。
     - 中心服务器服务定义的service_description值需要和分布服务器服务定义的service_description 值一致。

  2. 分布服务器配置

     分布服务器的配置增加和主动监控一样，先添加host，然后添加server即可。

  3. 中心服务器

     - 增加passive模式的主机和服务模板

       修改templates.cfg 文件，增加类似如下内容：

       ```shell
       # vi /usr/local/nagios/etc/objects/templates.cfg
       define host{
               name                            passive-host
               use                             generic-host
               check_period                    24x7
               check_interval                  5
               retry_interval                  1
               max_check_attempts              10
               check_command                   check-host-alive
               notification_period             24x7
               notification_interval           60
               notification_options            d,u,r
               contact_groups                  sysmaint
               register                        0
               check_freshness                 1  ;定义强制刷新检测
               freshness_threshold             600  ;指定服务检测结果应该在何时间内刷新，单位是s
               passive_checks_enabled          1  ;打开被动检测
               active_checks_enabled           0  ;关闭主动监测
       }
       define service{
               name                            passive-service
               use                             generic-service
               active_checks_enabled           0  ;关闭主动监测
               passive_checks_enabled          1  ;打开被动检测
               flap_detection_enabled          0  ;关闭状态抖动检测
               check_freshness                 1  ;定义强制刷新检测
               freshness_threshold             600  ;指定服务检测结果应该在何时间内刷新，单位是s
               max_check_attempts              4
               normal_check_interval           5
               retry_check_interval            1
               register                        0
               check_command                   service-is-stale ;定义强制检测的执行命令
       }
       ```

     - 增加强制检测命令

       ```shell
       # vi /usr/local/nagios/libexec/staleservice.sh
       #!/bin/bash

       /bin/echo "CRITICAL: Service results are stale!"
       exit 2
       # chmod +x /usr/local/nagios/libexec/staleservice.sh
       添加命令

       # vi /usr/local/nagios/etc/objects/commands.cfg
       define command{
               command_name    service-is-stale
               command_line    $USER1$/staleservice.sh
       }
       ```

     - 添加主机和服务

       复制分布服务器中的hosts和servers到nagios中心服务器中，修改定义主机中的use模板为定义的passive-host，之前默认一般为linux-server，修改定义服务器中的use模板为定义的passive-service，取消check_command值的定义。类似如下：

       ```shell
       define host{
               use             passive-host
               host_name       10.22.127.100
               alias           10.22.127.100
               address         10.22.127.100
       }
       define service{
               use                     passive-service,srv-pnp
               host_name               10.22.127.100
               service_description     check ssh login
       }
       ```

     - 检查重启

       添加完成后检查nagios配置是否正确，然后重启。

     - 日志

       日志一般是在/usr/local/nagios/var/nagios.log中，正常的话显示如下，如果配置有问题也可以通过日志查找原因。

  ​