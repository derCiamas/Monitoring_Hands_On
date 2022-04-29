## Initial configuration

### 1) Add configmaps (prometheus-custom-alerts.yaml and prometheus-custom-exporters.yaml):
```
kubectl apply -f <pathTo_prometheus-custom-alerts.yaml> -n kube-devops
kubectl apply -f <pathTo_prometheus-custom-exporters.yaml> -n kube-devops
```
### 2) Edit the "prometheus-config" config map and add to scrape configs:
```
kubectl edit cm prometheus-config -n kube-devops --save-config
```
- find the 'rule_files' and add the following line
```yaml
- /etc/customConfig/customAlertRules.yaml
```
the result should like similar to this:
```yaml
rule_files:
    - /etc/config/rules.yaml
    - /etc/config/alerts.yaml
    - /etc/customConfig/customAlertRules.yaml
```
- look for 'scrape_configs:' (should be just below 'rule_files') and insert at the beginning:
```yaml
 - job_name: customconfigurations
      tls_config:
          insecure_skip_verify: true
      file_sd_configs:
        - files:
          - /etc/customConfig/*_exporter.yaml
```
### 3) Edit the "prometheus" statefulset (modifying takes Prometheus down for a couple of minutes after the changes are saved)
```
kubectl edit statefulset prometheus -n kube-devops --save-config
```
- add to "volumes"
```yaml
      - name: custom-config
        projected:
          sources:
          - configMap:
              name: prometheus-custom-exporters
              optional: true
          - configMap:
              name: prometheus-custom-alerts
              optional: true
```
- add to pods.volumeMounts":
```yaml
      - mountPath: /etc/customConfig
        name: custom-config
```


## Updates
Whenever you change the *prometheus-custom-alerts.yaml* or *prometheus-custom-exporters.yaml* you can redeploy the file with:
```
kubectl apply -f <filePath> -n kube-devops
```

In order to let Prometheus know that the configuration has changed you need to send a request similar to the following one:
```
POST <your4WheelsClusterUrl>/prometheus/-/reload 
Authorization: Basic username:password
```
Alternatively you can restart the pod (will take Prometheus down for a couple of minutes):
```
kubectl scale statefulsets prometheus --replicas=0 -n kube-devops
kubectl scale statefulsets prometheus --replicas=1 -n kube-devops
```