first set up the k8s cluster with the following github link, which contains terraform files for infrastructure creation
    link:-   "https://github.com/YeshwanthYolar/kubernetes-cluster-files.git"

there is a script.sh attached to the masternode ec2-instance of the cluster 
it contains the installation of the following 
    - unzip , vim 
    - openjdk-17 (java)
    - maven
    - jenkins
    - docker
    - kubectl
    - awscli
    - aws-iam-authenticator
    - sonarqube docker container

if this script is attached it will install all necessary packages and tools

after the infra is ready ssh into the master_node using the pem_key and public_IP 
step-1 configure the aws using below command
    aws configure
step-2 update the context of the cluster in .kube/config, use the below command
    aws eks update-kubeconfig --region <cluster_region> --name <cluster_name>

note:- if the errors occurs provide the required permissions and owner access for user

as the sonarqube is running as a docker container it can be accessed on the web browser using the public_ip:9000
for loggin in by default username is 'admin'
                         password is 'admin'
    after logging in under security we can generate a access token for integrating with jenkins

now the main part is configuring jenkins for the CI process
as already jenkins is installed access it using the public_ip:8080
at first the deafault password can be found at file path :- /var/lib/jenkins/secrets/initialAdminPassword
give the output as password and complete the necessary login procedure and install suggested plugins

after its installed necessary plugins , navigate into manage_jenkins tab and on plugins tab install required plugins for the project
the required plugins are
    - pipeline stageview
    - docker 
    - sonarqube scanner
    - github integration
and install if any other as per your configuration

after installing the plugins the next steps would be to configure the credentials 
navigate to manage_jenkins tab click on credentials and click on global credentials and click add credentials 
    - first add the github token as the secret text (can be generated from your git repo) 
    - second add the sonarqube token as secret text (can be generated from the sonarqube UI)
    - add the docker hub username and password also 

after configuring all the necessary credentials, on the homepage click on new job to create new job
    - select pipeline as the type 
    - paste the provided pipeline at the end of the sceen in the pipeline section 

below is the explaination of the pipeline

after all that is configured 

install argocd operator
here is the link of the operatorhub.io 
    link:- https://operatorhub.io/operator/argocd-operator

click on install and you will get the commands to install olm (operator lifecycle manager) and argocd operator

after argocd operator is installed then install argocd controllers 
here is the yaml file for creating basic argocd controllers

save the below code in a file

        apiVersion: argoproj.io/v1alpha1
        kind: ArgoCD
        metadata:
           name: example-argocd
           labels:
            example: basic
        spec:
         server:
          service:
            𝘁𝘆𝗽𝗲: LoadBalancer

use below command to apply
    kubectl apply -f <file_name>.yml

access the argocd web UI using the loadbalancer URL with below command 
    kubectl get svc
after accessing the loadbalancer url in the browser, if it shows any risk click on advance and accept the risk

by default argocd username will be 'admin'
argocd by default stores the password in secrets called argocd-cluster
    kubectl edit secrets example-argocd-cluster   (this is the secret name usually )
    the password will be under data --> admin.password: <encoded_password>
copy the secret as it will be in the base64 encoded format decode using below command 
    echo <encoded_password> | base64 -d
the above command will output the decoded password of the argocd web UI

once it is logged in, 
click on the 
    - create application
         - fill all the neccesary fields like source of git url of deployment files path etc.

if there are no errors in the filled details the deployment will be succesfull  





