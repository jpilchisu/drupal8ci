# Drupal 8 CI

[![CircleCI](https://circleci.com/gh/Lullabot/drupal8ci.svg?style=svg)](https://circleci.com/gh/Lullabot/drupal8ci)

This repository provides the foundation to implement [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) in a Drupal 8
project using [CircleCI](https://circleci.com/), [GitLab CI](https://about.gitlab.com/features/gitlab-ci-cd/),
or [Travis CI](https://travis-ci.org) against a GitHub or GitLab repository.

To install, simply run the respective installer and allow the CI provider that you chose to watch repository changes
to start building on every pull request.

If you want to test an individual module instead of a Drupal project, see Andrew Berry's
[drupal_tests](https://github.com/deviantintegral/drupal_tests).

Here is a clip that shows [how it works for CircleCI](https://www.youtube.com/watch?v=wd_5mX0x4K8).

## Requirements

The scripts assume that you have a Drupal 8 project created using [drupal-project](https://github.com/drupal-composer/drupal-project),
which sets a well known foundation. If your project's directory
structure differs from what _drupal-project_ sets up, you will need to
adjust the CI scripts.

## Installation

Each CI tool has its own installer, which extracts the required files to run the jobs,
plus a set of sample PHPUnit and Behat tests.

Choose a CI tool from the list below and follow its installation steps.

### [CircleCI](https://circleci.com)

[Live demo](https://github.com/juampynr/drupal8-circleci)

Open a terminal and run the installer from the root of your project:
```bash
curl -L https://github.com/lullabot/drupal8ci/raw/master/setup-circleci.sh | bash
```

Sign up at [CircleCI](https://circleci.com/) and allow access to your project's repository.

![CircleCI watch](docs/images/circleci-watch.png)

Happy CI-ing! :-D. From now every time you create a pull request, CircleCI will run the
set of jobs and report their result like in the following screenshot:

![CircleCI pull request](docs/images/circleci-pr.png)

For an overview of the CircleCI features, have a look at
[this article from the Lullabot blog](https://www.lullabot.com/articles/continuous-integration-drupal-8-circleci).

#### Using a custom Docker image

The [CircleCI configuration file](dist/circleci/.circleci/config.yml) uses a
[custom Docker image](https://hub.docker.com/r/juampynr/drupal8ci/) that extends from
the [official Drupal image](https://hub.docker.com/_/drupal/) and it is [hosted at
Docker Hub](https://hub.docker.com/r/juampynr/drupal8ci/). If this image
does not fit your project's architecture, then have a look at its
[Dockerfile](https://github.com/Lullabot/drupal8ci/blob/master/circleci/.circleci/images/primary/Dockerfile)
and consider [creating your own image](https://circleci.com/docs/2.0/custom-images/)
based out of it.
   
#### Setting up the update path

The Behat job requires a running Drupal 8 site. The repository contains the code, but for running
tests in a realistic environment you need:

##### 1. A recent copy of the production environment's database

If you have Drush site aliases, then at the CircleCI dashboard go to the project's permissions
and add an SSH key. Next, add `drush @my.alias sql-cli` to the Behat job at `.circleci/config.yml`.

Alternatively, upload a [sanitized](https://drushcommands.com/drush-8x/sql/sql-sanitize/) database
dump somewhere. For example [the demo project uses a Dropbox URL](https://github.com/juampynr/drupal8-circleci/blob/master/.circleci/config.yml#L83)
via an environment variable which is set at the Circle CI web interface like in the following
screenshot:

![CircleCI database via environment variable](docs/images/circleci-db-env.png)

##### 2. The production environment's files directory

If you have a site alias, then add `drush rsync @my.alias @self` to the Behat job. Alternatively,
use [Stage File Proxy](https://www.drupal.org/project/stage_file_proxy) module.

### [Travis CI](https://travis-ci.org)

Open a terminal and run the installer from the root of your project:
```bash
curl -L https://github.com/lullabot/drupal8ci/raw/master/setup-travisci.sh | bash
```

Sign up at [Travis CI](https://travis-ci.com/) and allow access to your project's repository:

![Travis watch](docs/images/travis-watch.png)

Happy CI-ing! :-D. From now on every pull request will trigger a build in Travis and its
progress will be visible like in the following screenshot:

![Travis pull request](docs/images/travis-pr.png)

For you to see the result of the individual jobs, you need to click at the Details link
from the above screenshot:

![Travis CI jobs](docs/images/tracis-jobs.png)

#### Setting up the Behat job

The Behat job requires, in order to test the behavior of your project:

##### A recent copy of the production environment's database

If you have Drush site aliases, and your repository is private, then follow these
instructions to [add an SSH key](https://docs.travis-ci.com/user/private-dependencies/#User-Key).
Next, set up a drush site alias. Finally, adjust the Behat job to run `drush @my.alias sql-cli`.

Alternatively, upload a [sanitized](https://drushcommands.com/drush-8x/sql/sql-sanitize/) database
dump somewhere and set up the environment variable so the job can download it. For example
[the demo project uses a Dropbox URL](https://github.com/juampynr/d8cidemo/blob/master/.circleci/config.yml#L70)
via an environment variable referenced below:

![Travis CI db env var](docs/images/travisci-db-var.png)

##### 2. The production environment's files directory

If you have a site alias, then add `drush rsync @my.alias @self` to the Behat job. Alternatively,
use [Stage File Proxy](https://www.drupal.org/project/stage_file_proxy) module.

### [GitLab CI](https://about.gitlab.com/features/gitlab-ci-cd/)

Open a terminal and run the installer from the root of your project:
```bash
curl -L https://github.com/lullabot/drupal8ci/raw/master/setup-gitlabci.sh | bash
```

Follow the installation steps at the [GitLab CI website](https://about.gitlab.com/installation/) for completing the setup.


## Troubleshooting

### Class "\Drupal\Tests\Listeners\DrupalStandardsListener" does not exist

If you get this error at the unit and kernel tests jobs, then it means that your
project uses Drupal 8.5 or newer, which introduced a few changes at `web/core/phpunit.xml.dist`.

To fix this, overwrite `.circleci/config/phpunit.xml` with `.circleci/config/phpunit-drupal-8.5.xml`.
