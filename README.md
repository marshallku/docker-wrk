# Docker wrk

Minimal Docker image for [wrk](https://github.com/wg/wrk) and [wrk2](https://github.com/giltene/wrk2)

## Usage

### CLI

You can use both `wrk` and `wrk2` images.

```bash
docker run --rm ghcr.io/marshallku/alpine-wrk -t 1 -c 10 -d 5s https://www.example.com/
docker run --rm ghcr.io/marshallku/alpine-wrk2 -t 1 -c 10 -d 5s -R20 https://www.example.com/
```

### Kubernetes

And of course, you can also run it in Kubernetes.

#### Example with GET request

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: wrk-load-test-1
  namespace: wrk
spec:
  template:
    spec:
      containers:
        - name: wrk
          image: ghcr.io/marshallku/alpine-wrk:latest
          args:
            - "-t1"
            - "-c4"
            - "-d180s"
            - "--timeout"
            - "30s"
            - "http://service.namespace.svc.cluster.local/"
          resources:
            requests:
              cpu: "2"
              memory: "1Gi"
            limits:
              cpu: "2"
              memory: "1Gi"
      restartPolicy: Never
  backoffLimit: 4
```

#### Example with POST request

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wrk-scripts
  namespace: wrk
data:
  script.lua: |
    -- Read request body from file
    local file = io.open("/data/request.json", "r")
    wrk.body = file:read("*all")
    file:close()

    wrk.method = "POST"
    wrk.headers["Content-Type"] = "application/json"
    wrk.headers["Accept"] = "*/*"
```

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: wrk-load-test-1
  namespace: wrk
spec:
  template:
    spec:
      initContainers:
        - name: prepare-data
          image: curlimages/curl:latest
          command:
            - /bin/sh
            - -c
            - |
              set -e  # Exit on any error
              echo "Downloading JSON file..."
              curl -f -S -s -o /data/request.json "https://www.example.com/request.json"
              echo "Download complete. Checking file..."
              if [ ! -s /data/request.json ]; then
                echo "Error: Downloaded file is empty"
                exit 1
              fi
              echo "File check passed"
          volumeMounts:
            - name: data-volume
              mountPath: /data
      containers:
        - name: wrk
          image: ghcr.io/marshallku/alpine-wrk:latest
          args:
            - "-t1"
            - "-c4"
            - "-s"
            - "/scripts/script.lua"
            - "-d180s"
            - "--timeout"
            - "30s"
            - "http://service.namespace.svc.cluster.local/"
          resources:
            requests:
              cpu: "2"
              memory: "1Gi"
            limits:
              cpu: "2"
              memory: "1Gi"
          volumeMounts:
            - name: scripts-volume
              mountPath: /scripts
            - name: data-volume
              mountPath: /data
      volumes:
        - name: scripts-volume
          configMap:
            name: wrk-scripts
        - name: data-volume
          emptyDir: {}
      restartPolicy: Never
  backoffLimit: 4
```
