## **Zadania praktyczne dla sekcji 3: Deploy and Monitoring** 

## **1. Tworzenie niestandardowych metryk CloudWatch**  
- Skonfiguruj metrykę **CPUUtilization** dla instancji EC2.  
- Skonfiguruj regułę **Alarm**, która wysyła powiadomienie e-mail/SNS, gdy zużycie CPU przekroczy 80%. 

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie metryki CloudWatch dla EC2**  
1. Przejdź do **AWS Management Console** → **CloudWatch**.  
2. Kliknij **Metrics** → **Browse metrics**.  
3. Wybierz **EC2** → **Per-Instance Metrics**.  
4. Znajdź instancję EC2, którą chcesz monitorować, i zaznacz `CPUUtilization`.  
5. Kliknij **Actions** → **Create Alarm**.  

---

**2. Tworzenie alarmu CloudWatch dla metryki CPU**  
1. **Define Alarm**:  
   - W sekcji **Threshold type** wybierz **Static**.  
   - Wybierz **Greater than threshold** i ustaw wartość **80** (80% CPU).  
2. **Configure Actions**:  
   - Wybierz **Create new SNS topic** i wpisz nazwę np. `CPU-High-Alert`.  
   - Dodaj swój adres e-mail do powiadomień.  
   - Kliknij **Create topic** i **Confirm subscription** na swojej skrzynce e-mail.  
3. **Name and description**:  
   - Nazwa alarmu: `High CPU Usage Alarm`.  
   - Opis: `Powiadomienie o wysokim zużyciu CPU na EC2`.  
4. Kliknij **Create alarm**.  

📌 **CloudWatch teraz monitoruje CPU i wyśle powiadomienie, gdy CPU przekroczy 80%!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie metryki CloudWatch dla instancji EC2**  
📌 Pobierz **Instance ID** swojej instancji EC2:  
```sh
aws ec2 describe-instances --query "Reservations[*].Instances[*].InstanceId" --output text
```
Załóżmy, że wynik to `i-0123456789abcdef0`.

---

**2. Wysłanie danych do CloudWatch**  
Dodanie metryki CPU do CloudWatch:  
```sh
aws cloudwatch put-metric-data \
    --namespace "CustomEC2Metrics" \
    --metric-name "CPUUtilization" \
    --dimensions InstanceId=i-0123456789abcdef0 \
    --value 25
```

📌 **To symuluje użycie CPU na poziomie 25%.**  

---

**3. Tworzenie alarmu dla wysokiego użycia CPU**  
```sh
aws cloudwatch put-metric-alarm \
    --alarm-name "HighCPUAlarm" \
    --metric-name "CPUUtilization" \
    --namespace "AWS/EC2" \
    --statistic "Average" \
    --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --period 60 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:CPU-High-Alert
```

📌 **Alarm aktywuje się, gdy CPU przekroczy 80% przez co najmniej 2 minuty!**  

---

**4. Sprawdzenie statusu alarmu**  
```sh
aws cloudwatch describe-alarms --alarm-names "HighCPUAlarm"
```
</details>

---

## **2. Logowanie i analiza logów aplikacji w CloudWatch Logs**  
- Skonfiguruj **CloudWatch Logs** dla aplikacji działającej na instancji EC2.  
- Analizuj logi i znajdź błędy aplikacji.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie grupy logów w CloudWatch**  
1. Przejdź do **AWS Management Console** → **CloudWatch**.  
2. W lewym menu wybierz **Logs** → **Log groups**.  
3. Kliknij **Create log group**.  
4. Wpisz nazwę: `MyAppLogs`.  
5. Kliknij **Create**.  

---

**2. Instalacja i konfiguracja CloudWatch Agent na EC2**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Znajdź swoją instancję EC2, otwórz **Connect** → **Session Manager** lub **SSH**.  
3. Zainstaluj **CloudWatch Agent** (dla Amazon Linux/Ubuntu):  

   ```sh
   sudo yum install -y amazon-cloudwatch-agent
   ```

4. Skonfiguruj **CloudWatch Agent**, aby monitorował `/var/log` (np. logi aplikacji Nginx):  

   ```sh
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
   ```

5. Uruchom agent:  

   ```sh
   sudo systemctl start amazon-cloudwatch-agent
   ```

6. Zweryfikuj status agenta:  

   ```sh
   sudo systemctl status amazon-cloudwatch-agent
   ```

📌 **Teraz logi aplikacji są przesyłane do CloudWatch Logs!** 🎉  

---

**3. Analiza logów w AWS CloudWatch Logs**  
1. Przejdź do **CloudWatch Logs** → **Log groups** → `MyAppLogs`.  
2. Kliknij w **Latest Log Stream** i przeglądaj logi.  
3. Aby wyszukać błędy, użyj filtra:  
   ```
   ? ERROR
   ```
4. Możesz ustawić **CloudWatch Alarm**, aby powiadomił o błędach w logach.  

📌 **Teraz AWS CloudWatch zbiera logi aplikacji i umożliwia ich analizę!**  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie grupy logów**  
```sh
aws logs create-log-group --log-group-name MyAppLogs
```

---

**2. Tworzenie strumienia logów**  
```sh
aws logs create-log-stream --log-group-name MyAppLogs --log-stream-name MyAppInstance
```

---

**3. Instalacja CloudWatch Agent na EC2**  
Połącz się z instancją EC2:  
```sh
ssh -i my-key.pem ec2-user@<EC2_PUBLIC_IP>
```
Zainstaluj agent CloudWatch:  
```sh
sudo yum install -y amazon-cloudwatch-agent
```

Skonfiguruj monitorowanie logów `/var/log/messages`:  
```sh
sudo tee /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json <<EOF
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "MyAppLogs",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
EOF
```

Uruchom agent:  
```sh
sudo systemctl start amazon-cloudwatch-agent
```

📌 **CloudWatch Agent teraz przesyła logi do AWS CloudWatch!** 🎉  

---

**4. Analiza logów w CloudWatch**  
Pobranie 10 najnowszych wpisów logów:  
```sh
aws logs get-log-events --log-group-name MyAppLogs --log-stream-name MyAppInstance --limit 10
```

Wyszukiwanie błędów w logach:  
```sh
aws logs filter-log-events --log-group-name MyAppLogs --filter-pattern "ERROR"
```

📌 **Teraz możesz analizować logi aplikacji z poziomu CLI!** 🎉  

</details>

---

## **3. Tworzenie dashboardów CloudWatch**  
- Utwórz dashboard CloudWatch do monitorowania **CPU**, **dysków** i **sieci** dla instancji EC2.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie nowego dashboardu**  
1. Przejdź do **AWS Management Console** → **CloudWatch**.  
2. Wybierz **Dashboards** → **Create dashboard**.  
3. Wpisz nazwę dashboardu: `EC2-Monitoring-Dashboard`.  
4. Kliknij **Create dashboard**.  

---

**2. Dodanie metryk do dashboardu**  

#**Dodanie metryki CPU**
1. Kliknij **Add widget**.  
2. Wybierz **Line graph** → **Next**.  
3. Wybierz **Metrics** → **EC2** → **Per-Instance Metrics**.  
4. Zaznacz `CPUUtilization` dla swojej instancji.  
5. Kliknij **Create widget**.  

#**Dodanie metryki użycia dysku**
1. Kliknij **Add widget** → **Line graph**.  
2. Wybierz **Metrics** → **EC2** → **Per-Instance Metrics**.  
3. Wybierz metrykę `DiskReadOps` oraz `DiskWriteOps`.  
4. Kliknij **Create widget**.  

#**Dodanie metryki ruchu sieciowego**
1. Kliknij **Add widget** → **Line graph**.  
2. Wybierz **Metrics** → **EC2** → **Per-Instance Metrics**.  
3. Wybierz `NetworkIn` oraz `NetworkOut`.  
4. Kliknij **Create widget**.  

📌 **Dashboard CloudWatch teraz monitoruje CPU, dyski i sieć dla EC2!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie dashboardu**  
```sh
aws cloudwatch put-dashboard --dashboard-name EC2-Monitoring-Dashboard --dashboard-body '{
    "widgets": []
}'
```

---

**2. Dodanie metryk do dashboardu**  

#**Dodanie wykresu CPU**  
```sh
aws cloudwatch put-dashboard --dashboard-name EC2-Monitoring-Dashboard --dashboard-body '{
    "widgets": [
        {
            "type": "metric",
            "x": 0,
            "y": 0,
            "width": 6,
            "height": 6,
            "properties": {
                "metrics": [
                    [ "AWS/EC2", "CPUUtilization", "InstanceId", "i-0123456789abcdef0" ]
                ],
                "title": "CPU Usage",
                "view": "timeSeries",
                "stacked": false,
                "region": "us-east-1"
            }
        }
    ]
}'
```

---

#**Dodanie wykresu ruchu sieciowego**  
```sh
aws cloudwatch put-dashboard --dashboard-name EC2-Monitoring-Dashboard --dashboard-body '{
    "widgets": [
        {
            "type": "metric",
            "x": 6,
            "y": 0,
            "width": 6,
            "height": 6,
            "properties": {
                "metrics": [
                    [ "AWS/EC2", "NetworkIn", "InstanceId", "i-0123456789abcdef0" ],
                    [ "AWS/EC2", "NetworkOut", "InstanceId", "i-0123456789abcdef0" ]
                ],
                "title": "Network Traffic",
                "view": "timeSeries",
                "stacked": false,
                "region": "us-east-1"
            }
        }
    ]
}'
```

---

**3. Sprawdzenie utworzonego dashboardu**  
```sh
aws cloudwatch get-dashboard --dashboard-name EC2-Monitoring-Dashboard
```

📌 **Dashboard CloudWatch teraz monitoruje kluczowe metryki dla EC2!** 🎉  
</details>

---

## **4. Tworzenie grupy Auto Scaling dla instancji EC2**  
- Skonfiguruj grupę Auto Scaling z minimum **1** i maksimum **3** instancjami EC2.  
- Ustaw regułę automatycznego zwiększania instancji przy wysokim obciążeniu CPU.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie Launch Template (szablonu uruchamiania)**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz **Launch Templates** → **Create launch template**.  
3. Wpisz:  
   - **Launch template name**: `my-launch-template`.  
   - **Amazon Machine Image (AMI)**: Wybierz Amazon Linux 2.  
   - **Instance type**: `t2.micro`.  
   - **Key pair**: Wybierz istniejący lub utwórz nowy.  
   - **Security group**: Otwórz port **80** i **22**.  
4. Kliknij **Create launch template**.  

---

**2. Tworzenie grupy Auto Scaling**  
1. Przejdź do **EC2** → **Auto Scaling Groups**.  
2. Kliknij **Create Auto Scaling group**.  
3. Wpisz **Auto Scaling group name**: `my-auto-scaling-group`.  
4. Wybierz wcześniej utworzony **Launch Template** → `my-launch-template`.  
5. Wybierz **VPC** i subnety dla instancji.  
6. Kliknij **Next** i ustaw:  
   - **Minimum instances**: `1`  
   - **Maximum instances**: `3`  
   - **Desired capacity**: `1`  
7. Kliknij **Next** i **Create Auto Scaling Group**.  

📌 **Teraz Auto Scaling uruchomi i zarządza instancjami EC2!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie Launch Template**  
```sh
aws ec2 create-launch-template \
    --launch-template-name my-launch-template \
    --version-description "Initial version" \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t2.micro",
        "KeyName": "my-key-pair",
        "SecurityGroupIds": ["sg-0123456789abcdef0"]
    }'
```
📌 **Podmień `ami-0abcdef1234567890` na aktualny AMI dla Amazon Linux 2.**  

---

**2. Tworzenie grupy Auto Scaling**  
```sh
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name my-auto-scaling-group \
    --launch-template "LaunchTemplateName=my-launch-template,Version=1" \
    --min-size 1 --max-size 3 --desired-capacity 1 \
    --vpc-zone-identifier "subnet-0123456789abcdef0"
```
📌 **Podmień `subnet-0123456789abcdef0` na poprawne ID subnetu w Twojej VPC.**  

---

**3. Sprawdzenie statusu grupy Auto Scaling**  
```sh
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-auto-scaling-group
```

📌 **Grupa Auto Scaling działa i zarządza instancjami EC2!** 🎉  


</details>

---

## **5. Testowanie Auto Scaling**  
- Wymuś wysokie zużycie CPU na instancji i zweryfikuj, czy AWS dodał nowe instancje EC2.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Konfiguracja polityki skalowania w Auto Scaling**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz **Auto Scaling Groups** → Znajdź `my-auto-scaling-group`.  
3. Przejdź do zakładki **Automatic scaling** → **Create scaling policy**.  
4. Wybierz **Target tracking scaling policy**.  
5. W sekcji **Metric type** wybierz **Average CPU utilization**.  
6. Ustaw docelowe użycie CPU na **50%**.  
7. Kliknij **Create**.  

📌 **Teraz Auto Scaling będzie dodawał instancje, gdy średnie obciążenie CPU przekroczy 50%.**  

---

**2. Wymuszenie wysokiego zużycia CPU na instancji EC2**  
1. Przejdź do **EC2 Instances** i połącz się z instancją przez **Session Manager** lub **SSH**.  
2. Jeśli instancja korzysta z **Amazon Linux** lub **Ubuntu**, uruchom:  

   ```sh
   sudo yum install -y stress || sudo apt install -y stress
   ```
3. Uruchom sztuczne obciążenie CPU:  

   ```sh
   stress --cpu 2 --timeout 300
   ```
   📌 Obciążenie potrwa **5 minut** (300 sekund).  

4. Przejdź do **CloudWatch Metrics** → **EC2** → **CPUUtilization** i sprawdź wzrost obciążenia.  
5. Po kilku minutach Auto Scaling powinno automatycznie dodać nową instancję EC2.  

📌 **Instancje EC2 automatycznie się skalują w odpowiedzi na wysokie zużycie CPU!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Konfiguracja polityki skalowania**  
```sh
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name my-auto-scaling-group \
    --policy-name "ScaleOnHighCPU" \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "TargetValue": 50.0
    }'
```

📌 **Teraz Auto Scaling doda nowe instancje, gdy średnie zużycie CPU przekroczy 50%.**  

---

**2. Wymuszenie wysokiego zużycia CPU na instancji EC2**  
📌 Pobierz ID pierwszej instancji w grupie Auto Scaling:  
```sh
aws autoscaling describe-auto-scaling-groups \
    --auto-scaling-group-name my-auto-scaling-group \
    --query "AutoScalingGroups[0].Instances[0].InstanceId" --output text
```

Połącz się z instancją EC2:  
```sh
ssh -i my-key.pem ec2-user@<EC2_PUBLIC_IP>
```

Zainstaluj i uruchom test obciążenia CPU:  
```sh
sudo yum install -y stress || sudo apt install -y stress
stress --cpu 2 --timeout 300
```

📌 **Monitoruj metryki CPU w CloudWatch i sprawdź, czy Auto Scaling dodaje nowe instancje!** 🎉  

---

**3. Sprawdzenie dodanych instancji EC2 przez Auto Scaling**  
```sh
aws ec2 describe-instances --filters "Name=tag:aws:autoscaling:groupName,Values=my-auto-scaling-group"
```

📌 **Jeśli widzisz więcej niż jedną instancję, oznacza to, że Auto Scaling działa poprawnie!** 🎉  
</details>

---

## **6. Skonfigurowanie polityki zmniejszania liczby instancji**  
- Skonfiguruj Auto Scaling tak, aby zmniejszał liczbę instancji, gdy CPU spadnie poniżej 30%.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Edycja polityki Auto Scaling**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz **Auto Scaling Groups** → Znajdź `my-auto-scaling-group`.  
3. Przejdź do zakładki **Automatic scaling** → **Edit**.  
4. Kliknij **Create scaling policy**.  
5. Wybierz **Target tracking scaling policy**.  
6. **Metric type**: Wybierz **Average CPU utilization**.  
7. **Target value**: Ustaw **30%**.  
8. **Scale-in protection**: Wyłącz ochronę przed usunięciem instancji.  
9. Kliknij **Create policy**.  

📌 **Teraz Auto Scaling będzie usuwać instancje EC2, gdy CPU spadnie poniżej 30%!** 🎉  

---

**2. Testowanie skalowania w dół**  
1. Połącz się z instancją EC2 i **zakończ procesy zużywające CPU**:  
   ```sh
   sudo pkill -f stress
   ```
2. Przejdź do **CloudWatch Metrics** → **EC2** → **CPUUtilization**.  
3. Gdy CPU spadnie poniżej 30%, Auto Scaling **powinien automatycznie usunąć nadmiarowe instancje**.  

📌 **AWS teraz redukuje liczbę instancji w czasie niskiego obciążenia!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Konfiguracja polityki zmniejszania liczby instancji**  
```sh
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name my-auto-scaling-group \
    --policy-name "ScaleDownOnLowCPU" \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "TargetValue": 30.0
    }'
```

📌 **Teraz Auto Scaling usunie instancje, gdy średnie obciążenie CPU spadnie poniżej 30%!**  

---

**2. Weryfikacja działania polityki Auto Scaling**  
📌 Pobierz informacje o grupie Auto Scaling:  
```sh
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-auto-scaling-group
```

📌 Pobierz szczegóły polityki skalowania:  
```sh
aws autoscaling describe-policies --auto-scaling-group-name my-auto-scaling-group
```

📌 Sprawdź aktualną liczbę instancji EC2 w Auto Scaling:  
```sh
aws ec2 describe-instances --filters "Name=tag:aws:autoscaling:groupName,Values=my-auto-scaling-group"
```

📌 **Jeśli liczba instancji zmniejszy się po spadku CPU poniżej 30%, oznacza to, że polityka działa poprawnie!** 🎉  

</details>

---

## **7. Sprawdzenie rekomendacji Trusted Advisor**  
- Przejdź do **AWS Trusted Advisor** i przeanalizuj dostępne rekomendacje.  
- Zidentyfikuj potencjalne oszczędności kosztów i poprawki związane z bezpieczeństwem.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Przegląd rekomendacji w AWS Trusted Advisor**  
1. Przejdź do **AWS Management Console** → **Trusted Advisor**.  
2. Na stronie głównej zobaczysz **5 głównych kategorii rekomendacji**:  
   - **Cost Optimization** (Oszczędność kosztów)  
   - **Performance** (Wydajność)  
   - **Security** (Bezpieczeństwo)  
   - **Fault Tolerance** (Odporność na błędy)  
   - **Service Limits** (Limity usług)  
3. Kliknij w każdą kategorię, aby zobaczyć rekomendacje.  

---

**2. Analiza rekomendacji i podejmowanie działań**  

#**Cost Optimization (Oszczędność kosztów)**  
- Sprawdź, czy masz **niewykorzystane instancje EC2** lub **nieużywane woluminy EBS**.  
- Zweryfikuj, czy można użyć instancji Reserved Instances lub Savings Plans.  

#**Security (Bezpieczeństwo)**  
- Sprawdź, czy masz **publiczne grupy Security Groups** z otwartymi portami.  
- Zweryfikuj, czy **wielu użytkowników IAM nie posiada nadmiernych uprawnień**.  

#**Performance (Wydajność)**  
- Sprawdź, czy masz **przeciążone instancje EC2** i czy można dodać Auto Scaling.  
- Zweryfikuj, czy RDS nie wymaga optymalizacji bazy danych.  

📌 **Zapisz rekomendacje i podejmij działania optymalizacyjne!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Pobranie rekomendacji Trusted Advisor**  
Uruchom następujące polecenie, aby pobrać **wszystkie rekomendacje Trusted Advisor**:  
```sh
aws support describe-trusted-advisor-checks --language en
```
📌 **Zwróci listę dostępnych kontroli.**  

---

**2. Pobranie szczegółowych wyników dla poszczególnych kategorii**  

#**Sprawdzenie optymalizacji kosztów**  
```sh
aws support describe-trusted-advisor-check-result \
    --check-id e7f47a28-07c5-49db-a204-2f0a79c806c0 \
    --language en
```
📌 **Sprawdza niewykorzystane instancje EC2, woluminy EBS i S3.**  

#**Sprawdzenie bezpieczeństwa**  
```sh
aws support describe-trusted-advisor-check-result \
    --check-id 39f3e42a-2f37-4d3d-89de-7603f5b72412 \
    --language en
```
📌 **Sprawdza grupy Security Groups, uprawnienia IAM i dostęp publiczny.**  

#**Sprawdzenie wydajności**  
```sh
aws support describe-trusted-advisor-check-result \
    --check-id 9f38d7a2-9a0a-42c5-adaf-9cbb451f7c55 \
    --language en
```
📌 **Sprawdza przeciążone EC2, RDS i optymalizację sieci.**  

---

**3. Weryfikacja wyników i podjęcie działań**  
📌 **Wyniki Trusted Advisor są zwracane w formacie JSON – sprawdź, które zasoby wymagają uwagi i podejmij działania optymalizacyjne.**  

</details>

---

## **8. Optymalizacja bezpieczeństwa za pomocą Trusted Advisor**  
- Znajdź nieużywane uprawnienia IAM i zidentyfikuj potencjalne zagrożenia.  
- Wprowadź poprawki na podstawie rekomendacji Trusted Advisor.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Sprawdzenie rekomendacji Trusted Advisor dla bezpieczeństwa**  
1. Przejdź do **AWS Management Console** → **Trusted Advisor**.  
2. Wybierz kategorię **Security**.  
3. Przejrzyj kluczowe kontrole bezpieczeństwa, takie jak:  
   - **Security Groups - Unrestricted Access** (publiczne porty)  
   - **IAM Use** (nadmierne uprawnienia IAM)  
   - **S3 Bucket Permissions** (publiczny dostęp do danych)  
   - **MFA on Root Account** (brak MFA na koncie root)  

---

**2. Optymalizacja Security Groups**  
1. Kliknij w rekomendację **Security Groups - Unrestricted Access**.  
2. Sprawdź grupy zabezpieczeń, które mają otwarte porty (np. `0.0.0.0/0`).  
3. Otwórz **AWS EC2** → **Security Groups**.  
4. Edytuj problematyczne grupy zabezpieczeń i ogranicz dostęp do określonych adresów IP.  

📌 **Teraz tylko uprawnione adresy IP mają dostęp do zasobów AWS!** 🎉  

---

**3. Ograniczenie nadmiernych uprawnień IAM**  
1. Kliknij w rekomendację **IAM Use**.  
2. Sprawdź listę użytkowników IAM z uprawnieniami **AdministratorAccess**.  
3. Otwórz **AWS IAM** → **Users**.  
4. Usuń nadmiarowe uprawnienia i zastosuj zasadę **Least Privilege Access**.  

📌 **Tylko wymagane uprawnienia są teraz przypisane do użytkowników IAM!**  

---

**4. Sprawdzenie publicznych bucketów S3**  
1. Kliknij w rekomendację **S3 Bucket Permissions**.  
2. Sprawdź, czy są jakieś publicznie dostępne buckety S3.  
3. Otwórz **AWS S3** → **Permissions** dla każdego bucketa.  
4. Jeśli bucket nie powinien być publiczny, usuń dostęp **Public Access**.  

📌 **Dane S3 są teraz chronione przed nieautoryzowanym dostępem!**
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Pobranie rekomendacji Trusted Advisor dotyczących bezpieczeństwa**  
📌 Pobierz **listę wszystkich kontroli Trusted Advisor**:  
```sh
aws support describe-trusted-advisor-checks --language en
```
📌 **Sprawdź ID dla kategorii bezpieczeństwa.**  

---

**2. Sprawdzenie otwartych Security Groups**  
```sh
aws support describe-trusted-advisor-check-result \
    --check-id e52a62f0-5f4d-4e26-92dc-576f5f7b8d5b \
    --language en
```
📌 **Zwróci listę Security Groups z otwartymi portami (`0.0.0.0/0`).**  

#**Ograniczenie dostępu w Security Group**  
```sh
aws ec2 revoke-security-group-ingress \
    --group-id sg-0123456789abcdef0 \
    --protocol tcp --port 22 --cidr 0.0.0.0/0
```
📌 **Usuwa regułę, która pozwala na publiczny dostęp SSH.**  

---

**3. Sprawdzenie użytkowników IAM z nadmiernymi uprawnieniami**  
```sh
aws iam list-users
```
📌 **Sprawdź, czy użytkownicy IAM mają przypisane pełne uprawnienia (`AdministratorAccess`).**  

#**Usunięcie uprawnień AdministratorAccess**  
```sh
aws iam detach-user-policy --user-name "TestUser" \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
📌 **Teraz użytkownik `TestUser` nie ma nadmiernych uprawnień.**  

---

**4. Sprawdzenie publicznych bucketów S3**  
```sh
aws s3api list-buckets --query "Buckets[].Name"
```
📌 **Sprawdź, które buckety są publicznie dostępne.**  

#**Usunięcie dostępu publicznego do bucketa**  
```sh
aws s3api put-public-access-block --bucket my-public-bucket \
    --public-access-block-configuration '{
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }'
```
📌 **Bucket `my-public-bucket` nie jest już dostępny publicznie.** 🎉  


</details>

---

## **9. Optymalizacja kosztów w Trusted Advisor**  
- Przejrzyj raporty dotyczące niewykorzystanych instancji EC2 i storage S3.  
- Zastosuj zalecenia dotyczące oszczędności, np. zmniejszenie rozmiaru instancji lub przeniesienie danych do S3 Glacier.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Przegląd rekomendacji Trusted Advisor dla optymalizacji kosztów**  
1. Przejdź do **AWS Management Console** → **Trusted Advisor**.  
2. Wybierz kategorię **Cost Optimization**.  
3. Przejrzyj kluczowe rekomendacje, takie jak:  
   - **Underutilized Amazon EC2 Instances** (Nieefektywne instancje EC2).  
   - **Idle Load Balancers** (Nieużywane Load Balancery).  
   - **Unassociated Elastic IP Addresses** (Nieprzypisane adresy Elastic IP).  
   - **Low Utilization Amazon EBS Volumes** (Nieużywane woluminy EBS).  
   - **Amazon S3 Bucket Lifecycle** (Zalecenia dotyczące oszczędności na S3).  

---

**2. Optymalizacja instancji EC2**  
1. Kliknij w **Underutilized Amazon EC2 Instances**.  
2. Sprawdź instancje EC2, które mają niskie użycie CPU i pamięci RAM.  
3. Otwórz **AWS EC2** → **Instances** i zdecyduj, czy możesz:  
   - **Zatrzymać instancję** (jeśli nie jest używana).  
   - **Zmniejszyć typ instancji** (np. z `t3.large` na `t3.medium`).  

📌 **Redukcja kosztów poprzez zmniejszenie zasobów EC2!** 🎉  

---

**3. Usunięcie nieużywanych Load Balancerów**  
1. Kliknij w **Idle Load Balancers**.  
2. Sprawdź, które Load Balancery nie obsługują ruchu.  
3. Otwórz **AWS EC2** → **Load Balancers** i usuń zbędne zasoby.  

📌 **Eliminacja nieużywanych Load Balancerów obniża koszty!**  

---

**4. Optymalizacja przechowywania danych (S3, EBS)**  
1. Kliknij w **Low Utilization Amazon EBS Volumes**.  
2. Sprawdź, które woluminy EBS mają niski poziom operacji IOPS.  
3. Otwórz **AWS EC2** → **Volumes** i zdecyduj, czy można:  
   - **Usunąć wolumen** (jeśli nie jest podłączony).  
   - **Zredukować rozmiar wolumenu**.  

📌 **Zarządzanie przechowywaniem danych pozwala oszczędzać na EBS i S3!**  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**5. Zastosowanie polityki lifecycle dla Amazon S3**  
1. Kliknij w **Amazon S3 Bucket Lifecycle**.  
2. Sprawdź, które dane mogą zostać przeniesione do **S3 Glacier**.  
3. Otwórz **AWS S3** → **Bucket** → **Management** → **Lifecycle rules**.  
4. Utwórz regułę, która automatycznie przenosi dane starsze niż 30 dni do Glacier.  

📌 **Przenoszenie nieaktywnych danych do Glacier znacznie obniża koszty!** 🎉  

---

## **📌 Rozwiązanie w AWS CLI**  

**1. Pobranie rekomendacji Trusted Advisor dla kosztów**  
```sh
aws support describe-trusted-advisor-check-result \
    --check-id e7f47a28-07c5-49db-a204-2f0a79c806c0 \
    --language en
```
📌 **Sprawdzi nieużywane EC2, EBS, Load Balancery i S3.**  

---

**2. Znalezienie nieużywanych instancji EC2**  
```sh
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,Monitoring.State]"
```
📌 **Sprawdź, które instancje są w stanie `stopped` lub mają niski monitoring CPU.**  

#**Zmniejszenie typu instancji**  
```sh
aws ec2 modify-instance-attribute --instance-id i-0123456789abcdef0 --instance-type "t3.medium"
```

📌 **Zmniejszenie instancji obniża koszty operacyjne!**  

---

**3. Usunięcie nieużywanych Load Balancerów**  
📌 Pobierz listę Load Balancerów:  
```sh
aws elb describe-load-balancers --query "LoadBalancerDescriptions[*].LoadBalancerName"
```
📌 Usuń Load Balancer, jeśli nie obsługuje ruchu:  
```sh
aws elb delete-load-balancer --load-balancer-name my-unused-lb
```
📌 **Usunięcie Load Balancera eliminuje niepotrzebne opłaty!**  

---

**4. Znalezienie i usunięcie nieużywanych woluminów EBS**  
📌 Pobierz listę niepodłączonych wolumenów EBS:  
```sh
aws ec2 describe-volumes --query "Volumes[?State=='available'].[VolumeId,Size]"
```
📌 Usuń wolumen EBS, jeśli nie jest używany:  
```sh
aws ec2 delete-volume --volume-id vol-0123456789abcdef0
```
📌 **Niepłatne wolumeny EBS nadal generują koszty – usuwaj je!**  

---

**5. Optymalizacja przechowywania danych w S3**  
📌 Pobierz listę bucketów:  
```sh
aws s3api list-buckets --query "Buckets[].Name"
```
📌 Utwórz politykę lifecycle dla S3, aby przenosić dane do Glacier:  
```sh
aws s3api put-bucket-lifecycle-configuration --bucket my-data-bucket --lifecycle-configuration '{
    "Rules": [
        {
            "ID": "MoveToGlacier",
            "Prefix": "",
            "Status": "Enabled",
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "GLACIER"
                }
            ]
        }
    ]
}'
```
📌 **Automatyczna archiwizacja danych oszczędza koszty!** 🎉 
</details>