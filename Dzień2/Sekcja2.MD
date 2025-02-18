## **Zadania praktyczne dla sekcji 2: AWS ECR** 

## **1. Tworzenie prywatnego repozytorium w ECR**  
- Utwórz nowe **prywatne** repozytorium w Amazon ECR.  
- Skonfiguruj uprawnienia, aby tylko określone role IAM mogły przesyłać obrazy.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie prywatnego repozytorium ECR**  
1. Przejdź do **AWS Management Console** → **Amazon ECR**.  
2. Kliknij **Create repository**.  
3. W sekcji **Visibility settings** wybierz **Private** (domyślne ustawienie).  
4. Wpisz **Repository name**: `my-private-repo`.  
5. W sekcji **Tag immutability** (opcjonalnie):  
   - Wybierz **Enabled**, aby zapobiec nadpisywaniu istniejących tagów obrazów.  
6. W sekcji **Scan on push** (opcjonalnie):  
   - Wybierz **Enhanced scanning**, jeśli chcesz, aby AWS skanował obrazy w poszukiwaniu podatności.  
7. Kliknij **Create repository**.  

📌 **Repozytorium `my-private-repo` zostało utworzone!** 🎉  

---

**2. Konfiguracja uprawnień dostępu do repozytorium**  
1. Przejdź do zakładki **Permissions** w repozytorium `my-private-repo`.  
2. Kliknij **Edit policy**.  
3. Wklej następującą politykę (przykład nadania uprawnień użytkownikowi IAM):  

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "AWS": "arn:aws:iam::<AWS_ACCOUNT_ID>:user/dev-user"
               },
               "Action": [
                   "ecr:BatchGetImage",
                   "ecr:GetDownloadUrlForLayer"
               ]
           }
       ]
   }
   ```
4. Kliknij **Save policy**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie prywatnego repozytorium**  
```sh
aws ecr create-repository --repository-name my-private-repo
```
Przykładowa odpowiedź:  
```json
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:123456789012:repository/my-private-repo",
        "repositoryName": "my-private-repo",
        "repositoryUri": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-private-repo"
    }
}
```
📌 **Zapisz `repositoryUri`, ponieważ będzie potrzebne do wysyłania i pobierania obrazów Docker.**  

---

**2. Konfiguracja uprawnień do repozytorium**  
📌 Nadanie uprawnień użytkownikowi `dev-user`, aby mógł pobierać obrazy:  
```sh
aws ecr set-repository-policy \
    --repository-name my-private-repo \
    --policy-text '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::<AWS_ACCOUNT_ID>:user/dev-user"
                },
                "Action": [
                    "ecr:BatchGetImage",
                    "ecr:GetDownloadUrlForLayer"
                ]
            }
        ]
    }'
```
📌 **Teraz użytkownik `dev-user` ma dostęp do repozytorium!**  

---

**3. Sprawdzenie listy repozytoriów**  
Aby upewnić się, że repozytorium zostało utworzone, wykonaj:  
```sh
aws ecr describe-repositories
```
</details>

---

## **2. Tworzenie publicznego repozytorium w ECR**  
- Utwórz **publiczne** repozytorium w Amazon ECR.  
- Przetestuj dostępność repozytorium i sprawdź, czy można pobrać obraz bez uwierzytelniania.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie publicznego repozytorium ECR**  
1. Przejdź do **AWS Management Console** → **Amazon ECR**.  
2. Kliknij **Public registries** w menu bocznym.  
3. Kliknij **Create repository**.  
4. W sekcji **Visibility settings** wybierz **Public**.  
5. Wpisz **Repository name**: `my-public-repo`.  
6. (Opcjonalnie) Włącz **Tag immutability**, aby zapobiec nadpisywaniu obrazów.  
7. Kliknij **Create repository**.  

📌 **Repozytorium `my-public-repo` zostało utworzone i jest publicznie dostępne!** 🎉  

---

**2. Sprawdzenie publicznego dostępu do repozytorium**  
1. Przejdź do zakładki **Settings** repozytorium.  
2. Upewnij się, że **Visibility settings** to **Public**.  
3. Skopiuj **URI repozytorium**, np.:  
   ```
   public.ecr.aws/xyz123/my-public-repo
   ```
4. W przeglądarce otwórz stronę **Amazon ECR Public Gallery**:  
   ```
   https://gallery.ecr.aws/
   ```
5. Wyszukaj swoje repozytorium i sprawdź, czy jest dostępne.  

📌 **Twoje repozytorium jest teraz publiczne i dostępne dla innych użytkowników!**  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie publicznego repozytorium**  
```sh
aws ecr-public create-repository --repository-name my-public-repo
```
Przykładowa odpowiedź:  
```json
{
    "repository": {
        "repositoryArn": "arn:aws:ecr-public::123456789012:repository/my-public-repo",
        "repositoryName": "my-public-repo",
        "repositoryUri": "public.ecr.aws/xyz123/my-public-repo"
    }
}
```
📌 **Zapisz `repositoryUri`, ponieważ będzie potrzebne do wysyłania i pobierania obrazów Docker.**  

---

**2. Sprawdzenie statusu repozytorium**  
```sh
aws ecr-public describe-repositories
```
📌 Sprawdź, czy repozytorium `my-public-repo` znajduje się na liście.  

---

**3. Pobranie informacji o uprawnieniach repozytorium**  
```sh
aws ecr-public get-repository-policy --repository-name my-public-repo
```
📌 Domyślnie repozytorium publiczne powinno umożliwiać pobieranie obrazów wszystkim użytkownikom.  

</details>

---

## **3. Wysyłanie obrazu Docker do ECR**  
- Zaloguj się do ECR przy użyciu **AWS CLI**.  
- Utwórz lokalny obraz Docker (np. **Nginx**).  
- Wypchnij obraz do repozytorium ECR.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Pobranie i przygotowanie obrazu Docker**
1. Otwórz terminal na swojej maszynie.  
2. Pobierz obraz **Nginx**:  
   ```sh
   docker pull nginx:latest
   ```
3. Zweryfikuj, że obraz został pobrany:  
   ```sh
   docker images
   ```
   📌 Obraz **nginx** powinien być widoczny na liście.

---

**2. Logowanie do Amazon ECR Public**
1. Przejdź do **AWS Management Console** → **Amazon ECR**.  
2. Wybierz **Public registries** i znajdź swoje repozytorium **my-public-repo**.  
3. Skopiuj **Public repository URI**, np.:  
   ```
   public.ecr.aws/xyz123/my-public-repo
   ```

4. W terminalu zaloguj się do Amazon ECR Public:  
   ```sh
   aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
   ```

📌 **Jesteś teraz zalogowany do ECR Public i możesz wysyłać obrazy.**

---

**3. Przetagowanie obrazu Docker**
1. Nadaj obrazowi nową nazwę zgodną z repozytorium ECR:  
   ```sh
   docker tag nginx:latest public.ecr.aws/xyz123/my-public-repo/nginx:latest
   ```

---

**4. Wysłanie obrazu do ECR Public**
1. Prześlij obraz do repozytorium:  
   ```sh
   docker push public.ecr.aws/xyz123/my-public-repo/nginx:latest
   ```

2. Sprawdź, czy obraz pojawił się w **AWS Console** → **ECR Public** → **my-public-repo** → **Images**.  

📌 **Obraz został pomyślnie przesłany do publicznego ECR i jest dostępny dla każdego użytkownika!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Pobranie obrazu Docker**
```sh
docker pull nginx:latest
```

**2. Logowanie do Amazon ECR Public**
```sh
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
```

**3. Przetagowanie obrazu Docker**
```sh
docker tag nginx:latest public.ecr.aws/xyz123/my-public-repo/nginx:latest
```

**4. Wysłanie obrazu do Amazon ECR Public**
```sh
docker push public.ecr.aws/xyz123/my-public-repo/nginx:latest
```

**5. Sprawdzenie listy obrazów w repozytorium**
```sh
aws ecr-public describe-images --repository-name my-public-repo
```

📌 **Obraz jest teraz publicznie dostępny i może być pobrany przez każdego użytkownika!** 🎉  
</details>

---

## **4. Pobieranie obrazu z ECR i uruchamianie kontenera**  
- Pobierz obraz Docker z ECR.  
- Uruchom kontener lokalnie przy użyciu pobranego obrazu.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Znalezienie publicznego repozytorium ECR**  
1. Przejdź do **AWS Management Console** → **Amazon ECR**.  
2. Wybierz **Public registries** w menu bocznym.  
3. Znajdź repozytorium `my-public-repo` i skopiuj **Repository URI**, np.:  
   ```
   public.ecr.aws/xyz123/my-public-repo
   ```
4. Kliknij w repozytorium i przejdź do zakładki **Images**.  
5. Skopiuj pełny URI obrazu (np. `public.ecr.aws/xyz123/my-public-repo/nginx:latest`).  

---

**2. Pobranie obrazu Docker**  
1. Otwórz terminal i wykonaj komendę:  
   ```sh
   docker pull public.ecr.aws/xyz123/my-public-repo/nginx:latest
   ```
2. Sprawdź, czy obraz został pobrany:  
   ```sh
   docker images
   ```

📌 **Obraz został pobrany z Amazon ECR Public!**  

---

**3. Uruchomienie kontenera z pobranego obrazu**  
1. W terminalu uruchom kontener na podstawie obrazu:  
   ```sh
   docker run -d -p 8080:80 public.ecr.aws/xyz123/my-public-repo/nginx:latest
   ```
2. Sprawdź działające kontenery:  
   ```sh
   docker ps
   ```
3. Otwórz przeglądarkę i wpisz `http://localhost:8080` – powinieneś zobaczyć stronę domyślną **Nginx**.  

📌 **Obraz został pobrany i działa jako kontener Docker!** 🎉  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Pobranie obrazu Docker z Amazon ECR Public**  
```sh
docker pull public.ecr.aws/xyz123/my-public-repo/nginx:latest
```

**2. Sprawdzenie dostępnych obrazów Docker**  
```sh
docker images
```

**3. Uruchomienie kontenera z obrazu ECR Public**  
```sh
docker run -d -p 8080:80 public.ecr.aws/xyz123/my-public-repo/nginx:latest
```

**4. Sprawdzenie działających kontenerów**  
```sh
docker ps
```

📌 **Strona Nginx powinna być teraz dostępna na `http://localhost:8080`.**  
</details>

---

## **5. Konfiguracja polityki usuwania nieużywanych obrazów**  
- Skonfiguruj politykę **automatycznego usuwania** starych wersji obrazów.  
- Przetestuj, czy obrazy starsze niż np. 7 dni są usuwane.  
 
<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Znalezienie publicznego repozytorium ECR**  
1. Przejdź do **AWS Management Console** → **Amazon ECR**.  
2. Kliknij **Public registries** w menu bocznym.  
3. Znajdź repozytorium `my-public-repo` i kliknij w nie.  

---

**2. Konfiguracja polityki usuwania obrazów**  
1. Przejdź do zakładki **Lifecycle Policy**.  
2. Kliknij **Create lifecycle policy**.  
3. Kliknij **Create rule** i ustaw następujące wartości:  
   - **Rule name**: `DeleteOldImages`  
   - **Description**: `Remove images older than 7 days`  
   - **Tag prefix list**: `nginx`  
   - **Maximum image count**: `5` (pozostawi tylko 5 najnowszych wersji)  
   - **Action**: `Expire`  
4. Kliknij **Save**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie polityki usuwania nieużywanych obrazów**  
Utwórz plik JSON `lifecycle-policy.json` z następującą konfiguracją:  

```json
{
    "rules": [
        {
            "rulePriority": 1,
            "description": "Remove images older than 7 days",
            "selection": {
                "tagStatus": "tagged",
                "tagPrefixList": ["nginx"],
                "countType": "imageCountMoreThan",
                "countNumber": 5
            },
            "action": {
                "type": "expire"
            }
        }
    ]
}
```

**2. Zastosowanie polityki do repozytorium**  
```sh
aws ecr-public put-lifecycle-policy \
    --repository-name my-public-repo \
    --lifecycle-policy-text file://lifecycle-policy.json
```

**3. Sprawdzenie aktywnej polityki w repozytorium**  
```sh
aws ecr-public get-lifecycle-policy --repository-name my-public-repo
```

📌 **Teraz starsze obrazy będą automatycznie usuwane, gdy liczba wersji przekroczy 5!** 🎉  
</details>

---

## **6. Uruchamianie kontenera w Amazon ECS przy użyciu obrazu z ECR**  
- Skonfiguruj **klaster ECS** z typem **Fargate**.  
- Utwórz **zadanie ECS**, które korzysta z obrazu znajdującego się w ECR.  
- Uruchom zadanie i sprawdź jego status.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie klastra ECS w trybie Fargate**  
1. Przejdź do **AWS Management Console** → **Amazon ECS**.  
2. Kliknij **Clusters** → **Create Cluster**.  
3. Wybierz **Networking only (Fargate)** i kliknij **Next step**.  
4. Wpisz **Cluster name**: `my-ecs-cluster`.  
5. Kliknij **Create**.  

📌 **Klaster ECS został utworzony!** 🎉  

---

**2. Tworzenie definicji zadania (Task Definition)**  
1. Przejdź do **Task Definitions** → **Create new task definition**.  
2. Wybierz **Fargate** jako typ uruchomienia i kliknij **Next step**.  
3. Wpisz **Task definition name**: `nginx-task`.  
4. **Task Role**: pozostaw domyślną (`None`).  
5. **Task size**:  
   - **CPU**: `0.5 vCPU`  
   - **Memory**: `1 GB`  
6. Kliknij **Add container** i skonfiguruj:  
   - **Container name**: `nginx-container`.  
   - **Image**: `public.ecr.aws/xyz123/my-public-repo/nginx:latest` (skopiuj z repozytorium ECR Public).  
   - **Port mappings**: `Container port: 80`, **Protocol: TCP**.  
7. Kliknij **Create**.  

📌 **Definicja zadania jest gotowa do użycia!**  

---

**3. Tworzenie usługi ECS (Service)**  
1. Przejdź do **Clusters** → **my-ecs-cluster** → **Create Service**.  
2. Wybierz:  
   - **Launch type**: `Fargate`.  
   - **Task definition**: `nginx-task`.  
   - **Cluster**: `my-ecs-cluster`.  
   - **Service name**: `nginx-service`.  
   - **Number of tasks**: `1`.  
3. **Networking**:  
   - Wybierz domyślny **VPC** i subnety.  
   - **Security group**: Otwórz port `80` dla ruchu HTTP.  
4. Kliknij **Create Service**.  

📌 **ECS uruchomi teraz kontener Nginx na Fargate!** 🎉  

---

**4. Sprawdzenie uruchomionego kontenera**  
1. Przejdź do **Clusters** → **my-ecs-cluster** → **Tasks**.  
2. Kliknij zadanie `nginx-task`, aby zobaczyć szczegóły.  
3. Skopiuj **Public IP** i otwórz w przeglądarce:  
   ```
   http://<PUBLIC_IP>
   ```
4. Powinna wyświetlić się domyślna strona **Nginx**.  

📌 **Kontener działa na Amazon ECS!** 🎉  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie klastra ECS w trybie Fargate**  
```sh
aws ecs create-cluster --cluster-name my-ecs-cluster
```

---

**2. Tworzenie definicji zadania (Task Definition)**  
Utwórz plik `nginx-task-definition.json`:  
```json
{
  "family": "nginx-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "nginx-container",
      "image": "public.ecr.aws/xyz123/my-public-repo/nginx:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
```

Zarejestruj definicję zadania:  
```sh
aws ecs register-task-definition --cli-input-json file://nginx-task-definition.json
```

---

**3. Tworzenie usługi ECS**  
📌 Pobierz **ID subnetów** i **security group**:  
```sh
aws ec2 describe-subnets --query 'Subnets[*].SubnetId'
aws ec2 describe-security-groups --query 'SecurityGroups[*].GroupId'
```

📌 Utwórz usługę ECS:  
```sh
aws ecs create-service \
    --cluster my-ecs-cluster \
    --service-name nginx-service \
    --task-definition nginx-task \
    --desired-count 1 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxx],securityGroups=[sg-xxxxx],assignPublicIp=\"ENABLED\"}"
```

📌 **ECS uruchomi teraz kontener Nginx!** 🎉  

---

**4. Sprawdzenie uruchomionego kontenera**  
📌 Pobierz listę uruchomionych zadań:  
```sh
aws ecs list-tasks --cluster my-ecs-cluster
```

📌 Pobierz szczegóły zadania:  
```sh
aws ecs describe-tasks --cluster my-ecs-cluster --tasks <TASK_ID>
```

📌 Pobierz **Public IP** i sprawdź dostępność strony Nginx:  
```sh
aws ec2 describe-network-interfaces --query "NetworkInterfaces[*].Association.PublicIp"
```
Otwórz w przeglądarce:  
```
http://<PUBLIC_IP>
```

📌 **Kontener działa w Amazon ECS!** 🎉  
</details>

---

## **7. Konfiguracja automatycznego pobierania obrazów z ECR w ECS**  
- Skonfiguruj **ECS Task Definition**, aby zawsze pobierał najnowszą wersję obrazu z ECR.  
- Przetestuj wdrożenie nowej wersji aplikacji i sprawdź, czy ECS pobrał nowy obraz. 

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Modyfikacja definicji zadania (Task Definition)**  
1. Przejdź do **AWS Management Console** → **Amazon ECS**.  
2. Kliknij **Task Definitions** → Znajdź `nginx-task` → **Create new revision**.  
3. W sekcji **Container Definitions**, kliknij w `nginx-container`.  
4. W polu **Image**, wpisz:  
   ```
   public.ecr.aws/xyz123/my-public-repo/nginx:latest
   ```
5. Kliknij **Update** → **Create**.  

📌 **Teraz ECS będzie pobierał najnowszy obraz `nginx:latest` przy uruchamianiu nowej wersji!** 🎉  

---

**2. Aktualizacja usługi ECS do nowej wersji zadania**  
1. Przejdź do **Clusters** → **my-ecs-cluster** → **Services**.  
2. Kliknij w `nginx-service`.  
3. Kliknij **Update** → Wybierz nową wersję `nginx-task` (najnowsza).  
4. Kliknij **Update Service**.  
5. Poczekaj, aż ECS uruchomi nową wersję kontenera.  

📌 **ECS automatycznie pobierze nową wersję obrazu z publicznego ECR.**  

---

**3. Sprawdzenie, czy nowa wersja działa**  
1. Przejdź do **Clusters** → **my-ecs-cluster** → **Tasks**.  
2. Kliknij w nową instancję zadania.  
3. Skopiuj **Public IP** i otwórz w przeglądarce:  
   ```
   http://<PUBLIC_IP>
   ```
4. Powinna wyświetlić się **nowa wersja Nginx**.  

📌 **Kontener ECS zawsze pobiera najnowszą wersję obrazu z ECR!** 🎉  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie nowej rewizji zadania ECS**  
📌 Utwórz nową wersję definicji zadania, zapisując do pliku `nginx-task-update.json`:  

```json
{
  "family": "nginx-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "nginx-container",
      "image": "public.ecr.aws/xyz123/my-public-repo/nginx:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
```

📌 Zarejestruj nową wersję zadania:  
```sh
aws ecs register-task-definition --cli-input-json file://nginx-task-update.json
```

---

**2. Pobranie nowej rewizji zadania ECS**  
```sh
aws ecs list-task-definitions --family-prefix nginx-task --sort DESC
```
📌 Zwróci ID nowej rewizji, np. `nginx-task:2`.  

---

**3. Aktualizacja usługi ECS, aby używała nowej wersji obrazu**  
```sh
aws ecs update-service \
    --cluster my-ecs-cluster \
    --service nginx-service \
    --task-definition nginx-task:2
```

📌 **Teraz ECS uruchomi nową wersję kontenera z najnowszym obrazem!**  

---

**4. Sprawdzenie nowej instancji kontenera ECS**  
📌 Pobierz listę uruchomionych zadań:  
```sh
aws ecs list-tasks --cluster my-ecs-cluster
```

📌 Pobierz szczegóły nowej instancji:  
```sh
aws ecs describe-tasks --cluster my-ecs-cluster --tasks <TASK_ID>
```

📌 Pobierz **Public IP** kontenera i sprawdź w przeglądarce:  
```sh
aws ec2 describe-network-interfaces --query "NetworkInterfaces[*].Association.PublicIp"
```
Otwórz w przeglądarce:  
```
http://<PUBLIC_IP>
```

📌 **ECS teraz pobiera najnowszą wersję obrazu Nginx z ECR Public!** 🎉  
</details>