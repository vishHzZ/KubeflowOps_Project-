# KubeflowOps_Project-
Image pull ECR to Kubernetes. 
**KubeFlowOps - Image Push + YAML Deploy**

- Project Documentation: Push Docker Image to AWS ECR & Deploy on Kubernetes (EKS) using Deployment & Service YAML

**1️⃣ Access EC2 Instance**

Use SSH to access your EC2 instance securely. Ensure your PEM file is properly formatted.

- ssh -i &lt;Identity file&gt;. /pem ubuntu@&lt;public-ip-address&gt;
<img width="771" height="405" alt="image" src="https://github.com/user-attachments/assets/b5616f20-534a-4694-8c08-4075326e7823" />

**2️⃣ Install Docker**

- sudo \-i
- snap install docker
- <img width="771" height="250" alt="image" src="https://github.com/user-attachments/assets/304a800f-bf34-4f60-b152-d4072d45a2ea" />

**3️⃣ Create Project Directory**

- mkdir project
- cd project

**4️⃣ Create Dockerfile**

A Dockerfile defines how your Docker image is built. Here we use Nginx to host a static website.

- cat > &lt;dir_name&gt;/Dockerfile

FROM nginx:latest

COPY index.html /usr/share/nginx/html

EXPOSE 80

**5️⃣ Create index.html**

- cat > &lt;Dir_Name&gt;/ index.html

[Index.txt](file:///C:\Users\ADMIN\Downloads\Index.txt)(Ctrl +Click)

**6️⃣ Build Docker Image**

- Build your custom container image from the Docker file.

docker build -t myapp:v1

docker build -t myapp:v2

**7️⃣ Run Container Locally**

- Expose the container to map to the port to access your application from a browser.

docker run -it -d -p 8080:80 --name website myapp:v1

**8️⃣ Add Security Group Rule in AWS**

Go to:

EC2 → Security Groups → Inbound Rules → Edit → Add Rule

- Type: Custom TCP
- Port: 8080
- Source: 0.0.0.0/0
- <img width="771" height="336" alt="image" src="https://github.com/user-attachments/assets/6bd663b0-9a31-4d42-9288-68a48c423685" />

**9️⃣ Access Your Website**

Find public IP

- Cmd: curl ifconfig.me

Open in browser:

- http://&lt;public-ip&gt;:8080
- <img width="771" height="308" alt="image" src="https://github.com/user-attachments/assets/a3723af5-c2f2-4a7f-8d6b-f095b25e9cac" />



**🔟 Push Docker Image to AWS ECR**

- Store your Docker images securely in Amazon ECR.

**Step 1: Create an ECR Repository**

AWS Console → ECR → Create Repository  
Example repo name: **webapp**

<img width="771" height="405" alt="image" src="https://github.com/user-attachments/assets/0fad1584-7193-4ddf-a1a4-7c6a659eba05" />

<img width="771" height="405" alt="image" src="https://github.com/user-attachments/assets/6c1eccec-b2e5-4509-9ef6-6381efd4f206" />

**Step 2: View Push Commands**

Select the repository → Click **View Push Commands**

Follow these commands:

- **Login to ECR**

aws ecr get-login-password --region &lt;region&gt; | docker login -- username AWS --password-stdin &lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com

- **Tag image**

docker tag myapp:v1 &lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/webapp:v1

- **Push image**

docker push &lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/webapp:v1

<img width="771" height="405" alt="image" src="https://github.com/user-attachments/assets/71797ce3-5cec-4dec-80a6-33aef3f1aa10" />

**Create Cluster and then Node Group as follows.**
<img width="757" height="405" alt="image" src="https://github.com/user-attachments/assets/1d5cecb3-8ccf-4d60-86b8-b4036b2d20c1" />

<img width="771" height="326" alt="image" src="https://github.com/user-attachments/assets/425dede9-8d31-4315-a3b5-f07b2d8041cd" />

**Click on next:**
<img width="771" height="363" alt="image" src="https://github.com/user-attachments/assets/dcd8f4e2-34f9-4ffb-be6d-87adfa451794" />


**Click on next:**
<img width="771" height="418" alt="image" src="https://github.com/user-attachments/assets/fd10ba27-35c7-4d76-b7c3-7825daa687b1" />


**1️⃣1️⃣ Connect to EKS (Kubernetes) from CloudShell**

Configure kubectl to communicate with your EKS cluster.

- **Update kubeconfig**

aws eks update-kubeconfig --name &lt;cluster_name&gt; --region &lt;region&gt;

<img width="771" height="350" alt="image" src="https://github.com/user-attachments/assets/253e8265-847a-4d90-bec7-32df2b4183ab" />

- **Verify cluster**

kubectl get nodes

kubectl get svc

**1️⃣2️⃣ Create ECR Secret for K8S Pull**

- Kubernetes needs this secret to authenticate and pull images from ECR.

kubectl create secret docker-registry ecr-secret --docker-server=&lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com --docker-username=AWS --docker-password="\$(aws ecr get-login-password - region &lt;region&gt;)"

e.g, kubectl create secret docker-registry ecr-secret --docker-server=&lt;Account-Id&gt; --docker- username=AWS --docker-password="\$(aws ecr get-login-password -region&lt;regin name&gt;)"

<img width="771" height="364" alt="image" src="https://github.com/user-attachments/assets/eb730486-e481-4039-8a85-42aca8a714fe" />



**1️⃣3️⃣ Create Deployment YAML**

Defines how your application pods should run in Kubernetes.

cat > &lt;dir_Name&gt;/deployment.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

name: webapp

spec:

replicas: 2

selector:

matchLabels:

app: webapp

template:

metadata:

labels:

app: webapp

spec:

containers:

\- name: webapp

image: &lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/webapp:v1

ports:

\- containerPort: 80

imagePullSecrets:

\- name: ecr-secret

**1️⃣4️⃣ Create Service YAML**

Exposes your Kubernetes deployment to the outside world using a LoadBalancer.

cat > &lt;Dir_Name&gt;/svc.yaml

apiVersion: v1

kind: Service

metadata:

name: webapp-svc

spec:

type: LoadBalancer

selector:

app: webapp

ports:

\- port: 80

targetPort: 80

**1️⃣5️⃣ Apply Deployment & Service**

Deploy the application to Kubernetes and create required networking.

kubectl apply -f deployment.yaml

kubectl apply -f svc.yaml

**1️⃣6️⃣ Verify All Resources**

Check pods, services, and load balancer endpoints.

kubectl get all -o wide

<img width="771" height="329" alt="image" src="https://github.com/user-attachments/assets/38b8a194-58ba-43fb-9bda-9d1ed55aa9c9" />


- For updating the Image, first push the new image to ECR
- Repeat step 10 for pushing the image to ECR
- 
<img width="743" height="354" alt="image" src="https://github.com/user-attachments/assets/84e5ec72-0d97-40a5-ac4e-649ee30d4120" />

<img width="769" height="374" alt="image" src="https://github.com/user-attachments/assets/9280c570-7a55-4989-8f67-97a7b4c00460" />


**1️⃣7️⃣ Update Image (Rolling Update) V1 🡪 V2**

Kubernetes updates your running application with zero downtime.

Then set the new image:

- kubectl set image deployment/webapp webapp=&lt;aws_account_id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/webapp:3

Ex- kubectl set image deployment.apps/webapp [webapp=692701344220.dkr.ecr.eu-west-3.amazonaws.com/cbz/project1:myapp-v3](http://webapp/=692701344220.dkr.ecr.eu-west-3.amazonaws.com/cbz/project1:myapp-v3)

<img width="789" height="400" alt="image" src="https://github.com/user-attachments/assets/2c3116e9-51f9-473d-a54b-098db830b6c6" />

Paste the Load Balancer's DNS On Browser:

<img width="771" height="268" alt="image" src="https://github.com/user-attachments/assets/247f40ee-e662-4525-82a7-a9fa345b10b4" />


Check rollout status:

- kubectl rollout status deployment/webapp

**1️⃣8️⃣ Rollback to Previous Version V2 🡪 V1**

Revert to the previous stable version if something goes wrong.

- kubectl rollout undo deployment/webapp
  
- <img width="740" height="389" alt="image" src="https://github.com/user-attachments/assets/47444af0-7740-4891-a248-f95d96fde0e5" />


Refresh the browser page :

<img width="771" height="405" alt="image" src="https://github.com/user-attachments/assets/509fe0c2-4262-4def-a0fa-3a64638dfc67" />

- **Workflow Summary Table**

| **Step** | **Task** | **Description** |
| --- | --- | --- |
| 1   | Create Dockerfile | Set up Nginx-based image v1 v2 |
| 2   | Push Custom Image to ECR | Upload image to AWS registry |
| 3   | Deploy to EKS | Apply manifests file |
| 4   | Update Image | Rolling updates with zero downtime |
| 5   | Rollback the image | Rollback updates with zero downtime |
