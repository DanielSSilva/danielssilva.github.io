---
layout: post
title: Docker/SQL - MODIFY FILE encountered operating system error 31(A device attached to the system is not functioning.) while attempting to expand the physical file
tags: [Docker, SQL Server]
comments: true
---

I use a Mac as my development machine. As you might or might not now, SQL Server is not available for macOS. Instead, you need to use docker and have a container to hold your instance.
The whole Docker world is new to me, and every day I'm learning more and more things about it.

Recently I was testing my backup/restore procedure and I had this weird error:

`MODIFY FILE encountered operating system error 31(A device attached to the system is not functioning.) while attempting to expand the physical file '/var/opt/mssql/data/myDatabase.mdf'.`

To give you some context here's the restore command


```
docker exec -it sql19_demo3 /opt/mssql-tools/bin/sqlcmd \
    -S localhost -U SA -P 'MyDockerContainer231' \
    -Q 'RESTORE DATABASE Redacted FROM DISK = "/var/opt/mssql/data/Redacted_backup2.bak" WITH MOVE "Redacted" TO "/var/opt/mssql/data/Redacted.mdf", MOVE "Redacted_Log" TO "/var/opt/mssql/data/Redacted.ldf"'

```

And here's how I launched the container:

```
docker run \
    --name 'sql19_demo' \
    -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD='$PASSWORD \
    -p 1433:1433 \
    -v sqldata1:/var/opt/mssql \
    -v /Users/daniel/Documents/Work/databaseBackups:/backup \
    -d mcr.microsoft.com/mssql/server:2019-latest
```
### Quick explanation
This command creates a container that holds an image of Ubuntu and it has SQL Server2019 installed. It's mounted with a volume `sqldata1` that maps `/var/opt/mssql`, and another volume that maps my folder `/Users/daniel/Documents/Work/databaseBackups` to a folder `/backup` on the container.

# Looking for help
Because I am not familiar with backup/restores (and especially not with this error), I asked some folks for help and also jumped into google to look out for answers for such a similar problem.

As soon as I look at the results and open a few links, I assumed that it had something to do with the filesystem, so the volume part might play a role here... or so I thought.
In more than one post (i.e [this one](https://dba.stackexchange.com/questions/190654/sql-server-2017-linux-cu1-modify-file-encountered-operating-system-error-31), or [this one](https://github.com/Microsoft/mssql-docker/issues/180)), people were saying that the filesystem should be formatted to be `EXT4` instead of `EXT3`.

From my understanding, this would make sense if the `/var/opt/mssql` volume was mounted on one of my MacOS folders, but it wasn't.

As Anthony Nocentino ([b](http://www.centinosystems.com/blog/)\|[t](https://twitter.com/nocentino)) explains in his amazing blogpost series ["Persisting SQL Server Data in Docker Containers"](http://www.centinosystems.com/blog/sql/persisting-sql-server-data-in-docker-containers-part-1/)

>Docker on Mac (and Windows uses the same technique for now) exposes a small Linux VM to provide kernel services to Linux containers. When we create a Docker Volume without specifying a file location for the Volume it will be created *inside* this VM. So that’s where our Volume data is stored.

For this reason, it wasn't making sense to me that whole filesystem thing, because it would imply that the Linux VM provided by Docker would have the wrong filesystem configured. (Again, this is _my_ interpretation... might or might not be correct).

# So, what was my problem?

Turns out that the problem was way simpler to solve and I've found it "randomly".

When I was "playing" with the containers and trying different ways to restore, I suddenly was unable to start a new container because Docker ran out of space.

![out of space](/img/MODIFY_FILE_encountered_operating_system_error_31(A_device_attached_to_the_system_is_not_functioning.)_while_attempting_to_expand_the_physical_file/out_of_space.png)

Upon running `docker system df` to check what space was being used I saw that my containers were taking TONS of space.

![docker system df](/img/MODIFY_FILE_encountered_operating_system_error_31(A_device_attached_to_the_system_is_not_functioning.)_while_attempting_to_expand_the_physical_file/docker_system_df.png)

I went back to the container where I did the backup from and checked how much space the database I was trying to restore was using.

This was achieved by running `docker exec -it sql19 ls -lh /var/opt/mssql/data`

![database space](/img/MODIFY_FILE_encountered_operating_system_error_31(A_device_attached_to_the_system_is_not_functioning.)_while_attempting_to_expand_the_physical_file/database_space.png)

Oh wow! I wasn't expecting my dev database to be that large.

So, doing some quick maths, my docker was using around 91GB (6GB+60GB+25GB). This could be confirmed through the Docker app.

![docker app](/img/MODIFY_FILE_encountered_operating_system_error_31(A_device_attached_to_the_system_is_not_functioning.)_while_attempting_to_expand_the_physical_file/docker_app.png)

The Docker has 96GB allocated, but ~91GB is used.

Because the database is using around 25GB, this means that I would require an additional 20GB of space.

After clearing some space, I ran the same restore command, and at this time, everything went smoothly!

# Turns out it has nothing to do with volumes!

Because the error message says `(A device attached to the system is not functioning.)`, I thought that this could be related to the Docker volume created, but even if you create a container without any volume, the error will be the same.

# The error message might not be that clear in this case

Although the error message indicates that something might be wrong with space (it says `while attempting to expand the physical file '/var/opt/mssql/data/myDatabase.mdf'.`), from my interpretation, it seems that this is the result of the "bigger problem", which is `MODIFY FILE encountered operating system error 31(A device attached to the system is not functioning.)`.






