---
layout: post
title:  "Install Atlassian Confluence on a Mac with Docker"
date:   2018-01-29 08:22:00 +0200
categories: docker macOS confluence
tags: collect_list collect_set optimization spark groupby dataframe
---

To complete my migration from a Linux notebook to a Mac, I am finally moving my [Confluence](https://www.atlassian.com/software/confluence) installation to my MacBook Pro. (I know, it's a bit overkill to use Confluence just for my personal notes, but I really like it.)

As I want to keep my machine as clean as possible, I have used Docker for the installation. And this time I also gave a try to Kitematic, the new *beta* UI to handle the containers, by Docker.

## What you need to install

Basically:

* [Docker](https://www.docker.com/community-edition)
* [Kitematic](https://kitematic.com/)
* The MySQL JDBC Driver (more on this later)

And from within Kitematic
* confluence-server
* mysql

Using Kitematic, I have downloaded the **confluence-server** official image. I have configured to redirect the ports as `localhost:8090` and `localhost:8091`. It's just simpler to remember than the original redirect proposed by Docker. After an extreme effort of imagination, I have called this container ***confluence-server***.

I have also created a new instance of mysql. (I wrote before about [mysql on docker on a Mac](% post_url 2017-09-14-Run-MySQL-on-Mac-with-Docker %).) I have called it ***mysql-for-confluence***. Just remember to set the password by adding the variable `MYSQL_ROOT_PASSWORD`. Choose a good password. 

I have then updated the configuration of the *confluence-server* to have the *mysql-for-confluence* as dependency. You will find this setting in the networks tab.

![2018-01-28-confluence-mysql-container-dependency-network](/images/2018-01-28-confluence-mysql-container-dependency-network.png)

Just a personal note. Kitematic is good enough to hide where those settings are stored. I am no expert of Docker, and here I am using it just like another application. Just another note: start Docker before starting Kitematic. (Try the other way around and you will understand.)

I was hoping that Kitematic would automatically start the *mysql-for-confluence* container whenever you start the *confluence-server* one: it's just print a very fast error, lasting on screen for a fraction of a seconds, telling you to start the dependencies first. So first start the *mysql-for-confluence* and then the *confluence-server*.

However the main advantage of the dependency is that now you can see the *mysql-for-confluence* directly on the original port `3306`. And this is great.

## The MySQL JDBC driver for Confluence

Due to MySQL JDBC Driver licensing problems (about that: did you get your Confluence license from Atlassian?), you have to manually download the MySQL JDBC driver. I could have used the PostgreSQL as [Confluence already include the PostgreSQL Driver](https://confluence.atlassian.com/doc/database-jdbc-drivers-171742.html). But I already started writing this notes saying I would have used MySQL, and I am lazy. So I stayed on MySQL, [downloaded the MySQL JDBC driver](https://dev.mysql.com/downloads/connector/j/), and copied it to the *confluence-server* container.

```bash
docker cp mysql-connector-java-5.1.45-bin.jar confluence-server:/opt/atlassian/confluence/confluence/WEB-INF/lib
```

And restart the *confluence-server* container.

I did try to restart just Confluence, but it didn't work. I got a `$CATALINA_PID was set but the specified file does not exist. Is Tomcat running? Stop aborted` and I gave up.

## Setup MySQL

It's time to create the database on MySQL!

Start a shell on the `mysq-for-confluence` container and run `mysql -p` to get into MySQL. Then:

```mysql
CREATE DATABASE confluence;
CREATE USER confluence;
SET PASSWORD FOR confluence=anothergoodpassword
GRANT ALL confluence.* TO confluence;
```

(not sure the last line is necessary as there are no tables for the moment.)

You will also have to run the following to setup the database to use `utf8`([Confluence would have told you later](https://confluence.atlassian.com/kb/how-to-fix-the-collation-and-character-set-of-a-mysql-database-744326173.html) but I'm telling you now):

```mysql
ALTER DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
```

You will also have to update the [isolation level setting](https://confluence.atlassian.com/confkb/confluence-fails-to-start-and-throws-mysql-session-isolation-level-repeatable-read-is-no-longer-supported-error-241568536.html) in the `mysqld.cnf` file within the `mysql-for-confluence` container:

```bash
echo "transaction-isolation=READ-COMMITTED" >> /etc/mysql/mysql.conf.d/mysqld.cnf
```

**Make sure the quotes are correct** if you are copying and pasting this command!

Should you be willing to use a text editor, remember that you can install `vim` as you would do on Debian:

```bash
apt update
apt install vim
```

And guess what? Restart the `mysql-for-confluence` container once again.

Now back to the Confluence setup.

After choosing again the database in the setup procedure of Confluence, the connection should be ready.

![Settings to connect MySQL](/images/2018-01-28-setup-mysql-connection-success.png)

A few more **Next** buttons and I eventually could choose to restore the backup I have run from my old computer.

![Restoring the confluence backup](/images/2018-01-28-confluence-setup-restore-progress.png)

Not sure the last is required since there are no tables yet. Those will be created by the confluence installation.

Now all the notes I had on my Confluence are back. Although with some failing *Health Checks*.

![Confluence Failing Health Checks on Docker on Mac](/images/2018-01-28-confluence-health-checks-mac-docker.png)

I am leaving them for another post.
