# according to the value of $http_upgrade, construct and change the value of $connection_upgrade.The value of $connection_upgrade is 'upgrade.If $http_upgrade is an empty string, the value is 'close'.
#
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}


# set up an upstream context called ws_entry , this name will be available for use within proxy passes.
#
upstream ws_entry {
	
	#define the server to 10.1.206.67:10805
	#
    server 10.1.206.67:10805;
}

# set up an upstream context called ws_gate , this name will be available for use within proxy passes.
#
upstream ws_gate {

	#define the server to 10.1.206.67:10806
	#
    server 10.1.206.67:10806;
}
#Sets configuration for a virtual server
#
server {

	# sets the port 10805 for IP and accept on this port work in SSL mode on which the server will accept requests.
	#
    listen      10805 ssl;
	# sets names of a virtual server
	#
    server_name xingli20.com;

	# convert data to charsets utf-8
	#
    charset     utf-8;

	# access log storage path and use log_format main
	#
    access_log  /var/log/nginx/logics_access.log main;
	# error log storage path and warn level . It records messages of warn, error crit, alert, and emerg levels.
	#
    error_log   /var/log/nginx/logics_error.log warn;
	
	# provide the PEM format certificate to virtual server in /etc/nginx/ssl/fullchain.pem 
	#
    ssl_certificate      /etc/nginx/ssl/fullchain.pem;
	# provide the PEM format secret key to the virtual server in /etc/nginx/ssl/privkey.pem
	#
    ssl_certificate_key  /etc/nginx/ssl/privkey.pem;
	#enables the specified protocols TLSv1.2
	#
    ssl_protocols TLSv1.2;
	# specifies that server ciphers should be preferred over client ciphers
	#
    ssl_prefer_server_ciphers on;
	# specifies the enabled ciphers
	#
    ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384";

# sets configuration for location match /
#
    location / {
		# request proxy forwarded to upstream ws_entry 
		#
        proxy_pass http://ws_entry;

		# upgrade http1.1 to websocket protocol
		#
        proxy_http_version 1.1;
		# request server upgrade protocol to WebSocket
		#
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
		# X-Real-IP contains the original IP of the client, which can be captured by $remote_addr
		#
        proxy_set_header X-Real-IP $remote_addr;
		# X-Forwarded-For represents the IP marked in the client header, it can be captured by $ proxy_add_x_forward_for, if the header does not have X-Forwarded-For then it is the same as $ remoter_addr
		#	
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		# configure automatic protocol acquisition method, the client can use http or https protocol
		#
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

#Sets configuration for a virtual server
#
server {

	# sets the port 10806 for IP and accept on this port work in SSL mode on which the server will accept requests.
	#
    listen      10806 ssl;
	# sets names of a virtual server
	#
    server_name xingli20.com;
	
	# convert data to charsets utf-8
	#
    charset     utf-8;

	# access log storage path and use log_format main
	#
    access_log  /var/log/nginx/logics1_access.log main;
	# error log storage path and warn level . It records messages of warn, error crit, alert, and emerg levels.
	#
    error_log   /var/log/nginx/logics1_error.log warn;
	
	# provide the PEM format certificate to virtual server in /etc/nginx/ssl/fullchain.pem 
	#
    ssl_certificate      /etc/nginx/ssl/fullchain.pem;
	# provide the PEM format secret key to the virtual server in /etc/nginx/ssl/privkey.pem
	#
    ssl_certificate_key  /etc/nginx/ssl/privkey.pem;
	
	# enables the specified protocols TLSv1.2
	#
    ssl_protocols TLSv1.2;
	# specifies that server ciphers should be preferred over client ciphers
	#
    ssl_prefer_server_ciphers on;
	#Specifies the enabled ciphers
	#
    ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384";

# sets configuration for location match /
#
    location / {
		# request proxy forwarded to upstream ws_gate 
		#
        proxy_pass http://ws_gate;
		# To turn a connection between a client and server from HTTP/1.1 into WebSocket
		#

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
		# X-Real-IP contains the original IP of the client, which can be captured by $remote_addr
		#
        proxy_set_header X-Real-IP $remote_addr;
		# X-Forwarded-For represents the IP marked in the client header, it can be captured by $ proxy_add_x_forward_for, if the header does not have X-Forwarded-For then it is the same as $ remoter_addr
		#
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		# configure automatic protocol acquisition method, the client can use http or https protocol
		#
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
