# docker-compose scaling demo
A short demo on how to use docker-compose to create a Web Service connected to a load balancer and a Redis (In-Memory Cache).

## Start up and running
Once you've cloned the project to your host we can now start our demo project.  
```bash
cd py3-scale
docker-compose up
```

Open URL [http://localhost/] in your web-browser.  
Refresh the web-page several times - in the TTY-console you will see,
that your requests are served by 2 docker-containers of the `py3` service:
```text
py3_1    | 172.18.0.7 - - [22/Oct/2019 08:38:52] "GET / HTTP/1.1" 200 -
py3_2    | 172.18.0.7 - - [22/Oct/2019 08:38:54] "GET / HTTP/1.1" 200 -
py3_1    | 172.18.0.7 - - [22/Oct/2019 08:38:56] "GET / HTTP/1.1" 200 -
py3_2    | 172.18.0.7 - - [22/Oct/2019 08:38:57] "GET / HTTP/1.1" 200 -
```

This means that the composition is working as it should. Stop it and run Compose in Daemon mode:
```bash
   Press [CTRL]+[C] in TTY-console to stop Compose

Stopping py3-scale_lb_1    ... done
Stopping py3-scale_py3_1   ... done
Stopping py3-scale_py3_2   ... done
Stopping py3-scale_redis_1 ... done

docker-compose up -d
```

## Scaling
Right now we have **2** docker-containers of the  `py3` service, as described in `docker-compose.yml`:
```yaml
  py3:
    # ... cut few YAML lines
    scale: 2
```

Let's scale our `py3` service from 2 instance to 4 instances.
```bash
docker-compose scale py3=4

Starting py3-scale_py3_1 ... done
Starting py3-scale_py3_2 ... done
Creating py3-scale_py3_3 ... done
Creating py3-scale_py3_4 ... done
``` 
This will scale our `py3` service now.  
    
Now refresh URL [http://localhost/py3/hits] in web-browser several times
or **`curl http://localhost/py3/hits`** in TTY.  
We will see now, the number of "My hostname" values increased up to 4 different strings.

To get a deeper understanding tail the logs of the stack to watch what happens each time you access your web services.
```bash
docker-compose logs

    py3_4   | 172.17.0.7 - - [22/Oct/2019 08:39:41] "GET /py3/hits HTTP/1.1" 200 -
    py3_1   | 172.17.0.7 - - [22/Oct/2019 08:39:43] "GET /py3/hits HTTP/1.1" 200 -
    py3_2   | 172.17.0.7 - - [22/Oct/2019 08:39:46] "GET /py3/hits HTTP/1.1" 200 -
    py3_3   | 172.17.0.7 - - [22/Oct/2019 08:39:48] "GET /py3/hits HTTP/1.1" 200 -
    py3_4   | 172.17.0.7 - - [22/Oct/2019 08:39:49] "GET /py3/hits HTTP/1.1" 200 -
```
Here's the output from my docker-compose logs after I curled my application 5 times so it is clear that the round-robin is sent to all 5 web service containers.

# Notes

## Compose file version
**Version 2.4** of docker-compose file is required to use the **"scale: 2"** variable in `docker-compose.yml`.

Version **3** of docker-compose file is intended for [Docker-Swarm](https://docs.docker.com/get-started/part4/) usage,
and do not support **"scale: X"** var.

## HAproxy from dockercloud
A benefit of running the HAProxy from Dockercloud is it automatically
detects the coming and going of docker-containers and doesn't require any changes.

Without this load balancer (dockercloud/haproxy) you will need to edit and restart Compose for scaling service.

## container_name is'n applicable with SCALE
You CAN'T use **container_name** and **scale** together,
because the **scale** generates a unique name for each container,
but it is hard-coded by the **container_name**. 