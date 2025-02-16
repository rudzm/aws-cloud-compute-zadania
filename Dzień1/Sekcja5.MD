### **Zadania praktyczne dla sekcji 5: AWS Lambda**  

---

**Tworzenie i uruchomienie prostej funkcji Lambda**  
Stwórz funkcję AWS Lambda w dowolnym języku obsługiwanym przez AWS Lambda. Funkcja powinna zwracać prostą odpowiedź w formacie JSON. Wykonaj test lokalny w konsoli AWS i sprawdź wynik.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie funkcji Lambda**  
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch**.  
4. Wypełnij pola:  
   - **Function name**: `MyFirstLambda`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: Wybierz **Create a new role with basic Lambda permissions**  
5. Kliknij **Create function**.  

---

### **2. Edycja kodu funkcji Lambda**  
1. Przewiń do sekcji **Code**.  
2. Wklej poniższy kod w edytorze:  

   ```python
   import json

   def lambda_handler(event, context):
       return {
           "statusCode": 200,
           "body": json.dumps("Hello from AWS Lambda!")
       }
   ```
3. Kliknij **Deploy**.  

---

### **3. Testowanie funkcji Lambda**  
1. Kliknij **Test** → **Create a new test event**.  
2. Wpisz **Event name**: `TestEvent`.  
3. W polu **Event JSON** wklej `{}` (pusty event).  
4. Kliknij **Create** → **Test**.  
5. Sprawdź, czy odpowiedź zawiera:  
   ```json
   {
       "statusCode": 200,
       "body": "\"Hello from AWS Lambda!\""
   }
   ```
6. Przejdź do zakładki **Monitor** → **View logs in CloudWatch**, aby sprawdzić logi funkcji.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie roli IAM dla AWS Lambda**  
Lambda wymaga roli IAM do działania. Tworzymy nową rolę z podstawowymi uprawnieniami:  

```sh
aws iam create-role --role-name LambdaBasicExecutionRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

Następnie przypisujemy politykę pozwalającą na zapis logów do CloudWatch:  

```sh
aws iam attach-role-policy --role-name LambdaBasicExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

### **2. Tworzenie pliku z kodem funkcji Lambda**  
Utwórz plik `lambda_function.py` i dodaj kod funkcji:  

```sh
echo 'import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps("Hello from AWS Lambda!")
    }' > lambda_function.py
```

---

### **3. Pakowanie kodu do pliku ZIP**  
```sh
zip function.zip lambda_function.py
```

---

### **4. Pobranie ARN roli IAM**  
```sh
aws iam get-role --role-name LambdaBasicExecutionRole --query 'Role.Arn' --output text
```

Skopiuj **ARN** roli i użyj go w następnym kroku.

---

### **5. Tworzenie funkcji Lambda**  
Zastąp `<ARN_ROLI>` wartością uzyskaną w poprzednim kroku:  

```sh
aws lambda create-function --function-name MyFirstLambda \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **6. Wywołanie funkcji Lambda**  
```sh
aws lambda invoke --function-name MyFirstLambda --payload '{}' response.json
```

Sprawdź wynik:  
```sh
cat response.json
```

Oczekiwany wynik:  
```json
{
    "statusCode": 200,
    "body": "\"Hello from AWS Lambda!\""
}
```

---

### **7. Sprawdzenie logów w CloudWatch**  
```sh
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/MyFirstLambda"
aws logs tail "/aws/lambda/MyFirstLambda" --follow
```
</details>

**Tworzenie funkcji Lambda wywoływanej przez Amazon S3**   
Skonfiguruj funkcję Lambda tak, aby była automatycznie wywoływana w momencie przesłania nowego pliku do określonego bucketa Amazon S3. Funkcja powinna odczytać metadane pliku i zapisać je w logach CloudWatch.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie bucketa S3**
1. Przejdź do **AWS Management Console** → **S3**.  
2. Kliknij **Create bucket**.  
3. Wpisz unikalną nazwę bucketa, np. `lambda-s3-trigger-demo-<unikalny-id>`.  
4. Wybierz region, np. `us-east-1`.  
5. **Wyłącz opcję blokowania dostępu publicznego** (nie będzie potrzebne dla tego zadania).  
6. Kliknij **Create bucket**.  

---

### **2. Tworzenie funkcji Lambda**
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch**.  
4. Wypełnij dane:  
   - **Function name**: `S3TriggerLambda`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: **Create a new role with basic Lambda permissions**  
5. Kliknij **Create function**.  

---

### **3. Edycja kodu funkcji Lambda**
1. W sekcji **Code** wklej następujący kod:  

   ```python
   import json
   import boto3

   def lambda_handler(event, context):
       s3 = boto3.client("s3")
       for record in event["Records"]:
           bucket_name = record["s3"]["bucket"]["name"]
           file_name = record["s3"]["object"]["key"]
           print(f"Nowy plik przesłany: {file_name} do bucketu: {bucket_name}")
       return {
           "statusCode": 200,
           "body": json.dumps("S3 event processed successfully!")
       }
   ```
2. Kliknij **Deploy**.  

---

### **4. Dodanie wyzwalacza S3 do funkcji Lambda**
1. Przejdź do zakładki **Configuration** → **Triggers**.  
2. Kliknij **Add Trigger**.  
3. Wybierz **S3** jako źródło wyzwalania.  
4. Wybierz bucket utworzony w kroku 1.  
5. W sekcji **Event type** wybierz **PUT** (nowe przesłane pliki).  
6. Kliknij **Add**.  

📌 **Teraz każda operacja przesłania pliku do S3 uruchomi funkcję Lambda!**  

---

### **5. Testowanie funkcji**
1. Przejdź do **S3** → Otwórz swój bucket.  
2. Kliknij **Upload**, wybierz dowolny plik i prześlij go.  
3. Przejdź do **AWS Lambda** → **Monitor** → **View logs in CloudWatch**.  
4. Sprawdź, czy w logach pojawiła się informacja o nowym pliku.
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie bucketa S3**
```sh
aws s3 mb s3://lambda-s3-trigger-demo-<unikalny-id>
```

---

### **2. Tworzenie roli IAM dla AWS Lambda**
```sh
aws iam create-role --role-name LambdaS3ExecutionRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```
Przypisanie polityki pozwalającej Lambda na odczyt z S3 i zapis logów w CloudWatch:
```sh
aws iam attach-role-policy --role-name LambdaS3ExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

aws iam attach-role-policy --role-name LambdaS3ExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

### **3. Tworzenie pliku funkcji Lambda**
```sh
echo 'import json
import boto3

def lambda_handler(event, context):
    s3 = boto3.client("s3")
    for record in event["Records"]:
        bucket_name = record["s3"]["bucket"]["name"]
        file_name = record["s3"]["object"]["key"]
        print(f"Nowy plik przesłany: {file_name} do bucketu: {bucket_name}")
    return {
        "statusCode": 200,
        "body": json.dumps("S3 event processed successfully!")
    }' > lambda_function.py
```

Spakowanie pliku ZIP:
```sh
zip function.zip lambda_function.py
```

---

### **4. Pobranie ARN roli IAM**
```sh
aws iam get-role --role-name LambdaS3ExecutionRole --query 'Role.Arn' --output text
```

---

### **5. Tworzenie funkcji Lambda**
Zastąp `<ARN_ROLI>` uzyskaną wartością:
```sh
aws lambda create-function --function-name S3TriggerLambda \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **6. Konfiguracja wyzwalacza S3 dla funkcji Lambda**
Zastąp `<BUCKET_NAME>` nazwą swojego bucketa:
```sh
aws s3api put-bucket-notification-configuration --bucket <BUCKET_NAME> --notification-configuration '{
    "LambdaFunctionConfigurations": [{
        "LambdaFunctionArn": "'$(aws lambda get-function --function-name S3TriggerLambda --query "Configuration.FunctionArn" --output text)'",
        "Events": ["s3:ObjectCreated:*"]
    }]
}'
```

---

### **7. Testowanie funkcji Lambda**  
Prześlij testowy plik do S3:
```sh
aws s3 cp testfile.txt s3://lambda-s3-trigger-demo-<unikalny-id>/
```
Sprawdź logi w CloudWatch:
```sh
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/S3TriggerLambda"
aws logs tail "/aws/lambda/S3TriggerLambda" --follow
```
</details>

---


**Analiza logów funkcji Lambda w Amazon CloudWatch**  
Uruchom funkcję AWS Lambda kilka razy i przeanalizuj jej logi w Amazon CloudWatch. Zidentyfikuj czas wykonania oraz ewentualne błędy.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie funkcji Lambda**  
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch**.  
4. Wypełnij dane:  
   - **Function name**: `LambdaLoggingDemo`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: **Create a new role with basic Lambda permissions**  
5. Kliknij **Create function**.  

---

### **2. Edycja kodu funkcji Lambda**  
1. Przejdź do sekcji **Code** i wklej następujący kod:  

   ```python
   import json
   import logging

   # Konfiguracja loggera
   logger = logging.getLogger()
   logger.setLevel(logging.INFO)

   def lambda_handler(event, context):
       logger.info("Funkcja Lambda została uruchomiona.")
       logger.warning("To jest ostrzeżenie!")
       logger.error("To jest komunikat o błędzie!")

       return {
           "statusCode": 200,
           "body": json.dumps("Logowanie w CloudWatch działa poprawnie!")
       }
   ```
2. Kliknij **Deploy**.  

---

### **3. Uruchomienie funkcji i generowanie logów**  
1. Kliknij **Test** → **Create a new test event**.  
2. Wpisz **Event name**: `TestEvent`.  
3. W polu **Event JSON** wklej `{}` (pusty event).  
4. Kliknij **Create** → **Test**, aby uruchomić funkcję kilka razy.  

---

### **4. Analiza logów w Amazon CloudWatch**  
1. Przejdź do zakładki **Monitor** → **View logs in CloudWatch**.  
2. Znajdź grupę logów `/aws/lambda/LambdaLoggingDemo` i kliknij w nią.  
3. Przejrzyj logi, sprawdzając, czy pojawiły się wpisy INFO, WARNING i ERROR.  
4. Zidentyfikuj timestampy oraz ewentualne błędy.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie roli IAM dla AWS Lambda**  
Lambda wymaga roli IAM do działania. Tworzymy nową rolę z podstawowymi uprawnieniami:  

```sh
aws iam create-role --role-name LambdaLoggingExecutionRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```

Przypisujemy politykę pozwalającą na zapis logów do CloudWatch:  

```sh
aws iam attach-role-policy --role-name LambdaLoggingExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

### **2. Tworzenie pliku funkcji Lambda**  
```sh
echo 'import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info("Funkcja Lambda została uruchomiona.")
    logger.warning("To jest ostrzeżenie!")
    logger.error("To jest komunikat o błędzie!")

    return {
        "statusCode": 200,
        "body": json.dumps("Logowanie w CloudWatch działa poprawnie!")
    }' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

---

### **3. Pobranie ARN roli IAM**  
```sh
aws iam get-role --role-name LambdaLoggingExecutionRole --query 'Role.Arn' --output text
```

---

### **4. Tworzenie funkcji Lambda**  
Zastąp `<ARN_ROLI>` uzyskaną wartością:  
```sh
aws lambda create-function --function-name LambdaLoggingDemo \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **5. Wywołanie funkcji Lambda i generowanie logów**  
```sh
aws lambda invoke --function-name LambdaLoggingDemo --payload '{}' response.json
```

Sprawdzenie odpowiedzi:  
```sh
cat response.json
```

---

### **6. Sprawdzenie logów w CloudWatch**  
Wylistowanie grup logów:  
```sh
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/LambdaLoggingDemo"
```

Pobranie najnowszych logów funkcji Lambda:  
```sh
aws logs tail "/aws/lambda/LambdaLoggingDemo" --follow
```
</details>

---


**Wyzwalanie AWS Lambda za pomocą Amazon SNS**   
Skonfiguruj Amazon SNS tak, aby wiadomości publikowane w temacie SNS automatycznie wywoływały funkcję Lambda. Sprawdź, czy funkcja poprawnie przetwarza otrzymane wiadomości.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie tematu SNS**
1. Przejdź do **AWS Management Console** → **Amazon SNS**.  
2. Kliknij **Topics** → **Create topic**.  
3. Wybierz **Standard** jako typ tematu.  
4. Wpisz **Name**: `LambdaSNSTopic`  
5. Kliknij **Create topic**.  
6. Skopiuj **Topic ARN** – będzie potrzebny później.  

---

### **2. Tworzenie funkcji Lambda**
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch**.  
4. Wypełnij dane:  
   - **Function name**: `SNSLambdaHandler`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: **Create a new role with basic Lambda permissions**  
5. Kliknij **Create function**.  

---

### **3. Edycja kodu funkcji Lambda**
1. Przejdź do sekcji **Code** i wklej następujący kod:  

   ```python
   import json

   def lambda_handler(event, context):
       for record in event["Records"]:
           message = record["Sns"]["Message"]
           print(f"Otrzymano wiadomość z SNS: {message}")

       return {
           "statusCode": 200,
           "body": json.dumps("SNS event processed successfully!")
       }
   ```
2. Kliknij **Deploy**.  

---

### **4. Powiązanie SNS z funkcją Lambda**
1. Przejdź do zakładki **Configuration** → **Triggers**.  
2. Kliknij **Add Trigger**.  
3. Wybierz **SNS** jako źródło wyzwalania.  
4. Wybierz utworzony wcześniej temat **LambdaSNSTopic**.  
5. Kliknij **Add**.  

📌 **Teraz każda wiadomość wysłana do SNS wywoła funkcję Lambda!**  

---

### **5. Testowanie SNS i Lambda**
1. Przejdź do **Amazon SNS** → **LambdaSNSTopic** → **Publish message**.  
2. Wpisz dowolny tekst w **Message body** i kliknij **Publish message**.  
3. Przejdź do **AWS Lambda** → **Monitor** → **View logs in CloudWatch**.  
4. Sprawdź logi funkcji Lambda, powinny zawierać otrzymaną wiadomość z SNS.  
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie tematu SNS**
```sh
aws sns create-topic --name LambdaSNSTopic
```
Skopiuj zwrócony `TopicArn`.

---

### **2. Tworzenie roli IAM dla AWS Lambda**
```sh
aws iam create-role --role-name LambdaSNSExecutionRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```
Przypisujemy politykę pozwalającą na zapis logów do CloudWatch:  

```sh
aws iam attach-role-policy --role-name LambdaSNSExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

### **3. Tworzenie pliku funkcji Lambda**
```sh
echo 'import json

def lambda_handler(event, context):
    for record in event["Records"]:
        message = record["Sns"]["Message"]
        print(f"Otrzymano wiadomość z SNS: {message}")

    return {
        "statusCode": 200,
        "body": json.dumps("SNS event processed successfully!")
    }' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

---

### **4. Pobranie ARN roli IAM**
```sh
aws iam get-role --role-name LambdaSNSExecutionRole --query 'Role.Arn' --output text
```

---

### **5. Tworzenie funkcji Lambda**
Zastąp `<ARN_ROLI>` uzyskaną wartością:  
```sh
aws lambda create-function --function-name SNSLambdaHandler \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **6. Subskrypcja Lambda do tematu SNS**
Zastąp `<TOPIC_ARN>` wartością zwróconą w kroku 1:
```sh
aws sns subscribe --topic-arn <TOPIC_ARN> \
    --protocol lambda \
    --notification-endpoint $(aws lambda get-function --function-name SNSLambdaHandler --query "Configuration.FunctionArn" --output text)
```

### **7. Zezwolenie SNS na wywołanie Lambda**
```sh
aws lambda add-permission --function-name SNSLambdaHandler \
    --statement-id sns-permission \
    --action "lambda:InvokeFunction" \
    --principal sns.amazonaws.com \
    --source-arn <TOPIC_ARN>
```

---

Publikowanie wiadomości do SNS:
```sh
aws sns publish --topic-arn <TOPIC_ARN> --message "Testowa wiadomość dla Lambda!"
```

Sprawdzenie logów w CloudWatch:
```sh
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/SNSLambdaHandler"
aws logs tail "/aws/lambda/SNSLambdaHandler" --follow
```
</details>

**Automatyczne przetwarzanie wiadomości z Amazon SQS przez AWS Lambda**   
Stwórz kolejkę Amazon SQS i skonfiguruj AWS Lambda do automatycznego przetwarzania wiadomości. Upewnij się, że wiadomości są odbierane i poprawnie logowane przez funkcję.  

<details>
    <summary>Instrukcja krok po kroku AWS GUI</summary>

### **1. Tworzenie kolejki SQS**
1. Przejdź do **AWS Management Console** → **Amazon SQS**.  
2. Kliknij **Create queue**.  
3. Wybierz **Standard queue**.  
4. Wpisz **Name**: `LambdaSQSQueue`.  
5. W sekcji **Access policy** wybierz **Advanced** i dodaj uprawnienia dla Lambda (można je także dodać później).  
6. Kliknij **Create queue**.  
7. Skopiuj **Queue URL** i **Queue ARN** – będą potrzebne później.  

---

### **2. Tworzenie funkcji Lambda**
1. Przejdź do **AWS Management Console** → **Lambda**.  
2. Kliknij **Create function**.  
3. Wybierz **Author from scratch**.  
4. Wypełnij dane:  
   - **Function name**: `SQSProcessorLambda`  
   - **Runtime**: `Python 3.9`  
   - **Execution role**: **Create a new role with basic Lambda permissions**  
5. Kliknij **Create function**.  

---

### **3. Edycja kodu funkcji Lambda**
1. W sekcji **Code** wklej następujący kod:  

   ```python
   import json

   def lambda_handler(event, context):
       for record in event["Records"]:
           message_body = record["body"]
           print(f"Otrzymano wiadomość z SQS: {message_body}")

       return {
           "statusCode": 200,
           "body": json.dumps("SQS message processed successfully!")
       }
   ```
2. Kliknij **Deploy**.  

---

### **4. Powiązanie SQS z funkcją Lambda**
1. Przejdź do zakładki **Configuration** → **Triggers**.  
2. Kliknij **Add Trigger**.  
3. Wybierz **SQS** jako źródło wyzwalania.  
4. Wybierz utworzoną wcześniej kolejkę **LambdaSQSQueue**.  
5. Kliknij **Add**.  

---

### **5. Testowanie SQS i Lambda**
1. Przejdź do **Amazon SQS** → **LambdaSQSQueue** → **Send and receive messages**.  
2. Wpisz przykładową wiadomość, np. `"Testowa wiadomość dla Lambda"` i kliknij **Send message**.  
3. Przejdź do **AWS Lambda** → **Monitor** → **View logs in CloudWatch**.  
4. Sprawdź logi funkcji Lambda, powinny zawierać otrzymaną wiadomość z SQS.
</details>
<details>
    <summary>Instrukcja krok po kroku AWS CLI</summary>

### **1. Tworzenie kolejki SQS**
```sh
aws sqs create-queue --queue-name LambdaSQSQueue
```
Pobierz ARN kolejki:
```sh
aws sqs get-queue-attributes --queue-url $(aws sqs get-queue-url --queue-name LambdaSQSQueue --query "QueueUrl" --output text) --attribute-name QueueArn
```
Skopiuj wartość `QueueArn`.

---

### **2. Tworzenie roli IAM dla AWS Lambda**
```sh
aws iam create-role --role-name LambdaSQSExecutionRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
```
Przypisujemy polityki:
```sh
aws iam attach-role-policy --role-name LambdaSQSExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

aws iam attach-role-policy --role-name LambdaSQSExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess
```

---

### **3. Tworzenie pliku funkcji Lambda**
```sh
echo 'import json

def lambda_handler(event, context):
    for record in event["Records"]:
        message_body = record["body"]
        print(f"Otrzymano wiadomość z SQS: {message_body}")

    return {
        "statusCode": 200,
        "body": json.dumps("SQS message processed successfully!")
    }' > lambda_function.py
```

Spakowanie pliku ZIP:  
```sh
zip function.zip lambda_function.py
```

---

### **4. Pobranie ARN roli IAM**
```sh
aws iam get-role --role-name LambdaSQSExecutionRole --query 'Role.Arn' --output text
```

---

### **5. Tworzenie funkcji Lambda**
Zastąp `<ARN_ROLI>` uzyskaną wartością:  
```sh
aws lambda create-function --function-name SQSProcessorLambda \
    --runtime python3.9 \
    --role <ARN_ROLI> \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

---

### **6. Powiązanie kolejki SQS z funkcją Lambda**
Zastąp `<QUEUE_ARN>` wartością zwróconą w kroku 1:
```sh
aws lambda create-event-source-mapping \
    --function-name SQSProcessorLambda \
    --batch-size 10 \
    --event-source-arn <QUEUE_ARN>
```

---

### **7. Testowanie SQS i Lambda**  
Publikowanie wiadomości do SQS:
```sh
aws sqs send-message --queue-url $(aws sqs get-queue-url --queue-name LambdaSQSQueue --query "QueueUrl" --output text) --message-body "Testowa wiadomość dla Lambda!"
```

Sprawdzenie logów w CloudWatch:
```sh
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/SQSProcessorLambda"
aws logs tail "/aws/lambda/SQSProcessorLambda" --follow
```
</details>

---


