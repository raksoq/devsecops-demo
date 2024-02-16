## DockerHub

### password
Password needs to be in quotes '' since # was part of the password and breaks everything.


```bash
kubectl create secret docker-registry regcred \
--docker-server=https://index.docker.io/v1/ \
--docker-username=name \
--docker-password='pass' \
--docker-email=my@iemail.com \
-n ci 
```