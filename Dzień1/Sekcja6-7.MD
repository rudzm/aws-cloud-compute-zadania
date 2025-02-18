### **Zadania praktyczne dla sekcji 6 i 7: AWS Security i IAM** 

---

#### ** IAM (Identity and Access Management)**  
**Tworzenie użytkownika IAM i nadawanie mu odpowiednich uprawnień**
- Utwórz nowego użytkownika IAM i przypisz mu odpowiednie polityki dostępu.  
- Sprawdź, jakie zasoby użytkownik może modyfikować.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie użytkownika IAM**  
1. Przejdź do **AWS Management Console** → **IAM (Identity and Access Management)**.  
2. W menu bocznym kliknij **Users** → **Add users**.  
3. Wypełnij pola:  
   - **User name**: `NewIAMUser`  
   - **AWS access type**:  
     - **Zaznacz "Access key - Programmatic access"**, jeśli użytkownik ma korzystać z AWS CLI lub API.  
     - **Zaznacz "AWS Management Console access"**, jeśli użytkownik ma logować się do konsoli AWS.  
   - Jeśli wybrałeś dostęp do konsoli, ustaw **Custom password** i zdecyduj, czy użytkownik ma wymuszoną zmianę hasła przy pierwszym logowaniu.  
4. Kliknij **Next: Permissions**.  

---

### **2. Nadawanie uprawnień użytkownikowi IAM**  
1. Wybierz jedną z opcji:  
   - **Attach existing policies directly** – Przypisz istniejącą politykę, np. `AdministratorAccess` (pełny dostęp) lub `AmazonS3ReadOnlyAccess` (tylko odczyt dla S3).  
   - **Add user to group** – Dodaj użytkownika do wcześniej utworzonej grupy IAM z określonymi uprawnieniami.  
   - **Create a custom policy** – Utwórz niestandardową politykę (zaawansowane).  
2. Kliknij **Next: Tags** (możesz dodać metadane, np. dział lub projekt).  
3. Kliknij **Next: Review** i sprawdź konfigurację.  
4. Kliknij **Create user**.  

 **Po utworzeniu użytkownika pobierz Access Key ID i Secret Access Key, jeśli wybrałeś dostęp programowy!**  

---

### **3. Testowanie użytkownika IAM**  
1. Wyloguj się z konsoli AWS i zaloguj się na konto nowego użytkownika IAM.  
2. Sprawdź, czy użytkownik ma dostęp tylko do przydzielonych usług.  
3. Jeśli wybrałeś dostęp programowy, spróbuj zalogować się przez **AWS CLI** (instrukcja poniżej). 
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie użytkownika IAM**
```sh
aws iam create-user --user-name NewIAMUser
```

---

### **2. Nadanie uprawnień użytkownikowi**  
Przypisanie istniejącej polityki (np. dostęp tylko do S3 Read-Only):  
```sh
aws iam attach-user-policy --user-name NewIAMUser \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```
Alternatywnie, jeśli chcesz nadać pełne uprawnienia administratora:  
```sh
aws iam attach-user-policy --user-name NewIAMUser \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

### **3. Tworzenie kluczy dostępowych (programmatic access)**  
```sh
aws iam create-access-key --user-name NewIAMUser
```
Przykładowy wynik:
```json
{
    "AccessKey": {
        "UserName": "NewIAMUser",
        "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
    }
}
```
 **Zapisz klucz `AccessKeyId` i `SecretAccessKey`, ponieważ nie będzie można ich ponownie zobaczyć!**  

---

### **4. Testowanie dostępu przez AWS CLI**  
Zaloguj się jako nowy użytkownik IAM, konfigurując AWS CLI:
```sh
aws configure
```
Podaj dane użytkownika IAM:
- **AWS Access Key ID**: `AKIAIOSFODNN7EXAMPLE`  
- **AWS Secret Access Key**: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`  
- **Default region name**: `us-east-1`  
- **Default output format**: `json`  

Sprawdź dostęp do S3 (jeśli użytkownik ma politykę `AmazonS3ReadOnlyAccess`):
```sh
aws s3 ls
```
Jeśli użytkownik nie ma dostępu do S3, pojawi się komunikat **"Access Denied"** – oznacza to, że polityka IAM nie pozwala na tę operację.
</details>

**Tworzenie ról IAM i przypisywanie ich do usług AWS**
- Utwórz nową rolę IAM pozwalającą listować EC2 i przypisz ją do funkcji Lambda.  
- Przetestuj, czy instancja lub funkcja ma dostęp do wymaganych zasobów.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie roli IAM**  
1. Przejdź do **AWS Management Console** → **IAM (Identity and Access Management)**.  
2. W menu bocznym kliknij **Roles** → **Create role**.  
3. W sekcji **Trusted entity type** wybierz **AWS service**.  
4. W sekcji **Use case** wybierz **Lambda** i kliknij **Next**.  

---

### **2. Nadanie uprawnień do listowania instancji EC2**  
1. W sekcji **Permissions policies** kliknij **Create policy**.  
2. Przejdź do zakładki **JSON** i wklej następującą politykę:  

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "ec2:DescribeInstances",
               "Resource": "*"
           }
       ]
   }
   ```
3. Kliknij **Next**, nadaj polityce nazwę, np. `ListEC2InstancesPolicy`, i kliknij **Create policy**.  
4. Wróć do ekranu tworzenia roli, odśwież listę polityk i wyszukaj `ListEC2InstancesPolicy`.  
5. Zaznacz politykę i kliknij **Next**.  

---

### **3. Konfiguracja roli IAM**  
1. Wpisz **Role name**: `LambdaListEC2Role`.  
2. Kliknij **Create role**.  

---

### **4. Tworzenie funkcji Lambda**  
1. Przejdź do **AWS Lambda** → **Create function**.  
2. Wybierz **Author from scratch**.  
3. Wypełnij dane:  
   - **Function name**: `ListEC2InstancesLambda`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: Wybierz **Use an existing role** i wybierz `LambdaListEC2Role`.  
4. Kliknij **Create function**.  

---

### **5. Wprowadzenie kodu funkcji**  
1. W sekcji **Code** wklej następujący kod:  

   ```python
   import boto3
   import json

   def lambda_handler(event, context):
       ec2 = boto3.client("ec2")
       instances = ec2.describe_instances()
       return {
           "statusCode": 200,
           "body": json.dumps(instances, indent=2, default=str)
       }
   ```
2. Kliknij **Deploy**.  

---

### **6. Testowanie funkcji Lambda**  
1. Przejdź do zakładki **Test** → **Create new test event**.  
2. W polu **Event JSON** wpisz `{}`.  
3. Kliknij **Create** → **Test**, aby uruchomić funkcję.  
4. Sprawdź wynik – powinien zawierać listę instancji EC2.  

 **Funkcja Lambda została poprawnie utworzona i ma dostęp do listowania instancji EC2!**  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie roli IAM dla AWS Lambda**  
```sh
aws iam create-role --role-name LambdaListEC2Role \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

---

### **2. Tworzenie polityki IAM do listowania EC2**  
```sh
aws iam create-policy --policy-name ListEC2InstancesPolicy \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": "ec2:DescribeInstances",
            "Resource": "*"
        }]
    }'
```

---

### **3. Przypisanie polityki do roli IAM**  
```sh
aws iam attach-role-policy --role-name LambdaListEC2Role \
    --policy-arn arn:aws:iam::aws:policy/ListEC2InstancesPolicy
```

---

### **4. Pobranie ARN utworzonej roli**
```sh
aws iam get-role --role-name LambdaListEC2Role --query 'Role.Arn' --output text
```

---

### **5. Tworzenie pliku funkcji Lambda**
```sh
echo 'import boto3
import json

def lambda_handler(event, context):
    ec2 = boto3.client("ec2")
    instances = ec2.describe_instances()
    return {
        "statusCode": 200,
        "body": json.dumps(instances, indent=2, default=str)
    }' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

---

### **6. Tworzenie funkcji Lambda**  
Zastąp `<ARN_ROLI>` pobraną wartością:  
```sh
aws lambda create-function --function-name ListEC2InstancesLambda \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **7. Testowanie funkcji Lambda**  
```sh
aws lambda invoke --function-name ListEC2InstancesLambda response.json
cat response.json
```

Jeśli konfiguracja jest poprawna, wynik powinien zawierać listę instancji EC2.
</details>

**Tworzenie i zarządzanie politykami IAM**
- Stwórz niestandardową politykę IAM ograniczającą dostęp do określonych usług.  
- Przypisz politykę do użytkownika i sprawdź jej działanie.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie niestandardowej polityki IAM**  
1. Przejdź do **AWS Management Console** → **IAM (Identity and Access Management)**.  
2. W menu bocznym kliknij **Policies** → **Create policy**.  
3. Przejdź do zakładki **JSON** i wklej następującą politykę:  

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ec2:DescribeInstances",
                   "s3:ListBucket"
               ],
               "Resource": "*"
           },
           {
               "Effect": "Deny",
               "Action": [
                   "ec2:StartInstances",
                   "ec2:StopInstances",
                   "s3:PutObject",
                   "s3:DeleteObject"
               ],
               "Resource": "*"
           }
       ]
   }
   ```
4. Kliknij **Next: Tags** (możesz dodać tagi dla organizacji zasobów).  
5. Kliknij **Next: Review**.  
6. Wpisz **Policy name**: `ReadOnlyEC2AndS3Policy`.  
7. Kliknij **Create policy**.  

---

### **2. Przypisanie polityki do użytkownika IAM**  
1. Przejdź do **Users** w IAM i wybierz użytkownika, któremu chcesz przypisać politykę.  
2. Kliknij **Add permissions** → **Attach existing policies directly**.  
3. Wyszukaj `ReadOnlyEC2AndS3Policy`, zaznacz i kliknij **Next**.  
4. Kliknij **Add permissions**.  

 **Użytkownik IAM ma teraz uprawnienia do listowania instancji EC2 i bucketów S3, ale nie może ich modyfikować!**  

---

### **3. Testowanie uprawnień użytkownika IAM**  
1. **Zaloguj się na konto IAM użytkownika** z przypisaną polityką.  
2. **Przetestuj dostęp do EC2**:  
   - Przejdź do **EC2 Dashboard** → **Instances** → sprawdź, czy widzisz listę instancji.  
   - Spróbuj uruchomić lub zatrzymać instancję – powinieneś otrzymać błąd dostępu.  
3. **Przetestuj dostęp do S3**:  
   - Przejdź do **S3 Dashboard** → powinieneś widzieć bucket list.  
   - Spróbuj przesłać plik – powinieneś otrzymać błąd dostępu.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie niestandardowej polityki IAM**  
```sh
aws iam create-policy --policy-name ReadOnlyEC2AndS3Policy \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:DescribeInstances",
                    "s3:ListBucket"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Deny",
                "Action": [
                    "ec2:StartInstances",
                    "ec2:StopInstances",
                    "s3:PutObject",
                    "s3:DeleteObject"
                ],
                "Resource": "*"
            }
        ]
    }'
```

---

### **2. Przypisanie polityki do użytkownika IAM**  
Pobierz **ARN** nowo utworzonej polityki:  
```sh
aws iam list-policies --query 'Policies[?PolicyName==`ReadOnlyEC2AndS3Policy`].Arn' --output text
```

Przypisz politykę do użytkownika IAM (`TestUser` w przykładzie):  
```sh
aws iam attach-user-policy --user-name TestUser \
    --policy-arn arn:aws:iam::<AWS_ACCOUNT_ID>:policy/ReadOnlyEC2AndS3Policy
```

---

### **3. Testowanie dostępu dla użytkownika IAM**  

**Zalogowanie użytkownika IAM przez AWS CLI:**  
```sh
aws configure
```
Podaj:
- **AWS Access Key ID**  
- **AWS Secret Access Key**  
- **Region**  
- **Format output** (np. `json`)  

**Testowanie dostępu do EC2:**  
```sh
aws ec2 describe-instances
```
Powinieneś zobaczyć listę instancji.  

**Testowanie dostępu do S3:**  
```sh
aws s3 ls
```
Powinieneś zobaczyć listę bucketów.  

**Sprawdzenie, czy użytkownik NIE może uruchomić instancji:**  
```sh
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```
Powinieneś otrzymać komunikat **"Access Denied"**.  

**Sprawdzenie, czy użytkownik NIE może przesłać pliku do S3:**  
```sh
aws s3 cp testfile.txt s3://example-bucket/
```
Powinieneś otrzymać komunikat **"Access Denied"**.
</details>

**Konfiguracja uwierzytelniania wieloskładnikowego (MFA) dla użytkownika IAM**
- Włącz MFA dla użytkownika IAM i skonfiguruj aplikację uwierzytelniającą.  
- Przetestuj logowanie użytkownika z MFA.

---

#### ** EC2 Security**  
**Tworzenie i zarządzanie Security Groups dla instancji EC2**
- Utwórz Security Group pozwalającą na dostęp SSH tylko z Twojego IP.  
- Przetestuj połączenie z instancją EC2 po wprowadzeniu zmian.  


**Tworzenie i konfiguracja Network ACLs dla subnetów w VPC**
- Utwórz ACL ograniczającą dostęp do wybranych portów.  
- Przetestuj dostępność instancji EC2 po zastosowaniu ACL.  

---

#### ** AWS Key Management Service (KMS) & Encryption**  
**Tworzenie klucza KMS i szyfrowanie danych w S3**
- Utwórz klucz KMS i skonfiguruj szyfrowanie bucketu S3.  
- Prześlij plik do S3 i zweryfikuj, czy jest zaszyfrowany.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie klucza szyfrującego KMS**  
1. Przejdź do **AWS Management Console** → **AWS Key Management Service (KMS)**.  
2. W menu bocznym kliknij **Customer managed keys** → **Create key**.  
3. W sekcji **Key type** wybierz **Symmetric** (klucz symetryczny).  
4. Kliknij **Next**.  
5. Wpisz **Key alias**: `MyKMSKey`.  
6. (Opcjonalnie) Dodaj **Opis** i **Tagi**.  
7. W sekcji **Define key administrative permissions** wybierz użytkowników IAM, którzy będą zarządzać kluczem.  
8. W sekcji **Define key usage permissions** wybierz użytkowników/usługi AWS, które będą mogły używać klucza do szyfrowania danych.  
9. Kliknij **Finish**.  
10. Skopiuj **Key ARN** – będzie potrzebny do szyfrowania S3.  

---

### **2. Tworzenie bucketu S3 z szyfrowaniem KMS**  
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wpisz **Bucket name**: `my-secure-bucket`.  
4. Wybierz **Region**, np. `us-east-1`.  
5. Przejdź do sekcji **Default encryption** i wybierz:  
   - **Enable**  
   - **AWS Key Management Service key (SSE-KMS)**  
   - **Select a KMS key** → Wybierz `MyKMSKey`.  
6. Kliknij **Create bucket**.  

---

### **3. Testowanie szyfrowania – przesłanie pliku**  
1. Przejdź do **S3** → `my-secure-bucket`.  
2. Kliknij **Upload** i wybierz plik.  
3. Po przesłaniu sprawdź właściwości pliku:  
   - Powinno być widoczne **Server-side encryption: AWS-KMS**.  

 **Bucket S3 jest teraz zabezpieczony kluczem KMS!**  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie klucza KMS**  
```sh
aws kms create-key --description "Klucz KMS do szyfrowania S3" \
    --tags TagKey=Project,TagValue=SecureS3
```
Zapisz **KeyId** zwrócony w odpowiedzi.  

---

### **2. Tworzenie aliasu dla klucza KMS**  
Zastąp `<KEY_ID>` wartością uzyskaną w poprzednim kroku:  
```sh
aws kms create-alias --alias-name alias/MyKMSKey --target-key-id <KEY_ID>
```

---

### **3. Tworzenie bucketu S3**  
```sh
aws s3api create-bucket --bucket my-secure-bucket --region us-east-1 \
    --create-bucket-configuration LocationConstraint=us-east-1
```

---

### **4. Konfiguracja szyfrowania S3 przy użyciu KMS**  
Zastąp `<KEY_ARN>` wartością uzyskaną w kroku 1:  
```sh
aws s3api put-bucket-encryption --bucket my-secure-bucket --server-side-encryption-configuration '{
    "Rules": [{
        "ApplyServerSideEncryptionByDefault": {
            "SSEAlgorithm": "aws:kms",
            "KMSMasterKeyID": "<KEY_ARN>"
        }
    }]
}'
```

---

### **5. Testowanie szyfrowania – przesłanie pliku**  
```sh
aws s3 cp testfile.txt s3://my-secure-bucket/
```

Sprawdzenie, czy plik został zaszyfrowany:  
```sh
aws s3api head-object --bucket my-secure-bucket --key testfile.txt
```
Wynik powinien zawierać:  
```json
"ServerSideEncryption": "aws:kms",
"SSEKMSKeyId": "<KEY_ARN>"
```
</details>

**Konfiguracja szyfrowania woluminów EBS przy użyciu KMS**
- Utwórz nowy wolumin EBS i zaszyfruj go kluczem KMS.  
- Podłącz wolumin do instancji EC2 i sprawdź jego działanie.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie klucza szyfrującego KMS**  
1. Przejdź do **AWS Management Console** → **AWS Key Management Service (KMS)**.  
2. W menu bocznym kliknij **Customer managed keys** → **Create key**.  
3. W sekcji **Key type** wybierz **Symmetric** (klucz symetryczny).  
4. Kliknij **Next**.  
5. Wpisz **Key alias**: `MyKMSKeyEBS`.  
6. (Opcjonalnie) Dodaj **Opis** i **Tagi**.  
7. W sekcji **Define key administrative permissions** wybierz użytkowników IAM, którzy będą zarządzać kluczem.  
8. W sekcji **Define key usage permissions** wybierz użytkowników/usługi AWS, które będą mogły używać klucza do szyfrowania danych.  
9. Kliknij **Finish**.  
10. Skopiuj **Key ARN** – będzie potrzebny w kolejnych krokach.  

---

### **2. Tworzenie zaszyfrowanego woluminu EBS**  
1. Przejdź do **AWS Management Console** → **EC2** → **Elastic Block Store** → **Volumes**.  
2. Kliknij **Create Volume**.  
3. Wypełnij pola:  
   - **Size (GB)**: `10` (lub więcej w zależności od potrzeb).  
   - **Availability Zone**: Wybierz strefę, w której działa Twoja instancja EC2.  
   - **Volume Type**: `gp3` (domyślny).  
   - **Encryption**: Wybierz **Encrypt this volume** i użyj klucza `MyKMSKeyEBS`.  
4. Kliknij **Create Volume**.  

---

### **3. Przypisanie woluminu EBS do instancji EC2**  
1. Wróć do zakładki **Volumes** w EC2.  
2. Znajdź nowo utworzony wolumin i kliknij **Actions** → **Attach Volume**.  
3. Wybierz **Instance ID** instancji EC2, do której chcesz przypisać wolumin.  
4. W polu **Device** wpisz `/dev/xvdf` (lub inne dostępne).  
5. Kliknij **Attach Volume**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie klucza KMS**  
```sh
aws kms create-key --description "Klucz KMS do szyfrowania EBS" \
    --tags TagKey=Project,TagValue=SecureEBS
```
Zapisz **KeyId** zwrócony w odpowiedzi.  

Tworzenie aliasu dla klucza:  
```sh
aws kms create-alias --alias-name alias/MyKMSKeyEBS --target-key-id <KEY_ID>
```

---

### **2. Tworzenie zaszyfrowanego woluminu EBS**  
Zastąp `<KEY_ARN>` wartością uzyskaną w poprzednim kroku:  
```sh
aws ec2 create-volume --size 10 --volume-type gp3 --availability-zone us-east-1a \
    --encrypted --kms-key-id <KEY_ARN>
```

Zwróci identyfikator woluminu, np.:  
```json
{
    "VolumeId": "vol-0123456789abcdef0"
}
```

---

### **3. Przypisanie woluminu EBS do instancji EC2**  
Zastąp `<INSTANCE_ID>` oraz `<VOLUME_ID>` odpowiednimi wartościami:  
```sh
aws ec2 attach-volume --volume-id <VOLUME_ID> --instance-id <INSTANCE_ID> --device /dev/xvdf
```

---

### **4. Sprawdzenie statusu woluminu**  
```sh
aws ec2 describe-volumes --volume-ids <VOLUME_ID>
```

Sprawdzenie podłączonych woluminów w instancji:  
```sh
aws ec2 describe-instance-attribute --instance-id <INSTANCE_ID> --attribute blockDeviceMapping
```
</details>

---

#### ** AWS Secrets Manager**  
**Przechowywanie i odczytywanie haseł oraz kluczy API w AWS Secrets Manager** 
- Utwórz tajny klucz API w AWS Secrets Manager.  
- Odczytaj wartość klucza za pomocą AWS CLI lub SDK.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie sekretu w AWS Secrets Manager**  
1. Przejdź do **AWS Management Console** → **AWS Secrets Manager**.  
2. Kliknij **Store a new secret**.  
3. Wybierz **Secret type**:  
   - **Other type of secret** (dla kluczy API, haseł, tokenów itp.).  
4. W sekcji **Secret key/value** wprowadź wartości, np.:  
   - **Key**: `api_key` → **Value**: `123456789-abcdef`  
   - **Key**: `password` → **Value**: `SuperSecretPassword!`  
5. Kliknij **Next**.  
6. Nadaj **Secret name**: `MyAPISecret`.  
7. Kliknij **Store secret**.  

---

### **2. Tworzenie roli IAM dla AWS Lambda**  
1. Przejdź do **AWS IAM** → **Roles** → **Create role**.  
2. Wybierz **Trusted entity type** → **AWS Service**.  
3. W sekcji **Use case** wybierz **Lambda** i kliknij **Next**.  
4. Kliknij **Create policy**, przejdź do zakładki **JSON** i wklej:  
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "secretsmanager:GetSecretValue",
               "Resource": "arn:aws:secretsmanager:us-east-1:<AWS_ACCOUNT_ID>:secret:MyAPISecret-*"
           }
       ]
   }
   ```
5. Kliknij **Review**, nadaj nazwę `LambdaSecretsManagerPolicy`, i kliknij **Create policy**.  
6. Wróć do tworzenia roli, odśwież listę polityk i wybierz `LambdaSecretsManagerPolicy`.  
7. Nadaj rolę nazwę **LambdaSecretsRole** i kliknij **Create role**.  

---

### **3. Tworzenie funkcji AWS Lambda**  
1. Przejdź do **AWS Lambda** → **Create function**.  
2. Wybierz **Author from scratch**.  
3. Wypełnij dane:  
   - **Function name**: `SecretsManagerLambda`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: Wybierz **Use an existing role** i wybierz `LambdaSecretsRole`.  
4. Kliknij **Create function**.  

---

### **4. Edycja kodu funkcji AWS Lambda**  
1. Przejdź do sekcji **Code** i usuń istniejący kod.  
2. Wklej następujący kod:  

   ```python
   import boto3
   import json
   import os

   def lambda_handler(event, context):
       secret_name = "MyAPISecret"
       region_name = "us-east-1"

       # Inicjalizacja klienta AWS Secrets Manager
       client = boto3.client("secretsmanager", region_name=region_name)

       try:
           # Pobranie sekretu
           response = client.get_secret_value(SecretId=secret_name)
           secret = json.loads(response["SecretString"])

           return {
               "statusCode": 200,
               "body": json.dumps({"api_key": secret["api_key"], "password": "******"})
           }
       except Exception as e:
           return {
               "statusCode": 500,
               "body": json.dumps({"error": str(e)})
           }
   ```
3. Kliknij **Deploy**.  

---

### **5. Testowanie funkcji AWS Lambda**  
1. Przejdź do zakładki **Test** → **Create new test event**.  
2. Wpisz **Event name**: `TestSecretsRetrieval`.  
3. W polu **Event JSON** wpisz `{}`.  
4. Kliknij **Create** → **Test**, aby uruchomić funkcję.  
5. Sprawdź wynik – powinien zwrócić `api_key`, ale hasło będzie zamaskowane (`******`).
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie sekretu w AWS Secrets Manager**  
```sh
aws secretsmanager create-secret --name MyAPISecret \
    --description "Klucz API i hasło do aplikacji" \
    --secret-string '{"api_key": "123456789-abcdef", "password": "SuperSecretPassword!"}'
```

---

### **2. Tworzenie roli IAM dla AWS Lambda**  
```sh
aws iam create-role --role-name LambdaSecretsRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

Przypisanie polityki do roli:  
```sh
aws iam create-policy --policy-name LambdaSecretsManagerPolicy \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "arn:aws:secretsmanager:us-east-1:<AWS_ACCOUNT_ID>:secret:MyAPISecret-*"
        }]
    }'
```

Pobierz ARN polityki:  
```sh
aws iam list-policies --query 'Policies[?PolicyName==`LambdaSecretsManagerPolicy`].Arn' --output text
```

Przypisz politykę do roli:  
```sh
aws iam attach-role-policy --role-name LambdaSecretsRole \
    --policy-arn arn:aws:iam::<AWS_ACCOUNT_ID>:policy/LambdaSecretsManagerPolicy
```

---

### **3. Tworzenie funkcji AWS Lambda**  
```sh
echo 'import boto3
import json

def lambda_handler(event, context):
    secret_name = "MyAPISecret"
    region_name = "us-east-1"

    client = boto3.client("secretsmanager", region_name=region_name)

    try:
        response = client.get_secret_value(SecretId=secret_name)
        secret = json.loads(response["SecretString"])

        return {
            "statusCode": 200,
            "body": json.dumps({"api_key": secret["api_key"], "password": "******"})
        }
    except Exception as e:
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

Tworzenie funkcji Lambda:  
```sh
aws lambda create-function --function-name SecretsManagerLambda \
    --runtime python3.9 \
    --role arn:aws:iam::<AWS_ACCOUNT_ID>:role/LambdaSecretsRole \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

Testowanie funkcji Lambda:  
```sh
aws lambda invoke --function-name SecretsManagerLambda response.json
cat response.json
```

</details>

---

#### ** AWS Config & CloudTrail**  
**Włączenie AWS Config i monitorowanie zmian w zasobach AWS**
- Skonfiguruj AWS Config do śledzenia zmian w instancjach EC2 i Security Groups.  
- Wykonaj zmianę w Security Group i sprawdź, czy została zarejestrowana.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Włączenie AWS Config**  
1. Przejdź do **AWS Management Console** → **AWS Config**.  
2. Kliknij **Get started** (jeśli AWS Config nie jest jeszcze skonfigurowany).  

---

### **2. Wybór zasobów do monitorowania**  
1. W sekcji **Resource types to record** wybierz:  
   - **Specific types** i zaznacz:  
     - `AWS::EC2::Instance`  
     - `AWS::EC2::SecurityGroup`  
2. Kliknij **Next**.  

---

### **3. Tworzenie roli IAM dla AWS Config**  
1. Wybierz **Create a new role** i nazwij ją `AWSConfigRole`.  
2. Kliknij **Next**.  

---

### **4. Konfiguracja Amazon S3 dla AWS Config**  
1. AWS Config wymaga bucketu S3 do przechowywania historii zmian.  
2. Wybierz opcję **Create a new S3 bucket** lub podaj istniejący bucket.  
3. Kliknij **Next**.  

---

### **5. Włączenie AWS Config**  
1. Kliknij **Confirm and start recording**.  
2. AWS Config zacznie teraz monitorować zmiany w wybranych zasobach.  

---

### **6. Testowanie AWS Config – Wprowadzenie zmian w EC2 i Security Group**  
1. Przejdź do **EC2 Dashboard** → **Instances**.  
2. Zmień typ instancji EC2 (np. z `t2.micro` na `t3.micro`).  
3. Przejdź do **Security Groups**, edytuj grupę i zmień reguły np. dodaj `TCP: 8080`.  
4. Poczekaj kilka minut i przejdź do **AWS Config** → **Resources**.  
5. Znajdź swoją instancję EC2 i kliknij ją, aby zobaczyć historię zmian.  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Włączenie AWS Config**  
```sh
aws configservice put-configuration-recorder --configuration-recorder '{
    "name": "default",
    "roleARN": "arn:aws:iam::<AWS_ACCOUNT_ID>:role/service-role/AWSConfigRole",
    "recordingGroup": {
        "allSupported": false,
        "resourceTypes": ["AWS::EC2::Instance", "AWS::EC2::SecurityGroup"]
    }
}'
```

---

### **2. Tworzenie i konfiguracja bucketu S3 dla AWS Config**  
```sh
aws s3api create-bucket --bucket my-config-bucket --region us-east-1
```

---

### **3. Włączenie AWS Config z użyciem utworzonego bucketu**  
```sh
aws configservice put-delivery-channel --delivery-channel '{
    "name": "default",
    "s3BucketName": "my-config-bucket"
}'
```

Uruchomienie AWS Config:  
```sh
aws configservice start-configuration-recorder --configuration-recorder-name default
```

---

### **4. Testowanie AWS Config – Wprowadzenie zmian w EC2 i Security Group**  
1. **Zmiana typu instancji EC2**  
```sh
aws ec2 modify-instance-attribute --instance-id i-0123456789abcdef0 --instance-type "{\"Value\": \"t3.micro\"}"
```

2. **Dodanie nowej reguły do Security Group**  
```sh
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 8080 --cidr 0.0.0.0/0
```

3. **Sprawdzenie, czy AWS Config zarejestrował zmiany**  
```sh
aws configservice describe-configuration-recorders
```

4. **Pobranie historii zmian w instancji EC2**  
```sh
aws configservice get-resource-config-history --resource-type AWS::EC2::Instance --resource-id i-0123456789abcdef0
```
</details>

**Konfiguracja AWS CloudTrail do monitorowania operacji w AWS**  
- Włącz CloudTrail i skonfiguruj rejestrowanie operacji w wybranym regionie.  
- Przetestuj śledzenie aktywności – np. zaloguj się i utwórz nowy zasób.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Włączenie AWS CloudTrail**  
1. Przejdź do **AWS Management Console** → **CloudTrail**.  
2. Kliknij **Create trail**.  
3. Wpisz **Trail name**: `MyCloudTrail`.  
4. W sekcji **Storage location** wybierz:  
   - **Create new S3 bucket** → Wpisz nazwę, np. `my-cloudtrail-logs`.  
   - Jeśli masz istniejący bucket, wybierz **Use existing S3 bucket**.  
5. Kliknij **Next**.  

---

### **2. Wybór logowanych zdarzeń**  
1. W sekcji **Event type** wybierz **Management events**.  
2. W sekcji **Read/write events** wybierz **Read/Write events** (aby rejestrować zarówno operacje odczytu, jak i zapisu).  
3. Kliknij **Next**.  

---

### **3. Konfiguracja IAM dla CloudTrail**  
1. W sekcji **IAM role** wybierz **Create a new role**.  
2. Nazwij rolę **CloudTrailRole**.  
3. CloudTrail automatycznie przypisze wymagane uprawnienia.  
4. Kliknij **Next** → **Create trail**.  


---

### **4. Testowanie CloudTrail – Wykonanie operacji w AWS**  
1. Przejdź do **AWS IAM** → **Users** i utwórz nowego użytkownika.  
2. Po kilku minutach przejdź do **AWS CloudTrail** → **Event history**.  
3. Znajdź zdarzenie **CreateUser** i kliknij je, aby zobaczyć szczegóły.  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketu S3 dla CloudTrail**  
```sh
aws s3api create-bucket --bucket my-cloudtrail-logs --region us-east-1
```

---

### **2. Tworzenie roli IAM dla CloudTrail**  
```sh
aws iam create-role --role-name CloudTrailRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "cloudtrail.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

Przypisanie polityki:  
```sh
aws iam attach-role-policy --role-name CloudTrailRole \
    --policy-arn arn:aws:iam::aws:policy/AWSCloudTrailFullAccess
```

---

### **3. Tworzenie AWS CloudTrail**  
```sh
aws cloudtrail create-trail --name MyCloudTrail \
    --s3-bucket-name my-cloudtrail-logs \
    --is-multi-region-trail
```

Uruchomienie CloudTrail:  
```sh
aws cloudtrail start-logging --name MyCloudTrail
```

---

### **4. Testowanie CloudTrail – Wykonanie operacji w AWS**  
1. Utwórz użytkownika IAM:  
```sh
aws iam create-user --user-name TestUser
```
2. Sprawdź logi CloudTrail:  
```sh
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser
```
Przykładowa odpowiedź:
```json
{
    "Events": [
        {
            "EventId": "12345678-90ab-cdef-1234-567890abcdef",
            "EventName": "CreateUser",
            "EventTime": "2024-02-12T10:00:00Z",
            "Username": "TestUser"
        }
    ]
}
```
</details>

**Ustawienie zasad wymuszających silne hasła dla użytkowników IAM**
- Skonfiguruj politykę haseł IAM, aby wymagała minimum 12 znaków, co najmniej jednej wielkiej litery, cyfry i znaku specjalnego.  
- Przetestuj logowanie użytkownika IAM, aby sprawdzić, czy polityka jest przestrzegana.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Przejście do ustawień polityki haseł IAM**  
1. Zaloguj się do **AWS Management Console**.  
2. Przejdź do **IAM (Identity and Access Management)**.  
3. W menu bocznym kliknij **Account settings**.  
4. W sekcji **Password policy** kliknij **Edit password policy**.  

---

### **2. Konfiguracja polityki haseł**  
1. Zaznacz następujące opcje:  
   - **Require at least one uppercase letter** (Wymagaj co najmniej jednej wielkiej litery).  
   - **Require at least one lowercase letter** (Wymagaj co najmniej jednej małej litery).  
   - **Require at least one number** (Wymagaj co najmniej jednej cyfry).  
   - **Require at least one non-alphanumeric character** (Wymagaj co najmniej jednego znaku specjalnego).  
   - **Enable password expiration** (Wymuś zmianę hasła co X dni) – opcjonalnie, np. 90 dni.  
   - **Prevent password reuse** (Uniemożliwiaj ponowne użycie ostatnich haseł) – np. ostatnich 5 haseł.  
2. Ustaw **Minimum password length**: `12`.  
3. Kliknij **Save changes**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Sprawdzenie aktualnej polityki haseł IAM**  
```sh
aws iam get-account-password-policy
```
Jeśli polityka nie jest jeszcze ustawiona, zobaczysz błąd:  
```json
An error occurred (NoSuchEntity) when calling the GetAccountPasswordPolicy operation
```

---

### **2. Ustawienie nowej polityki haseł**  
```sh
aws iam update-account-password-policy \
    --minimum-password-length 12 \
    --require-uppercase-characters \
    --require-lowercase-characters \
    --require-numbers \
    --require-symbols \
    --password-reuse-prevention 5 \
    --max-password-age 90
```
</details>