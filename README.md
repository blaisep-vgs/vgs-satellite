<p align="center"><a href="https://www.verygoodsecurity.com/"><img src="https://avatars0.githubusercontent.com/u/17788525" width="128" alt="VGS Logo"></a></p>
<p align="center"><b>@vgs/vgs-satellite</b><br/>VGS Offline integration application.</p>
<p align="center">
<a href="https://circleci.com/gh/verygoodsecurity/vgs-satellite/tree/master"><img src="https://circleci.com/gh/verygoodsecurity/vgs-satellite/tree/master.svg?style=svg" alt="circleci-test"></a>
<a href="https://badge.fury.io/js/%40vgs%2Fvgs-satellite"><img src="https://badge.fury.io/js/%40vgs%2Fvgs-satellite.svg" alt="npm-version"></a>
<a href="https://badge.fury.io/js/%40vgs%2Fvgs-satellite"><img src="https://img.shields.io/npm/dw/@vgs/vgs-satellite?style=flat-square" alt="npm-downloads"></a>
<a href="https://opensource.org/licenses/ISC"><img src="https://img.shields.io/npm/l/@vgs/vgs-satellite?style=flat-square" alt="NPM"></a>
</p>
[![OpenFaaS](https://img.shields.io/badge/openfaas-cloud-blue.svg)](https://www.openfaas.com)


<!-- toc -->
* [Description](#description)
* [Environment variables](#environment-variables)
* [Running from github](#running-from-github)
* [Node Package Manager](#node-package-manager)
* [Docker image](#docker-image)
* [How to run](#how-to-use)
* [Mitmproxy](#running-mimtmproxy-separately)
<!-- tocstop -->

## Description

VGS Satellite is an  application for offline integration with Very Good Security.
This  application gives you an ability to run requests with your service and transform them into suitable VGS route configuration
without any need to sign up. VGS Satellite consists of two parts: UI application for building routes configurations based on HAR files 
and additional python script for mitmproxy, that transforms intercepted requests into JSON-HAR format per requests.

You can choose between different ways to run VGS Satellite:
1. Run vgs-satellite UI (with mitmproxy) from source provided on [Github](#running-from-github)
1. Install vgs-satellite UI via [NPM](#node-package-manager) and run mitmproxy with [script](https://github.com/verygoodsecurity/vgs-satellite/blob/master/script/mitm-requests-json.py) from sources
1. Run vgs-satellite in [docker](#docker-image) from quay and run mitmproxy with [script](https://github.com/verygoodsecurity/vgs-satellite/blob/master/script/mitm-requests-json.py) from sources

## Environment variables

- `PROXY_PORT` - port to run mitmproxy on
- `WEB_PORT` - port to run mitmproxy web UI on
- `SATELLITE_PORT` - port to run vgs-satellite web UI on
- `MITM_DIR` - directory that would be mapped inside containers to store mitmproxy intercepted requests
- `MITM_ARGS` - additional mitmproxy args (e.g. - "--mode reverse:http://localhost:8080/", empty by default)


## Running from github

1. Clone sources
    ```bash
        git clone git@github.com:verygoodsecurity/vgs-satellite.git && cd vgs-satellite
    ```

1. Create or use `.env` file inside application root. For environment variable nomenclature see [here](#environment-variables)
    
1. Run application

    ```bash
   docker-compose up 
   ```
   

## Node package manager
_See package on [npm](https://www.npmjs.com/package/@vgs/vgs-satellite)_

```sh-session
$ npm install -g yarn
$ yarn global add @vgs/vgs-satellite
running command...
$ SATELLITE_PORT=1234 MITM_DIR=/path/to/har/files vgs-satellite 
```

You can override `SATELLITE_PORT` and `MITM_DIR`, for details see [here](#environment-variables)
Note: This scenario does not run mitmproxy. To run mitmproxy separately see [here](#running-mimtmproxy-separately)


## Docker image
_See package on [quay](https://quay.io/repository/verygoodsecurity/vgs-satellite)_

```sh-session
$ docker pull quay.io/verygoodsecurity/vgs-satellite
$ export SATELLITE_PORT=1234
$ export MITM_DIR=/tmp/container_path
running command...
$ docker run -e SATELLITE_PORT -e MITM_DIR -v "/tmp/local_path:${MITM_DIR}" -p $SATELLITE_PORT:$SATELLITE_PORT quay.io/verygoodsecurity/vgs-satellite
```

You can override `SATELLITE_PORT` and `MITM_DIR`, for details see [here](#environment-variables)
Note: This scenario does not run mitmproxy. To run mitmproxy separately see [here](#running-mimtmproxy-separately)

## How to use 

_Note: this manual of how to use vgs-satellite assuming you are running from docker-compose_

1. Run application. 
   We assume that content of `.env` wasn't changed
1. Run example requests:
    ```bash
    curl -H "Content-Type: application/json" -x http://localhost:1230 http://httpbin.org/post -d '{"foo": "bar"}'
    ```
1. Open [http://localhost:1234](http://localhost:1234)
1. Wait for your requests to appear or add your HAR file by clicking `Upload HAR file(s)`

   _Note: you can find HAR file of your requests in /path/to/vgs-satallite/mitm-requests_
   
   ![requests-list](manual/1-requests-list.png)
   
1. Choose your request from the list

   ![requests-detail](manual/2-requests-detail.png)
    
1. Click secure you payload

   ![secure-payload](manual/3-secure-payload.png)
   
1. Check field you would like to secure.

   ![secure-check](manual/4-secure-check.png)

    For additional setting please reference the [nomenclature](https://www.verygoodsecurity.com/docs/terminology/nomenclature)

1. Click `Secure this payload`, then `View route configuration`

   ![route-config](manual/5-route-config.png)
   
1. Download inbound/outbound route and reference instructions provided to import your first route on VGS Dashboard!

## Running mimtmproxy separately

Mitmproxy script is available [here](https://github.com/verygoodsecurity/vgs-satellite/blob/master/script/mitm-requests-json.py)

If you rum mitmproxy separately, use the following command:

```bash
MITM_DIR=/path/to/har/files mitmweb -s script/mitm-requests-json.py
``` 

Note: VGS Satellite UI and mitmproxy additional script uses `MITM_DIR` env variable to sync on directory. Mitmproxy saves HAR-JSON files in `MITM_DIR` and VGS Satellite loads requests from `MITM_DIR`