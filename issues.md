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

https://senertugrul.medium.com/how-to-use-kaniko-to-build-docker-images-on-jenkins-216a68caf7b8

https://www.youtube.com/watch?v=EgwVQN6GNJg