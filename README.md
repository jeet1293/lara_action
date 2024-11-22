

# Lara Action

[![Tests](https://github.com/jeet1293/lara_action/actions/workflows/tests.yml/badge.svg)](https://github.com/jeet1293/lara_action/actions/workflows/tests.yml)
[![PHPStan](https://github.com/jeet1293/lara_action/actions/workflows/phpstan.yml/badge.svg)](https://github.com/jeet1293/lara_action/actions/workflows/phpstan.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This is a dummy Laravel project setup using [Laravel Sail](https://laravel.com/docs/11.x/sail) for local development. It is intended for testing [GitHub Actions](https://github.com/features/actions) and automating workflows such as running tests and performing static analysis with PHPStan.

## Project Setup

### Prerequisites

Ensure that you have the following tools installed on your system:

- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose](https://docs.docker.com/compose/)
- [Git](https://git-scm.com/)

### Setting Up the Project

1. Clone the repository to your local machine:

    ```bash
    git clone https://github.com/jeet1293/lara_action.git
    cd lara_action
    ```

2. Install Composer Dependencies:

    ```bash
    docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php83-composer:latest \
    composer install --ignore-platform-reqs
    ```

3. Build the Docker containers using Sail:

    ```bash
    ./vendor/bin/sail up -d
    ```

   This will start the Docker containers in detached mode. Sail sets up services like the web server, database, etc., using Docker.
   This will start the Docker containers in detached mode. Sail sets up services like the web server, database, etc., using Docker.

4. Configure environment variables:

    ```bash
        cp .env.example .env
        ./vendor/bin/sail artisan key:generate
    ```
   Please make the necessary change(i.e. Generate the application key)

5. Run the Laravel application:
   Open your browser and go to:

    ```plaintext
    http://localhost
    ```

   You should see the Laravel welcome page.

#### Running Artisan Commands

You can run Laravel Artisan commands within the Docker container by prefixing them with `./vendor/bin/sail`. For example, to run migrations:

```bash
./vendor/bin/sail artisan make:controller MyController
./vendor/bin/sail artisan make:Model MyModel
```

## GitHub Actions Workflow

The GitHub Actions workflow in this project automates tasks such as running tests and performing static analysis. It runs two primary jobs:

1. **Run Tests** - This job installs dependencies, prepares the Laravel application, and runs the tests using PHPUnit.
2. **Run Static Analysis** - This job installs dependencies and runs static analysis with PHPStan to check for code quality issues.

### Workflow Configuration

The workflow is triggered on:

- **Push** events to the `main` branch.
- **Pull Request** events targeting the `main` branch, specifically for changes to `.php` and `composer.*` files.

#### Job 1: Run Tests

This job runs [PHPUnit](https://phpunit.de/index.html) tests using the following steps:

1. **Checkout the code**: It checks out the code from the repository.
2. **Setup PHP**: It sets up PHP 8.3 and installs the necessary PHP extensions.
3. **Cache Composer packages**: It caches the Composer dependencies to speed up future builds.
4. **Install Composer dependencies**: It installs the dependencies defined in `composer.lock`.
5. **Prepare Laravel application**: It copies the `.env.ci` configuration file to `.env` and generates a new Laravel application key using `php artisan key:generate`.
6. **Run tests**: It runs the Laravel tests using `php artisan test`.

##### Example command to run the tests:

```bash
./vendor/bin/sail artisan test
```

#### Job 2: Run Static Analysis

This job runs [PHPStan](https://phpstan.org/), a static analysis tool, to check the code for potential issues. The steps for this job are:

1. **Checkout the code**: This step checks out the code from the repository using the `actions/checkout@v4` action.
2. **Setup PHP**: This step uses `shivammathur/setup-php@v2` to set up PHP 8.3 and ensure the necessary PHP extensions are available for the analysis.
3. **Cache Composer Packages**: To speed up subsequent runs, this step caches the Composer dependencies located in the `vendor/` directory. This prevents the need to reinstall dependencies on every run and helps to optimize build times.
4. **Install Composer dependencies**: This step installs the project’s Composer dependencies, including the PHPStan extension installer, which is required for running PHPStan.
5. **Run Static Analysis with PHPStan**: This step runs PHPStan, scanning the codebase for issues related to code quality, potential bugs, and type errors. The results are output in a GitHub-compatible format.

##### Example command to run PHPStan:

```bash
./vendor/bin/sail exec laravel.test ./vendor/bin/phpstan
```
