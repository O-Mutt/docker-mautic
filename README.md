Docker Mautic Image
===================

> [!NOTE]
> _This version refers to Docker images and examples for Mautic 5, previous Mautic versions aren't actively supported anymore. If you would like information about older versions, see <https://github.com/mautic/docker-mautic/tree/mautic4>._

You can access and customize Docker Mautic from [Official Docker Hub image](https://hub.docker.com/r/mautic/mautic/).

# Pulling image from Docker Hub

If you want to pull the latest stable image from DockerHub:

    docker pull mautic/mautic:latest

# Running Basic Container

Setting up MySQL Server:

    docker volume create mysql_data

* `5-apache`: latest stable version of Mautic 5 of the `apache` variant
* `5.0-fpm`: latest version in the 5.0 minor release in the `fpm` variant
* `5.0.3-apache`: specific point release of the `apache` variant

Running Mautic:

    $ docker volume create mautic_data

    $ docker run --name mautic -d \
        --restart=always \
        -e MAUTIC_DB_HOST=127.0.0.1 \
        -e MAUTIC_DB_USER=root \
        -e MAUTIC_DB_PASSWORD=mypassword \
        -e MAUTIC_DB_NAME=mautic \
        -e MAUTIC_RUN_CRON_JOBS=true \
        -e MAUTIC_TRUSTED_PROXIES=0.0.0.0/0 \
        -p 8080:80 \
        -v mautic_data:/var/www/html \
        mautic/mautic:latest

This will run a basic mysql service within Mautic on <http://localhost:8080>.

## Customizing Mautic Container

The following environment variables are also honored for configuring your Mautic instance:

#### Database Options

* `-e MAUTIC_DB_HOST=...` (defaults to the IP and port of the linked `mysql` container)

* `-e MAUTIC_DB_USER=...` (defaults to "root")
* `-e MAUTIC_DB_PASSWORD=...` (defaults to the value of the `MYSQL_ROOT_PASSWORD` environment variable from the linked `mysql` container)
* `-e MAUTIC_DB_NAME=...` (defaults to "mautic")
* `-e MAUTIC_DB_TABLE_PREFIX=...` (defaults to empty) Add prefix do Mautic Tables. Very useful when migrate existing databases from another server to docker.

If you'd like to use an external database instead of a linked `mysql` container, specify the hostname and port with `MAUTIC_DB_HOST` along with the password in `MAUTIC_DB_PASSWORD` and the username in `MAUTIC_DB_USER` (if it is something other than `root`).

* `mautic_web`: runs the Mautic webinterface
* `mautic_worker`: runs the worker processes to consume the messenger queues
* `mautic_cron`: runs the defined cronjobs

This allows you to use different scaling strategies to run the workers or crons, without having to maintain separate images.
The `mautic_cron` and `mautic_worker` require the codebase anyhow, as they execute console commands that need to bootstrap the full application.

### Enable / Disable Features

* `-e MAUTIC_TESTER=...` (defaults to empty) Enables Mautic Github Pull Tester  [documentation](https://github.com/mautic/mautic-tester)

### PHP options

* `-e PHP_INI_DATE_TIMEZONE=...` (defaults to `UTC`) Set PHP timezone

* `-e PHP_MEMORY_LIMIT=...` (defaults to `256M`) Set PHP memory limit
* `-e PHP_MAX_UPLOAD=...` (defaults to `20M`) Set PHP upload max file size
* `-e PHP_MAX_EXECUTION_TIME=...` (defaults to `300`) Set PHP max execution time

>
> [!WARNING]
> The examples **require `docker compose` v2**.
> Running the examples with the unsupported `docker-compose` v1 will result in a non-starting web container.

> [!IMPORTANT]
> Please take into account the purpose of those examples:
> it shows how it **could** be used, not how it **should** be used.
> Do not use those examples in production without reviewing, understanding and configuring them.

### Mautic Versioning

Mautic Docker has two ENV that you can specify an version do start your new container:

* the `.env` file:
  Should be used for all general variables for Mysql, PHP, ...
* the `.mautic_env` file:
  Should be used for all Mautic specific variables.

## Accesing the Instance

Access your new Mautic on `http://localhost:8080` or `http://host-ip:8080` in a browser.

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `mautic`:

```yaml
version: '2'

services:

  mauticdb:
    image: percona/percona-server:5.7
    container_name: mauticdb
    volumes:
      - mysql_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=mysecret
    command:
      --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
    networks:
      - mautic-net

  mautic:
    image: mautic/mautic:latest
    container_name: mautic
    links:
      - mauticdb:mysql
    depends_on:
      - mauticdb
    ports:
      - 8080:80
    volumes:
      - mautic_data:/var/www/html
    environment:
      - MAUTIC_DB_HOST=mauticdb
      - MYSQL_PORT_3306_TCP=3306
      - MAUTIC_DB_USER=root
      - MAUTIC_DB_PASSWORD=mysecret
      - MAUTIC_DB_NAME=mautic
      - MAUTIC_RUN_CRON_JOBS=true
    networks:
      - mautic-net

volumes:
  mysql_data:
    driver: local
  mautic_data:
    driver: local
networks:
  mautic-net:
    driver: bridge
```

Run `docker-compose up`, wait for it to initialize completely, and visit `http://localhost:8080` or `http://host-ip:8080`.

> This compose file was tested on compose file version 3.0+ (docker engine 1.13.0+), see the relation of compose file and docker engine [here](https://docs.docker.com/compose/compose-file/compose-versioning/).

* `config`: the local config folder containing `local.php`, `parameters_local.php`, ...
* `var/logs`: the folder with logs
* `docroot/media`: the folder with uploaded and generated media files

# Supported Docker versions

### Configuration

#### Environment Variables

The following environment variables can be used to configure how your setup should behave.

Support for older versions (down to 1.0) is provided on a best-effort basis.

* `MAUTIC_DB_HOST`: IP address or hostname of the MySQL server.
* `MAUTIC_DB_PORT`: port which the MySQL server is listening on. Defaults to `3306`.
* `MAUTIC_DB_DATABASE`: Database which holds Mautic's tables.
* `MAUTIC_DB_USER`: MySQL user which should be used by Mautic.
* `MAUTIC_DB_PASSWORD`: Passowrd of the MySQL user which should be used by Mautic.
* `DOCKER_MAUTIC_ROLE`: which role does the container has to perform.
   Defaults to `mautic_web`, other supported values are `mautic_worker` and `mautic_cron`.
* `DOCKER_MAUTIC_LOAD_TEST_DATA`: should the test data be loaded on start or not.
   Defaults to `false`, other supported value is `true`.
   This variable is only usable when using the `web` role.
* `DOCKER_MAUTIC_RUN_MIGRATIONS`: should the Doctrine migrations be executed on start.
   Defaults to `false`, other supported value is `true`.
   This variable is only usable when using the `web` role.
* `DOCKER_MAUTIC_WORKERS_CONSUME_EMAIL`: Number of workers to start consuming mails.
   Defaults to `2`
* `DOCKER_MAUTIC_WORKERS_CONSUME_HIT`: Number of workers to start consuming hits.
   Defaults to `2`
* `DOCKER_MAUTIC_WORKERS_CONSUME_FAILED`: Number of workers to start consuming failed e-mails.
   Defaults to `2`

##### PHP Settings

* `PHP_INI_VALUE_DATE_TIMEZONE`: defaults to `UTC`
* `PHP_INI_VALUE_MEMORY_LIMIT`: defaults to `512M`
* `PHP_INI_VALUE_UPLOAD_MAX_FILESIZE`: defaults to `512M`
* `PHP_INI_VALUE_POST_MAX_FILESIZE`: defaults to `512M`
* `PHP_INI_VALUE_MAX_EXECUTION_TIME`: defaults to `300`

#### Mautic settings

Technically, every setting of Mautic you can set via the UI or via the `local.php` file can be set as environment variable.

e.g. the `messenger_dsn_hit` can be set via the `MAUTIC_MESSENGER_DSN_HIT` environment variable.
See the general Mautic documentation for more info.

### Customization

Currently this image has no easy way to extend Mautic (e.g. adding extra `composer` dependencies or installing extra plugins or themes).
This is an ongoing effort we hope to support in an upcoming 5.x release.

For now, please build your own images based on the official ones to add the needed dependencies, plugins and themes.

## Day to day tasks

You can execute commands directly against the [Mautic CLI](https://docs.mautic.org/en/5.x/configuration/command_line_interface.html#mautic-commands). To do so you have two options:

1. Connect to the running container and run the commands.
1. Run the commands as `exec` via docker (compose).

Both cases will use `docker compose exec`/`docker exec`. Using `docker compose` uses the `docker-compose.yaml` and the container names listed for ease. More info can be learned about `exec` commands [here](https://docs.docker.com/engine/reference/commandline/compose_exec/).

Note - Two flags that are used commonly in docker Mautic:

1. `--user www-data`
   * execute as the `www-data` user, which is the same user as the webserver runs. Running commands as the correct user ensures things function as expected. e.g. file permissions after clearing the cache are correct.
2. `--workdir /var/www/html`
   * set the working directory to the `/var/www/html` folder, which is the project root of Mautic.

### Connect to the Container

```bash
docker compose exec --user www-data --workdir /var/www/html mautic_web /bin/bash
```

### Running a Mautic CLI command

```bash
docker compose exec -u www-data -w /var/www/html mautic_web php ./bin/console mautic:install https://mautic.example.com --admin_email="admin@mautic.local" --admin_password="Maut1cR0cks!"
```

## Issues

If you have any problems with or questions about this image, please contact us through a [GitHub issue](https://github.com/mautic/docker-mautic/issues).

You can also reach the Mautic community through its [online forums](https://www.mautic.org/community/) or the [Mautic Slack channel](https://www.mautic.org/slack/).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/mautic/docker-mautic/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.

# License

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/cibero42"><img src="https://avatars.githubusercontent.com/u/102629460?v=4?s=100" width="100px;" alt="Renato"/><br /><sub><b>Renato</b></sub></a><br /><a href="https://github.com/mautic/docker-mautic/commits?author=cibero42" title="Code">💻</a> <a href="https://github.com/mautic/docker-mautic/commits?author=cibero42" title="Documentation">📖</a> <a href="https://github.com/mautic/docker-mautic/pulls?q=is%3Apr+reviewed-by%3Acibero42" title="Reviewed Pull Requests">👀</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://academy.leewayweb.com"><img src="https://avatars.githubusercontent.com/u/1532615?v=4?s=100" width="100px;" alt="Mauro Chojrin"/><br /><sub><b>Mauro Chojrin</b></sub></a><br /><a href="https://github.com/mautic/docker-mautic/commits?author=mchojrin" title="Code">💻</a></td>
    </tr>
  </tbody>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
