```python
import json
import boto3
import os
import logging
from decimal import Decimal

# Configurar logger
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# URL do LocalStack (padrão)
LOCALSTACK_URL = os.getenv("LOCALSTACK_URL", "http://localhost:4566")

# Conectar ao DynamoDB e S3 apontando para o LocalStack
dynamodb = boto3.resource('dynamodb', endpoint_url=LOCALSTACK_URL)
s3 = boto3.client('s3', endpoint_url=LOCALSTACK_URL)

# Conectar tabela
table = dynamodb.Table('NotasFiscais')


def lambda_handler(event, context):
    logger.info(f"Evento recebido: {json.dumps(event)}")

    # 1) Evento disparado por upload no S3
    if "Records" in event and event["Records"][0].get("eventSource") == "aws:s3":
        return processar_upload_s3(event)

    # 2) Evento vindo do API Gateway (GET/POST)
    metodo = event.get("httpMethod")

    if metodo == "GET":
        return consultar_registros(event)

    if metodo == "POST":
        return inserir_registro_unico(event)

    return {
        'statusCode': 400,
        'body': json.dumps('Erro: Método HTTP não suportado.')
    }


# ==============================
# API: GET → Consulta no DynamoDB
# ==============================
def consultar_registros(event):
    try:
        response = table.scan()
        logger.info(f"Consulta realizada. Total de registros: {response['Count']}")

        return {
            'statusCode': 200,
            'body': json.dumps(response['Items'], default=str)
        }

    except Exception as e:
        logger.error(f"Erro na consulta: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps('Erro ao consultar registros.')
        }


# ===================================
# API: POST → Insere um único registro
# ===================================
def inserir_registro_unico(event):
    try:
        body = json.loads(event['body'])
        logger.info(f"Inserindo registro manual: {body}")

        # Validação básica
        if not all(key in body for key in ["id", "cliente", "valor", "data_emissao"]):
            return {
                'statusCode': 400,
                'body': json.dumps('Erro: Campos obrigatórios faltando.')
            }

        # Converter valor numérico para Decimal (DynamoDB requer isso)
        body['valor'] = Decimal(str(body['valor']))

        table.put_item(Item=body)

        return {
            'statusCode': 200,
            'body': json.dumps('Registro inserido com sucesso!')
        }

    except Exception as e:
        logger.error(f"Erro ao inserir registro: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps('Erro ao inserir registro.')
        }


# ==========================================
# TRIGGER S3 → importa notas_fiscais.json
# ==========================================
def processar_upload_s3(event):
    try:
        bucket = event['Records'][0]['s3']['bucket']['name']
        arquivo = event['Records'][0]['s3']['object']['key']

        logger.info(f"Arquivo recebido no S3: {arquivo}")

        if arquivo != "notas_fiscais.json":
            logger.info("Arquivo ignorado (nome não corresponde).")
            return {'statusCode': 200, 'body': 'Arquivo ignorado.'}

        response = s3.get_object(Bucket=bucket, Key=arquivo)
        conteudo = response['Body'].read().decode('utf-8')
        notas = json.loads(conteudo)

        for nota in notas:
            if 'valor' in nota:
                nota['valor'] = Decimal(str(nota['valor']))
            table.put_item(Item=nota)
            logger.info(f"Nota inserida: {nota.get('id')}")

        return {
            'statusCode': 200,
            'body': 'Notas importadas com sucesso do S3.'
        }

    except Exception as e:
        logger.error(f"Erro ao processar arquivo do S3: {str(e)}")
        return {'statusCode': 500, 'body': 'Erro ao processar arquivo do S3.'}

```
