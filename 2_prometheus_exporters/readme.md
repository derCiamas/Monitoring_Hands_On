# HOW TO
## Exporters config
There are two paths you can follow in order to change the Prometheus exporters configuration. First one, where you are the only one working currently on config maps and the second where there are many collaborators working in parallel.

### Working solo
1) Open the prometheus-custom-exporters map and edit it to fit your needs. The example file can be found under *./working_solo/prometheus-custom-exporters.yaml* . Save the file and close after editing.
2) In order to let Prometheus know that the configuration has changed you need to send a request similar to the following one:
```
POST <your4WheelsClusterUrl>/prometheus/-/reload 
Authorization: Basic username:password
```
Alternatively you can restart the pod (will take Prometheus down for a couple of minutes):
```
kubectl scale statefulsets prometheus --replicas=0 -n kube-devops
kubectl scale statefulsets prometheus --replicas=1 -n kube-devops
```

### Working in parallel

1) In *./working_in_parallel/exporters_config*, create a copy of *example_exporter_ignore.yaml* and rename it to <yournameofchoice>_exporter.yaml . You can create as many *_exporter.yaml files as you wish. These will be merged in the next step.
2) In order to create config map from many files invoke the following (if the configmap exists already you can remove it with ```kubectl delete cm prometheus-custom-exporters``` ). If working in repo it might be wise to pull/fetch the newest repo stand to provision all the required maps and not only your local ones:
```
kubectl create cm prometheus-custom-exporters --from-file=<path_to_exporter_config_directory>
```
Invoking the command will create the prometheus-custom-exporters config map with all file contents from the exporters_config directory.

### What happens under the hood?
So how does the Prometheus know where to read the exporters info from?

Basically, the config map we've just created is mapped in the Prometheus'es stateful set as a drive where all the file we have created in the exporters_config are being created. You can check it by invoking

```kubectl get statefulset prometheus -o yaml```

Look there for the following:
```yaml
        (...)
        - mountPath: /etc/customConfig
          name: custom-config
      (...)
      volumes:
      - configMap:
          defaultMode: 420
          name: prometheus-config
        name: config-volume
      - name: custom-config
        projected:
          defaultMode: 420
          sources:
          - configMap:
              name: prometheus-custom-exporters
              optional: true
          - configMap:
              name: prometheus-custom-alerts
              optional: true
  (...)
```

The Prometheus itself uses the prometheus-config map. Let's check it (as the file is pretty hard to read in the console we will download it to the local file):
```kubectl get configmap prometheus-config -o yaml > prometheus-config.yaml```

You can now open the file in Visual Studio Code with: 
```code  .\prometheus-config.yaml```
Look there for the following part:
```
 scrape_configs:
    - job_name: customconfigurations
      tls_config:
          insecure_skip_verify: false
      file_sd_configs:
      - files:
        - /etc/customConfig/*_exporter.yaml
```
We tell the Prometheus to read the exporters for a scraping job named *customconfigurations* from the files located in */etc/customConfig/* (our mounted volume), where the file name contains *_exporter.yaml*.