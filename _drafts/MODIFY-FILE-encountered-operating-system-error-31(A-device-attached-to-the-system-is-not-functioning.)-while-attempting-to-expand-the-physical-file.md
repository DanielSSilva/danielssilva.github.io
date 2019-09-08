---
layout: post
title: MODIFY FILE encountered operating system error 31(A device attached to the system is not functioning.) while attempting to expand the physical file
tags: [Docker, SQL Server]
comments: true
---

I use a Mac as my development machine. As you might or might not now, SQL Server is not available for MacOS. Instead, you need to use docker and have a container to hold your instance.
The whole Docker world is new to me, and everyday I'm learning more and more things about it.

Recently I was testing my backup/restore procedure and I had some weird error, which was:

`MODIFY FILE encountered operating system error 31(A device attached to the system is not functioning.) while attempting to expand the physical file '/var/opt/mssql/data/myDatabase.mdf'.`

In order to give you some context, here's the restore command


```

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
This creates a container with a volume `sqldata1` that holds /var/opt/mssql, and another volume that maps my folder `/Users/daniel/Documents/Work/databaseBackups` to a folder `/backup` on the container.


Because I am not familiar with backup/restores (and specially not with this error), I immediately jumped into google to look out for answers for such similar problem.

As soon as I look at the results and open a few links, I assume that this has something to do with filesystem, so the volume part might play a role here... or so I thought.
In more than one post (i.e [this one](https://dba.stackexchange.com/questions/190654/sql-server-2017-linux-cu1-modify-file-encountered-operating-system-error-31), or [this one](https://github.com/Microsoft/mssql-docker/issues/180)), people were saying that the filesystem should be formatted to be `EXT4` instead of `EXT3`.

From my understanding, this would make sense if the `/var/opt/mssql` volume was mounted on one of my MacOS folders, but it wasn't.

As Anthony Nocentino ([b](http://www.centinosystems.com/blog/)\|[t](https://twitter.com/nocentino)) explains in his amazing blogpost series ["Persisting SQL Server Data in Docker Containers"](http://www.centinosystems.com/blog/sql/persisting-sql-server-data-in-docker-containers-part-1/)

>Docker on Mac (and Windows uses the same technique for now) exposes a small Linux VM to provide kernel services to Linux containers. When we create a Docker Volume without specifying a file location for the Volume it will be created *inside* this VM. So that’s where our Volume data is stored.

For this reason, it wasn't making sense to me that whole filesystem thing.




> Shout out to Anthony Nocentino's ([b](http://www.centinosystems.com/blog/)\|[t](https://twitter.com/nocentino)) for helping me along this way and for writting an amazing and well detailed blogpost series ["Persisting SQL Server Data in Docker Containers"](http://www.centinosystems.com/blog/sql/persisting-sql-server-data-in-docker-containers-part-1/) about persisting data on the containers. This all started because I wasn't being able to correctly backup my databases. Checkout the [twitter thread](https://twitter.com/nocentino/status/1167089014123503617) that motivated this series (and my need to learn about volumes) if you are curious.


