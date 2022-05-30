1. Install Allure reports server as per https://github.com/eformat/allure
```
helm upgrade myallure --set security.user=admin --set security.password=changeme --create-namespace --namespace=allure --install .
```
2. Create allure-auth secret in sample-app-cicd project
```
kind: Secret
apiVersion: v1
metadata:
  name: allure-auth
stringData:
  password: changeme
  username: admin
type: Opaque
```
3. Modify zap proxy task to connect to your allure instance using created secret

4. After zap-proxy task will finish successfully your report will be available here:
```
https://<ALLURE_HOST>/allure-docker-service/projects/zap-scan/reports/latest/index.html
```
