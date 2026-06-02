# container-hardening

Credit to u/arnedam on [r/selfhosted](https://www.reddit.com/r/selfhosted/comments/1pr74r4/fell_victim_to_cve202566478/)


Hardening docker containers is also highly recommended. Here are some advices from the top of my head (this assuming docker-compose.yml files, but can also be set using docker directly or settings params in Unraid).

1: Make sure your docker is _not_ running as root:

> This is always good practice in my opinion

```
user: "99:100" 
(this example from Unraid - running as user "nobody" group "users"
```

2: Turn off tty and stdin on the container:

> Some cases where you might want to keep this

```
tty: false
stdin_open: false
```

3: Try switching the whole filesystem to read-only (ymmw):

> Usually a good idea, unless an application needs write access

```
read_only: true
```

4: Make sure that the container cant elevate any privileges after start by itself:

> 0 reason not to do this

```
security_opt:
  - no-new-privileges:true

```

5: By default, the container gets a lot of capabilities (12 if I don't remember wrong). Remove ALL of them, and if the container really needs one or a couple of them, add them spesifically after the DROP statement.

```
cap_drop:
  - ALL

or: (this from my Plex container)

cap_drop:
  - NET_RAW
  - NET_ADMIN
  - SYS_ADMIN
```

6: Set up the /tmp-area in the docker to be noexec, nosuid, nodev and limit it's size. If something downloads a payload to the /tmp within the docker, they won't be able to execute the payload. If you limit size, it won't eat all the resources on your host computer. Sometimes (like with Plex), the software auto-updates. Then set the param to exec instead of noexec, but keep all the rest of them.

> most of the time recommended, very few applications where this cannot b e done

tmpfs:
  - /tmp:rw,noexec,nosuid,nodev,size=512m

7: Set limits to your docker so it won't run off with all the RAM and CPU resources of the host:

```
pids_limit: 512
mem_limit: 3g
cpus: 3
```

8: Limit logging to avoid logging bombs within the docker:

```
logging:
  driver: json-file
  options:
    max-size: "50m"
    max-file: "5"
```


9: Mount your data read-only in the docker, then the docker cannot destroy any of the data. Example for Plex:

```
volumes:
  - /mnt/tank/tv:/tv:ro
  - /mnt/tank/movies:/movies:ro
```

10: You may want to run your exposed containers in a separate network DMZ so that any breach won't let them touch the rest of your network. Configure your network and docker host accordingly.

Finally, some of these might prohibit the container to run properly, but my advice in those cases is to open one thing after another to make the attack-surface minimal.

```
docker logs <container> 
```

ChatGPT / Claude / Whatever AI will help you pinpoint what is the choking-point.

Using these settings for publicly exposed containers are lowering the blast radius at a significant level, but it won't remove all risks. Then you need to run it in a VM or even better, separate machine.


