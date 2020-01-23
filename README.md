# OSH TEMPLATES
A template describes a set of objects that can be parameterized and processed to produce a list 
of objects for creation by OpenShift Container Platform.
A template can be processed to create anything you have permission to create within a project, 
for example services, build configurations, and deployment configurations. A template may also 
define a set of labels to apply to every object defined in the template.

## DEPLOYMENT

## A) PRETASK
 - SECRETS.
   Secret objects let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys.
   Putting this information in a secret is safer and more flexible than putting it verbatim in a Pod Lifecycle definition or in a container image
	
   Add Deployment Key to Secrets.
   ```sh
   $ oc secret new encrypt-key ./encrypt_key
   ```
## B) DEPLOY

### CREATE WEBSERVER NGINX OPENSHIFT:

```sh
oc process -f build_webserver_template.yaml -p APPLICATION_NAME=nginx-gestion \
-p SOURCE_REPOSITORY_URL=https://github.com/Telefonica/smartwifi_osh_nginx.git \
-p CONTEXT_DIR='{ ENVIRONMENT }' -p APPLICATION_PORT_CLIENTES=8448 \
-p SOURCE_SECRET={ DEPLOYMENT_USER } -p APPLICATION_TAG=v1.0 | oc apply -f-

```

### CREATE GESTION CLIENTES OPENSHIFT:

```sh
oc process -f build_clientesapp_template.yaml -p APPLICATION_NAME=clientesapp \
-p SOURCE_REPOSITORY_URL=https://github.com/Telefonica/smartwifi_osh_backendapp.git \
-p CONTEXT_DIR='{ SOFTWARE_VERSION }' -p SOURCE_SECRET={ DEPLOYMENT_USER } -p APPLICATION_PORT=9000 \
-p HOSTNAME_HTTP={ URL_SERVICE } -p BACKGROUND={ ENVIRONMENT } -p SW_VERSION={ SOFTWARE_VERSION } \
-p NGINX_SERVICE_NAME=nginx-gestion -p NGINX_PORT=8448 | oc apply -f-
```

## C) POST TASK 

- EDIT CLIENTES APP DEPLOYMENTCONFIG
```
    hostAliases:
     - hostnames:
         - DEV|PRE|PRO
       ip: { MYSQL_IP } # Nat IP 
```
- CONFIGMAPS
  ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable.
  This page provides a series of usage examples demonstrating how to create ConfigMaps and configure Pods using data stored in ConfigMaps.
 
  1) Create a ConfigMap from directories, specific files, or literal values:
   ```sh
     $ oc create configmap { SERVICE_NAME }-config --from-file=conf/
  ```
  2) Consuming ConfigMaps in Pods:
  ```sh
     $ oc volume dc/{ SERVICE_NAME } --overwrite --add -t configmap  \
       -m { PATH_CONFIGURATION }--name={ SERVICE_NAME }-config \
       --configmap-name={ SERVICE_NAME }-config 
  ```
- CREATE PERSISTENT VOLUME

 1) Create Volume:
   ```sh 
    $ oc create -f create_volume.yaml
   ```
 2) Add volume to CLIENTES SERVICE 
 ```sh
    $ oc volumes deploymentconfigs/sftpserver --add --name=pvXXg-XXX --type=persistentVolumeClaim --claim-name=sftpfiles  --mount-path=/var/sftp/sftp_user 
 ```
 3) Add volume to CLIENTES SERVICE
 ```sh
    $ oc volumes deploymentconfigs/mcprovisionapp --add --name=pvXXg-XXX --type=persistentVolumeClaim \
      --claim-name=sftpfiles --mount-path=/opt/mcprovision/backend/
  ```  
## REQUIRED DEPLOYMENT VARIABLES 
+ DEPLOYMENT_USER:  github-user
+ PROXY: 	        n/a|url del proxy 
+ ENVIRONMENT:	    dev|pre|pro
+ SOFTWARE_VERSION: software version of the component 
+ URL_SERVICE:
    * DEV:
      clientesapp route service url
    * PRE:
      clientesapp route service url
    * PRO:
      clientesapp route service url

## OTHER
- CLEAN WHORE APP
```sh 
for i in $(oc get all | grep -w "{ SERVICE_NAME }" | awk '{print $1}') ; do oc delete $i; done
```
- DELETE ALL ON A PROJECT
```sh 
oc delete all --all -n project
```
- NGINX FIRST TIME CREATE PROBLEM
```sh 
oc new-app --source-secret=pdihub-user https://pdihub.hi.inet/nijiforhome/image_nginx.git --context-dir={ ENVIRONMENT } --name=nginx-gestion
```
