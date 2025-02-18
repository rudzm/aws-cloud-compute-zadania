## **Zadania praktyczne dla sekcji 1: AWS RDS i DynamoDB** 

## **1. Tworzenie instancji bazy danych RDS**
- Utwórz instancję **Amazon RDS** dla PostgreSQL.  
- Skonfiguruj parametry bazy i ustawienia sieciowe (VPC, Security Groups).  
- Połącz się z bazą danych z poziomu terminala lub klienta SQL.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie instancji RDS PostgreSQL**  
1. Przejdź do **AWS Management Console** → **Amazon RDS**.  
2. Kliknij **Create database**.  

---

**2. Wybór silnika bazy danych**  
1. Wybierz **Engine type**:  
   - `PostgreSQL`  
2. Wybierz **Database creation method**:  
   - `Standard create`  
3. W sekcji **Templates** wybierz:  
   - `Free tier` (jeśli dostępne)  

---

**3. Konfiguracja instancji**  
1. **DB instance identifier**: `my-postgres-db`  
2. **Master username**: `admin`  
3. **Master password**: `SuperStrongPassword123!`  
4. **DB instance class**: `db.t3.micro` (dla Free Tier)  
5. **Allocated storage**: `20 GiB`  

---

**4. Konfiguracja sieci**  
1. **VPC**: Wybierz domyślną lub utwórz nową.  
2. **Public access**: `Yes` (aby umożliwić dostęp spoza VPC).  
3. **VPC security groups**:  
   - Jeśli tworzysz nową Security Group, upewnij się, że ma regułę **Inbound** umożliwiającą połączenia na porcie **5432** z Twojego IP.  

---

**5. Tworzenie i uruchomienie instancji**  
1. Kliknij **Create database** i poczekaj na zakończenie procesu (może potrwać kilka minut).  
2. Po zakończeniu przejdź do **Databases** i skopiuj **Endpoint** instancji (np. `my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com`).  

---

**6. Łączenie się z bazą za pomocą psql**  
Na lokalnym komputerze zainstaluj klienta **psql** (jeśli jeszcze go nie masz). Następnie połącz się z bazą używając poniższego polecenia:  

```sh
psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
```

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie instancji RDS PostgreSQL**  
```sh
aws rds create-db-instance \
    --db-instance-identifier my-postgres-db \
    --db-instance-class db.t3.micro \
    --engine postgres \
    --allocated-storage 20 \
    --master-username admin \
    --master-user-password SuperStrongPassword123! \
    --backup-retention-period 7 \
    --publicly-accessible \
    --vpc-security-group-ids sg-0123456789abcdef0
```
📌 **Zastąp `sg-0123456789abcdef0` identyfikatorem swojej Security Group.**  

---

**2. Pobranie Endpointu bazy danych**  
Po kilku minutach (gdy instancja jest gotowa) wykonaj:  
```sh
aws rds describe-db-instances --db-instance-identifier my-postgres-db --query "DBInstances[0].Endpoint.Address" --output text
```
Wynik będzie wyglądał np. tak:  
```
my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com
```

---

**3. Łączenie się z bazą za pomocą psql**  
Połącz się do bazy PostgreSQL za pomocą pobranego endpointu:  
```sh
psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
```
Podaj hasło `SuperStrongPassword123!`. Jeśli wszystko działa poprawnie, powinieneś zobaczyć prompt PostgreSQL:  
```
postgres=#
```
</details>

---

## **2. Tworzenie użytkowników i nadawanie ról w RDS**  
- Utwórz nowego użytkownika w bazie RDS.  
- Nadaj mu uprawnienia do odczytu i zapisu w określonej tabeli.  
- Zweryfikuj, czy użytkownik może wykonywać operacje zgodnie z nadanymi uprawnieniami.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Połączenie z instancją RDS PostgreSQL**  
1. Przejdź do **AWS Management Console** → **Amazon RDS**.  
2. W zakładce **Databases** znajdź swoją bazę danych PostgreSQL.  
3. Skopiuj **Endpoint** (np. `my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com`).  
4. Połącz się z bazą używając klienta **psql**:  

   ```sh
   psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
   ```

---

**2. Tworzenie nowego użytkownika w PostgreSQL**  
W terminalu **psql** wykonaj następujące polecenie, aby utworzyć użytkownika:  

```sql
CREATE USER new_user WITH PASSWORD 'SecurePass123!';
```

📌 **`new_user`** – Nazwa nowego użytkownika  
📌 **`SecurePass123!`** – Silne hasło dla użytkownika  

---

**3. Tworzenie nowej bazy danych (opcjonalnie)**  
Jeśli chcesz, aby użytkownik pracował na osobnej bazie, utwórz ją:  

```sql
CREATE DATABASE company_db OWNER new_user;
```

---

**4. Nadanie uprawnień do tabeli**  
Jeśli użytkownik ma pracować na konkretnej bazie, połącz się z nią:  

```sh
\c company_db
```

Tworzenie tabeli:  
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(100)
);
```

Przyznanie użytkownikowi **pełnych uprawnień do tabeli**:  
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO new_user;
```

📌 Teraz `new_user` może **odczytywać, dodawać, edytować i usuwać dane** w tabeli `employees`.  

---

**5. Testowanie dostępu nowego użytkownika**  
1. Wyloguj się z `psql` (`\q`).  
2. Połącz się ponownie jako `new_user`:  

   ```sh
   psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U new_user -d company_db
   ```

3. Przetestuj odczyt i zapis:  

   ```sql
   INSERT INTO employees (name, department) VALUES ('John Doe', 'Engineering');
   SELECT * FROM employees;
   ```
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Połączenie z instancją RDS PostgreSQL**  
Pobierz **Endpoint** instancji RDS:  
```sh
aws rds describe-db-instances --db-instance-identifier my-postgres-db --query "DBInstances[0].Endpoint.Address" --output text
```
Następnie połącz się z bazą:  
```sh
psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
```

---

**2. Tworzenie nowego użytkownika PostgreSQL przez CLI**  
```sh
echo "CREATE USER new_user WITH PASSWORD 'SecurePass123!';" | psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
```

---

**3. Tworzenie bazy danych (opcjonalnie)**  
```sh
echo "CREATE DATABASE company_db OWNER new_user;" | psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d postgres
```

---

**4. Nadanie uprawnień do tabeli**  
Tworzenie tabeli:  
```sh
echo "CREATE TABLE employees (id SERIAL PRIMARY KEY, name VARCHAR(100), department VARCHAR(100));" | psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d company_db
```

Przyznanie uprawnień:  
```sh
echo "GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO new_user;" | psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U admin -d company_db
```

---

**5. Testowanie dostępu nowego użytkownika**  
Zaloguj się jako `new_user`:  
```sh
psql -h my-postgres-db.abc123xyz.us-east-1.rds.amazonaws.com -U new_user -d company_db
```

Wstawienie danych i ich odczyt:  
```sql
INSERT INTO employees (name, department) VALUES ('Alice Johnson', 'HR');
SELECT * FROM employees;
```
</details>

---

## **3. Tworzenie automatycznych backupów i snapshotów w RDS**  
- Skonfiguruj automatyczne backupy instancji RDS.  
- Utwórz ręczny snapshot bazy danych.  
- Przywróć bazę z backupu i sprawdź, czy dane są poprawne.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Włączenie automatycznych backupów dla instancji RDS PostgreSQL**  
1. Przejdź do **AWS Management Console** → **Amazon RDS**.  
2. Kliknij w swoją instancję PostgreSQL.  
3. Przejdź do zakładki **Maintenance & backups**.  
4. W sekcji **Automated backups** sprawdź, czy są włączone.  
   - Jeśli nie, kliknij **Modify**.  
5. W sekcji **Backup retention period** wybierz okres przechowywania backupów (np. `7 days`).  
6. W sekcji **Backup window** ustaw przedział czasowy dla automatycznych backupów.  
7. Kliknij **Continue** → **Apply immediately**.  

📌 **Teraz AWS RDS będzie automatycznie tworzyć backupy Twojej bazy danych.**  

---

**2. Tworzenie ręcznego snapshotu bazy danych**  
1. Przejdź do **AWS RDS** → **Databases**.  
2. Wybierz swoją instancję PostgreSQL.  
3. Kliknij **Actions** → **Take snapshot**.  
4. Wpisz **Snapshot name**: `mydb-snapshot-2024-02-12`.  
5. Kliknij **Create snapshot**.  
6. Poczekaj kilka minut, aż snapshot zostanie utworzony.  

📌 **Snapshot został utworzony i można go użyć do przywracania bazy danych.**  

---

**3. Przywracanie bazy danych z ręcznego snapshotu**  
1. Przejdź do **AWS RDS** → **Snapshots**.  
2. Wybierz snapshot `mydb-snapshot-2024-02-12`.  
3. Kliknij **Actions** → **Restore snapshot**.  
4. Wpisz **DB instance identifier**: `restored-mydb-instance`.  
5. Kliknij **Restore DB instance**.  
6. Po zakończeniu procesu znajdź nową instancję i skopiuj jej **Endpoint**.  

📌 **Teraz możesz połączyć się z przywróconą bazą za pomocą `psql`.**  

---

**4. Testowanie połączenia do przywróconej bazy PostgreSQL**  
Połącz się do przywróconej instancji:  
```sh
psql -h restored-mydb-instance.xxxxxx.rds.amazonaws.com -U admin -d postgres
```

📌 **Baza danych została przywrócona i jest gotowa do użytku!**  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Konfiguracja automatycznych backupów RDS PostgreSQL**  
```sh
aws rds modify-db-instance \
    --db-instance-identifier my-postgres-db \
    --backup-retention-period 7 \
    --apply-immediately
```
📌 **To polecenie ustawia okres przechowywania backupów na 7 dni.**  

---

**2. Tworzenie ręcznego snapshotu instancji RDS**  
```sh
aws rds create-db-snapshot \
    --db-instance-identifier my-postgres-db \
    --db-snapshot-identifier mydb-snapshot-2024-02-12
```

Sprawdzenie statusu snapshotu:  
```sh
aws rds describe-db-snapshots --db-snapshot-identifier mydb-snapshot-2024-02-12
```

---

**3. Przywracanie bazy danych z ręcznego snapshotu**  
```sh
aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier restored-mydb-instance \
    --db-snapshot-identifier mydb-snapshot-2024-02-12
```

Sprawdzenie statusu przywracanej instancji:  
```sh
aws rds describe-db-instances --db-instance-identifier restored-mydb-instance
```

---

**4. Testowanie połączenia do przywróconej bazy PostgreSQL**  
Po odzyskaniu bazy uzyskaj jej **Endpoint**:  
```sh
aws rds describe-db-instances --db-instance-identifier restored-mydb-instance \
    --query "DBInstances[0].Endpoint.Address" --output text
```
Połącz się z nową bazą danych:  
```sh
psql -h restored-mydb-instance.xxxxxx.rds.amazonaws.com -U admin -d postgres
```

</details>

---

## **4. Tworzenie tabeli w DynamoDB i wstawianie danych**  
- Utwórz tabelę w **Amazon DynamoDB** z kluczem głównym.  
- Dodaj kilka rekordów do tabeli.  
- Pobierz dane przy użyciu operacji `GetItem` i `Scan`.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie tabeli w Amazon DynamoDB**  
1. Przejdź do **AWS Management Console** → **DynamoDB**.  
2. Kliknij **Create table**.  
3. Wypełnij podstawowe informacje:  
   - **Table name**: `Employees`  
   - **Partition key**: `EmployeeID` (Type: `Number`)  
   - (Opcjonalnie) **Sort key**: `Department` (Type: `String`)  
4. W sekcji **Settings** pozostaw domyślne wartości.  
5. Kliknij **Create table**.  
6. Poczekaj, aż tabela przejdzie do stanu **Active**.  

📌 **Tabela `Employees` jest gotowa do użycia!**  

---

**2. Dodawanie danych do tabeli DynamoDB**  
1. Przejdź do zakładki **Items**.  
2. Kliknij **Create item**.  
3. Wprowadź wartości:  
   ```json
   {
       "EmployeeID": 1001,
       "Name": "John Doe",
       "Department": "Engineering"
   }
   ```
4. Kliknij **Save**.  
5. Powtórz ten krok dla kilku różnych rekordów.  

📌 **Dane zostały dodane do tabeli!**  

---

**3. Pobieranie danych z tabeli**  
1. W zakładce **Items** użyj opcji **Scan**, aby zobaczyć wszystkie rekordy.  
2. Wyszukaj pracownika z `EmployeeID = 1001` przy użyciu opcji **Query**.  

</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie tabeli w DynamoDB**  
```sh
aws dynamodb create-table \
    --table-name Employees \
    --attribute-definitions AttributeName=EmployeeID,AttributeType=N \
    --key-schema AttributeName=EmployeeID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

Sprawdzenie statusu tabeli:  
```sh
aws dynamodb describe-table --table-name Employees --query "Table.TableStatus"
```
📌 Czekaj na status `ACTIVE`.  

---

**2. Dodawanie danych do tabeli DynamoDB**  
```sh
aws dynamodb put-item \
    --table-name Employees \
    --item '{
        "EmployeeID": {"N": "1001"},
        "Name": {"S": "John Doe"},
        "Department": {"S": "Engineering"}
    }'
```

Dodanie kolejnego rekordu:  
```sh
aws dynamodb put-item \
    --table-name Employees \
    --item '{
        "EmployeeID": {"N": "1002"},
        "Name": {"S": "Alice Johnson"},
        "Department": {"S": "HR"}
    }'
```

---

**3. Pobieranie danych z tabeli**  
📌 Pobranie wszystkich rekordów (`Scan`):  
```sh
aws dynamodb scan --table-name Employees
```

📌 Pobranie konkretnego pracownika (`Query`):  
```sh
aws dynamodb get-item --table-name Employees \
    --key '{"EmployeeID": {"N": "1001"}}'
```
</details>

---

## **5. Indeksowanie i optymalizacja zapytań w DynamoDB**  
- Utwórz **Global Secondary Index (GSI)** dla tabeli.  
- Wykonaj zapytania wykorzystujące indeks i porównaj wyniki z zapytaniami podstawowymi.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie tabeli DynamoDB (jeśli jeszcze jej nie ma)**  
1. Przejdź do **AWS Management Console** → **DynamoDB**.  
2. Kliknij **Create table**.  
3. Wpisz nazwę tabeli: `Employees`.  
4. **Partition key**: `EmployeeID` (Type: `Number`).  
5. (Opcjonalnie) **Sort key**: `Department` (Type: `String`).  
6. Kliknij **Create table** i poczekaj na status **Active**.  

---

**2. Dodanie Global Secondary Index (GSI)**  
1. Przejdź do **AWS DynamoDB** → **Tables** → **Employees**.  
2. Kliknij **Indexes** → **Create index**.  
3. Wprowadź wartości:  
   - **Partition key**: `Department` (Type: `String`).  
   - **Sort key (opcjonalnie)**: `EmployeeID` (Type: `Number`).  
   - **Projection type**: `All` (aby indeks zawierał wszystkie atrybuty).  
4. Kliknij **Create index**.  
5. Poczekaj, aż indeks przejdzie do stanu **Active**.  

📌 **Indeks został dodany i można go używać do szybszego wyszukiwania pracowników według działu!**  

---

**3. Testowanie zapytań z wykorzystaniem GSI**  
1. Przejdź do zakładki **Items**.  
2. Kliknij **Query**.  
3. Wybierz **Index name**: `Department-index`.  
4. Wprowadź `Department = "Engineering"`.  
5. Kliknij **Run query**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie tabeli DynamoDB (jeśli jeszcze jej nie ma)**  
```sh
aws dynamodb create-table \
    --table-name Employees \
    --attribute-definitions AttributeName=EmployeeID,AttributeType=N AttributeName=Department,AttributeType=S \
    --key-schema AttributeName=EmployeeID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

---

**2. Dodanie Global Secondary Index (GSI) do istniejącej tabeli**  
```sh
aws dynamodb update-table \
    --table-name Employees \
    --attribute-definitions AttributeName=Department,AttributeType=S \
    --global-secondary-index-updates '[{
        "Create": {
            "IndexName": "Department-index",
            "KeySchema": [
                {"AttributeName": "Department", "KeyType": "HASH"},
                {"AttributeName": "EmployeeID", "KeyType": "RANGE"}
            ],
            "Projection": {"ProjectionType": "ALL"}
        }
    }]'
```

Sprawdzenie statusu indeksu:  
```sh
aws dynamodb describe-table --table-name Employees --query "Table.GlobalSecondaryIndexes"
```

📌 **Poczekaj na status `ACTIVE` przed wykonaniem zapytań.**  

---

**3. Testowanie zapytań z GSI**  
📌 Pobranie wszystkich pracowników z działu "Engineering"  
```sh
aws dynamodb query \
    --table-name Employees \
    --index-name Department-index \
    --key-condition-expression "Department = :dept" \
    --expression-attribute-values '{":dept": {"S": "Engineering"}}'
```
</details>

---

## **6. Tworzenie reguł TTL dla automatycznego usuwania starych danych**  
- Skonfiguruj pole **TTL** w tabeli DynamoDB dla automatycznego usuwania rekordów po określonym czasie.  
- Zweryfikuj, czy stare rekordy są usuwane zgodnie z konfiguracją.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie tabeli DynamoDB (jeśli jeszcze jej nie ma)**  
1. Przejdź do **AWS Management Console** → **DynamoDB**.  
2. Kliknij **Create table**.  
3. Wpisz **Table name**: `Sessions`.  
4. **Partition key**: `SessionID` (Type: `String`).  
5. Kliknij **Create table** i poczekaj na status **Active**.  

---

**2. Włączenie TTL w tabeli DynamoDB**  
1. Przejdź do zakładki **Tables** → Wybierz `Sessions`.  
2. Przejdź do zakładki **Additional settings**.  
3. Kliknij **Edit** w sekcji **Time to Live (TTL) attribute**.  
4. Wpisz nazwę pola TTL: `ExpirationTime`.  
5. Kliknij **Enable TTL** i poczekaj, aż status zmieni się na **Enabled**.  

📌 **TTL jest teraz włączone i DynamoDB automatycznie usunie rekordy, gdy ich czas wygaśnie.**  

---

**3. Dodawanie danych z polem TTL**  
1. Przejdź do zakładki **Items**.  
2. Kliknij **Create item**.  
3. Dodaj wartości:  
   ```json
   {
       "SessionID": "abc123",
       "User": "JohnDoe",
       "ExpirationTime": 1707772800
   }
   ```
   📌 **`ExpirationTime`** to czas w **sekundach UNIX (Epoch Time)**, po którym rekord zostanie automatycznie usunięty.  
4. Kliknij **Save**.  

---

**4. Weryfikacja usunięcia rekordu przez TTL**  
1. Wykonaj **Scan** w tabeli i sprawdź, czy rekord nadal istnieje.  
2. Po upływie `ExpirationTime` wpis zostanie automatycznie usunięty.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie tabeli DynamoDB (jeśli jeszcze jej nie ma)**  
```sh
aws dynamodb create-table \
    --table-name Sessions \
    --attribute-definitions AttributeName=SessionID,AttributeType=S \
    --key-schema AttributeName=SessionID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

---

**2. Włączenie TTL na tabeli**  
```sh
aws dynamodb update-time-to-live \
    --table-name Sessions \
    --time-to-live-specification Enabled=true,AttributeName=ExpirationTime
```

Sprawdzenie statusu TTL:  
```sh
aws dynamodb describe-time-to-live --table-name Sessions
```
📌 Powinieneś zobaczyć:  
```json
{
    "TimeToLiveDescription": {
        "TimeToLiveStatus": "ENABLED",
        "AttributeName": "ExpirationTime"
    }
}
```

---

**3. Dodawanie rekordu z polem TTL**  
Poniższy rekord zostanie automatycznie usunięty po upływie 1 godziny:  
```sh
EXPIRATION_TIME=$(($(date +%s) + 3600)) 
aws dynamodb put-item \
    --table-name Sessions \
    --item '{
        "SessionID": {"S": "abc123"},
        "User": {"S": "JohnDoe"},
        "ExpirationTime": {"N": "'"$EXPIRATION_TIME"'"}
    }'
```
📌 Wartość `ExpirationTime` to aktualny czas UNIX + 1 godzina.  

---

**4. Weryfikacja usunięcia rekordu**  
Po kilku minutach sprawdź, czy rekord nadal istnieje:  
```sh
aws dynamodb scan --table-name Sessions
```
</details>

---

## **7. Przetwarzanie plików z S3 i zapisywanie wyników w DynamoDB**  
- Utwórz bucket S3 i przesyłaj pliki JSON.  
- Skonfiguruj **AWS Lambda**, aby automatycznie przetwarzała pliki i zapisywała dane w **DynamoDB**.  
- Zweryfikuj poprawność zapisanych danych w tabeli.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

**1. Tworzenie bucketu S3**  
1. Przejdź do **AWS Management Console** → **Amazon S3**.  
2. Kliknij **Create bucket**.  
3. Wpisz **Bucket name**: `my-json-bucket`.  
4. Wybierz region i kliknij **Create bucket**.  

---

**2. Tworzenie tabeli w DynamoDB**  
1. Przejdź do **AWS DynamoDB**.  
2. Kliknij **Create table**.  
3. Wpisz **Table name**: `ProcessedData`.  
4. **Partition key**: `RecordID` (Type: `String`).  
5. Kliknij **Create table**.  

---

**3. Tworzenie funkcji AWS Lambda**  
1. Przejdź do **AWS Lambda** → **Create function**.  
2. Wybierz **Author from scratch**.  
3. **Function name**: `S3ToDynamoDBLambda`.  
4. **Runtime**: `Python 3.9`.  
5. **Execution role**:  
   - Wybierz **Create a new role with basic permissions**.  
6. Kliknij **Create function**.  

---

**4. Modyfikacja roli IAM dla Lambda**  
1. Przejdź do **AWS IAM** → **Roles**.  
2. Znajdź utworzoną rolę dla Lambda (`S3ToDynamoDBLambda-role`).  
3. Kliknij **Attach policies** i dodaj:  
   - `AmazonS3ReadOnlyAccess` (do odczytu plików z S3).  
   - `AmazonDynamoDBFullAccess` (do zapisu danych w DynamoDB).  

---

**5. Edycja kodu AWS Lambda**  
1. Wróć do zakładki **Code** w funkcji Lambda.  
2. Wklej następujący kod:  

   ```python
   import json
   import boto3

   s3 = boto3.client('s3')
   dynamodb = boto3.resource('dynamodb')
   table = dynamodb.Table('ProcessedData')

   def lambda_handler(event, context):
       for record in event['Records']:
           bucket_name = record['s3']['bucket']['name']
           file_key = record['s3']['object']['key']

           response = s3.get_object(Bucket=bucket_name, Key=file_key)
           file_content = response['Body'].read().decode('utf-8')
           data = json.loads(file_content)

           for item in data:
               table.put_item(Item=item)

       return {"statusCode": 200, "body": "Data processed successfully"}
   ```

3. Kliknij **Deploy**.  

---

**6. Konfiguracja wyzwalacza S3 dla Lambda**  
1. Przejdź do zakładki **Configuration** → **Triggers**.  
2. Kliknij **Add trigger**.  
3. Wybierz **S3** jako źródło wyzwalania.  
4. Wybierz bucket `my-json-bucket`.  
5. **Event type**: `PUT` (aby Lambda uruchamiała się po przesłaniu pliku).  
6. Kliknij **Add**.  

---

**7. Testowanie**  
1. Prześlij plik JSON do bucketu S3:  

   ```json
   [
       {"RecordID": "1", "Name": "Alice", "Age": 30},
       {"RecordID": "2", "Name": "Bob", "Age": 25}
   ]
   ```

2. Po kilku sekundach sprawdź tabelę **DynamoDB** → **Items**.  
</details>

<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

**1. Tworzenie bucketu S3**  
```sh
aws s3api create-bucket --bucket my-json-bucket --region us-east-1
```

---

**2. Tworzenie tabeli w DynamoDB**  
```sh
aws dynamodb create-table \
    --table-name ProcessedData \
    --attribute-definitions AttributeName=RecordID,AttributeType=S \
    --key-schema AttributeName=RecordID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

Sprawdzenie statusu tabeli:  
```sh
aws dynamodb describe-table --table-name ProcessedData
```

---

**3. Tworzenie roli IAM dla Lambda**  
```sh
aws iam create-role --role-name LambdaS3DynamoDBRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

Dodanie uprawnień:  
```sh
aws iam attach-role-policy --role-name LambdaS3DynamoDBRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam attach-role-policy --role-name LambdaS3DynamoDBRole --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
```

Pobranie ARN roli:  
```sh
aws iam get-role --role-name LambdaS3DynamoDBRole --query 'Role.Arn' --output text
```

---

**4. Tworzenie funkcji Lambda**  
Tworzymy plik **lambda_function.py**:  
```sh
echo 'import json
import boto3

s3 = boto3.client("s3")
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("ProcessedData")

def lambda_handler(event, context):
    for record in event["Records"]:
        bucket_name = record["s3"]["bucket"]["name"]
        file_key = record["s3"]["object"]["key"]

        response = s3.get_object(Bucket=bucket_name, Key=file_key)
        file_content = response["Body"].read().decode("utf-8")
        data = json.loads(file_content)

        for item in data:
            table.put_item(Item=item)

    return {"statusCode": 200, "body": "Data processed successfully"}' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

Tworzenie funkcji Lambda:  
```sh
aws lambda create-function --function-name S3ToDynamoDBLambda \
    --runtime python3.9 \
    --role arn:aws:iam::<AWS_ACCOUNT_ID>:role/LambdaS3DynamoDBRole \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

**5. Konfiguracja wyzwalacza S3**  
```sh
aws lambda add-permission \
    --function-name S3ToDynamoDBLambda \
    --statement-id S3TriggerPermission \
    --action "lambda:InvokeFunction" \
    --principal s3.amazonaws.com \
    --source-arn arn:aws:s3:::my-json-bucket \
    --source-account <AWS_ACCOUNT_ID>
```

```sh
aws s3api put-bucket-notification-configuration \
    --bucket my-json-bucket \
    --notification-configuration '{
        "LambdaFunctionConfigurations": [{
            "LambdaFunctionArn": "arn:aws:lambda:us-east-1:<AWS_ACCOUNT_ID>:function:S3ToDynamoDBLambda",
            "Events": ["s3:ObjectCreated:Put"]
        }]
    }'
```

---

**6. Testowanie**  
Prześlij plik JSON do S3:  
```sh
echo '[{"RecordID": "1", "Name": "Alice", "Age": 30}, {"RecordID": "2", "Name": "Bob", "Age": 25}]' > data.json
aws s3 cp data.json s3://my-json-bucket/
```

Sprawdzenie zawartości tabeli:  
```sh
aws dynamodb scan --table-name ProcessedData
```
</details>