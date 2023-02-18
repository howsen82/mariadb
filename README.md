# Containerized MariaDB Installation
Start a new container MariaDB database server with automatically created volume
<pre><code>docker container run -d --name mariadb-test -e MYSQL_ROOT_PASSWORD=secret mariadb</code></pre>

To view the status for newly created container
<pre><code>docker container list</code></pre>

To view MariaDB installation logs
<pre><code>docker logs mariadb-test</code></pre>

To stop the container, using command:
<pre><code>docker container stop mariadb-test</code></pre>

The MariaDB stores all databases files inside the volume for container in the <pre><code>*/var/lib/msql*</code></pre> directory. However, in the MariaDB *Dockerfile*, that is in the description of the image, */var/lib/mysql* is marked as a *volume*.

To analyze the container volume details, use command

<pre><code>docker container inspect mariadb-test</code></pre>

View the volume mounts:

<pre><code>docker container inspect -f '{{ .Mounts }}' mariadb-test</code></pre>

To view all the volumes inside the docker

<pre><code>docker volume list</code></pre>

To view volume details, you can lookup using command:

<pre><code>docker volume inspect <64-character-hexadecimal-code></code></pre>

View the container disk spaces, using command:

<pre><code>docker system df -v</code></pre>

To connect to container MariaDB container to execute SQL statements on MariaDB server.

<pre><code>docker exec -it mariadb-test mysql -u root -p
show status;
exit
</code></pre>

Or

<pre><code>docker exec -it mariadb-test /bin/bash

# View the container image OS release
cat /etc/os-release

# View the MariaDB version
mysqld --version

exit</code></pre>

**Tidying Up MariaDB server**
When a container get deleted, the associated volume is always preserved. This is useful because volumes contains data you may need in other containers.

<pre><code>
# Lookup volume hex code
docker container inspect -f '{{ .Mounts }}' mariadb-test

# Stop container
docker container stop mariadb-test
docker rm mariadb-test
docker container volume rm <container-volume-hex-code>
</code></pre>

To delete all the unused container volumes, and this isn't dangerous
<pre><code>docker volume prune</code></pre>

<h4><u>Named Volume Container</u></h4>

Start new MariaDB server container with *named volume*
<pre><code>docker container run -d --name mariadb-test \
    -v myvolume:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret mariadb</code></pre>

View the volumes disk spaces
<pre><code>docker system df -v</code></pre>

**Container update without Data Loss**
The benefits of volumes become clear when you want to perform a container update (e.g., after a new version of the MariaDB image has been released in Docker Hub). To do this, you need to stop and delete the current container

<pre><code>
docker container stop mariadb-test
docker container rm mariadb-test
docker pull mariadb
docker run -d --name mariadb-test \
           -v myvolume:/var/lib/mysql mariadb
</code></pre>

**Tidying Up**
The task of tidying up is alsu much easier with named volumes. You only delete the volume if you no longer need the data it contains.

<pre><code>
docker container stop mariadb-test
docker container rm mariadb-test
docker volume rm myvolume
</code></pre>

<h4><u>Volumes in Custom Directories (Bind Mount)</u></h4>

<pre><code>
mkdir /home/<username>/mysql

# Ubuntu
docker run -d --name mariadb-test \
           -e MYSQL_ROOT_PASSWORD=secret \
           -v /home/<username>/mysql/:/var/lib/mysql mariadb

# RHEL, SELinux, Fedora
docker run -d --name mariadb-test \
           -e MYSQL_ROOT_PASSWORD=secret \
           -v /home/<username>/mysql/:/var/lib/mysql:z mariadb

# Windows on WSL2
docker run -d --name mariadb-test \
           -e MYSQL_ROOT_PASSWORD=secret \
           -v "C:\Users\username\dir":/var/lib/mysql mariadb
</code></pre>

**Tidying Up**
The task of tidying up is alsu much easier with named volumes. You only delete the volume if you no longer need the data it contains.

<pre><code>
docker container stop mariadb-test
docker container rm mariadb-test

sudo rm -rf mysql    # Linuxc, macOS
rmdir /s mysql  # Windows
</code></pre>

# MariaDB Server Network

Setting up a Docker network

<pre><code>docker network create testnet</code></pre>

Start a MariaDB server

<pre><code>docker run -d --name mariadb-test --network testnet \
    -e MYSQL_RANDOM_ROOT_PASSWORD=1 \
    -e MYSQL_DATABASE=wp \
    -e MYSQL_USER=wpuser \
    -e MYSQL_PASSWORD=secret \
    -v myvolume:/var/lib/mysql \
    mariadb
</code></pre>