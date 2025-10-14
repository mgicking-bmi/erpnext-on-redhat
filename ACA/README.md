## Running ERPNext in Azure Container Apps

### Start with why
Containers are easily scalable and (usually) maintainable

### List of Containers
We will base our architecture off of the non-customized example given in the [official frappe_docker repository](https://github.com/frappe/frappe_docker/blob/main/pwd.yml). This example has the following containers.

|container|purpose|note|
|---|---|---|
|backend|A [werkzeug](https://werkzeug.palletsprojects.com/en/2.0.x/) server that serves assets and data to the frontend|based off of the `frappe/erpnext` docker image|
|configurator|Updates common_site_config.json so Frappe knows how to access db and redis. It is executed on every docker-compose up (and exited immediately). Other services start after this container exits successfully|**We will need to figure out how to orchestrate this timing with Azure Pipelines, or replace this container with a container job or by manually configuring common_site_config.json**|
|create-site|Creates the actual bench site. We may need to change the startup command to include custom configuration|based off of the `frappe/erpnext` docker image|
|db|self-explanatory: it holds data|image: mariadb:10.6|
|frontend|[nginx](https://www.nginx.com) server that serves JS/CSS assets and routes incoming requests|based off of the `frappe/erpnext` docker image|
|queue-long|Python server that run job queues using [rq](https://python-rq.org) for background jobs|based off of the `frappe/erpnext` docker image|
|queue-short|Python server that run job queues using [rq](https://python-rq.org) for background jobs|based off of the `frappe/erpnext` docker image|
|scheduler|Python server that runs tasks on schedule using [schedule](https://schedule.readthedocs.io/en/stable/) for task automation|based off of the `frappe/erpnext` docker image|
|websocket|Node server that runs [Socket.IO](https://socket.io) to manage communication|based off of the `frappe/erpnext` docker image|

On top of these containers, we're creating a separate one for our custom app that will also be based off of the `frappe/erpnext` docker image. The reason for doing this as a separate container is two-fold. First, it means we can prove out running ERPNext in containers. Secondly, it gives us a sandbox to work in to accomplish some future architecture changes*.

### Dockerfiles
Since Azure Container Apps are designed to work off of individual dockerfiles instead of a Docker-Compose, I've done my best to separate all the containers we need out into individual Dockerfiles. **Once the ACA infrastructure exists, this is where you'll need to do the bulk of editing/tweaking to get things working!!!** The dockerfiles can be found in the `/dockerfiles` folder. The `redis-cache` and `redis-queue` dockerfiles shouldn't be needed as we're aiming to use Azure Managed Redis, but I've included them in case we run into issues.

.

.

.

.

_* Future architecture changes:_
1. Because of how many containers are based off of the `frappe/erpnext` docker image, I would like to look into replacing `frontend`, `backend`, and `websocket` with just one container that will also run our custom app.

1. Get rid of `configurator` by having Azure Pipelines or something else handle configuration

1. Get rid of `create-site` by having everything non-job related running on a single container

1. Incorporate [Traefik](https://doc.traefik.io/traefik/) with multiple instances of our singular, non-job containers to act as a load balancer across redundant containers