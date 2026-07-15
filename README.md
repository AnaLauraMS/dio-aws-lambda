# Automação AWS: S3 + Lambda

Projeto prático desenvolvido para o **Bootcamp GFT Start #6 - AWS** na DIO. O objetivo é demonstrar a integração de serviços serverless criando um fluxo automatizado de processamento de arquivos.

---

## Como Funciona o Fluxo

1. O usuário envia um arquivo de texto (`.txt`) para a pasta `input/` de um bucket no **Amazon S3**.
2. O upload ativa automaticamente um gatilho para a função **AWS Lambda**.
3. A Lambda (em **Python**) lê o arquivo, transforma todo o texto em letras maiúsculas e salva o resultado final na pasta `output/` do mesmo bucket.

---

## 💻 Código Python Utilizado (Lambda)

```python
import json
import urllib.parse
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')
    
    try:
        if "output/" in key:
            return {'statusCode': 200, 'body': json.dumps('Ignorado.')}
            
        response = s3.get_object(Bucket=bucket, Key=key)
        content = response['Body'].read().decode('utf-8')
        
        processed_content = content.upper()
        
        output_key = key.replace("input/", "output/")
        if output_key == key:
            output_key = "output/" + key
        
        s3.put_object(Bucket=bucket, Key=output_key, Body=processed_content.encode('utf-8'))
        
        print(f"Sucesso! Salvo em {output_key}")
        return {'statusCode': 200, 'body': json.dumps('Sucesso!')}
        
    except Exception as e:
        print(e)
        raise e
