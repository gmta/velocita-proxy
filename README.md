# Velocita

Velocita is a caching reverse proxy for Composer repositories such as Packagist and distribution locations like GitHub.

* Speeds up downloads for package metadata and dist files
* Serves cached files if the source location is unreachable or experiencing problems
* Can be used as a shared cache by multiple developers
* No changes required to your project's files

## Performance

Velocita can give you major performance improvements when a package is not present in the local cache. For example,
installing the PHPUnit dependencies from `composer.lock`:

| Configuration       | Duration       | Relative |
| ------------------- |:--------------:|:--------:|
| Composer            | 25.59s ± 1.14s |   100%   |
| Composer + Velocita | 1.14s ± 0.05s  |    4%    |

Benchmark: `composer install --profile` after `composer require phpunit/phpunit:8.0.4` and clearing both the local cache
and the vendor folder.

Symfony Flex's parallel prefetcher can also benefit from Velocita:

| Configuration           | Duration       | Relative |
| ----------------------- |:--------------:|:--------:|
| Symfony Flex            | 13.13s ± 0.17s |   100%   |
| Symfony Flex + Velocita | 10.59s ± 0.20s |    81%   |

Benchmark: `composer create-project symfony/skeleton symfony --profile` after clearing the local cache.

## Installation

There are two parts to Velocita:

* Velocita Proxy, which acts as a caching reverse proxy
* Composer-velocita, which instructs Composer to retrieve files from Velocita Proxy

### Velocita Proxy

Velocita is available as a Docker image. There are three supported ways to run this image:

1. Using [docker-compose](https://docs.docker.com/compose/):

    1. Clone this repository:

        ```
        git clone https://github.com/isaaceindhoven/velocita-proxy
        cd composer-velocita
        ```

    2. Copy `.env.dist` to `.env`
    3. Edit `.env` and set:

        * `VELOCITA_HOSTNAME`: the hostname (e.g. `localhost`) that Velocita will listen to
        * `VELOCITA_TLS_CERT_FILE`: the path to your X.509 PEM-encoded certificate (or chain) for the domain
        * `VELOCITA_TLS_KEY_FILE`: the path to the private key associated with the certificate

    4. Start Velocita:

        ```
        docker-compose -f docker-compose.yml -f docker-compose.https.yml up -d
        ```

    5. Done!

2. Run the image directly: see [the image's usage instructions](proxy/README.md).

3. Using Kubernetes: this is a work in progress.

### Composer-velocita

[Composer-velocita](https://github.com/isaaceindhoven/composer-velocita) is a Composer plugin that redirects downloads
to your Velocita instance for all repositories it supports.

Run the following two commands on the machine where you want to enable Velocita, replacing
`https://your.velocita.tld/` with the location of your instance:

```
composer global require isaac/composer-velocita
composer velocita:enable https://your.velocita.tld/
```

And you're all set!

## Authors

* Jelle Raaijmakers - [jelle.raaijmakers@isaac.nl](mailto:jelle.raaijmakers@isaac.nl) / [GMTA](https://github.com/GMTA)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
