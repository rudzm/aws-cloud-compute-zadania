### **Zadania praktyczne dla sekcji 3: Podstawowe usługi AWS (EC2, VPC, S3, EBS)**  

---

# **Amazon EC2**  
**Uruchomienie instancji EC2**  
- Utwórz nową instancję EC2 z systemem **Amazon Linux 2**.  
- Wybierz **typu t3.micro** (Free Tier).  
- Skonfiguruj klucz SSH i połącz się z instancją.  
- Zainstaluj serwer Apache i uruchom stronę testową (`sudo yum install httpd -y && sudo systemctl start httpd`). 

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

1. Przejdź do **AWS Management Console** → **EC2** → **Launch Instance**.  
2. Wybierz **Amazon Linux 2 AMI**.  
3. Wybierz typ instancji **t3.micro** (Free Tier).  
4. Skonfiguruj **Security Group**, pozwalając na **SSH (22), HTTP (80), HTTPS (443)**.  
5. Przypisz klucz SSH i uruchom instancję.  
6. Połącz się przez SSH:  
   ```sh
   ssh -i MyKey.pem ec2-user@<public-ip>
   ```
7. Zainstaluj serwer Apache:  
   ```sh
   sudo yum install httpd -y
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

```sh
aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t3.micro \
  --key-name MyKeyPair --security-groups WebServerSG
```
</details>

---

# **Amazon VPC**  
**Tworzenie własnej sieci VPC**  
- Utwórz **nowe VPC** z adresem **10.0.0.0/16**.  
- Skonfiguruj **dwa subnety**:  
  - Publiczny subnet: `10.0.1.0/24` (dostęp do Internetu).  
  - Prywatny subnet: `10.0.2.0/24` (bez dostępu do Internetu).  
- Skonfiguruj **Internet Gateway** dla publicznego subnetu.  
- Uruchom instancję EC2 w prywatnym subnecie i skonfiguruj dostęp przez **NAT Gateway**.  

<details>
  <summary>Instrukcja krok po kroku AWS GUI</summary>

  1. **Tworzenie VPC**  
    - Przejdź do **AWS Management Console** → **VPC** → **Create VPC**.  
    - Nazwij VPC: `MyCustomVPC`.  
    - Ustaw **IPv4 CIDR**: `10.0.0.0/16`.  
    - Kliknij **Create VPC**.  
  2. **Tworzenie Subnetów**  
    - Przejdź do **Subnets** → **Create Subnet**.  
    - Wybierz `MyCustomVPC`.  
    - Utwórz **publiczny subnet** (`10.0.1.0/24`).  
    - Utwórz **prywatny subnet** (`10.0.2.0/24`).  
  3. **Tworzenie Internet Gateway (IGW)**  
    - Przejdź do **Internet Gateways** → **Create IGW**.  
    - Nazwij IGW: `MyInternetGateway`.  
    - Przypisz go do **MyCustomVPC** → **Attach to VPC**.  
  4. **Konfiguracja Route Table dla Public Subnet**  
    - Przejdź do **Route Tables** → Wybierz domyślną tabelę.  
    - Kliknij **Edit Routes** → **Add Route**.  
    - Ustaw **Destination**: `0.0.0.0/0`.  
    - Wybierz **Target**: `MyInternetGateway`.  
    - **Save changes**. 

</details>
<details>
  <summary>Instrukcja krok po kroku AWS CLI</summary>

```sh
# Tworzenie VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Tworzenie subnetów
aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.2.0/24

# Tworzenie Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-12345678 --internet-gateway-id igw-12345678

# Tworzenie Route Table i dodanie reguły
aws ec2 create-route-table --vpc-id vpc-12345678
aws ec2 create-route --route-table-id rtb-12345678 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-12345678
```

</details>

**Konfiguracja Security Groups i Network ACLs**  
- Utwórz **Security Group**, która zezwala na ruch:  
  - `HTTP (80)` i `HTTPS (443)` dla wszystkich (`0.0.0.0/0`).  
  - `SSH (22)` tylko dla Twojego IP.  
- Skonfiguruj **Network ACLs**, blokując cały ruch spoza Twojego VPC.  

<details>
  <summary>Instrukcja krok po kroku AWS GUI</summary>

1. **Tworzenie Security Group dla EC2**  
   - Przejdź do **EC2** → **Security Groups** → **Create Security Group**.  
   - **Name**: `WebServerSG`.  
   - **Inbound Rules**:  
     - **Allow HTTP (80)** → `0.0.0.0/0`.  
     - **Allow HTTPS (443)** → `0.0.0.0/0`.  
     - **Allow SSH (22)** → **Your IP**.  

2. **Tworzenie Network ACL**  
   - Przejdź do **VPC** → **Network ACLs** → **Create Network ACL**.  
   - Wybierz **MyCustomVPC**.  
   - **Inbound Rules**:  
     - Rule #100: **Allow HTTP (80)** → `0.0.0.0/0`.  
     - Rule #110: **Allow HTTPS (443)** → `0.0.0.0/0`.  
     - Rule #120: **Allow SSH (22)** → **Your IP**.  
   - **Outbound Rules**:  
     - Rule #100: **Allow ALL Traffic** → `0.0.0.0/0`. 

</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

```sh
# Tworzenie Security Group
aws ec2 create-security-group --group-name WebServerSG --description "Security Group for Web Server" --vpc-id vpc-12345678

# Dodanie reguł do Security Group
aws ec2 authorize-security-group-ingress --group-name WebServerSG --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name WebServerSG --protocol tcp --port 22 --cidr <your-ip>/32

# Tworzenie Network ACL
aws ec2 create-network-acl --vpc-id vpc-12345678

# Dodanie reguł do Network ACL
aws ec2 create-network-acl-entry --network-acl-id acl-12345678 --rule-number 100 --protocol tcp --port-range From=80,To=80 --cidr-block 0.0.0.0/0 --rule-action allow
aws ec2 create-network-acl-entry --network-acl-id acl-12345678 --rule-number 110 --protocol tcp --port-range From=443,To=443 --cidr-block 0.0.0.0/0 --rule-action allow
```

</details>

**Połączenie dwóch instancji EC2 w różnych subnetach**  
- Uruchom dwie instancje EC2 w różnych subnetach tej samej VPC.  
- Skonfiguruj trasowanie w tabeli routingu, aby mogły się komunikować.  
- Przetestuj połączenie za pomocą `ssh`.  


<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

1. **Uruchomienie dwóch instancji EC2**  
   - Przejdź do **EC2** → **Launch Instance**.  
   - Wybierz **Amazon Linux 2**.  
   - **Publiczna instancja** → **PublicSubnet**, włącz **Auto-assign Public IP**.  
   - **Prywatna instancja** → **PrivateSubnet**, wyłącz **Auto-assign Public IP**.  
   - Wybierz **WebServerSG**.  

2. **Połączenie między instancjami**  
   - Połącz się z **publiczną instancją**:  
     ```sh
     ssh -i MyKey.pem ec2-user@<public-ip>
     ```
   - Sprawdź **Private IP** drugiej instancji.  
   - Połącz się do niej z publicznej instancji:  
     ```sh
     ssh ec2-user@<private-ip>
     ```

</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

```sh
# Uruchomienie publicznej instancji
aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t3.micro --subnet-id subnet-12345678 --key-name MyKey --security-groups WebServerSG

# Uruchomienie prywatnej instancji
aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t3.micro --subnet-id subnet-87654321 --key-name MyKey --security-groups WebServerSG
```
</details>

---

# **Amazon EBS**  
**Tworzenie i podłączanie woluminu EBS do EC2**  
- Utwórz **wolumin EBS** o pojemności **10 GB**.  
- Podłącz go do istniejącej instancji EC2.  
- Sformatuj dysk (`mkfs -t xfs /dev/xvdf`) i zamontuj (`mount /dev/xvdf /mnt`).  
- Zapisz plik testowy i sprawdź, czy dane są przechowywane po restarcie instancji. 

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie woluminu EBS**  
1. Przejdź do **AWS Management Console** → **EC2 Dashboard**.  
2. W menu bocznym kliknij **Volumes** (sekcja "Elastic Block Store").  
3. Kliknij **Create Volume**.  
4. Skonfiguruj wolumin:  
   - **Size (GB)**: `10` (lub więcej, według potrzeb).  
   - **Volume Type**: `gp3` (General Purpose SSD).  
   - **Availability Zone**: Wybierz **tę samą AZ**, w której działa Twoja instancja EC2.  
   - **Encryption**: Opcjonalne, jeśli chcesz szyfrować dane.  
5. Kliknij **Create Volume**.  

---

### **2. Podłączanie woluminu do instancji EC2**  
1. Wróć do zakładki **Volumes** w EC2.  
2. Znajdź utworzony wolumin i zaznacz go.  
3. Kliknij **Actions** → **Attach Volume**.  
4. Wybierz swoją instancję EC2.  
5. Zmień urządzenie na `/dev/xvdf` lub pozostaw domyślne `/dev/sdf`.  
6. Kliknij **Attach Volume**.  

---

### **3. Formatowanie i montowanie dysku w systemie operacyjnym EC2**  
1. Połącz się z instancją EC2 przez SSH:  
   ```sh
   ssh -i MyKey.pem ec2-user@<public-ip>
   ```
2. Sprawdź, czy wolumin jest widoczny w systemie:  
   ```sh
   lsblk
   ```
   Oczekiwany wynik (nowy dysk `/dev/xvdf` lub `/dev/nvme1n1` dla instancji w Nitro):  
   ```
   NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   xvda    202:0    0   10G  0 disk /
   xvdf    202:1    0   10G  0 disk 
   ```

3. Jeśli wolumin jest nowy, sformatuj go (np. w systemie plików XFS):  
   ```sh
   sudo mkfs -t xfs /dev/xvdf
   ```
4. Utwórz punkt montowania:  
   ```sh
   sudo mkdir /mnt/my-ebs-volume
   ```
5. Zamontuj wolumin:  
   ```sh
   sudo mount /dev/xvdf /mnt/my-ebs-volume
   ```
6. Sprawdź, czy wolumin jest zamontowany:  
   ```sh
   df -h
   ```
   Oczekiwany wynik:  
   ```
   Filesystem      Size  Used Avail Use% Mounted on
   /dev/xvdf       10G   0G   10G  0%   /mnt/my-ebs-volume
   ```

---

### **4. Automatyczne montowanie woluminu po restarcie**  
1. Pobierz identyfikator UUID nowego woluminu:  
   ```sh
   sudo blkid /dev/xvdf
   ```
   Przykładowy wynik:  
   ```
   /dev/xvdf: UUID="1234abcd-5678-efgh-ijkl-98765mnopqrs" TYPE="xfs"
   ```
2. Edytuj plik `/etc/fstab`:  
   ```sh
   sudo nano /etc/fstab
   ```
3. Dodaj nową linię, aby automatycznie montować dysk po restarcie:  
   ```
   UUID=1234abcd-5678-efgh-ijkl-98765mnopqrs /mnt/my-ebs-volume xfs defaults,nofail 0 2
   ```
4. Zapisz zmiany i zamknij plik (`CTRL + X`, potem `Y`, następnie `Enter`).  
5. Sprawdź poprawność wpisów w `/etc/fstab`:  
   ```sh
   sudo mount -a
   ```
6. Uruchom ponownie instancję i sprawdź, czy wolumin jest zamontowany po restarcie:  
   ```sh
   df -h
   ```
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie woluminu EBS**  
```sh
aws ec2 create-volume --size 10 --region us-east-1 --availability-zone us-east-1a --volume-type gp3
```

### **2. Podłączanie woluminu do instancji**  
```sh
aws ec2 attach-volume --volume-id vol-12345678 --instance-id i-12345678 --device /dev/xvdf
```

### **3. Formatowanie i montowanie w systemie operacyjnym EC2**  
```sh
sudo mkfs -t xfs /dev/xvdf
sudo mkdir /mnt/my-ebs-volume
sudo mount /dev/xvdf /mnt/my-ebs-volume
df -h
```

### **4. Automatyczne montowanie po restarcie**  
```sh
sudo blkid /dev/xvdf
sudo nano /etc/fstab
```
Dodaj linię:  
```
UUID=<UUID_Z_KOMENDY_BLKID> /mnt/my-ebs-volume xfs defaults,nofail 0 2
```
Zapisz (`CTRL + X`, `Y`, `Enter`) i sprawdź:  
```sh
sudo mount -a
df -h
```
</details>

---

# **Amazon S3**  
**Tworzenie i konfiguracja bucketu S3**  
- Utwórz **nowy bucket** S3 o unikalnej nazwie.  
- Prześlij do niego plik `.txt`.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie bucketu S3**  
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wypełnij pola:  
   - **Bucket name**: `my-s3-bucket-<unikalna-nazwa>`  
   - **AWS Region**: Wybierz region bliski Twojej lokalizacji.  
   - **Block all public access**: **Zaznaczone** (domyślne).  
   - **Versioning**: Możesz włączyć dla przechowywania wersji plików.  
4. Kliknij **Create bucket**.  

---

### **2. Przesyłanie pliku do S3**  
1. Otwórz swój bucket.  
2. Kliknij **Upload** → **Add files**.  
3. Wybierz plik np. `myfile.txt`.  
4. Kliknij **Upload**.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketu S3**
```sh
aws s3 mb s3://my-s3-bucket-<unikalna-nazwa>
```

### **2. Przesyłanie pliku do S3**
```sh
aws s3 cp myfile.txt s3://my-s3-bucket-<unikalna-nazwa>/
```
</details>

**Hosting statycznej strony internetowej w S3**  
- Włącz opcję **Static Website Hosting** w swoim buckecie.  
- Prześlij plik `index.html` i ustaw odpowiednie uprawnienia.  
- Sprawdź, czy strona działa, wpisując **URL bucketu** w przeglądarce.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie bucketu S3**  
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wypełnij pola:  
   - **Bucket name**: `my-static-website-bucket-<unikalna-nazwa>`  
   - **AWS Region**: Wybierz najbliższy region.  
   - **Block all public access**: **Odznacz** (aby strona była publiczna).  
   - **Bucket Versioning**: Możesz włączyć dla przechowywania wersji plików.  
4. Kliknij **Create bucket**.  

---

### **2. Włączenie hostingu statycznej strony**  
1. Otwórz swój bucket → **Properties**.  
2. Przejdź do **Static website hosting** → **Edit**.  
3. Wybierz **Enable** i ustaw:  
   - **Index document**: `index.html`.  
   - **Error document**: `error.html` (opcjonalnie).  
4. Kliknij **Save changes**.  

---

### **3. Przesyłanie plików do S3**  
1. Przygotuj plik `index.html`:  
   ```html
   <html>
   <head><title>Moja Strona S3</title></head>
   <body>
       <h1>Witaj na mojej stronie hostowanej w Amazon S3!</h1>
   </body>
   </html>
   ```
2. W **S3 Console**, przejdź do bucketu → **Upload** → **Add files**.  
3. Wybierz `index.html` i kliknij **Upload**.  

---

### **4. Ustawienie uprawnień publicznych**  
1. Otwórz bucket → **Permissions** → **Bucket Policy** → **Edit**.  
2. Wklej poniższą politykę (zastąp `my-static-website-bucket-<unikalna-nazwa>` nazwą swojego bucketu):  
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::my-static-website-bucket-<unikalna-nazwa>/*"
       }
     ]
   }
   ```
3. Kliknij **Save changes**.  

---

### **5. Otwieranie strony w przeglądarce**  
1. Wróć do **Properties** → **Static website hosting**.  
2. Skopiuj **Endpoint URL** i otwórz go w przeglądarce.  
3. Jeśli widzisz stronę **"Witaj na mojej stronie hostowanej w Amazon S3!"**, konfiguracja jest poprawna! 🎉  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketu**  
```sh
aws s3 mb s3://my-static-website-bucket-<unikalna-nazwa> --region us-east-1
```

### **2. Włączenie hostingu statycznej strony**  
```sh
aws s3 website s3://my-static-website-bucket-<unikalna-nazwa>/ --index-document index.html
```

### **3. Przesyłanie plików**  
```sh
aws s3 cp index.html s3://my-static-website-bucket-<unikalna-nazwa>/ --acl public-read
```

### **4. Ustawienie uprawnień publicznych**  
```sh
aws s3api put-bucket-policy --bucket my-static-website-bucket-<unikalna-nazwa> --policy '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-website-bucket-<unikalna-nazwa>/*"
    }
  ]
}'
```

### **5. Pobranie adresu strony**  
```sh
aws s3api get-bucket-website --bucket my-static-website-bucket-<unikalna-nazwa>
```
📌 Otrzymasz URL strony w odpowiedzi, np. `http://my-static-website-bucket-<unikalna-nazwa>.s3-website-us-east-1.amazonaws.com/`.  
</details>

**Tworzenie wersji plików w S3 (Versioning)**  
- Włącz **Versioning** dla bucketu S3.  
- Prześlij kilka wersji tego samego pliku.  
- Przywróć starszą wersję pliku.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie bucketu S3 (jeśli nie istnieje)**  
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wypełnij pola:  
   - **Bucket name**: `my-versioned-bucket-<unikalna-nazwa>`  
   - **AWS Region**: Wybierz najbliższy region.  
   - **Block all public access**: **Zaznaczone** (zalecane).  
4. Kliknij **Create bucket**.  

---

### **2. Włączenie wersjonowania**  
1. Przejdź do **S3 Dashboard** → Otwórz swój bucket.  
2. Przejdź do zakładki **Properties**.  
3. Znajdź sekcję **Bucket Versioning** → Kliknij **Edit**.  
4. Wybierz **Enable** → Kliknij **Save changes**.  

📌 **Teraz każda zmiana plików w tym buckecie będzie automatycznie zapisywana jako nowa wersja.**  

---

### **3. Przesyłanie i edycja wersji plików**  
1. Przejdź do **Objects** w buckecie.  
2. Kliknij **Upload** → **Add files** i wybierz `document.txt`.  
3. Kliknij **Upload**.  
4. **Zmień plik**:  
   - Edytuj `document.txt` na swoim komputerze.  
   - Ponownie prześlij ten plik do S3, używając tej samej nazwy (`document.txt`).  
   - AWS automatycznie zapisze nową wersję.  

---

### **4. Wyświetlanie i przywracanie wcześniejszych wersji plików**  
1. W buckecie kliknij **Objects** → Znajdź `document.txt`.  
2. Kliknij **Versions** (obok nazwy pliku).  
3. Znajdziesz listę wersji pliku. Kliknij na starszą wersję, aby ją pobrać.  
4. Aby przywrócić starszą wersję:  
   - Wybierz ją i kliknij **Download**.  
   - Ponownie prześlij ją jako najnowszą wersję, aby nadpisać bieżący plik.  

---

### **5. Usuwanie wersji plików**  
- Jeśli usuniesz `document.txt`, plik pozostanie, ale jako **marker usunięcia**.  
- Aby **całkowicie usunąć plik**, przejdź do **Versions**, wybierz wszystkie wersje i usuń je ręcznie.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketu S3 z wersjonowaniem**  
```sh
aws s3 mb s3://my-versioned-bucket-<unikalna-nazwa>
aws s3api put-bucket-versioning --bucket my-versioned-bucket-<unikalna-nazwa> --versioning-configuration Status=Enabled
```

### **2. Przesyłanie pliku do S3**
```sh
echo "To jest pierwsza wersja" > document.txt
aws s3 cp document.txt s3://my-versioned-bucket-<unikalna-nazwa>/
```

### **3. Nadpisanie pliku i utworzenie nowej wersji**
```sh
echo "To jest druga wersja" > document.txt
aws s3 cp document.txt s3://my-versioned-bucket-<unikalna-nazwa>/
```

### **4. Wyświetlanie wersji pliku**
```sh
aws s3api list-object-versions --bucket my-versioned-bucket-<unikalna-nazwa>
```

Przykładowy wynik:
```json
{
    "Versions": [
        {
            "Key": "document.txt",
            "VersionId": "3z4fE...",
            "IsLatest": true,
            "LastModified": "2024-02-12T12:00:00.000Z"
        },
        {
            "Key": "document.txt",
            "VersionId": "8a2fG...",
            "IsLatest": false,
            "LastModified": "2024-02-12T11:50:00.000Z"
        }
    ]
}
```

### **5. Pobieranie konkretnej wersji pliku**
```sh
aws s3api get-object --bucket my-versioned-bucket-<unikalna-nazwa> --key document.txt --version-id 8a2fG... old-document.txt
```

### **6. Usuwanie wersji plików**  
1. Oznaczenie pliku jako usuniętego (dodanie marker delete):  
   ```sh
   aws s3 rm s3://my-versioned-bucket-<unikalna-nazwa>/document.txt
   ```
2. Całkowite usunięcie konkretnej wersji:  
   ```sh
   aws s3api delete-object --bucket my-versioned-bucket-<unikalna-nazwa> --key document.txt --version-id 3z4fE...
   ```
</details>

**Automatyczne przenoszenie danych do tańszej klasy storage**  
- Skonfiguruj **Lifecycle Policy**, aby pliki starsze niż 30 dni były przenoszone do **S3 Glacier**. 

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie bucketu S3 (jeśli nie istnieje)**  
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wypełnij pola:  
   - **Bucket name**: `my-lifecycle-bucket-<unikalna-nazwa>`  
   - **AWS Region**: Wybierz region bliski Twojej lokalizacji.  
   - **Block all public access**: **Zaznaczone** (zalecane).  
4. Kliknij **Create bucket**.  

---

### **2. Tworzenie Lifecycle Policy**  
1. Otwórz swój **bucket S3**.  
2. Przejdź do zakładki **Management** → **Lifecycle rules**.  
3. Kliknij **Create lifecycle rule**.  
4. **Podaj nazwę reguły**, np. `MoveToGlacierAfter30Days`.  
5. W sekcji **Choose rule scope** wybierz:  
   - **Apply to all objects in the bucket** (jeśli chcesz zastosować regułę do całego bucketu)  
   - Lub określ prefiks dla konkretnego katalogu, np. `logs/`  
6. Wybierz **Lifecycle rule actions**:  
   - **Move current versions of objects to Glacier**  
   - Wybierz **Glacier Flexible Retrieval** lub **Glacier Instant Retrieval**  
   - Ustaw **30 days**  
7. Kliknij **Create rule**.  

📌 **Teraz każdy plik w buckecie będzie przenoszony do Glacier po 30 dniach.**  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketu (jeśli nie istnieje)**  
```sh
aws s3 mb s3://my-lifecycle-bucket-<unikalna-nazwa>
```

### **2. Tworzenie pliku polityki Lifecycle**
Utwórz plik `lifecycle.json`:
```json
{
    "Rules": [
        {
            "ID": "MoveToGlacierAfter30Days",
            "Filter": {
                "Prefix": ""
            },
            "Status": "Enabled",
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "GLACIER"
                }
            ]
        }
    ]
}
```

### **3. Przypisanie polityki do bucketu**
```sh
aws s3api put-bucket-lifecycle-configuration --bucket my-lifecycle-bucket-<unikalna-nazwa> --lifecycle-configuration file://lifecycle.json
```

### **4. Sprawdzenie ustawień Lifecycle Policy**
```sh
aws s3api get-bucket-lifecycle-configuration --bucket my-lifecycle-bucket-<unikalna-nazwa>
```

---

## **📌 Testowanie polityki Lifecycle**  
1. **Prześlij plik do S3**  
   ```sh
   echo "Testowy plik do archiwizacji" > testfile.txt
   aws s3 cp testfile.txt s3://my-lifecycle-bucket-<unikalna-nazwa>/
   ```
2. **Poczekaj 30 dni lub zmodyfikuj politykę na 1 dzień** (do testów).  
3. **Sprawdź klasę przechowywania pliku**  
   ```sh
   aws s3api head-object --bucket my-lifecycle-bucket-<unikalna-nazwa> --key testfile.txt
   ```
   Jeśli plik został przeniesiony, zobaczysz `StorageClass: GLACIER`.  
</details>