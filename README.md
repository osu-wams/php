# php
Docker image for base

To enable xdebug tracing

set the enviromnet variable in docker run or docker compose


`XDEBUG_CONFIG="xdebug.mode=develop,debug xdebug.start_with_request=yes"`


You can also set xdebug to only run on trigger with

`XDEBUG_CONFIG="xdebug.mode=develop,debug xdebug.start_with_request=trigger xdebug.idekey=PHPSTORM"`