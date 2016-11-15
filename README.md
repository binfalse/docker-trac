# Docker::Trac

This image will create a Docker container running [tracd](https://trac.edgewall.org/wiki/TracStandalone).

It provides a [mount point](https://docs.docker.com/engine/reference/builder/#/volume) at `/trac` and it expects the trac projects in `/trac/projects` and a `/trac/.htpasswd` to authenticate the users.
The tracd webserver will be listening at port 80 of the container.

## Run the tracd

Assuming your trac projects are in `/var/lib/trac/projects` and you created a `.htpasswd` with user credentials in `/var/lib/trac/.htpasswd` you would run the Docker container with

    docker run --name trac --rm -it -p 0.0.0.0:8080:80 -v /var/lib/trac:/trac binfalse/trac:latest

Then you should be able to access the trac at port `8080` of the server's IPv4 address.


## Use Nginx to proxy the tracd

At least in my opinion it makes sense to use a proxy for the tracd webserver.
Nginx can then do the authentication of users and would handle SSL connections etc.
Just add the following snippet to your Nginx' `/etc/nginx/conf.d/default.conf`:

    location / {
                proxy_pass http://trac/;
                proxy_pass_header Authorization;
                proxy_redirect http://trac/ http://trac.binfalse.de/;
                proxy_set_header REMOTE_USER $remote_user;
                auth_basic "Restricted";
                auth_basic_user_file htpasswd;
    }

This requires htpasswd file in `/etc/nginx/htpasswd` and assumes that the tracd server runs at `http://trac/`.
If you just want the authentication for the login page change the above to

    location / {
                proxy_pass http://trac:80;
                proxy_pass_header Authorization;
                proxy_set_header REMOTE_USER $remote_user;
                proxy_redirect http://trac/ http://trac.binfalse.de/;
    }
    location ~ /login$ {
                proxy_pass http://trac:80;
                proxy_pass_header Authorization;
                proxy_redirect http://trac/ http://trac.binfalse.de/;
                proxy_set_header REMOTE_USER $remote_user;
                auth_basic "Restricted";
                auth_basic_user_file htpasswd;
    }


## Docker Compose

A Docker compose configuration for a **trac** container, and an **Nginx** container, and an **Nginx-Proxy** container may look like this:


    version: '2'
    services:
        nginx-proxy:
            restart: always
            image: binfalse/nginx-proxy
            ports:
                - "80:80"
                - "443:443"
            volumes:
                - /var/run/docker.sock:/tmp/docker.sock:ro
                - /srv/web/certs:/etc/nginx/certs:ro
            environment:
                DEFAULT_HOST: binfalse.de
        nginx-trac:
            restart: always
            image: nginx
            volumes:
                - /srv/docker/configs/nginx-site-trac.conf:/etc/nginx/conf.d/default.conf:ro
                - /srv/repositories/trac/.htpasswd:/etc/nginx/htpasswd:ro
            environment:
                - VIRTUAL_HOST=trac.binfalse.de
            links:
                - trac
        trac:
            restart: always
            image: binfalse/trac
            volumes:
                - /srv/repositories/trac/:/trac
                - /srv/repositories/git/repositories/:/trac/git/:ro


The above configuration expects:

* SSL certificates in `/srv/web/certs`
* The trac-Nginx configuration in `/srv/docker/configs/nginx-site-trac.conf` (see above on how to configure Nginx)
* The htpasswd file in `/srv/repositories/trac/.htpasswd`
* The trac repositories in `/srv/repositories/trac/projects`
* The corresponding git repositories in `/srv/repositories/git/repositories/` (but that just depends on how you configure your trac environment)


## See Also
* [Using Nginx as your main web server for multiple Trac projects](https://trac.edgewall.org/wiki/TracNginxRecipe)
* [Running Trac Standalone using tracd](https://trac.edgewall.org/wiki/TracStandalone)
* [Docker Nginx-Proxy](https://github.com/binfalse/nginx-proxy)


## License

    Docker Container for running trac projects
    Copyright (C) 2014-2016: Martin Scharm <martin@binfalse.de>
    
    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.
    
    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.
    
    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.

