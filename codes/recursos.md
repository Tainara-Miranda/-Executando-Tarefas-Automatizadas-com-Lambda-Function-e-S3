1. Criar o container no docker desktop:
 ```bash
   docker run -d --name localstack -p 4566:4566 -p 4571:4571 -e SERVICES=ALL -e DEBUG=1 -v /var/run/docker.sock:/var/run/docker.sock localstack/localstack
 ```
2. Criar o bucket S3:
 ```bash
   aws s3api create-bucket --bucket notas-fiscais-upload --endpoint-url=http://localhost:4566
 ```
3. Criar a tabela DynamoDB:
```bash
   aws dynamodb create-table --endpoint-url=http://localhost:4566 --table-name NotasFiscais --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```
4. Criar o Lambda Function:
```bash
   aws lambda create-function --function-name ProcessarNotasFiscais --runtime python3.9 --role arn:aws:iam::000000000000:role/lambda-role --handler grava_db.lambda_handler --zip-file fileb://lambda_function.zip --endpoint-url=http://localhost:4566
 ```
5. Criar a Permissão ao S3 para invocar o Lambda Function:
```bash
aws lambda add-permission --function-name ProcessarNotasFiscais --statement-id s3-trigger-permission --action "lambda:InvokeFunction" --principal s3.amazonaws.com --source-arn "arn:aws:s3:::notas-fiscais-upload" --endpoint-url=http://localhost:4566
 ```
6. Configurar a notificação
```bash
aws s3api put-bucket-notification-configuration --bucket notas-fiscais-upload --notification-configuration file://notification.json --endpoint-url=http://localhost:4566
 ```
7. Validar a notificação no S3:
```bash
aws s3api get-bucket-notification-configuration --bucket notas-fiscais-upload --endpoint-url=http://localhost:4566
 ```
8. Criar o API:
```bash
	aws apigateway create-rest-api --name "NotasFiscaisAPI" --endpoint-url=http://localhost:4566
 ```
