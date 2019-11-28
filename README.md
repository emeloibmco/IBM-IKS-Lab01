# IBM-Kubernetes-Service-PHP-LAB

## Índice

* Introducción
* Creación del Dockerfile
* Envío de la imagen a IBM Cloud Container Registry
* Despliegue de la aplicación	
* Opción 2 de despliegue de la aplicación
* Eliminación del despliegue de la aplicación y eliminación del servicio
* Referencias

## Introducción 

Con Docker puede construir imágenes automáticamente leyendo las instrucciones desde un archivo Dockerfile. A Dockerfile es un documento de texto que contiene todos los comandos que un usuario podría llamar en la línea de comandos (CLI) para ensamblar una imagen. Con c usuarios pueden crear una compilación automatizada que ejecuta varias instrucciones de línea de comandos en sucesión [1].

<img width="650" alt="1" src="https://user-images.githubusercontent.com/50923637/69677427-f15ce500-1070-11ea-907d-acb91389c1de.png">

## Creación del Dockerfile

Las imágenes en Docker son construidas a partir de una imagen de referencia desde Docker hub. A continuación, se muestra las imanes disponibles para el Framework PHP.

<img width="650" alt="2" src="https://user-images.githubusercontent.com/50923637/69677548-2f5a0900-1071-11ea-9ee4-3b30b7aa4e72.png">

Con el comando ```Sudo vi Dockerfile``` se abre un Dockerfile en modo edición. A continuación, se detallan las instrucciones para la creación de la Imagen en .Net en Dockefile 

- **FROM**: le dice a Docker que vamos a utilizar el mcr.microsoft.com/dotnet/core/sdk:2.2 AS build-env como contenedor base llamada build-env .
- **WORKDIR**: La instrucción crea el directorio de trabajo
- **COPY** : se utiliza para copiar todos los archivos, como por ejemplo de la carpeta WebApiRest  del proyecto  a la carpeta WebApiRest  en el contenedor Docker 
- **RUN**: Ejecuta el contenedor con el nombre asignado
- **ENTRYPOINT**: Permite configurar un contenedor como un ejecutable.

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69677805-caeb7980-1071-11ea-9706-d6819655401c.png">

## Envío de la imagen a IBM Cloud Container Registry

- **ENTRYPOINT**: Permite configurar un contenedor como un ejecutable.

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69733847-03379a00-10fc-11ea-81bd-08a661d46362.png">

A continuación, podrá ver el paso para realizar el envió de la imagen al container registry de IBM Cloud.

* **Primero** Abra la terminal desde la carpeta donde se encuentre el proyecto

```cd <nombre del proyecto>```
* **Segundo** Inicie sesión en IBM Cloud CLI. Para especificar una región de IBM Cloud, incluya el API endpoind.

```ibmcloud login``` y Si tiene un ID federado, utilice ```ibmcloud login –sso```

* **Tercero** Ejecute ```ibmcloud cr login``` e inicie sesión con sus credenciales de IBM Cloud. Esto le permitirá enviar imágenes al IBM Cloud Container Registry.

Las credenciales de ingreso también las puede obtener desde le dasboard de IBM Cloud

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69734481-097a4600-10fd-11ea-93dd-39cc61cdb03b.png">

* **Cuarto** Cree un espacio de nombres en IBM Cloud Container Registry donde pueda almacenar sus imágenes:

```ibmcloud cr namespace-add <my_namespace>```

* **Quinto** Cree el ejemplo de la imagen Docker:

```docker build --tag us.icr.io/<my_namespace>/<nombre del proyecto> .```

* **Sexto** Verifique que la imagen esté construida:

```docker images```

* **Septimo** Agregue la imagen a IBM Cloud Container Registry:

```docker push us.icr.io/<namespace>/<nombre del proyecto>```

* **Octavo** Asegúrese de que el clúster que creó anteriormente esté listo para usar:

o	Ejecute ibmcloud cs clusters y asegúrese de que su clúster esté en estado Normal.

o	Ejecute ibmcloud cs workers <yourclustername> y asegúrese de que todos los trabajadores estén en estado Normal con un estado Listo.
  
o	Tome nota de la IP pública del trabajador.
  
**¡Ahora está listo para usar Kubernetes para desplegar la aplicación wepapirest!**

## Despliegue de la aplicación	

Primero debe acceder a su Kubernetes Service, para poder seguir el paso a paso del despliegue de la aplicación.

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69735481-bf925f80-10fe-11ea-808a-9c9ae0e10bc0.png">

A continuación, vera el paso a paso necesario para realizar el despliegue de la aplicación.

* **Primero** Obtenga el comando para establecer la variable de entorno y descargue los archivos de configuración de Kubernetes:

```ibmcloud cs cluster-config <cluster_name_or_ID>```

Cuando finaliza la descarga de los archivos de configuración, se muestra un comando que puede usar para establecer la ruta al archivo de configuración de Kubernetes local como una variable de entorno, por ejemplo, para OS X:

**export KUBECONFIG=/Users/<user_name>/.bluemix/plugins/container-service/clusters/pr_firm_cluster/kube-config-prod-par02-pr_firm_cluster.yml**

Copie y pegue el comando que se muestra en su terminal para establecer la variable de entorno KUBECONFIG.

**Segundo** Ejecute su imagen como una implementación: 

``` kubectl run hello-world --image=us.icr.io/<namespace>/hello-world ```

Esta acción tomará un poco de tiempo. Para verificar el estado de su implementación, ejecute este comando:

``` kubectl get pods ```

Debería ver una salida como esta:


```
=> kubectl get pods
NAME                          READY     STATUS              RESTARTS           AGE
webapirest-world-562211614    0/1       ContainerCreating   0          1m
```

* **Tercero**
Cuando el estado dice en ejecución, exponga esa implementación como un servicio, al que se accede a través de la IP de los nodos de trabajo. El ejemplo para este laboratorio escucha en el puerto 8080. Ejecute este comando:

``` kubectl expose deployment/hello-world --type="NodePort" --port=8080 ```

* **Cuarto** 
Encuentre el puerto que se usa en ese nodo de trabajo y examine su nuevo servicio:

``` kubectl describe service <name-of-deployment> ```

Tome nota de la línea "NodePort:" como <nodeport>.

* **Quinto**
Ejecute el siguiente comando y observe la IP pública como <public-IP>.
  
``` ibmcloud cs workers <name-of-cluster> ```

Ahora puede acceder a su contenedor / servicio utilizando curl <public-IP>: <nodeport> (o un navegador web). Ha terminado si ve esto:
  
**"¡la aplicación hellow-world! ¡Tu aplicación está funcionando en un clúster! "**

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69736445-8529c200-1100-11ea-85a5-820a2bc1325f.png">

## Opción 2 de despliegue de la aplicación

A partir del **archivo.yaml** puede ejecutar la lista de instrucciones del despliegue con el comando:

``` vi hello-world-deploy.yaml ```

<img width="650" alt="3" src="https://user-images.githubusercontent.com/50923637/69736618-e3ef3b80-1100-11ea-84da-5d4383f2f300.png">

Puede desplegar la aplicación con el siguiente comando:

``` kubectl apply  -f hello-world-deploy.yaml ```

## Eliminación del despliegue de la aplicación y eliminación del servicio

*	El siguiente comando se utiliza para eliminar la implementación:

```kubectl delete deployment <nombre del proyecto>``` 

* Para eliminar el servicio se debe utilizar el siguiente comando:

```kubectl delete service <nombre del servicio>```

## Referencias

[1] Docker. (s.f.). Obtenido de https://docs.docker.com/

[2] IBM Kubernetes Service Obtenido de https://www.ibm.com/developerworks/ssa/library/mw-1706-feillet-bluemix/1706-feillet.html
