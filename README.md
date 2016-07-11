# monit-in-docker
base on ubuntu image

monit:
	/etc/monit/monitrc: allow admin:admin      # require user 'admin' with password 'monit'

	nginxmonitor:
		check process nginx with pidfile /var/run/nginx.pid
		   group www
		   group nginx
		   start program = "/etc/init.d/nginx start"
		   stop program = "/etc/init.d/nginx stop"
		#  if failed port 80 protocol http request "/" then restart
		   if 5 restarts with 5 cycles then timeout
		   depend nginx_bin
		   depend nginx_rc

		check file nginx_bin with path /usr/sbin/nginx
		   group nginx
		   include /etc/monit/templates/rootbin

		check file nginx_rc with path /etc/init.d/nginx
		   group nginx
		   include /etc/monit/templates/rootbin

	postgresqlmonitor:
		check process postgres with pidfile /var/run/postgresql/9.4-main.pid
		   group database
		   start program = "/etc/init.d/postgresql start"
		   stop  program = "/etc/init.d/postgresql stop"
		   if failed unixsocket /var/run/postgresql/.s.PGSQL.5432 protocol pgsql
		      then restart
		   if failed host localhost port 5432 protocol pgsql then restart

	uwsgimonitor:
		check process uwsgi with pidfile /var/run/uwsgi.pid
		  start program = "/usr/local/bin/uwsgi --emperor /var/www/html/stockmaster/conf/uwsgi.ini --daemonize /var/run/uwsgidaemon --pidfile /var/run/uwsgi.pid"
		  stop program = "/usr/local/bin/uwsgi --stop /var/run/uwsgi.pid"
		  restart program = "/usr/local/bin/uwsgi --reload /var/run/uwsgi.pid"
		  group nginx

	vsftpdmonitor
		check process vsftpd with pidfile /var/run/vsftpd/vsftpd.pid
		  start program = "/etc/init.d/vsftpd start"
		  stop program = "/etc/init.d/vsftpd stop"
		  if failed port 21 protocol ftp then restart
		  if 5 restarts within 5 cycles then timeout
