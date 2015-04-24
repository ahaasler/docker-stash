# docker-stash: A Docker image for Stash.

[![Release](https://img.shields.io/github/release/ahaasler/docker-stash.svg?style=flat)](https://github.com/ahaasler/docker-stash/releases/latest)

## Features

* Runs on a production ready *OpenJDK* 8 - [Zulu](http://www.azulsystems.com/products/zulu "Zulu: Multi-platform Certified OpenJDK") by Azul Systems.
* Ready to be configured with *Nginx* as a reverse proxy (https available).
* Built on top of *Debian* for a minimal image size.

## Usage

```bash
docker run -d -p 7990:7990 -p 7999:7999 ahaasler/stash
```

### Parameters

You can use this parameters to configure your stash instance:

* **-s:** Enables the connector security and sets `https` as connector scheme.
* **-n &lt;proxyName&gt;:** Sets the connector proxy name.
* **-p &lt;proxyPort&gt;:** Sets the connector proxy port.
* **-c &lt;contextPath&gt;:** Sets the context path (do not write the initial /).

This parameters should be given to the entrypoint (passing them after the image):

```bash
docker run -d -p 7990:7990 -p 7999:7999 ahaasler/stash <parameters>
```

> If you want to execute another command instead of launching stash you should overwrite the entrypoint with `--entrypoint <command>` (docker run parameter).

### Nginx as reverse proxy

Lets say you have the following *nginx* configuration for stash:

```
server {
	listen                          80;
	server_name                     example.com;
	return                          301 https://$host$request_uri;
}
server {
	listen                          443;
	server_name                     example.com;

	ssl                             on;
	ssl_certificate                 /path/to/certificate.crt;
	ssl_certificate_key             /path/to/key.key;
	location /stash {
		proxy_set_header X-Forwarded-Host $host;
		proxy_set_header X-Forwarded-Server $host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://127.0.0.1:7990;
		proxy_redirect off;
	}
}
```

> This is only an example, please secure you *nginx* better.

For that configuration you should run your stash container with:

```bash
docker run -d -p 7990:7990 -p 7999:7999 ahaasler/stash -s -n example.com -p 443 -c stash
```

### Persistent data

The stash home is set to `/data/stash`. If you want to persist your data you should use a data volume for `/data/stash`.

#### Binding a host directory

```bash
docker run -d -p 7990:7990 -p 7999:7999 -v /home/user/stash-data:/data/stash ahaasler/stash
```

Make sure that the stash user (with id 782) has read/write/execute permissions.

If security is important follow the Atlassian recommendation:

> Ensure that only the user running Stash can access the Stash home directory, and that this user has read, write and execute permissions, by setting file system permissions appropriately for your operating system.

#### Using a data-only container

1. Create the data-only container and set proper permissions:

	* **Lazy way (preferred)** - Using [docker-stash-data](https://github.com/ahaasler/docker-stash-data "A data-only container for docker-stash"):

		```bash
docker run --name stash-data ahaasler/stash-data
		```

	* *I-wan't-to-know-what-I'm-doing* way:

		```bash
docker run --name stash-data -v /data/stash busybox true
docker run --rm -it --volumes-from stash-data debian bash
		```

		The last command will open a *debian* container. Execute this inside that container:

		```bash
chown 782:root /data/stash; chmod 770 /data/stash; exit;
		```

2. Use it in the stash container:

	```bash
docker run --name stash --volumes-from stash-data -d -p 7990:7990 -p 7999:7999 ahaasler/stash
	```

### PostgreSQL external database

A great way to connect your Stash instance with a PostgreSQL database is
using the [docker-stash-postgres](https://github.com/ahaasler/docker-stash-postgres "A PostgreSQL container for docker-stash")
image.

1. Create and name the database container:

	```bash
docker run --name stash-postgres -d ahaasler/stash-postgres
	```

2. Use it in the Stash container:

	```bash
docker run --name stash --link stash-postgres:stash-postgres -d -p 7990:7990 -p 7999:7999 ahaasler/stash
	```

3. Connect your Stash instance following the Atlassian documentation:
[Connecting Stash to PostgreSQL](https://confluence.atlassian.com/display/STASH/Connecting+Stash+to+PostgreSQL#ConnectingStashtoPostgreSQL-ConnectStashtothePostgreSQLdatabase "Connecting Stash to PostgreSQL").

>  See [docker-stash-postgres](https://github.com/ahaasler/docker-stash-postgres "A PostgreSQL container for docker-stash")
for more information an configuration options.

## Thanks

* [Docker](https://www.docker.com/ "Docker") for this amazing container engine.
* [PostgreSQL](http://www.postgresql.org/) for this advanced database.
* [Atlassian](https://www.atlassian.com/ "Atlassian") for making great products. Also for their work on [atlassian-docker](https://bitbucket.org/atlassianlabs/atlassian-docker "atlassian-docker repo") which inspired this.
* [Azul Systems](http://www.azulsystems.com/ "Azul Systems") for their *OpenJDK* docker base image.
* And specially to you and the entire community.

## License

This image is licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full license text.
