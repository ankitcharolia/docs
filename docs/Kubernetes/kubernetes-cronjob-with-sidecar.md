---
layout: default
title: Gracefully Exit Google cloud-sql-proxy Sidecar Container with Kubernetes Cronjob
parent: Kubernetes
nav_order: 1
---

# How to gracefully exit google cloud-sql-proxy container with kubernetes cronjob

NOTE: `The Proxy includes support for an admin server on localhost. By default, the the admin server is not enabled. To enable the server, pass the --debug or --quitquitquit flag. This will start the server on localhost at port 9091`

 **The admin server exits gracefully when it receives a POST request at /quitquitquit. `curl -X POST localhost:9091/quitquitquit`**

```shell
apiVersion: batch/v1
kind: CronJob
metadata:
  name: "release-name-cronjob-example"
  labels:
    helm.sh/chart: cronjob-0.0.1
    app.kubernetes.io/managed-by: Helm
    app: release-name
    app.kubernetes.io/name: release-name
    app.kubernetes.io/instance: release-name
    version: "latest"
    app.kubernetes.io/version: "latest"
spec:
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        metadata:
          labels:
            app: release-name
            cron: cronjob-example
        spec:
          terminationGracePeriodSeconds: 30
          serviceAccountName: release-name
          securityContext:
            {}
          volumes:
            - name: nginx-conf
              configMap:
                name: release-name-nginx-config
            - name: data-www-public
              emptyDir: {}
          containers:
          - name: cloud-sql-proxy
            image: eu.gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.2.0-alpine
            args:
              - --private-ip
              - --address=0.0.0.0
              - --quiet
              - --quitquitquit
              - --structured-logs
              - --health-check
              - --http-port=9090
              # Tell the proxy to exit gracefully if it receives a SIGTERM
              - --exit-zero-on-sigterm
              - --max-sigterm-delay=20s
              - --run-connection-test
              - --port=3306
              - dummy-project:europe-west3:dummy-instance-name
            # Configure kubernetes to call the /quitquitquit endpoint on the
            # admin server before sending SIGTERM to the proxy before stopping
            # the pod. This will give the proxy more time to gracefully exit.
            lifecycle:
              preStop:
                httpGet:
                  path: /quitquitquit
                  port: 9091
                  scheme: HTTP
            startupProbe:
              # The /startup probe returns OK when the proxy is ready to receive
              # connections from the application. In this example, k8s will check
              # once a second for 20 seconds.
              # We strongly recommend adding a startup probe to the proxy sidecar
              # container. This will ensure that service traffic will be routed to
              # the pod only after the proxy has successfully started.
              httpGet:
                path: /startup
                port: 9090
              periodSeconds: 1
              timeoutSeconds: 5
              successThreshold: 1
              failureThreshold: 20
            livenessProbe:
              # The /liveness probe returns OK as soon as the proxy application has
              # begun its startup process and continues to return OK until the
              # process stops.
              # We recommend adding a liveness probe to the proxy sidecar container.
              httpGet:
                path: /liveness
                port: 9090
              # Number of seconds after the container has started before the first probe is scheduled. Defaults to 0.
              # Not necessary when the startup probe is in use.
              initialDelaySeconds: 0
              # Frequency of the probe.
              periodSeconds: 10
              # Number of seconds after which the probe times out.
              timeoutSeconds: 10
              # Number of times the probe is allowed to fail before the transition
              # from healthy to failure state.
              #
              # If periodSeconds = 60, 5 tries will result in five minutes of
              # checks. The proxy starts to refresh a certificate five minutes
              # before its expiration. If those five minutes lapse without a
              # successful refresh, the liveness probe will fail and the pod will be
              # restarted.
              failureThreshold: 5
            readinessProbe:
              # The /readiness probe returns OK when the proxy can establish
              # a new connections to its databases.
              # Please use the readiness probe to the proxy sidecar with caution.
              # An improperly configured readiness probe can cause unnecessary
              # interruption to the application. See README.md for more detail.
              httpGet:
                path: /readiness
                port: 9090
              initialDelaySeconds: 10
              periodSeconds: 10
              timeoutSeconds: 10
              # Number of times the probe must report success to transition from failure to healthy state.
              # Defaults to 1 for readiness probe.
              successThreshold: 1
              failureThreshold: 6
            resources:
              requests:
                memory: "64Mi"
                cpu: "100m"
            securityContext:
              runAsNonRoot: true
              privileged: false
              allowPrivilegeEscalation: false
              readOnlyRootFilesystem: true
              capabilities:
                drop:
                  - ALL
          - image: 'gcr-repository/nginx:latest'
            imagePullPolicy: Always
            name: cronjob-example
            env:
              - name: MYSQL_DB_NAME
                value: example_db
          # env vars coming from values.yaml
              - name: ADMIN_EXAMPLE_URI
                value: http://example.com
            envFrom:
              - secretRef:
                  name: release-name
            lifecycle:
              preStop:
                exec:
                  # Wait before issuing SIGTERM to the application so that
                  # Kubernetes itself has enough time to stop all active connections
                  # and incoming requests from reaching the application.
                  # See: https://learnk8s.io/graceful-shutdown
                  command: ["/bin/sleep", "10"]
            command: [/bin/bash]
            args:
            - -c
            - curl -4 -s -XPUT http://ek8s-service:9001/token/refresh && curl -X
              POST localhost:9091/quitquitquit
            resources:
              {}
          restartPolicy: Never
  schedule: "*/5 * * * *"
  successfulJobsHistoryLimit: 1
```

## successfully terminate sidecar container
```shell
# reference: https://stackoverflow.com/questions/41679364/kubernetes-stop-cloudsql-proxy-sidecar-container-in-multi-container-pod-job/76989549#76989549
<cronjob-commands>;exit_code=$?; curl -X POST localhost:9091/quitquitquit && exit $exit_code
```

## References
* [Use Sidecard native feature](https://github.com/GoogleCloudPlatform/cloud-sql-proxy/issues/128#issuecomment-2264787327)