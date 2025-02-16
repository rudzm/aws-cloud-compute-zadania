### **Zadania praktyczne dla sekcji 4: AWS Compute i Networking (EC2, Lambda, ECS, ELB/ALB)**  

---

#### **Amazon EC2**  
**Uruchomienie instancji EC2 i skonfigurowanie serwera WWW  ***
- Utwórz instancję EC2 w AWS.  
- Skonfiguruj dostęp przez Security Group.  
- Zainstaluj i uruchom serwer WWW.  
- Przetestuj działanie serwera przez przeglądarkę.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie instancji EC2**  
1. Przejdź do **AWS Management Console** → **EC2 Dashboard**.  
2. Kliknij **Launch Instance**.  
3. Wypełnij pola:  
   - **Name**: `WebServerEC2`  
   - **Amazon Machine Image (AMI)**: `Amazon Linux 2`  
   - **Instance Type**: `t3.micro` (Free Tier)  
   - **Key Pair**: Utwórz lub wybierz istniejący klucz SSH  
   - **Network Settings**:  
     - **VPC**: Domyślna lub Twoja własna  
     - **Subnet**: Wybierz subnet publiczny  
     - **Auto-assign public IP**: **Enabled**  
     - **Security Group**:  
       - Otwórz port **22 (SSH)** – dostęp tylko dla Twojego IP  
       - Otwórz port **80 (HTTP)** – dostęp dla wszystkich (`0.0.0.0/0`)  
4. Kliknij **Launch Instance**.  

---

### **2. Połączenie z instancją przez SSH**  
1. Wróć do **EC2 Dashboard** → **Instances**.  
2. Skopiuj **Public IP** uruchomionej instancji.  
3. Połącz się z instancją przez terminal:  
   ```sh
   ssh -i MyKey.pem ec2-user@<public-ip>
   ```

---

### **3. Instalacja i uruchomienie serwera Apache**  
1. Zaktualizuj system:  
   ```sh
   sudo yum update -y
   ```
2. Zainstaluj Apache:  
   ```sh
   sudo yum install httpd -y
   ```
3. Uruchom serwer WWW:  
   ```sh
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
4. Dodaj stronę testową:  
   ```sh
   echo "<h1>Witaj w AWS EC2</h1>" | sudo tee /var/www/html/index.html
   ```
5. Sprawdź działanie serwera, wpisując **Public IP** w przeglądarce:  
   ```
   http://<public-ip>
   ```
   Jeśli konfiguracja jest poprawna, zobaczysz komunikat **„Witaj w AWS EC2”**.
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie instancji EC2**
```sh
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name MyKey \
  --security-groups WebServerSG \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServerEC2}]'
```

### **2. Pobranie publicznego IP instancji**
```sh
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WebServerEC2" \
  --query "Reservations[].Instances[].PublicIpAddress" --output text
```

### **3. Połączenie z instancją**
```sh
ssh -i MyKey.pem ec2-user@<public-ip>
```

### **4. Instalacja Apache i uruchomienie serwera**
```sh
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Witaj w AWS EC2</h1>" | sudo tee /var/www/html/index.html
```

### **5. Sprawdzenie dostępności strony**
```sh
curl http://<public-ip>
```
Jeśli wynik zwróci **„Witaj w AWS EC2”**, serwer działa poprawnie.
</details>

#### **AWS Lambda**  
**Tworzenie funkcji AWS Lambda i monitorowanie jej działania za pomocą AWS X-Ray**
- Utwórz funkcję AWS Lambda w wybranym języku programowania.  
- Skonfiguruj jej rolę IAM i uprawnienia.  
- Włącz AWS X-Ray i skonfiguruj śledzenie żądań.  
- Wykonaj kilka testów i przeanalizuj dane w X-Ray.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie funkcji AWS Lambda**
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch** i skonfiguruj:  
   - **Function name**: `LambdaXRayDemo`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: Utwórz nową rolę z podstawowymi uprawnieniami (`AWSLambdaBasicExecutionRole`).  
4. Kliknij **Create function**.  

---

### **2. Dodanie kodu funkcji**
1. Przewiń do sekcji **Code** i wklej następujący kod:  
   ```python
   import json
   import time
   import boto3
   from aws_xray_sdk.core import patch_all, xray_recorder

   patch_all()

   def lambda_handler(event, context):
       xray_recorder.begin_segment("LambdaXRaySegment")
       time.sleep(1)  # Symulacja opóźnienia
       response = {
           "statusCode": 200,
           "body": json.dumps("Hello from AWS Lambda with X-Ray!")
       }
       xray_recorder.end_segment()
       return response
   ```
2. Kliknij **Deploy**.  

---

### **3. Włączenie AWS X-Ray dla funkcji Lambda**
1. Przewiń do sekcji **Monitoring tools** → **AWS X-Ray tracing**.  
2. Kliknij **Edit** i zaznacz **Active tracing**.  
3. Kliknij **Save**.  

---

### **4. Testowanie funkcji Lambda**
1. Przejdź do **Test** i utwórz nowy testowy event (może być pusty JSON `{}`).  
2. Kliknij **Test** i zobacz wynik w **Logs**.  

---

### **5. Analiza śledzenia w AWS X-Ray**
1. Przejdź do **AWS X-Ray** → **Traces**.  
2. Wybierz najnowszy ślad i przeanalizuj wykres.  
3. Sprawdź, jak długo trwało wykonanie funkcji i jakie segmenty były monitorowane.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie roli IAM dla Lambda**
```sh
aws iam create-role --role-name LambdaXRayRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

### **2. Przypisanie uprawnień X-Ray do roli**
```sh
aws iam attach-role-policy --role-name LambdaXRayRole \
    --policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess
aws iam attach-role-policy --role-name LambdaXRayRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

### **3. Tworzenie funkcji Lambda**
```sh
echo 'import json
import time
import boto3
from aws_xray_sdk.core import patch_all, xray_recorder

patch_all()

def lambda_handler(event, context):
    xray_recorder.begin_segment("LambdaXRaySegment")
    time.sleep(1)  # Symulacja opóźnienia
    response = {
        "statusCode": 200,
        "body": json.dumps("Hello from AWS Lambda with X-Ray!")
    }
    xray_recorder.end_segment()
    return response' > lambda_function.py

zip function.zip lambda_function.py

aws lambda create-function --function-name LambdaXRayDemo \
    --runtime python3.9 \
    --role arn:aws:iam::<AWS_ACCOUNT_ID>:role/LambdaXRayRole \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

### **4. Włączenie AWS X-Ray dla funkcji Lambda**
```sh
aws lambda update-function-configuration --function-name LambdaXRayDemo \
    --tracing-config Mode=Active
```

### **5. Wywołanie funkcji Lambda**
```sh
aws lambda invoke --function-name LambdaXRayDemo --payload '{}' response.json
cat response.json
```

### **6. Sprawdzenie śledzenia w AWS X-Ray**
```sh
aws xray get-trace-summaries --start-time $(date -d '5 minutes ago' +%s) --end-time $(date +%s)
```
</details>

#### **Amazon ECS (Elastic Container Service)**  
**Uruchomienie kontenera z Nginx w Amazon ECS (Fargate)**
- Skonfiguruj **klaster ECS** działający na Fargate.  
- Utwórz **Task Definition**, wykorzystując oficjalny obraz **nginx** z Docker Hub.  
- Skonfiguruj **Service**, aby uruchomić aplikację w Fargate.  
- Sprawdź działanie aplikacji, wpisując adres publiczny Load Balancera lub Taska w przeglądarce.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie klastra ECS**
1. Przejdź do **AWS Management Console** → **ECS (Elastic Container Service)**.  
2. Kliknij **Create Cluster**.  
3. Wybierz **Networking only (Fargate)** i kliknij **Next step**.  
4. Nazwij klaster np. `nginx-fargate-cluster`.  
5. Kliknij **Create** i poczekaj na zakończenie procesu.  

---

### **2. Tworzenie definicji zadania (Task Definition)**
1. Przejdź do **Task Definitions** → **Create new Task Definition**.  
2. Wybierz **Fargate** jako typ uruchomienia.  
3. W sekcji **Task Definition Name** podaj np. `nginx-fargate-task`.  
4. Wybierz **Operating System Family**: **Linux**.  
5. W sekcji **Task execution role** wybierz istniejącą rolę **ecsTaskExecutionRole** lub utwórz nową.  
6. W sekcji **Container Definitions** kliknij **Add container** i skonfiguruj:  
   - **Container name**: `nginx-container`  
   - **Image**: `nginx` (oficjalny obraz z Docker Hub)  
   - **Port mappings**: `80` → `80`  
7. Kliknij **Create**.  

---

### **3. Tworzenie usługi ECS (Service)**
1. Przejdź do **ECS Dashboard** → **Clusters** → `nginx-fargate-cluster`.  
2. Kliknij **Create Service**.  
3. Wybierz:  
   - **Launch Type**: `Fargate`  
   - **Task Definition**: `nginx-fargate-task`  
   - **Cluster**: `nginx-fargate-cluster`  
   - **Service Name**: `nginx-fargate-service`  
   - **Number of Tasks**: `1`  
4. Wybierz istniejącą **VPC** i subnety publiczne.  
5. W **Security Group** otwórz port `80` dla wszystkich (`0.0.0.0/0`).  
6. Kliknij **Create Service** i poczekaj na uruchomienie zadania.  

---

### **4. Sprawdzenie działania aplikacji**
1. Przejdź do **ECS Dashboard** → **Clusters** → `nginx-fargate-cluster`.  
2. Kliknij w zakładkę **Tasks** i wybierz uruchomiony Task.  
3. Znajdź **Public IP** kontenera i otwórz go w przeglądarce:  
   ```
   http://<public-ip>
   ```
4. Jeśli zobaczysz stronę startową Nginx, konfiguracja działa poprawnie! 🎉  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie klastra ECS**
```sh
aws ecs create-cluster --cluster-name nginx-fargate-cluster
```

### **2. Tworzenie definicji zadania ECS**
```sh
aws ecs register-task-definition --family nginx-fargate-task \
  --network-mode awsvpc --requires-compatibilities FARGATE \
  --execution-role-arn arn:aws:iam::<AWS_ACCOUNT_ID>:role/ecsTaskExecutionRole \
  --cpu 256 --memory 512 \
  --container-definitions '[
      {
        "name": "nginx-container",
        "image": "nginx",
        "portMappings": [
          {
            "containerPort": 80,
            "hostPort": 80,
            "protocol": "tcp"
          }
        ]
      }
    ]'
```

### **3. Tworzenie usługi ECS**
```sh
aws ecs create-service --cluster nginx-fargate-cluster \
  --service-name nginx-fargate-service \
  --task-definition nginx-fargate-task \
  --desired-count 1 --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345678],securityGroups=[sg-12345678],assignPublicIp=ENABLED}"
```

📌 **Zmień `subnet-12345678` oraz `sg-12345678` na odpowiednie wartości dla Twojej VPC i Security Group.**  

### **4. Sprawdzenie statusu usługi**
```sh
aws ecs describe-services --cluster nginx-fargate-cluster --services nginx-fargate-service
```

### **5. Pobranie publicznego IP kontenera**
```sh
aws ecs list-tasks --cluster nginx-fargate-cluster
TASK_ARN=$(aws ecs list-tasks --cluster nginx-fargate-cluster --query "taskArns[0]" --output text)
aws ecs describe-tasks --cluster nginx-fargate-cluster --tasks $TASK_ARN \
  --query "tasks[].attachments[].details[?name=='networkInterfaceId'].value" --output text
```

**Skopiowany `networkInterfaceId` możesz znaleźć w AWS Console i sprawdzić jego publiczny adres IP.**  

</details>

#### **Elastic Load Balancer (ELB) / Application Load Balancer (ALB)**  
**Tworzenie Load Balancera dla EC2**  
- Utwórz Application Load Balancer (ALB).  
- Skonfiguruj Target Group i dodaj instancje EC2.  
- Utwórz listenera na porcie 80 i skonfiguruj reguły routingu.  
- Przetestuj działanie Load Balancera, sprawdzając dostępność aplikacji.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie Load Balancera**  
1. Przejdź do **AWS Management Console** → **EC2 Dashboard**.  
2. W menu bocznym wybierz **Load Balancers** → Kliknij **Create Load Balancer**.  
3. Wybierz **Application Load Balancer** i kliknij **Create**.  
4. Wypełnij dane Load Balancera:  
   - **Name**: `MyALB`  
   - **Scheme**: `Internet-facing`  
   - **IP address type**: `IPv4`  
   - **VPC**: Wybierz VPC, w której działają instancje EC2.  
   - **Availability Zones**: Wybierz co najmniej 2 subnety publiczne.  

---

### **2. Konfiguracja listenera**  
1. W sekcji **Listeners** → **Add Listener**, wybierz:  
   - **Protocol**: `HTTP`  
   - **Port**: `80`  
2. Wybierz **Forward to: New target group**.  

---

### **3. Tworzenie Target Group (Grupy docelowej)**  
1. Nazwij **Target Group**: `MyTargetGroup`.  
2. **Target type**: `Instance`.  
3. **Protocol**: `HTTP`, **Port**: `80`.  
4. **VPC**: Ta sama, w której działają instancje EC2.  
5. Kliknij **Next** i wybierz instancje EC2, które mają obsługiwać ruch.  
6. Kliknij **Create Target Group**.  

---

### **4. Powiązanie Load Balancera z Target Group**  
1. Wróć do ekranu konfiguracji ALB.  
2. Wybierz **MyTargetGroup** jako domyślną grupę docelową.  
3. Kliknij **Create Load Balancer**.  

---

### **5. Testowanie Load Balancera**  
1. Przejdź do **EC2 Dashboard** → **Load Balancers**.  
2. Znajdź **DNS name** Load Balancera i otwórz w przeglądarce:  
   ```
   http://<DNS-ALB>
   ```
3. Jeśli zobaczysz stronę z jednej z instancji EC2, oznacza to, że Load Balancer działa poprawnie.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie Load Balancera**  
```sh
aws elbv2 create-load-balancer \
    --name MyALB \
    --type application \
    --scheme internet-facing \
    --subnets subnet-12345678 subnet-87654321 \
    --security-groups sg-12345678
```

### **2. Tworzenie Target Group**  
```sh
aws elbv2 create-target-group \
    --name MyTargetGroup \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-12345678
```

### **3. Rejestracja instancji EC2 do Target Group**  
```sh
aws elbv2 register-targets \
    --target-group-arn arn:aws:elasticloadbalancing:region:account-id:targetgroup/MyTargetGroup/1234567890abcdef \
    --targets Id=i-12345678 Id=i-87654321
```

### **4. Tworzenie Listenera dla Load Balancera**  
```sh
aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/MyALB/abcdef1234567890 \
    --protocol HTTP --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account-id:targetgroup/MyTargetGroup/1234567890abcdef
```

### **5. Pobranie adresu DNS Load Balancera**
```sh
aws elbv2 describe-load-balancers \
    --names MyALB \
    --query "LoadBalancers[0].DNSName"
```

### **6. Testowanie Load Balancera**
Otwórz przeglądarkę i wpisz zwrócony **DNS Load Balancera**:
```
http://<DNS-ALB>
```
</details>