[![](https://images.microbadger.com/badges/version/ymedlop/npm-cache-resource:0.10.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.10 "Get your own version badge on microbadger.com") [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:0.10.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.10 "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/commit/ymedlop/npm-cache-resource:0.10.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.10 "Get your own commit badge on microbadger.com") [![](https://images.microbadger.com/badges/license/ymedlop/npm-cache-resource:0.10.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.10 "Get your own license badge on microbadger.com")

npm-cache-resource
==================

a Concourse resource for caching dependencies downloaded by NPM - built on [mhart/alpine-node](https://hub.docker.com/r/mhart/alpine-node).

[![EXAMPLE PIPELINE](https://raw.githubusercontent.com/ymedlop-sandbox/npm-cache-resource/master/images/example-pipeline.png)](https://raw.githubusercontent.com/ymedlop-sandbox/npm-cache-resource/master/images/example-pipeline.png)


Versions
--------

* [latest](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/master/Dockerfile) ( npm 4.0.5 - alpine-node:latest ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource "Get your own image badge on microbadger.com")
* [7](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/alpine-node-v7/Dockerfile) ( npm 4.0.5 - alpine-node:7 ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:7.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:7 "Get your own image badge on microbadger.com")
* [6](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/alpine-node-v6/Dockerfile) ( npm 3.10.10 - alpine-node:6 ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:6.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:6 "Get your own image badge on microbadger.com")
* [4](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/alpine-node-v4/Dockerfile) ( npm 2.15.11 - alpine-node:4 ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:4.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:4 "Get your own image badge on microbadger.com")
* [0.12](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/alpine-node-v0.12/Dockerfile) ( npm 2.15.11 - alpine-node:0.12 ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:0.12.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.12 "Get your own image badge on microbadger.com")
* [0.10](https://github.com/ymedlop-sandbox/npm-cache-resource/blob/alpine-node-v0.10/Dockerfile)( npm 2.15.11 - alpine-node:0.10 ) [![](https://images.microbadger.com/badges/image/ymedlop/npm-cache-resource:0.10.svg)](https://microbadger.com/images/ymedlop/npm-cache-resource:0.1' "Get your own image badge on microbadger.com")


Resource Configuration
----------------------

```
resource_types:

  - name: npm-cache
    type: docker-image
    source: {repository: ymedlop/npm-cache-resource, tag: 0.10}
```


Source Configuration
--------------------

* `<<:`: *Required.* The source is the same as the corresponding git resource

### Configuration to access a Git Private Repo

#### SSH:

* `private_key`: *Required.* Private key.

#### HTTP(S):

* `username_key`: *Required.* Username for HTTP(S) auth. This is needed when only HTTP/HTTPS protocol for git is available (which does not support private key auth) and auth is required.
* `password_key`: *Required.* Password for HTTP(S) auth.
* `skip_ssl_verification`: *Optional.* Skips git ssl verification by exporting GIT_SSL_NO_VERIFY=true.

### Configuration to access a NPM Registry or Private Registry by User and Password.

* `registry-url`: *Optional.* Private NPM registry to log in to (Default: https://registry.npmjs.org)
* `registry-user`: *Required.* Registry Username.
* `registry-pass`:  *Required.* Registry User Password.
* `registry-email`: *Required.* Registry User Email.
* `registry-scope`: *Optional.* Registry Scope.

### Configuration to access a Private Registry by Base64 Token

* `registry`: *Required.* The location our private npm registry.
* `token`: *Required.* Our npm token.

```
Whatever tool you use to generate the encoded username and password string, try to encode the string admin:admin123, which should result in YWRtaW46YWRtaW4xMjM=. `
Another example for a valid setup is jane:testpassword123 resulting in amFuZTp0ZXN0cGFzc3dvcmQxMjM=.

In our demo we are using admin:123456 resulting in YWRtaW46MTIzNDU2
```

### Example

```
resources:

  # a perfectly normal source repository with lashings and lashings of dependencies
  - name: react-redux-badges-repo
    type: git
    source: &react-redux-badges-repo-source # apply a YAML anchor so we can refer to this in the cache resource
      uri: https://github.com/ymedlop-sandbox/react-redux-badges.git

  # a resource caching the dependencies listed in the source repository
  - name: npm-repo-cache
    type: npm-cache # as defined above
    source:
      <<: *react-redux-badges-repo-source # the source is the same as the corresponding git resource ...
      paths: # ... except that it's only interested in files listing dependencies
        - package.json
```

## Behavior

### `check`: Check for new commits

The repository is cloned (or pulled if already present), and any commits from the given version on are returned. If no version is given, the ref for HEAD is returned.

Any commits that contain the string [ci skip] will be ignored. This allows you to commit to your repository without triggering a new version.

### `in`: Pulls a package from npm

Clones the repository to the destination, and locks it down to a given ref. It will return the same given ref as version. And fetch npm package from the package.json`.

#### Examples

```
jobs:

  - name: cache
    plan:

      - get: react-redux-badges-repo
        trigger: true

      - get: npm-repo-cache

  - name: test
    plan:

      - get: react-redux-badges-repo
        trigger: true
        passed: [cache]

      - get: npm-repo-cache
        passed: [cache]

      - task: run tests
        config:

          platform: linux
          image_resource:
            type: docker-image
            source: {repository: mhart/alpine-node, tag: "0.10"}

          inputs:
            - name: react-redux-badges-repo
              path: /src
            - name: npm-repo-cache
              path: /cache

          run:
            path: sh
            args:
              - -exc
              - |
                mv cache/node_modules src
                cd src && npm test
```

### `out`: Nothing to do here....

Getting Started
---------------

Examples...

0. [Simple Example](examples/simple/README.md)
1. [Private Registry Example](examples/private-registry/README.md)

Credits:
========

* [projectfalcon/gradle-cache-resource](https://github.com/projectfalcon/gradle-cache-resource) We are following this resource to create our npm cache resource.

License
-------

See the [LICENSE file](LICENSE) for license text and copyright information.