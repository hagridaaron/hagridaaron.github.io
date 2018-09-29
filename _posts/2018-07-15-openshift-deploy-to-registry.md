---
layout: post
title: Deploying Openshift BuildConfig to Private Docker Registry  
--
OpenShift Container Platform comes with an internal registry. By default when you create an application the build configuration is set up to push the images into the internal registry and the deployment configuration is set up to pull images from this internal registry.

Some people may be interested in OpenShift pushing the resultant application docker image ,after it is built, into a separate docker registry that they run outside OpenShift.

## Seup Docker Credentials as a secret

We will supply .docker/config.json file with valid Docker Registry credentials in order to push the output image into a private Docker Registry or pull the builder image from the private Docker Registry that requires authentication.

The .docker/config.json file is found in your home directory by default and has the following info (it may be a json file instead of yaml) :

```
auths:
  https://index.docker.io/v1/: 
    auth: "YWRfbGzhcGU6R2labnRib21ifTE=" 
    email: "user@example.com" 
```

If you connected to DockerHub before, you will already have this in your home directory. If you are using a different internal registry you will have the respective registry config here.

Run the following command to add a secret to your project that pulls the auth details from the docker config. We are naming the secret as `dockerhub-secret`. The output of the command shows `secret/dockerhub-secret` is created.

```
$ oc secrets new dockerhub-secret ~/.docker/config.json
secret/dockerhub-secret
```

you can also verify the list of secrets by running `oc get secrets` as shown below. This also lists the secret `dockerhub-secret` that we just created.

```
NAME                                   TYPE                                  DATA      AGE
dockerhub-secret                       kubernetes.io/dockerconfigjson        1         9m
```

Alternately, (let's say you have not logged into DockerHub from your workstation before,) you can also create the secret by passing the credentials directly.

```shell
oc secrets new-dockercfg dockerhub --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

## Add secret to your Builder service account
Each build in your project runs with using the builder service account. In this example we're using the `jeknins` service account. To get a list of service accounts in your project run and it shows the builder service account as one of the service accounts that are automatically created for you when you created a new project. **Note** `sa` is short form for service account.

```
$ oc get sa
NAME       SECRETS   AGE
builder    2         99d
default    2         99d
deployer   2         99d
jenkins    3         99d
```

We will now edit the `jenkins` SA and add the secret in there. Run the following command and it brings up the builder service account configuration in your default editor

```
$ oc edit sa builder
```

Edit the file to add the `dockerhub-secret` secret to the secrets list as shown below. Be careful about the indentation.

```
apiVersion: v1
imagePullSecrets:
- name: jenkins-dockercfg-d54lx
kind: ServiceAccount
metadata:
  labels:
    app: jenkins-persistent
    template: jenkins-persistent-template
  name: jenkins
  namespace: YOUR-NAMESPACE
  ownerReferences:
  - apiVersion: template.openshift.io/v1
    blockOwnerDeletion: true
    kind: TemplateInstance
  selfLink: /api/v1/namespaces/YOUR-NAMESPACE/serviceaccounts/jenkins
secrets:
- name: dockerhub-secret
```

Now your builder can use the newly created secret.

## Edit BuildConfig to push to your Docker Registry

In your BuildConfig there will be a line that looks something similar to the following:

```
spec:
  output:
    to:
      kind: ImageStreamTag
      name: my-app:latest
```

Note that this is telling the builder to push the output to the ImageStream `my-app` with the ImageStreamTag of `my-app:latest`


Now, we  will change the imagestream location to point to our Docker Registry as follows.


```
spec:
  output:
    to:
      kind: DockerImage   
      name: hub.docker.io/my-team/my-app:latest
    pushSecret:
      name: my-app-secret
```


In addition we also add the pushSecret here to let the builder use that particular secret while pushing into this registry.

Now you know a bit more about how to change up the target location when building images within OpenShift using BuildConfigs.