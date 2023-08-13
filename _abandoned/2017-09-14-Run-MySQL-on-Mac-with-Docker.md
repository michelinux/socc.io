---
# layout: post
title:  "Run MySQL on Mac with Docker"
date:   2017-09-14 08:22:00 +0200
# categories: docker macOS
# tags: docker macOS mac mysql
published: false
---

I don't like clutter and I don't like to install too many things on my computer. I am confident that Docker can save me. Should you still have to install Docker, have a quick look at https://docs.docker.com/docker-for-mac/install/. 

Installing a MySQL to do some quick experiment has never been easier:

```sh
docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:latest
```

Quick recap of the options:

* `-e` Set environment variables
* `-d` Run container in background
* `--name` Assign a name to the container
* `-p` Let the external world connect to the container. By default containers cannot be reached from the external and `3306` is the default port of MySQL.

When running this command you will download and start the *latest* version of the image `mysql` with `"password"` set as `root` password. You will be able to connect to the container from the external world via the port `3306`. The container is named `some-mysql`. You can verify with:
```sh
docker ps
```

```
CONTAINER ID  IMAGE         COMMAND                 PORTS                      NAMES
a335b0d1a4c3  mysql:latest  "docker-entrypoint..."  0.0.0.0:3306->3306/tcp     some-mysql
```

Now one may want to create a get inside the database and create a simple table.

To connect to the machine, we can run the following:

```sh
docker exec -it some-mysql mysql -p
```

The `some-mysql` is the name of the running container. If when you started the container you named it `john`, then you would use `-it john`.

The `mysql -p` is the command we want to execute. Note that the `-p` is not an option of the `docker` command, but it's an option passed to the `mysql` executable (it's to tell `mysql` that we will provide a password). The command is asking docker to run the command `mysql -p` inside the container `some-mysql`.

Now you can enjoy the power of `SQL` to fully express yourself

```sql
create database MY_DATABASE;
use MY_DATABASE

create table HEROES (NAME varchar(255), COUNTRY varchar(255), BORN date);
insert into HEROES values("Borsellino", "Italy", "1940-01-19");
insert into HEROES values("Falcone", "Italy", "1939-05-18");
insert into HEROES values("Garibaldi", "Italy", "1807-06-02");
insert into HEROES values("Gandhi", "India", "1869-10-02");

select * from HEROES;
```

You can stop the database with a simple

```sh
docker stop some-mysql
```

And restart it whenever you need with

```sh
docker start some-mysql
```

Your tables and data will still be there.

I find really amazing how easy this kind of setup has become compared to my early days in IT.


##References

[https://docs.docker.com/docker-for-mac/](https://docs.docker.com/docker-for-mac/)

[https://store.docker.com/images/mysql/](https://store.docker.com/images/mysql/) from which I shamelessly ripped off the examples in this page.

