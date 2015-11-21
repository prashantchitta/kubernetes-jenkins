# Kubernetes Jenkins

Running Jenkins on Kubernetes with Persistent Volume. This setup can also be used to build kubernetes code base inside jenkins.

Lets first get the master ready

1. Cluster admin will make a few persistent volumes (pv), which are essentially cinder volumes, available for cluster use. 
   Create a claim to use one of those volumes in Kubernetes
    ```shell
    	kubectl create -f master/claim.yml
     ```
2. Check if the claim is created 
    ```shell
 	kubectl get pvc
    ```
3. Create a replication controller with 1 replica and persistent volume attached to it
    ```shell
 	kubectl create -f master/rc.yml
    ```
4. Create a service for the Jenkins master
    ```shell
 	kubectl create -f master/service-http.yml
    ```

Now your jenkins master server will be up and running at http://{serviceip}:{port}



Now lets get the slave ready

1. Run the docker file to create the slave image.
   ```shell
	docker build -t jenkins-slave .
   ```
2. Upload the image to either public/private registry


#Setup Jenkins

1. Goto Manage Jenkins -> Manage Plugins -> Install Github plugin, Github Authentication plugin, Go Plugin
2. Goto Credentials -> Global Credentials -> Add Credential -> Choose "Kubernetes Service Account" and click OK.
3. Goto Manage Jenkins -> Configure Systems -> Click Add new cloud at the bottom of the page.
	* Kubernetes URL: https://kubernetes.default.svc.kubernetes.local
	* Credential: Add the service account credentials you created in step 2.
	* Jenkins URL: http://{serviceip}:{port}
	* Click Add Pod Template:
		* Docker image: jenkins-slave
		* Jenkins slave root directory: /home/jenkins

The above setup will make Jenkins master contact Kubernetes API server to spin slave pods to do the builds.


#Setup Kubernetes Project to run on Jenkins
1. On Jenkins Home page Click "New Item" -> Choose Freestyle project and a name. 
	* Check "Discard Old builds" -> Log rotation -> Set Max # of builds to keep to a {value} and Days to keep builds to a {value}
	* Check "This build is parameterized" -> Choose String Parameter from dropdown -> Name: BRANCH, Default Value: tess-master
	* Choose Git radio button -> Repositories -> Repository URL: {URL of kube repo}, Branches to build: */tess-master
	* Check "Build when a changed is pushed to github repo."
	* Build Environment: Check "Setup GO programming language" -> Go version: Choose the version you setup previously using the Go Plugin
	* Execute shell: Enter the shell commands that needs to be executed as part of the build.
