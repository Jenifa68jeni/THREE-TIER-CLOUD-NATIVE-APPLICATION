# INFRASTRUCTURE SETUP – AWS EC2 & RDS ARCHITECTURE

## 🔹 Operating System
- **Amazon Linux AMI** used for Jenkins Master, Jenkins Slave, and EKS cluster nodes.

---

## 🔹 EC2 & RDS Instances
| Service        | Instance Name  | Purpose                                                   | Default Port |
|----------------|----------------|-----------------------------------------------------------|--------------|
| Jenkins Master | jenkinsmaster  | CI/CD pipeline orchestration & job scheduling             | 8080         |
| Jenkins Slave  | jenkinsslave   | Distributed build execution (runs jobs assigned by master)| 22 (SSH)     |
| EKS Cluster    | eks-cluster    | Kubernetes cluster creation & management (kubectl, eksctl, AWS CLI) | 22 (SSH) |
| Docker Hub     | Cloud Service  | Artifact repository for container images                  | –            |
| AWS RDS (MySQL)| jenifa-db      | Persistent database for application data                  | 3306         |

---

## 🔹 Security Group – Inbound Ports
- **22 (SSH)** → Restrict to your IP only (for EC2 & RDS admin access)  
- **8080** → Jenkins Master web UI  
- **3306** → MySQL (RDS database access from Jenkins/EKS pods)  

---

![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/0723d41280ce0549d445f54e2a505252a077a324/Screenshot%202026-04-08%20181255.png)
# RDS SETUP – AWS MySQL Database

## 🔹 Database Creation
- Created a new AWS RDS MySQL instance with identifier: **jenifa-db**  
- Engine: **MySQL Community Edition**  
- Instance type: **db.t4g.micro** (suitable for testing and small workloads)  
- Region: **ap-south-1 (Mumbai)**  
- Port: **3306** (default MySQL port)  
- Publicly accessible: **Yes** (to allow Jenkins/EKS pods to connect)  
![iamge alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/7b4d73cce9a9c550754003b2a01b3e4998ea8a5a/Screenshot%202026-04-08%20182722.png)
# RDS SETUP – AWS MySQL Database

## 🔹 Connectivity
- **Endpoint:** jenifa-db.cjqsqwumim0r.ap-south-1.rds.amazonaws.com  
- **Username:** jenifa  
- **Database:** jenifa_db  
- **Verified Connection:** Using MySQL Workbench → Test Connection successful  
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/30e3197966e5df1654f3e10269a515f4843c45dd/Screenshot%202026-04-08%20182918.png)
## Database Installation
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/9d0bdbf0768f8bbfe03ad5bafd34a0e79fd5687e/Screenshot%202026-04-08%20183141.png)
## Verification
Ran queries in MySQL Workbench:
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/3abb1104111cc00ee4fb74d70883f732047965ca/Screenshot%202026-04-08%20183352.png)
# JENKINS SETUP – MASTER & SLAVE CONFIGURATION

- Jenkins Master installed on EC2 instance **jenkins-master**  
- Jenkins Slave configured and connected via **SSH** from the master dashboard  
- **Port 8080** opened for Jenkins web UI access  
- Slave node added under **Manage Jenkins → Nodes → New Node** with proper labels and credentials  
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/d9074e1c94fb98e97532be130045fc55ec4d0071/Screenshot%202026-04-08%20183608.png)
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/0fbc14aa82773310de857cbee1d1138aa0264a47/Screenshot%202026-04-08%20183753.png)
# GLOBAL TOOL CONFIGURATION

- Navigate to: **Manage Jenkins → Global Tool Configuration**  
- Configure:  
  - **Maven** → Name: `maven`  
    - Version: `3.8.4`  
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/3176392dd82303caf3c36c08043a79fe307f885d/Screenshot%202026-04-08%20183943.png)
# CREDENTIALS SETUP

- Added **SSH credentials (ec2-user)** under **Manage Jenkins → Credentials**  
- Scope: **System → Global**  
- Credential ID: **jenkins-sl
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/db8a380ab1bd3c18a7d4970787167c88ef0c735b/Screenshot%202026-04-08%20184116.png)
# EKS HOST VM SETUP

- The EKS Host VM was launched and configured with **AWS CLI**, **kubectl**, and **eksctl** to manage the Kubernetes cluster.  

## 🔹 IAM Role Setup
An IAM role was created and attached to the EKS Host VM with the following permissions to enable secure management of AWS resources and Kubernetes cluster operations:

- IAM Full Access  
- VPC Full Access  
- EC2 Full Access  
- CloudFormation Full Access  
- Administrator Access
# CLUSTER CREATION – Amazon EKS

- Kubernetes cluster created using **eksctl**  
- Configured with required IAM role and permissions (IAM, VPC, EC2, CloudFormation, Administrator Access)  
- Region: **ap-south-1 (Mumbai)**  
- Nodes provisioned and attached to the cluster  
- Verified cluster connectivity with:  

  kubectl get nodes
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/bc9a1958fb6042c6bbcbdeec394e7bad85d0b148/Screenshot%202026-04-08%20184349.png)
# KUBERNETES DEPLOYMENT SETUP (JENKINS CLUSTER CONNECTION)

## 1. Install Required Tools on Jenkins Server
- **AWS CLI** → to connect Jenkins with AWS EKS  
- **kubectl** → to interact with the Kubernetes cluster  
- Ensure both are installed and available in Jenkins environment  

---

## 2. Copy Kubeconfig File to Jenkins
- When you create the cluster in the EKS Host VM, a **kubeconfig** file is generated  
- Copy this file into the Jenkins server (e.g., `/var/lib/jenkins/.kube/config`)  

---

## 3. Verify Cluster Nodes
Run the following command from Jenkins server to confirm connectivity:  
kubectl get nodes
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/7f5d1747e352ef57652b86f60c3d8a7e10c95706/Screenshot%202026-04-08%20184539.png)
## Job Creation
# APPLICATION CONFIGURATION – Spring Datasource

- Add **RDS endpoint, username, and password** to `application.properties`  
- This ensures the application knows how to connect to the database when deployed  

Example configuration:
spring.datasource.url=jdbc:mysql://jenifa-db.cjqsqwumim0r.ap-south-1.rds.amazonaws.com:3306/jenifa_db
spring.datasource.username=jenifa
spring.datasource.password=<your-password>
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/f2ce642307144fc00bf668dffcbcc489f63fa734/Screenshot%202026-04-08%20184753.png)
# KUBERNETES DEPLOYMENT – deployment.yaml (Slave VM)

## 🔹 Steps
1. On your Jenkins server (or local machine), go to your project workspace  
2. Create the file: `deployment.yaml`  
3. Paste the following content:
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/12e8c9404a136980db3bf7ad9a12eed23018b3b8/Screenshot%202026-04-08%20185047.png)
## save and exit
# KUBERNETES SERVICE – service.yaml (Slave VM)

## 🔹 Steps
1. On your Jenkins server (or local machine), go to your project workspace  
2. Create the file: `service.yaml`  
3. Paste the following content:
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/4c6c2774b0e9967ee3b6fc6d6f935fd5bf9200c3/Screenshot%202026-04-08%20185426.png)
## save and exit.
## Pipeline Execution for Deployment
# JENKINS PIPELINE SCRIPT (Declarative Pipeline)

pipeline {
    agent { label 'slave' }
    tools {
        maven 'maven'
    }
    environment {
        DOCKER_HUB_USER = 'jenkinsdemo'
        DOCKER_HUB_PASS = 'Jenkins@123'
        IMAGE_NAME = 'product-api'
    }
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/suffi2002/01_product01_api.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: '''
                    FROM eclipse-temurin:17-jre-alpine
                    WORKDIR /usr/app
                    COPY target/*.jar ./product_api.jar
                    CMD ["java", "-jar", "product_api.jar"]
                    '''
                    sh "docker build -t ${IMAGE_NAME} ."
                    sh "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASS}"
                    sh "docker tag ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: '''
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: product-api-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: product-api
                      template:
                        metadata:
                          labels:
                            app: product-api
                        spec:
                          containers:
                          - name: product-api
                            image: jenkinsdemo/product-api:latest
                            ports:
                            - containerPort: 8080
                            env:
                            - name: SPRING_DATASOURCE_URL
                              value: jdbc:mysql://jenkins-db.ciopwmndr.ap-south-1.rds.amazonaws.com:3306/jenkins_db
                            - name: SPRING_DATASOURCE_USERNAME
                              value: jenkinsdemo
                            - name: SPRING_DATASOURCE_PASSWORD
                              value: Jenkins@123
                            - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
                              value: com.mysql.cj.jdbc.Driver
                    '''
                    writeFile file: 'service.yaml', text: '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: product-api-service
                    spec:
                      selector:
                        app: product-api
                      ports:
                      - port: 8080
                        targetPort: 8080
                      type: LoadBalancer
                    '''
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    sh 'kubectl get svc product-api-service'
                }
            }
        }
    }
}

## TESTING AND VERIFICATION
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/17d8968829c703737876bb0736aa5f7735d05e1b/Screenshot%202026-04-08%20185933.png)
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/b393fe61962e9e060c65949e05ce00b3a01fe2ee/Screenshot%202026-04-08%20190039.png)
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/47f8ae0a672118a109d028f6cb7a85c4b4411d6e/Screenshot%202026-04-08%20190146.png)
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/a005cbd22fabf427753c55c5b75e2081fc945ab0/Screenshot%202026-04-08%20190255.png)
## Application Access:
Copy the LoadBalancer DNS and paste in browser as :
a172fedd77afa492fbd6b83c8cca7d5f-1029677772.ap-south1.elb.amazonaws.com/api/products
![image alt](https://github.com/Jenifa68jeni/THREE-TIER-CLOUD-NATIVE-APPLICATION/blob/c819b84ec265181b9b933173179c8a6839015cb9/Screenshot%202026-04-08%20190500.png)







