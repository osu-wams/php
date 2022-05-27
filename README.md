# php
Docker image for base

To enable xdebug tracing

set the enviromnet variable in docker run or docker compose

`XDEBOG_MODE=develop,debug`
`XDEBUG_CONFIG="xdebug.start_with_request=yes"`


You can also set xdebug to only run on trigger with
`XDEBOG_MODE=develop,debug`
`XDEBUG_CONFIG="xdebug.start_with_request=trigger xdebug.idekey=PHPSTORM"`