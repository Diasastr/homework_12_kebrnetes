# Інструкція по розгортанню Kubernetes за допомогою Minikube на AWS EC2

Цей README описує кроки для створення EC2 інстансу на AWS, налаштування Minikube для Kubernetes, створення та використання ресурсів Kubernetes, включаючи розгортання MySQL та Nginx.

## Створення та Налаштування EC2 Інстансу

### Створення EC2 Інстанс

1. **Запуск EC2 Інстансу**:
   Створила інстанс з потрібною security group і обрала t3.small для достатньої пам'яті

2. **SSH-доступ**:
   Підключилась до інстанс за допомогою AWS EC2_CONNECT

### Встановлення Minikube та Kubectl

1. **Оновила систему і встановила докер**:
   Оновила залежності і встановила докер для подальшого використання його кубернетес. Додала юзера до групи з доступами і оновила групу.
   ```bash
   sudo yum update -y
   sudo yum docker install -y
   sudo usermod -aG docker $USER
   newgrp docker
   ```

2. **Встановлення Minikube**:
   Використала команди:
   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```
   
3. **Встановлення Kubectl**:
   Використала команди:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```
   
4. **Запуск Minikube**:
   Використала команди:
   ```bash
   minikube start
   ```
   ![1  Start minikube](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/cca8f7ea-9f05-46b4-b5b8-6c8adbd3d3d7)
   
### Робота з Kubernetes

1. **Створила Namespace**:
   ```bash
   kubectl create namespace mynamespace
   ```

2. **Створила Secret для MySQL**:
   ```bash
   kubectl create secret generic mysql-pass --from-literal=password='yourRootPassword' -n mynamespace
   ```
   ![2  Created namespace and sql secret](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/004a7dcb-88bc-40c8-b2b3-cdf042ca11e7)

   
3. **Розгорнула MySQL**:
   Створила деплоймент файл
   ```yaml
   # mysql-deployment.yaml
   apiVersion: v1
   kind: Service
   ...
   apiVersion: apps/v1
   kind: Deployment
   ...
   ```
   і заранила його
   ```bash
   kubectl apply -f mysql-deployment.yaml
   ```
   ![3  Created mysql deploqment](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/9aa02fea-5e4d-4c5f-9e9e-5c231e3ca5f5)

4. **Розгорнула Nginx**:
   Створила деплоймент файл
   ```yaml
   # nginx-deployment.yaml
   apiVersion: v1
   kind: Service
   ...
   apiVersion: apps/v1
   kind: Deployment
   ...
   ```
   і заранила його
   ```bash
   kubectl apply -f nginx-deployment.yaml
   ```   
   ![4  Created nginx deployment](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/399a685f-55da-4290-9b13-4e4a81bfec6a)

...

![5  Checked pods running](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/acf4d945-f136-4a65-9107-2c5a2e027f9e)

### Доступ до Nginx Сервера

Після розгортання Nginx, ми можемо отримати доступ до веб-сервера за допомогою `NodePort`.

1. **Отримання доступу до Nginx**:
   - Використала команду нижче, щоб знайти `NodePort`, який використовується сервісом Nginx.
     ```bash
     kubectl get svc -n mynamespace
     ```
   - Відкрила веб-браузер та перейшла за адресою `http://<EC2-instance-public-IP>:<NodePort>`.

   Такоє можна перевірити роботу веббраузера використовуючи порт форвардінг і команду curl:
   
   ![6  Accessed nginx server](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/1c54dd22-945c-4931-a536-9d6ae0fa7152)

### Доступ до MySQL Бази Даних

1. **Логін до MySQL**:
   - Використала наступну команду для доступу до MySQL поду:
     ```bash
     kubectl exec -it <mysql-pod-name> -n mynamespace -- mysql -u root -p$(kubectl get secret mysql-pass -n mynamespace -o go-template='{{.data.password | base64decode}}')
     ```

   ![7  Accessed MySQL](https://github.com/Diasastr/homework_12_kebrnetes/assets/80010947/cc049105-aaa1-4337-bd07-87502c05eab7)

### Видалення Ресурсів

1. **Видалила Розгортання (Deployments) і Сервіси**:
     ```bash
     kubectl delete deployment mysql nginx -n mynamespace
     kubectl delete service mysql nginx -n mynamespace
     kubectl delete namespace mynamespace
     ```






   

