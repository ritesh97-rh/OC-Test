# Pipeline Setup Steps  

&nbsp;
#### 1.    Set up project to deploy C++ application using s2i

* ##### Create project named test-app

    `oc new project test-app`
    
* ##### Create image binary and resources under s2i-4-toolchain label

    `oc new-app {REPO_GITHUB_ADDR} --context-dir s2i/devtoolset/4-toolchain --name s2i-4-toolchain` 
    
* ##### Create and deploy C++ application resources under label test-app from image binary created in previous step
    `oc new-app s2i-4-toolchain~{REPO_GITHUB_ADDR} --context-dir s2i/devtoolset/4-toolchain/test/test-app --name test-app` 

* ##### Add the edit role to the service account associated with the Jenkins deployment for the test-app project
    `oc policy add-role-to-user edit system:serviceaccount:test-app-pipeline:jenkins -n test-app`

&nbsp;
#### 2. Set up Jenkins project

* ##### Create new project named test-app-pipeline

    `oc new-project test-app-pipeline`
    
* ##### Set up an app from jenkins-ephemeral template

    `oc new-app jenkins-ephemeral` &nbsp;
    
    To view available templates, run `oc new-app --search jenkins`
    
* ##### Apply the build config with name esri-example-pipeline to jenkins app (NOTE: replace {REPO_GITHUB_ADDR} with github repository)

    `oc create -f Pipeline/jenkinsBuildConfig.yaml` 
    
* ##### Get route for Jenkins web console

    `oc get route` 

* ##### Go to the route url to access Jenkins Web Console; on first log-in give Jenkins requested permissions  
&nbsp;

#### 3. Set up Jenkins sync and run pipeline

* ##### Configure the Jenkins Sync Plugin inside Jenkins Web Console: 

     Click Manage Jenkins (left menu) -> Click Configure System -> go to Openshift Jenkins Sync section -> Add ' test-app' to the namespace. Namespace field should read 'test-app-pipeline test-app'.
    
* ##### Kick off pipeline in Jenkins
    `oc start-build esri-example-pipeline` 

&nbsp;
#### 4. Expected Results

* ##### The esri-example-pipeline build config (Pipeline/jenkinsBuildConfig.yaml in repo) contains Github repository address and path to Jenkinsfile

* ##### In Jenkins console, a build should run and successfully terminate

* ##### Do the following as Jenkins build is running, before it terminates

    `oc project test-app` (switch to test-app project) &nbsp;
    
    `watch oc get all` (watch mode to view changes in resource status)
    
* ##### Will see source code get cloned from Github, test-app resources terminating (first stage in pipeline) and then getting recreated from the existing s2i-4-toolchain imagestream (second pipeline stage)

* ##### In the Openshift console, will see pod (and its logs) get removed and recreated (it will start printing "Hello, world" in test-app pod log)

 

