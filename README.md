# ☁️ Executando Tarefas Automatizadas com Lambda Function e S3 (LocalStack)

A ideia para este desafio foi criar um **Bucket S3**, uma tabela no **DynamoDB** e uma função **Lambda Function** que se comunicassem entre si de forma **local**. Para realizar este desafio, utilizou-se o **LocalStack** em conjunto com o **Docker Desktop**, simulando um ambiente AWS.

## 🧭 Conteúdo do Repositório

* Na pasta **[`images`](/images)**: Você encontrará o fluxograma do projeto e as imagens que acompanham os resultados no LocalStack.
* Na pasta **[`codes`](/codes)**: Estão os códigos utilizados para gerar os recursos e as funções Lambda.

---

## 📝 A Ideia por Trás do Desafio: Ingestão e Processamento de Dados

A *case* apresentada é a ideia de realizar um upload de arquivos com processamento e registro no DynamoDB. O fluxo de trabalho segue a seguinte sequência (conforme o [fluxograma](/images/fluxograma)):

1.  **Upload:** O usuário faz o upload de um arquivo **JSON** em um **Bucket S3**.
2.  **Trigger:** Um evento no **S3** dispara uma **Lambda Function** escrita em Python.
3.  **Processamento e Persistência:** A Lambda Function processa o conteúdo do arquivo (ex: extrai dados relevantes) e grava as informações em uma tabela no **DynamoDB**.
4.  **Exposição de Dados:** Outra **Lambda Function** irá consultar a tabela e expor os dados por meio de uma **API Gateway**.

---

## ⚠️ Insights Obtidos: Solução de Problemas com Triggers (LocalStack)

Durante o processo, houve um erro ao tentar criar uma *trigger* para o Bucket S3:

> *An error occurred (InvalidArgument) when calling the PutBucketNotificationConfiguration operation: Unable to validate the following destination configurations*.

Após pesquisa, entendeu-se que o erro se dava ao fato do serviço de **EVENTS** não estar acionado no LocalStack. A solução seguiu os seguintes passos cruciais:

1.  Parar o container que estava rodando: `docker stop localstack-main`
2.  Apagar o container: `docker rm localstack-main`
3.  **Recriar com as novas configurações (incluindo o suporte a EVENTOS):**
    ```bash
    docker run -d --name localstack-main -p 4566:4566 -e SERVICES=ALL -e DEBUG=1 -v /var/run/docker.sock:/var/run/docker.sock localstack/localstack:latest
    ```
4.  Recriar os recursos (S3, Lambda, DynamoDB) criados anteriormente.

---

Com este desafio, o aprendizado focado foi em realizar **tarefas automatizadas** com **Lambda Function** e **Bucket S3**, a partir de uma configuração **Local** através do **LocalStack**, onde o contato com o LocalStack foi realizado com o **Docker Desktop**.

<br/>

## 👩‍💻 Realizado por:

<p>
    <img
      align=left
      margin=10
      width=80
      src="https://avatars.githubusercontent.com/u/194859576?s=400&u=beab67e10b95d1a468f6e7b1e6c763498e63e210&v=4"
      alt="Avatar de Tainara Miranda"
    />
    <p>&nbsp&nbsp&nbsp**Tainara Miranda**<br>
    &nbsp&nbsp&nbsp
    <a
        href="https://github.com/Tainara-Miranda">
        GitHub
    </a>
    &nbsp;|&nbsp;
    <a
        href="https://www.linkedin.com/in/tainara-miranda-979954108/">
        LinkedIn
    </a>
    &nbsp;|&nbsp;</p>
</p>
<br/><br/>
