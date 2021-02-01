----¿Qué es Kubernetes?----

Container Orchestration Tool  that helps you manage
containerized applications in different environments

Surge con el avance de micro-servicios y subsecuente
aumento de aplicaciones en contenedores.

No Downtime; Scalability; Disaster Recovery;
esto es lo que las herramientas de orquestación
de contenedores ofrecen.

----Componentes principales de Kubernetes---

-	Unidad básica: “Pod”. Es una abstracción sobre el
contenedor, para no trabajar directamente con la
aplicación de contenedor como Docker o la que sea.
Una aplicación por Pod.
	Cada Pod tiene su propia IP en Kubernetes.
	Las Pod son “efímeras”, esto es, pueden morir
espontáneamente.
	Cuando ocurra, el nuevo Pod tendría una IP distinta,
así que se usan
	“Service” en vez de IP. El Service es permanente y
es el mismo para las nuevas Pod creadas tras la muerte
de una.
	Los Services pueden ser Externos (accesibles por
fuentes externas) o Internos (no accesibles externamente
, útil para las bases de datos por 	ej.). Las URLs de
los Servicios Externos son bonitas por un “Ingress”
que traduce la IP del Servicio Externo.
	Los Services también son "Load Balancers", i.e,
escogen la Pod más ociosa para cada pedido del mismo
Servicio.

-	ConfigMap: configuración externa de la aplicación,
por si cambia al desplegar algo que la aplicación usa,
no tener que rebuild todos los contenedores necesarios,
sino simplemente cambiar el ConfigMap.

-	Secret: igual que ConfigMap pero para mantenerlo
encriptado, como passwords, usuarios y tal.

-	Volumes: hacer que los datos almacenados en la Pod
de la base de datos sean persistentes pese a la efemeri-
dad de las Pod. Los Volumes adjuntan los datos a un
almacenamiento físico, así que si el servicio reinicia o
la Pod peta, los datos persisten en un hardware.

-	Deployments: si la Pod de la aplicación peta, el ser
vicio se cae. Una solución es replicar esta Pod en otra
conectada al mismo Service. Un Deployment es un prototi-
po para esto, donde eliges la cantidad de clones de la
Pod. Es una abstracción por encima de la Pod. Las bases
de datos no se pueden replicar así, pues habría que tra-
tar inconsistencias de datos (Leer y escribir a la vez).

-	StatefulSet: Deployments para bases de datos o apli-
caciones que dependen de un estado.

----Arquitectura de Kubernetes----

Hay 2 tipos de nodos: Maestro y Esclavo.

-	Esclavo: tiene varios Pods. Realiza el trabajo de
las aplicaciones. Necesita de 3 procesos:
		-   Kubelet: interfaz contenedor-nodo. Inicializa
		el Pod con el contenedor dentro.
		-   Kube Proxy: forwards the requests entre
		Service y Pods, de forma que se minimizan los
		cambios de Nodos para las réplicas.
		-   Container Runtime independiente.
- Maestro: tiene 4 procesos que controlan el estado del
cluster y los nodos esclavo. Necesitan menos recursos.
		-   API server: interfaz usuario-cluster. Gate-
		keeper de autentificación.
		-   Scheduler: decide en qué Nodos
		según planificación inteligente los Pods pedidos
		serán iniciados por Kubelet.
		-   Controller Manager: detecta cambios de
		estado en Pods y hace pedidos a Scheduler de re-
		inicio por ejemplo.
		-   etcd: key-value store of a cluster store.
		Guarda los cambios realizados en el cluster.
		Las planificaciones del Scheduler usan esto
		para la decisión. No guarda las bases de datos.

----Minikube y Kubectl----

Habrá varios nodos esclavo y maestro en un setup de pro-
ducción. Puede ser complicado. Hay una herramienta Open,
Minikube, de forma que los procesos Maestro y Esclavo
están en el mismo nodo, con un contenedor Docker preins-
talado, ejecutándose en Virtual Box.
	MiniKube: un cluster de K8s de un nodo, ejecutado en
Virtual Box, para propósitos de tests. Tiene kubctl
como dependencia.
Cómo interactuar con el cluster? Con kubectl.
	kubectl: command line tool for K8s clusters, que
	sirve para comunicarse con la API Server. Es un
	cliente poderoso.
brew update
brew install hyperkit
brew install minikube
minikube start --vm-driver=hyperkit

----kubectl commands----

kubectl get all
kubectl create deployment NAME --image=IMAGE
	Crea Deployment desde IMAGE
kubectl get deployment
kubectl get pod
kubectl describe pod PODNAME
kubectl get replicaset
kubectl edit deployment NAME
kubectl logs PODNAME
kubectl exec -it PODNAME -- bin/bash
kubectl delete deployment NAME
kubectl apply -f CONFNAME
kubectl get deployment NAME -o yaml > deployconf.yaml
kubectl delete -f conf.yaml

----Configuration file syntax----

.yaml file -> indentación estricta. (Validación online
si es muy grande?) Se suelen guardar con el código de
despliege del servicio.
Tienen 3 partes. Conectan deployments a servicios a pods
	-   apiVersion y qué crea (kind)
	-   Metadata
	-   Especificación, específicas para cada kind.
	-   Status, añadido automáticamente por K8s, para
	comprobar que todo va bien con respecto a las spec.
Hay configuration files dentro de configuration files,
para definir Pods bajo Deployments por ej.
Relaciones Labels(en metadata) <-> selector(en spec)
	Service selector <-> Deployment label
	Pods selector matchLabels <-> Deployment label
labels:
	app: name
selector:
	app: name	
ports: comunicación entre los Servicios. En app conf.
	port: de DB a appPod; targetPort: appPod a container

----Demo Project: MongoDB con Mongo Express----
 
 Deployments: obtiene DB Url, DB User, Db Pwd como 
	env vars desde sus .yaml
 Services: Internal(MongoDB) o External(Para acceder a la
	Mongo Express desde el browser).
 ConfigMap: tiene por ej. la DB Url
 Secret: tiene por ej. el DB User y DB Pwd

----Persiting data with Persisting Volume----

Para DBs o directorios preconfigurados.
	Que no dependa del status de los Pods
	Que esté disponible en todos los nodos
	Que sobreviva incluso si todo el cluster se cae
Persistent Volume es otro tipo de recurso.
Creado en el YAML.
Los Volumes locales no cumplen 2 y 3 => Mejor Volumes 
externos (cloud) para DBs.


- ¿FTPS? -> curl ftp://ID:PW@IP/PATH -T [upload filename]; curl ftp://ID:PW@IP/PATH/File -o [download file name]
- ssh-keygen -R IP

General instructions
General instructions
- Check if all the necessary configuration files of your application are in the folder srcs.
- Check if the setup.sh file is at the root of the repository.
- Execute "setup.sh"
Mandatory part
This project consist of clusturing an application and deploying it with Kubernetes. Check if all the next points are respected. At the first error, you stop the correction and put a zero.
Services environment
- Verify if the application is deployed with all containers only with "setup.sh" script.
- Check using the dashboard if the evaluated has all the different containers.
They are nginx, ftps, wordpress, mysql, grafana and influxDB.
These containers must have the same name. If this is not the case or if a container is missing, the correction stops.
- Check if all containers are build with Alpine Linux. If not, the correction stops here.
- Also, check that each container has its Dockerfile, which was built using setup.sh.
The evaluated have to build himself the images that he will use. It is forbidden to take already build images
or use services like DockerHub.
If one is missing, or if one does not start with "FROM: alpine" or any other local image, the correction stops here.
Expose it !
- Check that redirections to your services are done with a Load Balancer.
To do that use "kubectl get services":
ftps, grafana, nginx & wordpress must be "LoadBalancer" type and EXTERNAL-IP field cannot be 127.0.0.1 or end with .0 or .255.
influxdb & mysql must be "ClusterIP" type.
Other entry can appear, but none must be "NodePort" type.
Every EXTERNAL-IP must be unique.
Also, check that "setup.sh" file don't use any "kubectl port-forward" command.
Nginx
- Try to access http (port 80) and verify that you are automatically redirected to https (port 443).
Then, execute the command "curl -I http://IP" and check if return code is a
"301 Moved Permanently" and the "Location" line is the same IP but in https.
Page displayed does not matter, as long it's not a web error (404, 503 etc).
SSL certificate are not necessarily recognized, certificate error on https is normal.
If one point listed previously differ, the correction stops.
FTP(s) server
- Check if the FTPS server is listening on port 21. Make sure it is a FTPS server (s for secure)
and not a basic FTP server.
If the FTPS server does not work as it should, the correction stops here.
- Check if you can upload and download files without any errors.
Hello Word(Press), MySQL and PhpMyAdmin
- Check if the WordPress website works under port 5050 on his IP. Connect to it using the administrator account,
check if several users is present, after leave a comment under post. Make sure your comment is added to the database entries.
For that, you can access PhpMyAdmin interface, must be under port 5000 on his IP and check the database ("wp_comments" table).
The database must persist in case failure.
To test this, you can remove the MySQL container with the Kubernetes dashboard.
It must recreate itself automatically, after check if your comment is still in the database.
If something does not work as expected, the correction stops here.
Grafana and InfluxDB
- Check if grafana is running under port 3000 on his IP. Connect to the interface.
You have to check if grafana is monitoring all containers. To do that search for the dashboard list,
click on the dashboards one by one.
Like MySQL database, after deleting the InfluxDB container, check that the data prior to deletion is still present in Grafana.
If something does not work as expected, the correction stops here.
Hello SSH
- Check if you have SSH access to the Nginx server with the same IP as http & https.
If not, the correction stops here.
Persistence!
- Check that in the event of a crash/shutdown of one of the services, the associated container relaunches correctly.
 To do this, use "kubectl exec deploy/SERVICE -- pkill APP" (replacing SERVICE and APP of course).
 Stop web containers (usually nginx or php-fpm APP) and stop grafana APP for this same container.
 Stop the ftps server (usually vsftpd APP).
 Stop mysqld and influxd APP for corresponding databases (recheck if the data persists).
 Stop the ssh server on the Nginx container (sshd APP).
 If a container hasn't relaunched after several minutes, if the MySQL or InfluxDB data is lost
 or if a service has restarted badly (in particular ftps), the correction stops here.
