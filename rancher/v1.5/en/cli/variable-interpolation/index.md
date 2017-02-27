---
title: Variable Interpolation in Rancher CLI
layout: rancher-default-v1.5
version: v1.5
lang: en
redirect_from:
  - rancher/cli/environment-interpolation
---

## Variable Interpolation
---

Using `rancher up`, environment variables from the machine running `rancher` can be used within the `docker-compose.yml` and `rancher-compose.yml` files. This is only supported in `rancher` commands and not in the Rancher UI.  

### How to use it

With the `docker-compose.yml` and `rancher-compose.yml` files, you can reference the environment variables on your machine. If there are no environment variables on the machine, it will replace the variable with a blank string. `Rancher` will provide a warning on which environment variables are not set.  If using environment variables for image tags, please note that `rancher` will not strip the `:` from the image to fetch the latest image. Since the image name, i.e. `<imagename>:` is an invalid image name, no container will be deployed. It's up to the user to ensure that all environment variables are present and valid on the machine.

#### Example

On our machine running `rancher`, we have an environment variable, `IMAGE_TAG=14.04`.

```bash
# Image tag is set as environment variable
$ env | grep IMAGE
IMAGE_TAG=14.04
# Run rancher
$ rancher up
```

**Example `docker-compose.yml`**

```yaml
version: '2'
services:
  ubuntu:
    tty: true
    image: ubuntu:$IMAGE_TAG
    stdin_open: true
```

<br>

In Rancher, an `ubuntu` service will be deployed with an `ubuntu:14.04` image.

### Variable Interpolation Formats

`Rancher` supports the same formats as `docker-compose`.

```yaml
version: '2'
services:
  web:
    # unbracketed name
    image: "$IMAGE"

    # bracketed name
    command: "${COMMAND}"

    # array element
    ports:
    - "${HOST_PORT}:8000"

    # dictionary item value
    labels:
      mylabel: "${LABEL_VALUE}"

    # unset value - this will expand to "host-"
    hostname: "host-${UNSET_VALUE}"

    # escaped interpolation - this will expand to "${ESCAPED}"
    command: "$${ESCAPED}"
```

### Templating

More advanced use cases such as conditional logic are supported by using the [Go template system](https://golang.org/pkg/text/template/).

As an example, the following template will produce a service with `ports` set if the `public` variable is true and `expose` will be set otherwise.


```yaml
version: '2'
services:
  web:
    image: nginx
    {{- if eq .Values.public "true" }}
    ports:
    - 8000
    {{- else }}
    expose:
    - 8000
    {{- end }}
```