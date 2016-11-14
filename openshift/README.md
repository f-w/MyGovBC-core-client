This folder contains following artifacts to facilitate deploying a myGovBC-client derived web site to OpenShift:

* an instant-app template adopting s2i build strategy with binary source input 
* a nginx-based builder image

The prerequisites of the deployment are:

* minimum edit access to a project of OpenShift origin 1.3 or compatible cluster. This implies you know and have access to following URL end points:
  * OpenShift console, by default https://console.pathfinder.gov.bc.ca:8443/console/
  * OpenShift docker registry, by default docker-registry.pathfinder.gov.bc.ca
* has following software installed on the deployment client:
  * git
  * docker
  * oc

The deployment consists of these steps

1. deploy builder image

   ```sh
   $ git clone https://github.com/bcgov/MyGovBC-core-client.git
   $ cd MyGovBC-core-client
   $ docker build -t s2i-nginx openshift/builder-image
   $ docker tag s2i-nginx docker-registry.pathfinder.gov.bc.ca/<yourprojectname>/s2i-nginx
   $ oc login -u <username> https://console.pathfinder.gov.bc.ca:8443  
   $ docker login docker-registry.pathfinder.gov.bc.ca -u <username> -p `oc whoami -t`
   $ docker push docker-registry.pathfinder.gov.bc.ca/<yourprojectname>/s2i-nginx  
   ```
2. deploy template

   ```sh
   $ oc project <yourprojectname>
   $ oc create -f openshift/templates/s2i-binary-src.yaml
   ```
   After this step you will find an instant app template called *mygovbc-client* available in the project 
3. create OpenShift instant app. You can do so by clicking *mygovbc-client* template from *Add to Project* in web console or by running
   
   ```sh
   $ oc process mygovbc-client|oc create -f -
   ```
4. build runtime image

   ```
   $ npm install
   $ npm run build
   $ oc project <yourprojectname>
   $ oc start-build <app_name> --from-dir=dist/ -Fw
   ```
   where \<app_name\> is set in previous step with default value *mygovbc-client*.

The last step can be easily automated using Jenkins to enable CI. Jenkins should have same set of CLI listed in prerequisites. If you use default Jenkins hosted by OpenShift, which is recommended, oc is already installed. In such case you only need to install and configure [NodeJS Plugin](https://wiki.jenkins-ci.org/display/JENKINS/NodeJS+Plugin) to meet all prerequisites. 

Proper authorization is needed for Jenkins to launch build. If Jenkins is in the same project \<yourprojectname\>, no further configuration is required. Otherwise, run following command to grant access to Jenkins service account:

```
oc project <yourprojectname>
oc policy add-role-to-group edit system:serviceaccount:<jenkins_project>:<jenkins_service_name>
```
where \<jenkins_project\> and \<jenkins_service_name\> need to be substituted properly.

If everything goes well, you will be able to access the web app using the URL provided in the Overview page of OpenShift project.
