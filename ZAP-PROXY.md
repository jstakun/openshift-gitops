1. Install Allure reports server as per https://github.com/eformat/allure
```
helm upgrade myallure --set security.user=admin --set security.password=changeme --create-namespace --namespace=allure --install .
```
2. In sample-app-cicd project create allure-auth secret
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
3. Modify zap proxy task to connect to your allure instance and secret

4. After task will finish successfully you report will be availabel here:
```
https://<ALLURE_HOST>/allure-docker-service/projects/zap-scan/reports/latest/index.html
```
