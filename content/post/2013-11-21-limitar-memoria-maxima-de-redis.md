---
author: fiunchinho
categories:
- Linux
date: 2013-11-21T17:59:32Z
dsq_thread_id:
- "2650647098"
guid: http://armesto.net/?p=10
id: 10
tags:
- redis
title: Limitar memoria máxima de Redis
url: /limitar-memoria-maxima-de-redis/
---

Haciendo pruebas en uno de los proyectos que tengo, utilicé <a title="Redis" href="http://redis.io/" target="_blank">Redis</a> como sistema de caché, en vez de utilizar <a title="Memcached" href="http://memcached.org/" target="_blank">Memcached</a> que es el que normalmente uso. Por defecto, Memcached va llenando su memoria hasta que esta se llena, y es entonces cuando empieza a borrar valores existentes para hacer sitio a las nuevas. La estrategia elegida para borrar es eliminar aquellas keys de la cache que han sido menos utilizadas.

Cuando activé Redis para mi proyecto, el proceso _daemon_ del servidor empezó a ocupar más y más memoria hasta que ocupaba casi la totalidad de la RAM del servidor. Poco después, el propio sistema operativo decidió matar el proceso de redis para no morir en el intento.

<!--more-->

Si no quería tener que conectarme todos los días a levantar manualmente el daemon de Redis debía encontrar una manera de que esto no ocurriese. Así que decidí que Redis se comportase igual que Memcached: decidir una **memoria límite**, y que cuando esté llena, **empiece a descartar las claves menos utilizadas**. Para ello abrí el archivo de configuración de Redis (en mi caso se encontraba bajo _/etc/redis/redis.conf_), y me llevé una grata sorpresa ya que parecía casi un manual. Todas las opciones estaban muy bien explicadas, incluso con ejemplos.

Buscando la opción `maxmemory`, me encontré con lo siguiente en el propio archivo de configuración

<pre># Don't use more memory than the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys with an
# EXPIRE set. It will try to start freeing keys that are going to expire
# in little time and preserve keys with a longer time to live.
# Redis will also try to remove objects from free lists if possible.
#
# If all this fails, Redis will start to reply with errors to commands
# that will use more memory, like SET, LPUSH, and so on, and will continue
# to reply to most read-only commands like GET.
#
# WARNING: maxmemory can be a good idea mainly if you want to use Redis as a
# 'state' server or cache, not as a real DB. When Redis is used as a real
# database the memory usage will grow over the weeks, it will be obvious if
# it is going to use too much memory in the long run, and you'll have the time
# to upgrade. With maxmemory after the limit is reached you'll start to get
# errors for write operations, and this may even lead to DB inconsistency.
#
# maxmemory &lt;bytes&gt;</pre>

Así que solo tenía que decidir cuanta memoria quería designar como máximo para Redis. Como lo estaba utilizando de caché, y no como base de datos, no hacía falta que fuese demasiado. Así que decidí poner **256Mb**

<pre>maxmemory 268435456</pre>

Lo siguiente, era decidir qué estrategia debía seguir Redis para descartar claves guardadas en memoria. Una línea más abajo de donde estaba en el fichero de configuración me encuentro esto

<pre># MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached? You can select among five behavior:
# 
# volatile-lru -&gt; remove the key with an expire set using an LRU algorithm
# allkeys-lru -&gt; remove any key accordingly to the LRU algorithm
# volatile-random -&gt; remove a random key with an expire set
# allkeys-&gt;random -&gt; remove a random key, any key
# volatile-ttl -&gt; remove the key with the nearest expire time (minor TTL)
# noeviction -&gt; don't expire at all, just return an error on write operations
# 
# Note: with all the kind of policies, Redis will return an error on write
# operations, when there are not suitable keys for eviction.
#
# At the date of writing this commands are: set setnx setex append
# incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
# sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
# zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
# getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy volatile-lru</pre>

Así que cualquiera de las 2 primeras estrategias podía valerme para lo que yo quería conseguir. Añadí la siguiente línea

<pre>maxmemory-policy volatile-lru</pre>

Y listo. Ahora Redis tenía un ímite de memoria, y cuando se llenase, iría borrando valores que hubiesen sido menos utilizados recientemente. Podemos comprobar que funciona correctamente, controlando cuánta memoria está utilizando Redis en un momento determinado. Para ello, tan solo tenemos que utilizar el siguiente comando

<pre>redis-cli info | grep memory</pre>

Y debería mostrarnos algo parecido a esto

<pre>used_memory:268227192
used_memory_human:255.80M
used_memory_rss:293154816</pre>

Ahí podemos ver que la memoria no supera el límite que elegimos. El sistema operativo ya no tendrá que volver a preocuparse.