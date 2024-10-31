# Docker PHP images
Specific images for PHP that contain the necessary PHP extensions for running Modern Drupal.

## Develop Version
The images with `-dev` contain xdebug and other local development tools.
To enable xdebug tracing, set the environment variable in docker run or docker compose

`PHP_XDEBOG_MODE=develop,debug`
`PHP_XDEBUG_IDEKEY="myIdeKey"`

### Updates/New Version
When a new Major/Minor version of PHP is released,
a new folder should be created with that Version and a corresponding Dockerfile created.
The GitHub Actions will need to be updated, the action uses a Matrix strategy to simplify building multiple versions.
Update the version array with the newly created version, and GitHub Actions will build and tag the images accordingly.

#### Testing Actions
To test the GitHub Actions YAML, the tool [act](https://github.com/nektos/act) can be used locally.
This will require setting the environment variables DOCKERHUB_USERNAME, DOCKERHUB_TOKEN, and GITHUB_TOKEN.
An example of passing multiple secrets to be available to the actions:
```shell
act -s DOCKERHUB_USERNAME=USER -s DOCKERHUB_TOKEN=TOKEN -s GITHUB_TOKEN=GH_TOKEN
```
