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
                proxy_pass http://trac:80;
                proxy_pass_header Authorization;
                proxy_set_header REMOTE_USER $remote_user;
                auth_basic "Restricted";
                auth_basic_user_file htpasswd;
    }

If you just want the authentication for the login page change the above to

    location / {
                proxy_pass http://trac:80;
                proxy_pass_header Authorization;
                proxy_set_header REMOTE_USER $remote_user;
    }
    location ~ /login$ {
                proxy_pass http://trac:80;
                proxy_pass_header Authorization;
                proxy_set_header REMOTE_USER $remote_user;
                auth_basic "Restricted";
                auth_basic_user_file htpasswd;
    }


## See Also
* [Using Nginx as your main web server for multiple Trac projects](https://trac.edgewall.org/wiki/TracNginxRecipe)
* [Running Trac Standalone using tracd](https://trac.edgewall.org/wiki/TracStandalone)
* [Docker Nginx-Proxy](https://github.com/binfalse/nginx-proxy)


