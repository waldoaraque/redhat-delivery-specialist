# redhat-delivery-specialist
Repo with some code viewed in the course

oc get pods -> ver pods del entorno
	-n (namespace)
	
etcd
Estado actual y deseado contenido en el almacén de datos que usa etcd como almacén de pares de clave-valor distribuidos.

etcd también contiene:

Reglas de RBAC

Información del entorno de la aplicación

Datos de usuarios que no son de la aplicación


Dos implementaciones:

ReplicaSet = Kubernetes

ReplicationController = OpenShift

OLM Operator Lifecycle Manager 

CRD (definición de recursos personalizados...)

Cada proyecto tiene sus propios:

Objetos: pods, servicios, controladores de replicación, etc.

Políticas: reglas que especifican qué usuarios pueden o no realizar acciones en los objetos.

Limitaciones: cuotas para objetos que pueden limitarse.

Cuentas de servicio: usuarios que interactúan automáticamente con acceso a objetos del proyecto.

Se usa la herramienta oc para iniciar sesión en la misma dirección que la consola web.

Debes proporcionar las credenciales de inicio de sesión para obtener el token para realizar llamadas a la API.

oc login -u <my-user-name> --server="<master-api-public-addr>:<master-public-port>"


El objeto ResourceQuota enumera los límites de uso de recursos estrictos por proyecto.

El objeto ClusterResourceQuota enumera los límites de uso de recursos estrictos para los usuarios del clúster.

LimitRanges expresa los requisitos de CPU y memoria de los contenedores de los pods.

Establece request (solicitud) y limit (limite) de la CPU y la memoria que los pods particulares pueden consumir.

Ayuda al planificador de OpenShift a asignar pods a los nodos.

LimitRanges expresa los niveles de calidad de servicio:

Best Effort (mejor esfuerzo)

Burstable (flexible)

Guaranteed (Garantizada)

Se puede establecer un valor predeterminado de LimitRange para todos los pods/contenedores de cada proyecto.

pods

Cantidad total de pods en el estado no terminal que puede existir en un proyecto (el pod tiene el estado terminal si status.phase in (Failed, Succeeded) es verdadero).

replicationcontrollers

Cantidad total de controladores de replicación que puede existir en un proyecto.

resourcequotas

Cantidad total de cuotas de recursos que puede existir en un proyecto.

services

Cantidad total de servicios que puede existir en un proyecto.

secrets

Cantidad total de contraseñas que puede existir en un proyecto.

configmaps

Cantidad total de ConfigMap que puede existir en un proyecto.

persistentvolumeclaims

Cantidad total de reclamaciones de volúmenes persistentes que puede existir en un proyecto.

openshift.io/imagestreams

Cantidad total de flujos de imagen que puede existir en un proyecto.

RQ compute-resources (recursos de cómputos RQ):

Se muestran los informes de uso y las cuotas del tipo de recurso específico.

Se muestran límites y solicitudes de pods y contenedores combinados.

$ oc get quota -n demoproject

$ oc describe quota core-object-counts -n demoproject

Valores predeterminados para los contenedores:

kind: LimitRange
apiVersion: v1
metadata:
  name: test-core-resource-limits
  namespace: test
spec:
  limits:
    - type: Container
      max:
        memory: 6Gi
      min:
        memory: 10Mi
      default:
        cpu: 500m
        memory: 1536Mi
      defaultRequest:
        cpu: 50m
        memory: 256Mi
    - type: Pod
      max:
        memory: 12Gi
      min:
        memory: 6Mi


La pila (stack) de registro EFK consta de tres componentes:

Elasticsearch: almacén de objetos donde se guardan todos los registros.

Fluentd: recopila los registros de los nodos y los envía a Elasticsearch.

Kibana: IU web para Elasticsearch.

autoescalador de pods horizontales (HPA)


Plantilla

Ejemplo:

Establece generate (generación) en expression (expresión) para especificar los valores generados.

from (desde) especifica el patrón para generar un valor con una sintaxis de expresión pseudoregular.

 parameters:
   - name: PASSWORD
     description: "The random user password"
     generate: expression
     from: "[a-zA-Z0-9]{12}"
     
Para exportar objetos desde un proyecto en el formulario de la plantilla:

$ oc export all --as-template=<template_name>


Los CR se administran de la misma manera que los objetos de OpenShift almacenados.

create, get, describe, delete, etc.

Ejemplo:

oc get tomcats

Definición de recurso personalizado
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tomcats.apache.org
spec:
  group: apache.org
  names:
    kind: Tomcat
    listKind: TomcatList
    plural: tomcats
    singular: tomcat
    shortNames:
    - tc
  scope: Namespaced
  version: v1alpha1
  
Para crear una instancia de CR:

Cree el archivo YAML con la definición del CR:

apiVersion: apache.org/v1alpha1
kind: Tomcat
metadata:
  name: mytomcat
spec:
  replicaCount: 2
Cree el recurso en OpenShift:

oc create -f mytomcat.yaml

Para manipular y examinar los CR, usa los comandos oc:

oc get tomcats
oc describe tomcat mytomcat
oc scale tomcats --replicas=2 # only if Operator supports scaling

Action: Click the Dashboard tab:

Point out the Dashboard, which shows combined metrics for the project.

Action: Click the Metrics tab:

Point out the Metrics tab, where custom graphs of Prometheus Metrics can be created.

Action: Click the Events tab:

Point out the Events tab, where events are reported as a stream, and may be filtered.

Explain the relationship between ReplicationControllers and ReplicaSets. It is similar to the relationship between OpenShift Deployment Configs and Kubernetes Deployments. All of them are supported by OpenShift.


$ env |grep -i ${keyword}


Amplía a la nueva implementación en uno o más pods (según el valor maxSurge).

Espera que se completen las verificaciones de disponibilidad.

Reduce la implementación antigua en uno o más pods (según el valor maxUnavailable).


Estrategia de implementación personalizada
Ejemplo:

"strategy": {
  "type": "Custom",
  "customParams": {
    "image": "organization/strategy",
    "command": ["command", "arg1"],
    "environment": [
      {
        "name": "ENV_1",
        "value": "VALUE_1"
      }
    ]
  }
}
La imagen de contenedor organization/strategy brinda el comportamiento de la implementación.

La variedad command (comando) opcional anula la directiva de CMD especificada en Dockerfile en la imagen.

Se agregan variables environment (entorno) opcionales al entorno de ejecución del proceso de la estrategia.

La estrategia de compilación de contenedores invoca el comando simple podman build.


S2I: herramienta para compilar imágenes de contenedores reproducibles.

Produce imágenes listas para ejecutar.

Inyecta la fuente de aplicación en la imagen de contenedor y ensambla la nueva imagen de contenedor.

Esta nueva imagen incorpora la imagen básica (compilador) y compila la fuente.

Está lista para ser usada con el comando podman run.

Los scripts de S2I pueden inyectar el código de la aplicación en la mayoría de las imágenes de contenedores, lo que aprovecha el ecosistema existente.

Usa tar para inyectar la fuente de aplicación.

S2I restringe las operaciones realizadas con el usuario root y ejecuta scripts con usuarios no root.

Implementación de muestra de la imagen personalizada del compilador:

Imagen openshift/origin-custom-docker-builder en

Mention that using this deployment strategy is good for minimizing application downtime when the new and old deployments can live side by side for a short while.

Explain that this kind of config change (a Deployment spec change) does not trigger an automatic rollout. Only changes to the Pod spec trigger an automatic rollout.



//JENKINS ON OPENSHIFT

Deploy Jenkins to control your builds and deployment pipeline:

$ oc new-app jenkins-persistent -p ENABLE_OAUTH=false -e JENKINS_PASSWORD=openshiftpipelines -n pipeline-${GUID}-dev

Make a note of the Jenkins admin password.

The password is defined by the JENKINS_PASSWORD parameter.

The above command created a Jenkins deployment and service accounts in the Dev project. Since you want to be able to use Jenkins to run images in the Test and Prod projects, you need to give the Jenkins pservice account authorization to manage resources in those projects. To do so, use oc policy to enable the Jenkins service account to manage resources in the pipeline-${GUID}-test and pipeline-${GUID}-prod projects:

$ oc policy add-role-to-user edit system:serviceaccount:pipeline-${GUID}-dev:jenkins -n pipeline-${GUID}-test
$ oc policy add-role-to-user edit system:serviceaccount:pipeline-${GUID}-dev:jenkins -n pipeline-${GUID}-prod
When an application is deployed in a project, the service accounts in the project do the work. Since the sample application’s image will be owned by the Dev project, it is not by default accessible to the Test and Prod projects. The service accounts in the Test and Prod projects need to be able to pull images in the Dev projects. Use oc policy to enable pulling of images from the pipeline-${GUID}-dev project to the pipeline-${GUID}-test and pipeline-${GUID}-prod projects:

$ oc policy add-role-to-group system:image-puller system:serviceaccounts:pipeline-${GUID}-test -n pipeline-${GUID}-dev
$ oc policy add-role-to-group system:image-puller system:serviceaccounts:pipeline-${GUID}-prod -n pipeline-${GUID}-dev


//2nd MODULE

$ oc completion bash > oc_bash_completion

$ sudo cp oc_bash_completion /etc/bash_completion.d/

oc login -u USERNAME https://api.${CLUSTER_DOMAIN_NAME}:6443

Options:

-p: Provide password (not recommended)

--certificate-authority='': Path to cert file (to avoid warnings)

--insecure-skip-tls-verify=false: Removes HTTPS protection

--token: Bearer token for authentication

Pre-condition: existent project and at least one pod

oc get pods -l app=nginx

oc set resources deployment nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi

oc <ACTION> <RESOURCE> <OPTIONS>

Troubleshooting

oc status

oc describe <RESOURCE-TYPE> <RESOURCE-NAME>

oc get events --sort-by='.lastTimestamp'

oc --loglevel=9 <OPERATION>

oc debug dc/DC-NAME

oc debug node/NODE-NAME to access specific node

You can switch between clusters and users by changing only the KUBECONFIG environment variable.

If you are planning to access the same cluster from another machine, you can bring the config file with your credentials there. Note that anybody who has access to this config file can log in to the cluster using your username. That means you must always keep it safe and secret. Note the owner-only read-write permissions on this file.

If you log in to a cluster from someone else’s computer, your credentials are recorded in that person’s $HOME/.kube/config file (or in the file where KUBECONFIG is pointing at that time). Be sure to remove your session information from anyone else’s computer and oc logout before leaving that computer.

Create project:

oc new-project NAME --display-name=DISPLAYNAME --description=DESCRIPTION

Use route for external access to application

Use oc expose command to create route

Use oc create route command to create secured route (HTTPS)

For horizontal scaling add/remove similar pods to ReplicaSet

Use oc scale --replicas= command

Use --replicas=0 to pause application

Scaling can be automated

Use oc autoscale command

Use oc api-versions command to list all

Example (truncated):

. . . .
apps.openshift.io/v1
apps/v1
apps/v1beta1
apps/v1beta2
. . . .

Use oc api-resources command to list all

Example (truncated):

. . . .
NAME                                  SHORTNAMES       APIGROUP                              NAMESPACED   KIND
bindings                                                                                     true         Binding
componentstatuses                     cs                                                     false        ComponentStatus
configmaps                            cm                                                     true         ConfigMap
endpoints                             ep                                                     true         Endpoints
events                                ev                                                     true         Event
limitranges                           limits                                                 true         LimitRange
namespaces                            ns                                                     false        Namespace
nodes                                 no                                                     false        Node
persistentvolumeclaims                pvc                                                    true         PersistentVolumeClaim
persistentvolumes                     pv                                                     false        PersistentVolume
pods                                  po                                                     true         Pod
. . . .

Use oc explain to find information about OpenShift objects

Examples:

oc explain pod

oc explain pod.spec

oc explain pod.spec.containers

oc explain pod.spec.containers.readinessProbe

oc explain pod.spec.containers.readinessProbe.httpGet

Or, use oc explain pod --recursive to list entire structure

Example of complex template—list of OpenShift objects:

objects:
- apiVersion: v1
  kind: Secret
  metadata:
    name: ${NAME}
. . . .
- apiVersion: v1
  kind: Service
  metadata:
    name: ${NAME}
  spec:
    ports:
    - name: web
      port: 8080
      targetPort: 8080
    selector:
      name: ${NAME}
- apiVersion: v1
  kind: Route
  metadata:
    name: ${NAME}
  spec:
    host: ${APPLICATION_DOMAIN}
    to:
      kind: Service
      name: ${NAME}
      


oc get pods --field-selector status.phase=Running

metadata.labels: OpenShift uses labels to identify and select components.

metadata.name: The name of the Pod.

spec.containers.image: The image used to deploy this container.

spec.containers.ports: The network ports used by this container.

spec.containers.resources: The resources requested by this container.

spec.volumes: Storage volumes used by this Pod. In this case, a Secret is mounted as a volume in the application container.

status.phase: The status of the Pod. In this case, it is running. You used this field to filter the Pods. You can use other fields for that purpose as well.


Check Status
oc describe deployment NAME

oc status

oc get all

Scaling Deployment
oc scale deployment NAME --replicas=NUMBER

oc rollout status deployment NAME

To update Deployment when image updated:

oc edit deployment NAME → change container image tag

Or oc set image deployment NAME CONTAINER=NEW_IMAGE → point to new image tag

DeploymentConfig created when you run oc new-app command

ReplicationController also created

DeploymentConfig uses special -deploy Pod to run deployment process

You can track revisions of DeploymentConfig

If new version results in error, DeploymentConfig automatically rolls back to last good version

Blue-Green Deployment Strategy
Sometimes you need more testing to make sure new version works properly

You want to run the new deployment in parallel with current

You switch to new version, but do not delete old one automatically

Use Route for this

Switch between back-end Services

Delete old Deployment only when sure new one is good

Canary Deployment Strategy
Use Route to split application traffic between old and new versions

Only small fraction of users sent to new version

Application needs to support running two different versions simultaneously

A-B Testing Strategy
Example: Test two different styles of front page and measure users' reaction to it

You run two versions simultaneously and send traffic in 50/50 proportion

Technically possible to run more than two versions

Use more than two backing Services

Application needs to support running two different versions simultaneously

IP addresses of Pods are not stable. Each time a Pod restarts, it is assigned a new IP address. To access a Pod, you may want the Pod to have a stable IP address. Or you may want to run several identical Pods and load balance between them, in which case you must have a stable IP address for the load balancer. In both cases, you use a Service.

Services, like ReplicaSets, use selectors to find the Pods associated with them. Selectors select Pods with specific labels. For example, a Service called service-web provides a stable IP address (called a ClusterIP) for the Pods with the app: web label.

A ClusterIP provided by a Service is internal to the cluster. To access the application from outside the cluster requires a Route. The Route object gets information about associated Pods from the back-end Service and acts as an external load balancer for those Pods. Routes are usually implemented using HAProxy. Routes provide domain name addresses, which can be used to access the application.

To get more information about that, run the oc autoscale -h command for help on autoscale.

One important difference between Deployments and DeploymentConfigs is the properties of the CAP theorem that each design has chosen for the rollout process. DeploymentConfigs prefer consistency, whereas Deployments have a preference for availability over consistency.

This means that you cannot delete the Pod to unstick the rollout, as the kubelet is responsible for deleting the associated Pod.

In the case of Deployments, the process of Pod creation is managed by the cluster’s controller manager. But in the case of DeploymentConfigs, a separate Pod deploys the application Pods.

 Rolling strategy and its main goal is to prevent any downtime for the running applications. This is the default.
 
 Recreate strategy. It stops all of the running Pods first and then starts the new version Pods. You can configure additional steps before (pre) stopping the old Pods, after stopping them but before starting the new ones (mid), or after starting the new Pods (post).
 
 Add a section with recreation parameters and specify pre, mid, and post steps, similar to this (remove the comments):

strategy:
  type: Recreate
  recreateParams:
    pre: {}  //Executes any pre lifecycle hook.
    mid: {}  //Executes any mid lifecycle hook.
    post: {} //Executes any post lifecycle hook.
    
This approach is called blue/green deployment and it uses two versions of the application running simultaneously. First, you use the Route object to point to the current version. Then you start the new version and switch the Route to the new pods. After you test the new version and are satisfied with it, you can scale down the previous one and delete that Deployment.

create a Route using the oc expose command to make it visible from outside the cluster. You use the --name parameter to specify the Route’s name. This Route serves both the blue and green Deployments, so call it bluegreen.

Create a Route specifying the --name parameter bluegreen:

$ oc expose svc green --name=bluegreen
route.route.openshift.io/bluegreen exposed
Discover the URL so that you can access this application from outside the cluster:

$ oc get route
NAME        HOST/PORT                                                           PATH   SERVICES   PORT       TERMINATION   WILDCARD
bluegreen   bluegreen-55bb-bluegreen.apps.shared-na4.na4.openshift.opentlc.com          green      8080-tcp                 None
Use curl to check the version of the deployed application, using the URL of your actual application’s Route from the previous command’s output. Or, better, create an environment variable for the Route:

$ export ROUTE=$(oc get route bluegreen -o jsonpath='{.spec.host}')
$ curl $ROUTE/version
0.4

This time you do not need to run oc expose to expose the Service because you already have an external Route pointing to the green Service.

In this section, you switch the Route to the blue Service. There are a couple ways to do this. First, you use the oc edit command as it is the easiest way and helps you to find the right place in the Route’s manifest. Later, you use the oc patch command to perform the switch. Using oc patch is useful if you want to automate switching between blue and green deployments.

 oc patch command to switch between deployments. This is useful if you want to automate the process or if you want to include this step in a CI/CD pipeline.

Switch it back to green using the command line by editing the route and changing the service name

$ oc patch route/bluegreen -p '{"spec":{"to":{"name":"green"}}}'

route.route.openshift.io/bluegreen patched
Verify that the version reverted to 0.4:

$ curl $ROUTE/version
0.4


You may have noticed the weight: parameter in the Route’s manifest. In blue/green scenarios, you switch the Route from one backing Service to another with a weight equal to 100%. The weight parameter is particularly suited to canary deployments because it controls the amount of traffic, such as five percent, to send to either of the two active deployments.

It is also used in A/B testing to split traffic equally so that you can compare the response rate or some other metric.

NOTE:  alternateBackends is a YAML list whose elements start with - (hyphen).

You can achieve the same results using the command line. Imagine you have completed your testing and want to switch 90 percent of the traffic to the 0.5 version but leave 10 percent on the 0.4 version just in case.

Set the Route back-end weights:

$ oc set route-backends bluegreen blue=9 green=1
route.route.openshift.io/bluegreen backends updated 

ssh waaraque-atsistemas.com@studentvm.c184.example.opentlc.com

echo ${GUID}

wget --no-check-certificate https://downloads-openshift-console.apps.shared-na4.na4.openshift.opentlc.com/amd64/linux/oc

sudo mv oc /usr/local/bin/oc

sudo chmod a+x /usr/local/bin/oc

oc login -h

oc login -u OPENTLC_USERID https://api.shared-na4.na4.openshift.opentlc.com:6443


oc version

oc whoami

oc whoami --show-server

oc whoami --show-token #interesting

oc whoami --show-context

cat $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/new-config
