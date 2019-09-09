### Setting up HLS live streaming server using NGINX + nginx-rtmp-module on Ubuntu

# Compile nginx with rtmp module
#### **Clone nginx-rtmp-module**

     git clone https://github.com/sergey-dryabzhinsky/nginx-rtmp-module.git

#### **Install nginx dependencies**

    sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev

#### **Download NGINX**

    wget http://nginx.org/download/nginx-1.10.1.tar.gz
    tar -xf nginx-1.10.1.tar.gz
    cd nginx-1.10.1`

#### **Compile nginx**

	./configure --with-http_ssl_module --add-module=../nginx-rtmp-module
	make
	sudo make install

# Create nginx configuration file

#### **the complete nginx.conf**

> The default location for nginx conf is /usr/local/nginx/conf/nginx.conf or /etc/nginx/nginx.conf

		worker_processes  auto;
	events {
		worker_connections  1024;
			}
	
	rtmp {
		server {
			listen 1935; # Listen on standard RTMP port
			chunk_size 4000;

        application show {
            live on;
            # Turn on HLS
            hls on;
            hls_path /mnt/hls/;
            hls_fragment 3;
            hls_playlist_length 60;
            # disable consuming the stream from nginx as rtmp
            deny play all;
      	  }
    	}
	}

		http {
		 sendfile off;
		 tcp_nopush on;
			#aio on;
			directio 512;
			default_type application/octet-stream;

			server {
				listen 8080;

			location / {
				# Disable cache
				add_header 'Cache-Control' 'no-cache';

				# CORS setup
				add_header 'Access-Control-Allow-Origin' '*' always;
				add_header 'Access-Control-Expose-Headers' 'Content-Length';

				# allow CORS preflight requests
				if ($request_method = 'OPTIONS') {
					add_header 'Access-Control-Allow-Origin' '*';
					add_header 'Access-Control-Max-Age' 1728000;
					add_header 'Content-Type' 'text/plain charset=UTF-8';
					add_header 'Content-Length' 0;
					return 204;
				}

				types {
					application/dash+xml mpd;
					application/vnd.apple.mpegurl m3u8;
					video/mp2t ts;
				}

				root /mnt/;
			}
		}
	}


# Start nginx

> The nginx binary is located wherever you compiled it to - /usr/local/nginx/sbin/nginx by default. Change it to reflect your path.

###### Test the configuration file
	/usr/local/nginx/sbin/nginx -t
###### Start nginx in the background
	/usr/local/nginx/sbin/nginx
###### Start nginx in the foreground
	/usr/local/nginx/sbin/nginx -g 'daemon off;'
###### Reload the config on the go
	/usr/local/nginx/sbin/nginx -t && nginx -s reload
###### Kill nginx
	/usr/local/nginx/sbin/nginx -s stop


