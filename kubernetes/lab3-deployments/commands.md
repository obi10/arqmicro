__Deployment de una aplicacion en OkE a partir de una imagen docker en OCIR__
https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-and-registry/index.html

---------
previamente se debio subir una imagen el OCIR
datos necesarios:
	OCI account: okepro1
	object storage namespace: idtu5rnw5dhj
	user: junior.palomino@oracle.com
	auth token: GRSt;>xF<rnZnPqxP{9K
```sh
$ docker login iad.ocir.io
username: idtu5rnw5dhj/junior.palomino@oracle.com
password: GRSt;>xF<rnZnPqxP{9K

$ docker push iad.ocir.io/idtu5rnw5dhj/httpd-k8s:v1 (respetar esa estructura del nombre de la image - tag)
```
---------



crear otro namespace (exlusivo para este lab - lab3)
```sh
$ kubectl apply -f 00-namespace.yaml
```

crear un secret. A Kubernetes cluster uses the Secret of docker-registry type to authenticate with a container registry to pull a private image.
https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
```sh
$ kubectl -n lab3 create secret docker-registry ocirsecret --docker-server=iad.ocir.io --docker-username='idtu5rnw5dhj/junior.palomino@oracle.com' --docker-password='GRSt;>xF<rnZnPqxP{9K' --docker-email='junior.palomino@oracle.com'
```
realizar el deployment de una aplicacion (en el archivo yaml, se especifica el servicio de lb y el deployment)
```sh
$ kubectl -n lab3 apply -f 01-apache-app.yaml
```

obs: apply -f solo ejecuta cambios de configuracion, no de la propia imagen de docker
Para actualizar la imagen de docker se debe ejecutar el siguiente comando:
```sh
$ kubectl -n lab3 set image deployment/httpd-k8s-deployment httpd-k8s-app=iad.ocir.io/idtu5rnw5dhj/httpd-k8s:v2 --record
```
La imagen iad.ocir.io/idtu5rnw5dhj/httpd-k8s:v2 debe estar en el OCIR
Fuente: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment



----Aux---
Como crear una nueva imagen a partir de la original httpd:latest
realizar un pull de la imagen en la PC local
```sh
$ docker pull httpd
$ docker run -it -d --name apache httpd:latest bash

$ docker exec -it apache bash

modificar el archivo /usr/local/apache2/htdocs/index.html
$ echo 'Hola Junior' > /usr/local/apache2/htdocs/index.html

crear la nueva imagen a partir de la original
$ docker commit -m "index.html modified" -a "junicode" apache iad.ocir.io/idtu5rnw5dhj/httpd-k8s:v2

$ docker images
```
---------
