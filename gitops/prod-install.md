### Single Server Example

In this use case we have a single server with a static IP attached to it. It can be used in scenarios where one powerful VM has multiple benches and applications or one entry level VM with single site. For single bench, single site setup follow only up to the point where first bench and first site is added. If you choose this setup you can only scale vertically. If you need to scale horizontally you'll need to backup the sites and restore them on to cluster setup.

We will setup the following:

- Install docker and docker compose v2 on linux server.
- Setup project called `rsis-prod` and create sites `rsis.reshinecarstudio.com` in the project.

Explanation:

Each instance of ERPNext project (bench) will have its own redis, socketio, gunicorn, nginx, workers and scheduler. It will connect to internal MariaDB by connecting to MariaDB network. It will expose sites to public through Traefik by connecting to Traefik network.

### Install Docker

Easiest way to install docker is to use the [convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script).

```shell
curl -fsSL https://get.docker.com | bash
```

Note: The documentation assumes Ubuntu LTS server is used. Use any distribution as long as the docker convenience script works. If the convenience script doesn't work, you'll need to install docker manually.

### Install Compose V2

Refer [original documentation](https://docs.docker.com/compose/cli-command/#install-on-linux) for updated version.

```shell
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

### Prepare

Clone `frappe_docker` repo for the needed YAMLs and change the current working directory of your shell to the cloned repo.

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

Create configuration and resources directory

```shell
mkdir gitops-rsis-prod
```

The `gitops-rsis-prod` directory will store all the resources that we use for setup. We will also keep the environment files in this directory as there will be multiple projects with different environment variables. You can create a private repo for this directory and track the changes there.

### Install ERPNext

#### Create first bench

Create first bench called `rsis-prod` with site `rsis.reshinecarstudio.com`

Create a file called `rsis-prod.env` in `gitops-rsis-prod`

```shell
cp example.env gitops-rsis-prod/rsis-prod.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=rsisprodDbP@ss/g' gitops-rsis-prod/rsis-prod.env
# sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' gitops-rsis-prod/rsis-prod.env
# sed -i 's/DB_PORT=/DB_PORT=3306/g' gitops-rsis-prod/rsis-prod.env
sed -i 's/SITES=`erp.example.com`/SITES=\`rsis.reshinecarstudio.com\`/g' gitops-rsis-prod/rsis-prod.env
echo 'ROUTER=rsis-prod' >> gitops-rsis-prod/rsis-prod.env
echo "BENCH_NETWORK=rsis-prod" >> gitops-rsis-prod/rsis-prod.env
```

env file is generated at location `gitops-rsis-prod/rsis-prod.env`.

Create a yaml file called `rsis-prod.yaml` in `gitops-rsis-prod` directory:

```shell
docker compose --project-name rsis-prod \
  --env-file gitops-rsis-prod/rsis-prod.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.noproxy.yaml \
  config > gitops-rsis-prod/rsis-prod.yaml
```

Use the above command after any changes are made to `rsis-prod.env` file to regenerate `gitops-rsis-prod/rsis-prod.yaml`. e.g. after changing version to migrate the bench.

Deploy `rsis-prod` containers:

```shell
docker compose --project-name rsis-prod -f gitops-rsis-prod/rsis-prod.yaml up -d
```

Create sites `rsis.reshinecarstudio.com`:

```shell
# rsis.reshinecarstudio.com
docker compose --project-name rsis-prod exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password rsisprodDbP@ss --install-app erpnext --admin-password rsisprodP@ss rsis.reshinecarstudio.com
```
