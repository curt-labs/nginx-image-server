# Nginx Image Server

Docker Image for [Nginx](http://nginx.org/) server for image processing with [ngx_small_light](https://github.com/cubicdaiya/ngx_small_light).
It supports resizing/cropping/formatting (`png`, `webp`...etc) of images stored in local storages or AWS S3.

Please see https://github.com/cubicdaiya/ngx_small_light for more information about image processing.

## SUPPORTED TAGS

* `latest`
 * Nginx 1.10.0
 * ngx_small_light 0.8.0
 * ImageMagick 6.8.6-8 (Q16) with WebP support

## HOW TO USE

```bash
# Get the docker image
$ docker pull us.gcr.io/curtmfg/nginx-image-server

# Start the image server
$ docker run \
    --rm \
    -it \
    --name nginx-image-server \
    -p 80:80 \
    -p 8090:8090 \
    -e "SERVER_NAME=image.example.com" \
    us.gcr.io/curtmfg/nginx-image-server:latest
```

Then you can try image-processing by accessing

* **Images in S3**: `http://<YOUR-SERVER.com>/small_light(dh=400,da=l,ds=s)/<PATH-TO-IMAGE-IN-S3>`
* **Images in Local**: `http://<YOUR-SERVER.com>/local/small_light(dh=400,da=l,ds=s)/images/example.jpg`

And `http://<YOUR-SERVER.com>:8090/status` retruns the nginx status.

### Custom configuration
You can build a docker image includes your own `nginx.conf`:

```
FROM us.gcr.io/curtmfg/nginx-image-server
COPY nginx.conf /etc/nginx/nginx.conf
```

Then build with `docker build -t your-nginx-image-server .` and run:

```bash
$ docker run \
    -d \
    --name your-nginx-image-server \
    -p 80:80 \
    your-nginx-image-server
```

Be sure to include `daemon off;` in your custom configuration to run Nginx in the foreground.
Otherwise your container will stop immediately after starting.

## HOW TO DEVELOP

```bash
# on your local machine
$ git clone https://github.com/curtmfg/nginx-image-server.git && cd nginx-image-server
$ script/bootstrap
$ cp .env.sample .env
```

```
# .env
TIMEZONE=Asia/Tokyo
```

```
$ vagrant up
$ vi Dockerfile

# login to VM and test it
$ vagrant ssh
@core-01 $ cd share
@core-01 $ docker build -t=us.gcr.io/curtmfg/nginx-image-server .
@core-01 $ script/test
```


### Feature(behavior) test
Behavior test with [infrataster](https://github.com/ryotarai/infrataster).
Test files are under `test/feature` directory. You can run this test with follwing script:

```bash
$ script/test
```

### Performance test
Performance test with [locust](http://locust.io/).
Test files are under `test/performance` directory. You can run locust with follwing script:

```bash
# Run target container
$ script/run

# Export target IP
$ export TARGET_IP=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' nginx-image-server)

# Run locust as WebTool
$ script/run-locust -f locustfile.py -H http://${TARGET_IP}

# Run locust as CLI
$ script/run-locust -f locustfile.py -H http://${TARGET_IP} --no-web -c 5 -r 1 -n 10
```

## LICENSE
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)
