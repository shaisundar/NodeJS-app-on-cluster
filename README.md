APP/API kubernetes cluster


------------------------------------------------------------------
Making the APP and API ---

Make app.js and api.js
Install node modules

Make package.json

{
  "name": "nodejs-image-demo",
  "version": "1.0.0",
  "description": "nodejs image demo",
  "author": "Sammy the Shark <sammy@example.com>",
  "license": "MIT",
  "main": "app.js",
  "keywords": [
    "nodejs",
    "bootstrap",
    "express"
  ],
  "dependencies": {
    "express": "^4.16.4"
  }
}

From <https://www.digitalocean.com/community/tutorials/how-to-build-a-node-js-application-with-docker> 

This will mention all dependencies of the project

CD into project directory - 
 $ npm install

This will install all dependecies from the .json and lock files

$Node app.js  ----- will start the application in specified port.

In our case app is on port 3000 and api is on port 4000

But will exposing over internet, we will expose on port 443 (HTTPS) and 80 (HTTP)



-----------------------------------------------------


Now let Dockerize it

To containerize 1 image - the dockerfile will look like this in the apps root dir

FROM node:10-alpine
RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
WORKDIR /home/node/app
COPY package*.json ./
USER node
RUN npm install
COPY --chown=node:node . .
EXPOSE 3000
CMD [ "node", "app.js" ]


Change the exposable port

Then run this dockerfile and upload it to dockerhub/acr

Docker build 
docker tag <container> <myregistry.azurecr.io/samples/nginx>
az acr login --name shaiacr
docker push  <container> <myregistry.azurecr.io/samples/nginx>

From <https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli> 



Also make .dockerignore file

For both app and API

--------------------------------------------------------------------

Deploying to Kubernetes


To set kubectl to point yo correct cluster -
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

From <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough> 

$ Kubectl get pods --- gets all pods

Then you can follow this link - --- https://blog.cloud66.com/setting-up-secure-endpoints-in-kubernetes/ 

Here, we will use our own uploaded image instead of his image (his seems to have some problem).

For using image from private repo, you need to create and use secret

kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL

From <https://stackoverflow.com/questions/32726923/pulling-images-from-private-registry-in-kubernetes> 

Then in your deployment .yaml file - 

imagePullSecrets:
- name: myregistrykey

From <https://stackoverflow.com/questions/32726923/pulling-images-from-private-registry-in-kubernetes> 



Now your deployment should be done and services pointing to these deployments on the respective ports

Make sure service is using type NodePort instead of Cluster IP

Then you create service accounts, roles and role binding from the same blog post (most in kube-system namespace)

Create a load balancer in the kube-system namespace that the ingress controller can point and talk to.

Then you setup an Ingress controller using the Service account - and set the publish service as the load balancer, default backend and config map

Create a default backend on the Kube-system namespace

Then set up Ingress resources with rules as per your need to redirect to your correct services as per request





To Test it locally , you can PORT forward the ports and browse from your local browser
$ kubectl port-forward app-9c5fbd865-8bl6n 3000:3000
$ curl http://localhost:3000
Hello! This is my app!

From <https://blog.cloud66.com/setting-up-secure-endpoints-in-kubernetes/> 

To test it over the internet when everything is done, just look for the load balancers public ip and try accessing it via the configured routes.


Final code is present in the repo ---


Then to scale your application  , run
$kubectl scale --replicas=x deployment app












