

# Laravel Stone - HTTP server using the  <a href="https://reactphp.org" target="_blank">ReactPHP</a>



With ReactPHP we keep our application alive between requests so we only execute this bootstrap once when starting the server: the footprint is absent from Requests.
However now the tables are turned: we're vulnerable to memory consumption, fatal error, statefulness and code update worries.

## Installation

Installer using Composer:

```bash
$ composer require orchid/stone
```

## Example


The easiest way to run:

```bash
$ php vendor/bin/stone
```

You can also specify a port:

```bash
$ php vendor/bin/stone -p 8000
```

It does not interfere with your application, so do not worry

## Making production ready

### Supervisord

Install <a href="http://supervisord.org/" target="_blank">Supervisord</a>

```bash
$ sudo apt-get install -y supervisor
```

Here's a configuration example (create a `*.conf` file in `/etc/supervisord/conf.d`):


```bash
[program:laravel-stone-server]
command=php /path/to/your/code/vendor/bin/stone -p 800%(process_num)
process_name=%(program_name)s-%(process_num)d
numprocs=4
umask=022
user=foobar
stdout_logfile=/var/log/supervisord/%(program_name)s-%(process_num)d.log              ; stdout log path, NONE for none; default AUTO
stderr_logfile=/var/log/supervisord/%(program_name)s-%(process_num)d-error.log        ; stderr log path, NONE for none; default AUTO
autostart=true
autorestart=true
startretries=3
```

It will:

- run 4 servers on ports 8000, 8001, 8002 and 8003
- it restarts them automatically when they crash (will try a maximum of 3 times, then give up)

### Nginx as Load-Balancer

What we do here is to proxy only requests that don’t point to a local file.

```bash
upstream stone  {
    server 127.0.0.1:8000;
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}


server {
    listen 80;
    server_name example.com;
    root /example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        proxy_pass http://stone;
        break;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Donate & Support

If you would like to support development by making a donation you can do so [here](https://www.paypal.me/tabuna/10usd). &#x1F60A;

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

---

> [orchid.software](https://orchid.software) &nbsp;&middot;&nbsp;
> GitHub [@tabuna](https://github.com/tabuna) &nbsp;&middot;&nbsp;
> Twitter [@orchid_platform](https://twitter.com/orchid_platform)
