## **Zadania praktyczne dla sekcji 4: EC2 Zarządzanie** 

## **1. Planowanie automatycznego uruchamiania i zatrzymywania instancji**  
- Skonfiguruj **AWS EventBridge** do automatycznego włączania i wyłączania instancji o określonych godzinach.

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie reguły EventBridge do automatycznego uruchamiania instancji**  
1. Przejdź do **AWS Management Console** → **EventBridge**.  
2. Kliknij **Rules** → **Create rule**.  
3. Wpisz **Rule name**: `StartEC2Instances`.  
4. W sekcji **Event source** wybierz **Schedule**.  
5. Wybierz **Cron expression** i ustaw harmonogram:  
   - Przykład: `0 7 * * ? *` → codziennie o **07:00 UTC**.  
6. W sekcji **Target** kliknij **EC2**.  
7. Wybierz **Action** → **StartInstances**.  
8. Wybierz instancję EC2 do uruchomienia.  
9. Kliknij **Create rule**.  

📌 **Instancja EC2 będzie automatycznie uruchamiana o 07:00 UTC!** 🎉  

---

### **2. Tworzenie reguły EventBridge do automatycznego zatrzymywania instancji**  
1. Wróć do **AWS EventBridge** → **Rules** → **Create rule**.  
2. Wpisz **Rule name**: `StopEC2Instances`.  
3. W sekcji **Event source** wybierz **Schedule**.  
4. Wybierz **Cron expression** i ustaw harmonogram:  
   - Przykład: `0 19 * * ? *` → codziennie o **19:00 UTC**.  
5. W sekcji **Target** kliknij **EC2**.  
6. Wybierz **Action** → **StopInstances**.  
7. Wybierz instancję EC2 do zatrzymania.  
8. Kliknij **Create rule**.  

📌 **Instancja EC2 będzie automatycznie zatrzymywana o 19:00 UTC!** 🎉  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie reguły EventBridge do uruchamiania instancji**  
```sh
aws events put-rule \
    --name "StartEC2Instances" \
    --schedule-expression "cron(0 7 * * ? *)"
```
📌 **Reguła uruchomi się codziennie o 07:00 UTC.**  

Dodanie instancji EC2 jako celu reguły:  
```sh
aws events put-targets \
    --rule "StartEC2Instances" \
    --targets "Id"="1","Arn"="arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
```

Przyznanie EventBridge uprawnień do uruchamiania instancji:  
```sh
aws iam put-role-policy \
    --role-name "AWSServiceRoleForEC2ScheduledEvents" \
    --policy-name "StartEC2Policy" \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "ec2:StartInstances",
                "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
            }
        ]
    }'
```

---

### **2. Tworzenie reguły EventBridge do zatrzymywania instancji**  
```sh
aws events put-rule \
    --name "StopEC2Instances" \
    --schedule-expression "cron(0 19 * * ? *)"
```
📌 **Reguła zatrzyma instancję codziennie o 19:00 UTC.**  

Dodanie instancji EC2 jako celu reguły:  
```sh
aws events put-targets \
    --rule "StopEC2Instances" \
    --targets "Id"="1","Arn"="arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
```

Przyznanie EventBridge uprawnień do zatrzymywania instancji:  
```sh
aws iam put-role-policy \
    --role-name "AWSServiceRoleForEC2ScheduledEvents" \
    --policy-name "StopEC2Policy" \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "ec2:StopInstances",
                "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
            }
        ]
    }'
```
</details>

---

## **2. Tworzenie własnego obrazu AMI z instancji EC2**  
- Uruchom instancję EC2, zainstaluj dodatkowe oprogramowanie i utwórz **obraz AMI**.  
- Uruchom nową instancję z utworzonego obrazu.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Przygotowanie instancji EC2 przed utworzeniem AMI**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz instancję, którą chcesz sklonować do obrazu AMI.  
3. (Opcjonalnie) Jeśli chcesz uwzględnić najnowsze zmiany, uruchom w terminalu:  
   ```sh
   sudo yum update -y  # Dla Amazon Linux / RHEL
   sudo apt update -y  # Dla Ubuntu / Debian
   ```
4. (Opcjonalnie) Uruchom ponownie instancję, aby zastosować zmiany:  
   ```sh
   sudo reboot
   ```

---

### **2. Tworzenie obrazu AMI**  
1. Przejdź do **AWS EC2** → **Instances**.  
2. Wybierz instancję, którą chcesz sklonować.  
3. Kliknij **Actions** → **Image and templates** → **Create Image**.  
4. Wpisz **Image name**, np. `MyCustomAMI`.  
5. (Opcjonalnie) Dodaj opis obrazu.  
6. Wybierz **No reboot**, jeśli nie chcesz restartować instancji.  
7. Kliknij **Create Image**.  

📌 **AWS rozpocznie tworzenie obrazu AMI. Proces ten może potrwać kilka minut.**  

---

### **3. Sprawdzenie gotowego obrazu AMI**  
1. Przejdź do **AWS EC2** → **Images** → **AMIs**.  
2. Znajdź utworzony obraz `MyCustomAMI`.  
3. Po zakończeniu procesu status zmieni się na **Available**.  

📌 **Teraz możesz użyć AMI do tworzenia nowych instancji EC2!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Pobranie ID instancji EC2**  
```sh
aws ec2 describe-instances --query "Reservations[*].Instances[*].InstanceId" --output text
```
Załóżmy, że ID instancji to `i-0123456789abcdef0`.

---

### **2. Tworzenie obrazu AMI z instancji EC2**  
```sh
aws ec2 create-image \
    --instance-id i-0123456789abcdef0 \
    --name "MyCustomAMI" \
    --description "Custom AMI created from instance i-0123456789abcdef0" \
    --no-reboot
```
📌 **Dodaj `--no-reboot`, jeśli nie chcesz restartować instancji podczas tworzenia obrazu.**  

---

### **3. Sprawdzenie statusu obrazu AMI**  
```sh
aws ec2 describe-images --owners self --query "Images[*].[ImageId,State,Name]" --output table
```
📌 **Kiedy `State` zmieni się na `available`, AMI jest gotowe do użycia!** 🎉  

---

### **4. Tworzenie nowej instancji EC2 z obrazu AMI**  
Po utworzeniu AMI możesz z niego uruchomić nową instancję:  
```sh
aws ec2 run-instances \
    --image-id ami-0123456789abcdef0 \
    --count 1 \
    --instance-type t2.micro \
    --key-name my-key \
    --security-groups my-security-group
```
📌 **Podmień `ami-0123456789abcdef0` na ID swojego AMI.**  

</details>

---

## **3. Udostępnianie AMI innym kontom AWS**  
- Skonfiguruj uprawnienia AMI tak, aby było dostępne dla wybranych kont AWS.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Znalezienie utworzonego obrazu AMI**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. W lewym menu kliknij **AMIs** (Images).  
3. Znajdź obraz AMI, który chcesz udostępnić.  
4. Zaznacz obraz i kliknij **Actions** → **Modify Image Permissions**.  

---

### **2. Udostępnienie AMI innemu kontu AWS**  
1. W sekcji **Image Permissions** wybierz **Private**.  
2. W polu **Account ID** wpisz identyfikator konta AWS, któremu chcesz udostępnić obraz.  
3. Kliknij **Add Permission**.  
4. Kliknij **Save Changes**.  

📌 **Teraz AMI jest dostępne dla innego konta AWS!** 🎉  

---

### **3. Udostępnienie powiązanych snapshotów (jeśli AMI korzysta z EBS)**  
Jeśli obraz AMI używa wolumenów **EBS**, musisz również udostępnić powiązany snapshot:  
1. Przejdź do **AWS EC2** → **Snapshots**.  
2. Znajdź snapshot powiązany z AMI.  
3. Kliknij **Modify Permissions**.  
4. Dodaj identyfikator konta AWS i kliknij **Save Changes**.  

📌 **Teraz inne konto może korzystać zarówno z AMI, jak i z jego snapshotu!**  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Pobranie listy dostępnych AMI**  
```sh
aws ec2 describe-images --owners self --query "Images[*].[ImageId,Name]" --output table
```
Załóżmy, że AMI ma ID: `ami-0123456789abcdef0`.

---

### **2. Udostępnienie AMI innemu kontu AWS**  
```sh
aws ec2 modify-image-attribute \
    --image-id ami-0123456789abcdef0 \
    --launch-permission "Add=[{UserId='111122223333'}]"
```
📌 **Zmień `111122223333` na ID konta, któremu chcesz udostępnić obraz.**  

---

### **3. Udostępnienie powiązanego snapshotu (jeśli AMI używa EBS)**  
📌 Najpierw znajdź snapshot powiązany z AMI:  
```sh
aws ec2 describe-images --image-ids ami-0123456789abcdef0 \
    --query "Images[*].BlockDeviceMappings[*].Ebs.SnapshotId"
```
Załóżmy, że snapshot ma ID: `snap-0123456789abcdef0`.

📌 Udostępnij snapshot innemu kontu AWS:  
```sh
aws ec2 modify-snapshot-attribute \
    --snapshot-id snap-0123456789abcdef0 \
    --attribute createVolumePermission \
    --operation-type add \
    --user-ids 111122223333
```

📌 **Teraz inne konto może używać AMI oraz jego snapshotu!** 🎉  

---

### **4. Sprawdzenie, czy AMI zostało poprawnie udostępnione**  
```sh
aws ec2 describe-image-attribute \
    --image-id ami-0123456789abcdef0 \
    --attribute launchPermission
```
📌 **Powinieneś zobaczyć ID konta, któremu udostępniono AMI.**  

</details>

---

## **4. Tworzenie i przywracanie snapshotów EBS**  
- Utwórz **snapshot** wolumenu EBS dla instancji EC2.  
- Przywróć snapshot do nowego wolumenu i podłącz go do innej instancji.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie snapshotu EBS**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. W lewym menu kliknij **Volumes**.  
3. Wybierz wolumen powiązany z Twoją instancją EC2.  
4. Kliknij **Actions** → **Create snapshot**.  
5. Wpisz **Description**, np. `Backup for EC2 Instance`.  
6. Kliknij **Create snapshot**.  

📌 **AWS rozpocznie tworzenie snapshotu, co może potrwać kilka minut.** 🎉  

---

### **2. Sprawdzenie statusu snapshotu**  
1. Przejdź do **AWS EC2** → **Snapshots**.  
2. Znajdź utworzony snapshot – status powinien zmienić się na **Completed**.  

---

### **3. Tworzenie nowego wolumenu EBS z istniejącego snapshotu**  
1. Przejdź do **AWS EC2** → **Snapshots**.  
2. Zaznacz snapshot i kliknij **Actions** → **Create Volume**.  
3. Wybierz ten sam **region** co instancja EC2.  
4. Wybierz **typ wolumenu** (np. **gp3** dla SSD).  
5. Kliknij **Create Volume**.  

📌 **Wolumen EBS został utworzony na podstawie snapshotu!** 🎉  

---

### **4. Podłączenie nowego wolumenu do instancji EC2**  
1. Przejdź do **AWS EC2** → **Volumes**.  
2. Zaznacz nowy wolumen i kliknij **Actions** → **Attach Volume**.  
3. Wybierz instancję EC2 i nazwę urządzenia (np. `/dev/sdf`).  
4. Kliknij **Attach**.  

📌 **Teraz instancja EC2 ma dostęp do nowego wolumenu!**  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie snapshotu EBS**  
📌 Pobierz **ID wolumenu**, który chcesz zbackupować:  
```sh
aws ec2 describe-volumes --query "Volumes[*].[VolumeId,Size,State]" --output table
```
Załóżmy, że wolumen ma ID: `vol-0123456789abcdef0`.  

📌 Utwórz snapshot:  
```sh
aws ec2 create-snapshot --volume-id vol-0123456789abcdef0 --description "Backup for EC2 Instance"
```

---

### **2. Sprawdzenie statusu snapshotu**  
```sh
aws ec2 describe-snapshots --query "Snapshots[*].[SnapshotId,State,StartTime]" --output table
```
📌 **Gdy `State` zmieni się na `completed`, snapshot jest gotowy.**  

---

### **3. Tworzenie nowego wolumenu EBS ze snapshotu**  
📌 Pobierz **ID snapshotu**:  
```sh
aws ec2 describe-snapshots --query "Snapshots[*].[SnapshotId]" --output text
```
Załóżmy, że snapshot ma ID: `snap-0123456789abcdef0`.  

📌 Utwórz nowy wolumen w tym samym regionie:  
```sh
aws ec2 create-volume --snapshot-id snap-0123456789abcdef0 --availability-zone us-east-1a --volume-type gp3
```
📌 **Zmień `us-east-1a` na strefę, w której działa Twoja instancja.**  

---

### **4. Podłączenie wolumenu do instancji EC2**  
📌 Pobierz **ID instancji EC2**, do której chcesz podłączyć nowy wolumen:  
```sh
aws ec2 describe-instances --query "Reservations[*].Instances[*].InstanceId" --output text
```
Załóżmy, że instancja ma ID: `i-0123456789abcdef0`.  

📌 Podłącz nowy wolumen do instancji EC2:  
```sh
aws ec2 attach-volume --volume-id vol-0123456789abcdef0 --instance-id i-0123456789abcdef0 --device /dev/sdf
```

📌 **Instancja EC2 ma teraz dostęp do nowego wolumenu EBS opartego na snapshotcie!** 🎉  

</details>

---

## **5. Automatyzacja tworzenia snapshotów EBS**  
- Skonfiguruj **AWS Data Lifecycle Manager**, aby automatycznie tworzył snapshoty co 24 godziny.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie polityki automatycznego tworzenia snapshotów**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. W lewym menu wybierz **Lifecycle Manager** (w sekcji Elastic Block Store).  
3. Kliknij **Create lifecycle policy**.  
4. Wybierz **Policy type**: **EBS Snapshot Policy**.  
5. Kliknij **Next**.  

---

### **2. Konfiguracja reguł tworzenia snapshotów**  
1. **Target resources**:  
   - Wybierz **Instance or Volume tags** i dodaj tag, np.  
     - **Key**: `Backup`  
     - **Value**: `Daily`  
   - Snapshoty będą tworzone dla wolumenów EBS z tym tagiem.  
2. **Schedule**:  
   - Wpisz **Policy Name**: `Daily-EBS-Snapshots`.  
   - Wybierz **Frequency**: `Every 24 hours`.  
   - Wybierz **Start Time**: np. `02:00 UTC`.  
   - Określ liczbę snapshotów do zachowania, np. **7** (stare snapshoty będą automatycznie usuwane).  
3. **IAM Permissions**:  
   - System automatycznie przypisze wymagane uprawnienia.  
4. Kliknij **Create Policy**.  

📌 **Teraz AWS DLM będzie automatycznie tworzył snapshoty EBS co 24 godziny!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie polityki Data Lifecycle Manager (DLM)**  
```sh
aws dlm create-lifecycle-policy \
    --execution-role-arn arn:aws:iam::123456789012:role/AWSDataLifecycleManagerDefaultRole \
    --description "Daily snapshot policy for EBS volumes" \
    --state ENABLED \
    --policy-details '{
        "ResourceTypes": ["VOLUME"],
        "TargetTags": [{"Key": "Backup", "Value": "Daily"}],
        "Schedules": [{
            "Name": "Daily EBS Snapshots",
            "TagsToAdd": [{"Key": "SnapshotType", "Value": "Automated"}],
            "CreateRule": {"Interval": 24, "IntervalUnit": "HOURS", "Times": ["02:00"]},
            "RetainRule": {"Count": 7}
        }]
    }'
```
📌 **Zmień `123456789012` na swój numer konta AWS.**  

---

### **2. Sprawdzenie statusu polityki**  
```sh
aws dlm get-lifecycle-policies
```
📌 **Wynik powinien pokazać politykę z `State: ENABLED`.**  

---

### **3. Dodanie tagu do wolumenu EBS, aby polityka działała**  
📌 Pobierz **ID wolumenu**, który chcesz objąć polityką:  
```sh
aws ec2 describe-volumes --query "Volumes[*].[VolumeId]" --output text
```
Załóżmy, że wolumen ma ID: `vol-0123456789abcdef0`.  

📌 Dodaj tag `Backup: Daily`:  
```sh
aws ec2 create-tags --resources vol-0123456789abcdef0 --tags Key=Backup,Value=Daily
```

📌 **Teraz AWS DLM automatycznie tworzy snapshoty dla wolumenów oznaczonych tagiem `Backup: Daily`.** 🎉  

---

### **4. Sprawdzenie utworzonych snapshotów**  
```sh
aws ec2 describe-snapshots --owner-id 123456789012 --query "Snapshots[*].[SnapshotId,StartTime]" --output table
```
📌 **Jeśli snapshoty są tworzone zgodnie z harmonogramem, polityka działa poprawnie!**  

</details>

---

## **6. Kopiowanie snapshotów do innego regionu AWS**  
- Skopiuj snapshot EBS do innego regionu w AWS Console i CLI.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Znalezienie snapshotu do skopiowania**
1. Przejdź do **AWS Management Console** → **EC2**.  
2. W lewym menu kliknij **Snapshots**.  
3. Znajdź snapshot, który chcesz skopiować, i zaznacz go.  
4. Kliknij **Actions** → **Copy Snapshot**.  

---

### **2. Konfiguracja kopiowania snapshotu**
1. W polu **Destination region** wybierz region docelowy (np. `us-west-2`).  
2. Wpisz **Description**, np. `Copy of snapshot for backup`.  
3. (Opcjonalnie) Wybierz **Encryption**:  
   - Jeśli snapshot był zaszyfrowany, zostanie skopiowany z tą samą konfiguracją.  
   - Jeśli snapshot nie był zaszyfrowany, możesz wybrać nowy klucz KMS.  
4. Kliknij **Copy Snapshot**.  

📌 **AWS rozpocznie kopiowanie snapshotu. Może to potrwać kilka minut, w zależności od jego rozmiaru.** 🎉  

---

### **3. Sprawdzenie skopiowanego snapshotu**
1. Zmień region w prawym górnym rogu na region docelowy (`us-west-2`).  
2. Przejdź do **EC2** → **Snapshots**.  
3. Znajdź nowo skopiowany snapshot – powinien mieć status **completed**.  

📌 **Snapshot został pomyślnie skopiowany do nowego regionu!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Pobranie listy snapshotów w regionie źródłowym**  
```sh
aws ec2 describe-snapshots --owner-id 123456789012 --region us-east-1 \
    --query "Snapshots[*].[SnapshotId,StartTime]" --output table
```
📌 **Zmień `123456789012` na swój numer konta AWS.**  

Załóżmy, że snapshot do skopiowania ma ID: `snap-0123456789abcdef0`.

---

### **2. Kopiowanie snapshotu do nowego regionu**  
```sh
aws ec2 copy-snapshot \
    --source-region us-east-1 \
    --source-snapshot-id snap-0123456789abcdef0 \
    --destination-region us-west-2 \
    --description "Backup copy of snapshot snap-0123456789abcdef0"
```
📌 **Zmień `us-east-1` i `us-west-2` na odpowiednie regiony.**  

---

### **3. Sprawdzenie statusu kopiowanego snapshotu**  
```sh
aws ec2 describe-snapshots --owner-id 123456789012 --region us-west-2 \
    --query "Snapshots[*].[SnapshotId,State,StartTime]" --output table
```
📌 **Gdy `State` zmieni się na `completed`, snapshot jest gotowy do użycia w nowym regionie!** 🎉  

---

### **4. Tworzenie wolumenu EBS z nowego snapshotu w regionie docelowym**  
```sh
aws ec2 create-volume \
    --snapshot-id snap-0123456789abcdef0 \
    --availability-zone us-west-2a \
    --volume-type gp3 \
    --region us-west-2
```
📌 **Zmień `snap-0123456789abcdef0` na ID skopiowanego snapshotu i `us-west-2a` na odpowiednią strefę dostępności.**  
</details>

---

## **7. Automatyczne patchowanie instancji EC2 za pomocą AWS Systems Manager**  
- Skonfiguruj **AWS Systems Manager Patch Manager**, aby automatycznie aktualizował system operacyjny instancji EC2.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Konfiguracja AWS Systems Manager na instancji EC2**
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz instancję EC2 i przejdź do zakładki **Permissions**.  
3. Upewnij się, że instancja ma rolę IAM z polityką **AmazonSSMManagedInstanceCore**:  
   - Jeśli nie ma, utwórz nową rolę IAM dla EC2 i dodaj ją do instancji.  

---

### **2. Tworzenie polityki aktualizacji w AWS Systems Manager**
1. Przejdź do **AWS Systems Manager** → **Patch Manager**.  
2. Kliknij **Configure patch policies**.  
3. Wybierz opcję **Create patch policy**.  
4. W sekcji **Targets** wybierz instancje do aktualizacji (np. wszystkie z określonym tagiem).  
5. Wybierz **Operating System**:  
   - **Amazon Linux 2**, **Ubuntu**, **Windows**, itp.  
6. W sekcji **Patch classification** wybierz:  
   - `Security updates` (dla bezpieczeństwa)  
   - `Critical updates` (dla ważnych poprawek)  
7. Wybierz harmonogram (np. co tydzień, w soboty o 02:00 UTC).  
8. Kliknij **Create**.  

📌 **AWS Systems Manager będzie teraz automatycznie aktualizować instancje EC2!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Dodanie roli IAM do instancji EC2**
📌 Sprawdź, czy instancja EC2 ma odpowiednią rolę:  
```sh
aws ec2 describe-instances \
    --query "Reservations[*].Instances[*].[InstanceId,IamInstanceProfile.Arn]" --output table
```
📌 Jeśli brak roli IAM, utwórz ją:  
```sh
aws iam create-role --role-name SSM-EC2-Role \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": { "Service": "ec2.amazonaws.com" },
                "Action": "sts:AssumeRole"
            }
        ]
    }'
```
📌 Przypisz politykę **AmazonSSMManagedInstanceCore** do roli:  
```sh
aws iam attach-role-policy \
    --role-name SSM-EC2-Role \
    --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```
📌 Przypisz rolę IAM do instancji EC2:  
```sh
aws ec2 associate-iam-instance-profile \
    --instance-id i-0123456789abcdef0 \
    --iam-instance-profile Name=SSM-EC2-Role
```

---

### **2. Tworzenie polityki aktualizacji w Systems Manager**
📌 Utwórz politykę Patch Manager dla Amazon Linux 2:  
```sh
aws ssm create-patch-baseline \
    --name "SecurityUpdates" \
    --operating-system AMAZON_LINUX_2 \
    --approval-rules "PatchRules=[{PatchFilterGroup={PatchFilters=[{Key='CLASSIFICATION',Values=['Security']}]},ApproveAfterDays=0}]"
```
📌 **Zmień `AMAZON_LINUX_2` na odpowiedni system operacyjny, jeśli używasz innego.**  

---

### **3. Przypisanie polityki do instancji EC2**  
📌 Pobierz **ID polityki** Patch Manager:  
```sh
aws ssm describe-patch-baselines --query "BaselineIdentities[*].[BaselineId,Name]" --output table
```
📌 Przypisz politykę do instancji:  
```sh
aws ssm register-patch-baseline-for-patch-group \
    --baseline-id pb-0123456789abcdef0 \
    --patch-group "Production"
```

---

### **4. Wymuszenie aktualizacji systemu operacyjnego**
📌 Uruchom proces aktualizacji:  
```sh
aws ssm send-command \
    --document-name "AWS-RunPatchBaseline" \
    --targets "Key=tag:Environment,Values=Production" \
    --parameters "Operation=Install"
```

---

### **5. Sprawdzenie statusu aktualizacji**  
📌 Monitorowanie statusu aktualizacji:  
```sh
aws ssm list-command-invocations \
    --query "CommandInvocations[*].[CommandId,InstanceId,Status]" --output table
```
📌 **Gdy `Status` zmieni się na `Success`, aktualizacja została zakończona!** 🎉  
</details>

---

## **8. Konfiguracja AWS Systems Manager Session Manager**  
- Połącz się do instancji EC2 za pomocą Session Manager, bez potrzeby korzystania z klucza SSH.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Upewnienie się, że instancja EC2 ma odpowiednią rolę IAM**  
1. Przejdź do **AWS Management Console** → **EC2**.  
2. Wybierz swoją instancję EC2 i przejdź do zakładki **Security**.  
3. Sprawdź, czy instancja ma przypisaną rolę IAM.  
4. Jeśli brak roli IAM, utwórz nową rolę:  
   - Przejdź do **AWS IAM** → **Roles** → **Create role**.  
   - Wybierz **AWS service** → **EC2**.  
   - Wyszukaj i dodaj politykę: **AmazonSSMManagedInstanceCore**.  
   - Nazwij rolę np. `SSM-Session-Manager-Role` i kliknij **Create role**.  
   - Wróć do EC2 → **Modify IAM role** i przypisz nowo utworzoną rolę do instancji.  

📌 **Teraz instancja EC2 ma uprawnienia do korzystania z Systems Manager!** 🎉  

---

### **2. Włączenie AWS Systems Manager Agent na instancji**  
📌 Jeśli korzystasz z **Amazon Linux 2** lub **Ubuntu**, agent jest domyślnie zainstalowany. Sprawdź jego status:  
1. Połącz się do instancji (jeśli masz dostęp SSH).  
2. Uruchom polecenie:  
   ```sh
   sudo systemctl status amazon-ssm-agent
   ```
3. Jeśli agent nie jest uruchomiony, uruchom go:  
   ```sh
   sudo systemctl start amazon-ssm-agent
   ```
4. Jeśli nie masz zainstalowanego agenta, zainstaluj go:  
   ```sh
   sudo yum install -y amazon-ssm-agent  # Amazon Linux / RHEL
   sudo apt install -y amazon-ssm-agent  # Ubuntu / Debian
   ```

📌 **Agent SSM jest teraz gotowy do obsługi Session Managera!** 🎉  

---

### **3. Połączenie z instancją EC2 przez Session Manager**  
1. Przejdź do **AWS Systems Manager** → **Session Manager**.  
2. Kliknij **Start Session**.  
3. Wybierz swoją instancję EC2 i kliknij **Start Session**.  
4. Otworzy się terminal w przeglądarce, gdzie możesz zarządzać instancją.  

📌 **Teraz możesz zarządzać instancją EC2 bez SSH!** 🎉  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Sprawdzenie, czy instancja EC2 ma przypisaną rolę IAM**  
```sh
aws ec2 describe-instances \
    --query "Reservations[*].Instances[*].[InstanceId,IamInstanceProfile.Arn]" --output table
```
📌 **Jeśli instancja nie ma przypisanej roli IAM, wykonaj następne kroki, aby dodać rolę.**  

---

### **2. Tworzenie roli IAM dla Session Manager**  
📌 Utwórz rolę IAM dla EC2:  
```sh
aws iam create-role --role-name SSM-Session-Manager-Role \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": { "Service": "ec2.amazonaws.com" },
                "Action": "sts:AssumeRole"
            }
        ]
    }'
```
📌 Dodaj uprawnienia **AmazonSSMManagedInstanceCore** do roli:  
```sh
aws iam attach-role-policy \
    --role-name SSM-Session-Manager-Role \
    --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```
📌 Przypisz rolę do instancji EC2:  
```sh
aws ec2 associate-iam-instance-profile \
    --instance-id i-0123456789abcdef0 \
    --iam-instance-profile Name=SSM-Session-Manager-Role
```

---

### **3. Połączenie do instancji EC2 przez Session Manager w CLI**  
📌 Aby połączyć się do instancji EC2 przez AWS CLI:  
```sh
aws ssm start-session --target i-0123456789abcdef0
```
📌 **Teraz możesz zarządzać instancją bez SSH!** 🎉  

---

### **4. Sprawdzenie aktywnych sesji w Session Manager**  
```sh
aws ssm describe-sessions --state Active
```

📌 **Zobaczysz listę aktywnych sesji dla swojej instancji EC2.**  

---

### **5. Zakończenie sesji Session Manager**  
📌 Aby zakończyć sesję, użyj:  
```sh
aws ssm terminate-session --session-id session-0a1b2c3d4e5f6g7h8
```

📌 **Teraz sesja została zamknięta!** 🎉  

</details>
