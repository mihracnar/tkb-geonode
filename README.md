# TKB GeoNode Project

GeoNode template project. Generates a django project with GeoNode support.

## Table of Contents

- [Quick Docker Start](#quick-docker-start)
- [Developer Workshop](#developer-workshop)
- [Create a custom project](#create-a-custom-project)
- [Start your server using Docker](#start-your-server-using-docker)
- [Run the instance in development mode](#run-the-instance-in-development-mode)
- [Run the instance on a public site](#run-the-instance-on-a-public-site)
- [Stop the Docker Images](#stop-the-docker-images)
- [Backup and Restore from Docker Images](#backup-and-restore-the-docker-images)
- [Recommended: Track your changes](#recommended-track-your-changes)
- [Hints: Configuring requirements.txt](#hints-configuring-requirementstxt)

## Quick Docker Start

```bash
python3.10 -m venv ~/.venvs/tkb_geonode_project
source ~/.venvs/tkb_geonode_project/bin/activate

pip install Django==4.2.9

mkdir ~/tkb_geonode_project

GN_VERSION=master # Define the branch or tag you want to generate the project from
django-admin startproject --template=https://github.com/GeoNode/geonode-project/archive/refs/heads/$GN_VERSION.zip -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile tkb_geonode_project ~/tkb_geonode_project

cd ~/tkb_geonode_project
python create-envfile.py
```

The project can also be generated from a local checkout of the geonode-project repository:

```bash
git clone https://github.com/GeoNode/geonode-project
git checkout $GN_VERSION
django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile tkb_geonode_project ~/tkb_geonode_project
```

### create-envfile.py Arguments

`create-envfile.py` accepts the following arguments:

- `--https`: Enable SSL. It's disabled by default
- `--env_type`: 
  - When set to `prod`: DEBUG is disabled and the creation of a valid SSL is requested to Letsencrypt's ACME server
  - When set to `test`: DEBUG is disabled and a test SSL certificate is generated for local testing
  - When set to `dev`: DEBUG is enabled and no SSL certificate is generated
- `--hostname`: The URL that will serve GeoNode (localhost by default)
- `--email`: The administrator's email. Notice that a real email and a valid SMTP configuration are required if `--env_type` is set to `prod`. Letsencrypt uses the email for issuing the SSL certificate
- `--geonodepwd`: GeoNode's administrator password. A random value is set if left empty
- `--geoserverpwd`: GeoServer's administrator password. A random value is set if left empty
- `--pgpwd`: PostgreSQL's administrator password. A random value is set if left empty
- `--dbpwd`: GeoNode DB user role's password. A random value is set if left empty
- `--geodbpwd`: GeoNode data DB user role's password. A random value is set if left empty
- `--clientid`: Client id of GeoServer's GeoNode OAuth2 client. A random value is set if left empty
- `--clientsecret`: Client secret of GeoServer's GeoNode OAuth2 client. A random value is set if left empty

```bash
docker compose build
docker compose up -d
```

## Developer Workshop

Available at: http://geonode.org/dev-workshop

## Create a custom project

**NOTE:** You can call your geonode project whatever you like except 'geonode'. Follow the naming conventions for python packages (generally lower case with underscores (_)). In the examples below, replace `tkb_geonode_project` with whatever you would like to name your project.

To setup your project follow these instructions:

### Generate the project

```bash
git clone https://github.com/GeoNode/geonode-project.git -b <your_branch>
source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
mkvirtualenv --python=/usr/bin/python3 tkb_geonode_project
pip install Django==3.2.16

django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile tkb_geonode_project

cd tkb_geonode_project
```

### Create the .env file

An `.env` file is required to run the application. It can be created from the `.env.sample` either manually or with the `create-envfile.py` script.

The script accepts several parameters to create the file, in detail:

- `hostname`: e.g. master.demo.geonode.org, default localhost
- `https`: (boolean), default value is False
- `email`: Admin email (this is required if https is set to True since a valid email is required by Letsencrypt certbot)
- `env_type`: prod, test or dev. It will set the DEBUG variable to False (prod, test) or True (dev)
- `geonodepwd`: GeoNode admin password (required inside the .env)
- `geoserverpwd`: GeoServer admin password (required inside the .env)
- `pgpwd`: PostgreSQL password (required inside the .env)
- `dbpwd`: GeoNode DB user password (required inside the .env)
- `geodbpwd`: Geodatabase user password (required inside the .env)
- `clientid`: OAuth2 client id (required inside the .env)
- `clientsecret`: OAuth2 client secret (required inside the .env)
- `secret key`: Django secret key (required inside the .env)
- `sample_file`: absolute path to a env_sample file used to create the env_file. If not provided, the one inside the GeoNode project is used.
- `file`: absolute path to a json file that contains all the above configuration

**NOTE:**
- If the same configuration is passed in the json file and as an argument, the CLI one will overwrite the one in the JSON file
- If some value is not provided, a random string is used

#### Example Usage

```bash
python create-envfile.py -f /opt/core/geonode-project/file.json \
  --hostname localhost \
  --https \
  --email random@email.com \
  --geonodepwd gn_password \
  --geoserverpwd gs_password \
  --pgpwd pg_password \
  --dbpwd db_password \
  --geodbpwd _db_password \
  --clientid 12345 \
  --clientsecret abc123
```

#### Example JSON expected:

```json
{
  "hostname": "value",
  "https": "value",
  "email": "value",
  "geonodepwd": "value",
  "geoserverpwd": "value",
  "pgpwd": "value",
  "dbpwd": "value",
  "geodbpwd": "value",
  "clientid": "value",
  "clientsecret": "value"
}
```

## Start your server

Skip this part if you want to run the project using Docker instead (see [Start your server using Docker](#start-your-server-using-docker))

### Setup the Python Dependencies

**NOTE:** Important: modify your requirements.txt file, by adding the GeoNode branch before continue!
(see [Hints: Configuring requirements.txt](#hints-configuring-requirementstxt))

```bash
cd src
pip install -r requirements.txt --upgrade
pip install -e . --upgrade

# Install GDAL Utilities for Python
pip install pygdal=="`gdal-config --version`.*"

# Dev scripts
mv ../.override_dev_env.sample ../.override_dev_env
mv manage_dev.sh.sample manage_dev.sh
mv paver_dev.sh.sample paver_dev.sh

source ../.override_dev_env

# Using the Default Settings
sh ./paver_dev.sh reset
sh ./paver_dev.sh setup
sh ./paver_dev.sh sync
sh ./paver_dev.sh start
```

### Access GeoNode from browser

**NOTE:** default admin user is `admin` (with password: `admin`)

```
http://localhost:8000/
```

## Start your server using Docker

You need Docker 1.12 or higher, get the latest stable official release for your platform.
Once you have the project configured run the following command from the root folder of the project.

### Run docker-compose to start it up

```bash
docker-compose build --no-cache
docker-compose up -d
```

**For Windows users:** Set the following environment variable before running docker-compose up:

```bash
set COMPOSE_CONVERT_WINDOWS_PATHS=1
```

Access the site on: http://localhost/

## Run the instance in development mode

Use dedicated docker-compose files while developing.

**NOTE:** In this example we are going to keep localhost as the target IP for GeoNode

```bash
docker-compose -f docker-compose.development.yml -f docker-compose.development.override.yml up
```

## Run the instance on a public site

### Preparation of the image (First time only)

**NOTE:** In this example we are going to publish to the public IP http://123.456.789.111

```bash
vim .env
# --> replace localhost with 123.456.789.111 everywhere
```

### Startup the image

```bash
docker-compose up --build -d
```

## Stop the Docker Images

```bash
docker-compose stop
```

### Fully Wipe-out the Docker Images

**WARNING:** This will wipe out all the repositories created until now.
**NOTE:** The images must be stopped first

```bash
docker system prune -a
```

## Backup and Restore from Docker Images

### Run a Backup

```bash
SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./tkb_geonode_project/br/backup.sh $BKP_FOLDER_NAME
```

**Parameters:**
- `BKP_FOLDER_NAME`: Default value = backup_restore. Shared Backup Folder name. The scripts assume it is located on "root" e.g.: /$BKP_FOLDER_NAME/
- `SOURCE_URL`: Source Server URL, the one generating the "backup" file.
- `TARGET_URL`: Target Server URL, the one which must be synched.

**Example:**

```bash
docker exec -it django4tkb_geonode_project sh -c 'SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./tkb_geonode_project/br/backup.sh $BKP_FOLDER_NAME'
```

### Run a Restore

```bash
SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./tkb_geonode_project/br/restore.sh $BKP_FOLDER_NAME
```

**Parameters:**
- `BKP_FOLDER_NAME`: Default value = backup_restore. Shared Backup Folder name. The scripts assume it is located on "root" e.g.: /$BKP_FOLDER_NAME/
- `SOURCE_URL`: Source Server URL, the one generating the "backup" file.
- `TARGET_URL`: Target Server URL, the one which must be synched.

**Example:**

```bash
docker exec -it django4tkb_geonode_project sh -c 'SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./tkb_geonode_project/br/restore.sh $BKP_FOLDER_NAME'
```

## Recommended: Track your changes

**Step 1.** Install Git (for Linux, Mac or Windows).

**Step 2.** Init git locally and do the first commit:

```bash
git init
git add *
git commit -m "Initial Commit"
```

**Step 3.** Set up a free account on github or bitbucket and make a copy of the repo there.

## Hints: Configuring requirements.txt

You may want to configure your requirements.txt, if you are using additional or custom versions of python packages. For example:

```python
Django==3.2.16
git+git://github.com/<your organization>/geonode.git@<your branch>
```

### Increasing PostgreSQL Max connections

In case you need to increase the PostgreSQL Max Connections, you can modify the `POSTGRESQL_MAX_CONNECTIONS` variable in `.env` file as below:

```
POSTGRESQL_MAX_CONNECTIONS=200
```

In this case PostgreSQL will run accepting 200 maximum connections.

## Test project generation and docker-compose build Vagrant usage

Testing with vagrant works like this:

### What vagrant does:

Starts a vm for test on docker swarm:
- configures a GeoNode project from template every time from your working directory (so you can develop directly on geonode-project).
- exposes service on localhost port 8888
- rebuilds everytime everything with cache [1] to avoid banning from docker hub with no login.
- starts, reboots to check if docker services come up correctly after reboot.

```bash
vagrant plugin install vagrant-reload
# test things for docker-compose
vagrant up
# check services are up upon reboot
vagrant ssh geonode-compose -c 'docker ps'
```

Test geonode on http://localhost:8888/

To clean up things and delete the vagrant box:

```bash
vagrant destroy -f
```

## Test project generation and Docker swarm build on vagrant

### What vagrant does:

Starts a vm for test on docker swarm:
- configures a GeoNode project from template every time from your working directory (so you can develop directly on geonode-project).
- exposes service on localhost port 8888
- rebuilds everytime everything with cache [1] to avoid banning from docker hub with no login.
- starts, reboots to check if docker services come up correctly after reboot.

To test on a docker swarm enable vagrant box:

```bash
vagrant up
VAGRANT_VAGRANTFILE=Vagrantfile.stack vagrant up
# check services are up upon reboot
VAGRANT_VAGRANTFILE=Vagrantfile.stack vagrant ssh geonode-compose -c 'docker service ls'
```

Test geonode on http://localhost:8888/

Again, to clean up things and delete the vagrant box:

```bash
VAGRANT_VAGRANTFILE=Vagrantfile.stack vagrant destroy -f
```

For direct development on geonode-project after first vagrant up to rebuild after changes to project, you can do vagrant reload like this:

```bash
vagrant reload
```

### What vagrant does (swarm or compose cases):

Starts a vm for test on plain docker service with docker-compose:
- configures a GeoNode project from template every time from your working directory (so you can develop directly on geonode-project).
- rebuilds everytime everything with cache [1] to avoid banning from docker hub with no login.
- starts, reboots.

**[1]** To achieve `docker-compose build --no-cache` just destroy vagrant boxes `vagrant destroy -f`