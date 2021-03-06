# Workshop Fuse Online API

Demonstração de como criar uma API no Fuse Online para inserir e obter registros de um banco de dados.

> Nehum código é necessário para este workshop.

## Pré-requisitos



### Red Hat Integrately Environment

Para completar esta atividade é necessário instalar e configurar o Fuse Online no Openshift. Veja: [INSTALLING AND OPERATING FUSE ONLINE ON OPENSHIFT CONTAINER PLATFORM](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.7/html/installing_and_operating_fuse_online_on_openshift_container_platform/index)

## Laboratórios:

0. [Criando um banco de dados no Openshift](#testdrive-step-0)
1. [Criando a tabela no banco de dados](#testdrive-step-1)
2. [Criando uma Conexão com um banco de dados no Fuse Online ](#testdrive-step-2)
3. [Criando uma API Rest no Fuse Online utilizando API First Approach](#testdrive-step-3)
4. [Utilizando Conditional Flows](#testdrive-step-4)
4. [SoapToRest](https://developers.redhat.com/blog/2018/07/24/migrate-soap-to-rest-with-camel/)
5. [API First](https://developers.redhat.com/blog/2018/07/12/contract-first-api-design-with-apicurio-and-red-hat-fuse/)


### Requisitos de Negócio 

Nesta atividade a intenção é simular o desenvolvimento de uma API que irá receber transações dos pedidos efetuados pelos consumidores de uma empresa. 
Após receber estas transações a integração irá gravar na tabela de banco de dados ou irá enviar esta transação para outro sistema através de mensageria.

### Criando um banco de dados no Openshift <a name="testdrive-step-0"></a>

Realize o login no Openshift selecionando o identity provider **testing-idp**

![](fuseonline/01.png)

Informe seu usuário e senha:

![](fuseonline/02.png)

Clique em 'Create Project' e crie um projeto para o seu banco de dados conforme os dados a seguir:

![](fuseonline/04.png)

Clique no projeto criado (no nosso caso order-db) e do lado esquerdo acima mude para **Developer**:

![](fuseonline/06.png)

Ainda do lado esquerdo da tela clique em 'Add' e logo em seguida clique em 'Database'.

![](fuseonline/07.png)

Selecione a opção **PostgreSQL (Ephemeral)**

![](fuseonline/07.png)

Clique em **Instantiate Template**

![](fuseonline/07.png)

Informe **Conection Username** e **Conection Password** (guarde estas informações pois elas serão necessárias para se conectar ao banco de dados).

![](fuseonline/12.png)

Clique em **Topology** e acompanhe o processo de deploy que estará finalizado quando o pod tiver o círculo azul escuro conforme a imagem abaixo:

![](fuseonline/13.png)

### Criando a tabela no banco de dados  <a name="testdrive-step-1"></a>

Após finalizar o processo de deploy do banco de dados, volte para a view de **Administrator**.

![](fuseonline/14.png)

Selecione a opção **Workloads** e logo abaixo **Pods**. 
Verifique em **Status** se o pod do banco de dados está em **Running** para se certificar que não tivemos nenhum problema até aqui.

![](fuseonline/15.png)

Clique no pod do banco de dados na coluna **Name** e logo em seguida selecione a aba **Terminal**.

![](fuseonline/16.png)

Utilize o comando abaixo no terminal do seu pod para se conectar ao banco de dados informando o username e a senha do banco de dados.

```bash
psql -h postgresql sampledb username
```
Agora que nos conectamos ao banco de dados iremos criar uma tabela chamada **customer_order** conforme o comando a seguir:

```bash
CREATE TABLE customer_order (id serial PRIMARY KEY, transaction_id VARCHAR(50) NOT NULL, status VARCHAR (50) NOT NULL, system VARCHAR (50) NOT NULL);
```
![](fuseonline/17.png)

Agora que a tabela foi criada, iremos verificar qual o endereço de conexão ao banco de dados. Do lado esquerdo clique em **Networking** para expandir e depois clique em **Services**. Na coluna **Location** você verá o IP e a porta para acesso ao banco de dados. Iremos utilizar esta informação mais adiante.

![](fuseonline/18.png)


### Criando uma Conexão com um banco de dados no Fuse Online <a name="testdrive-step-2"></a>

Após logar no seu ambiente do Fuse Online, do lado esquerdo clique em **Conections** e logo depois clique em **Create Conection** do lado direito acima.

![](fuseonline/19.png)

Todos os conectores do Fuse Online serão listados, selecione a opção **Database**.

![](fuseonline/20.png)

Informe a URL de conexão com o banco, bem como o usuário e senha conforme a seguir. Lembre-se que o IP e porta deverão ser os mesmos que aparecem em **Location** quando selecionamos **Networking** e **Services** conforme destacamos 3 passos anteriores. Após preencher os dados clique em **Validate** para verificar se a conexão está correta e depois em **Next**.

![](fuseonline/21.png)

Clique em **Save** para salvar a conexão criada. 

![](fuseonline/22.png)

### Criando uma API Rest no Fuse Online utilizando API First Approach <a name="testdrive-step-3"></a>

Agora iremos criar a nossa API Rest que possibilitará inserir novas transações no banco de dados **(POST)** ou obter a lista de transações do nosso banco **(GET)**.

No lado esquerdo acima clique em **Integrations** e logo depois em **Create Integration** na parte central:

![](fuseonline/23.png)

Logo depois selecione a opção **API Provider**.

![](fuseonline/24.png)

Selecione a opção de **Create a new OpenAPI 2.x document** 

![](fuseonline/25.png)

Mude algumas informações da API como **Title, Description**

![](fuseonline/26.png)

Logo após cliquem em Add Path do lado esquerdo e defina o **path /customer_order** e clique em **Add**

![](fuseonline/27.png)

Depois clique em **Add a data type** do lado esquerdo acima.

![](fuseonline/28.png)

Agora preencha os campos **Name, Description** e em seguida no campo **Enter JSON Example** informe o exemplo abaixo.

```json
{
  "transaction_id": "131314",
  "status": "completed",
  "system": "db"
}
```

**O JSON acima contem os que serão enviados pelos clientes de nossa API.**

![](fuseonline/29.png)

Obs: Não esqueça de salvar as suas alterações ao final.

Clique novamente no path que foi definido anteriormente **customer_order** para que possamos adicionar operações. Na parte central, selecione **POST** e clique em **Add Operation**

![](fuseonline/30.png)

Preencha as informação de **Summary, Description e Operation Id**.

![](fuseonline/31.png)

Mais abaixo ainda na operação **POST** clique em **Add a request body** e especifique o Data Type que definimos anteriormente, no nosso caso **customer_transaction**. Este serão os dados recebidos pela API. 

![](fuseonline/32.png)

Logo em seguida defina o **Response Type** como o retorno **201 - Created** e logo após selecione **customer_transaction** e também preencha uma **Description** para o retorno da consulta.

![](fuseonline/33.png)

Agora iremos definir a operação **GET** que possibilitará a consulta de todos as transações gravadas no banco de dados.

![](fuseonline/34.png)

Preencha as informações do **Summary, Description e Operation ID**

![](fuseonline/35.png)

A seguir em **Responses** selecione a opção **200 OK** definindo como **Type** array of customer_transaction (a nossa API retornará um array com todas as transações gravadas no banco). Clique em **Save**.

![](fuseonline/36.png)

Verifique se não tem nenhum erro em sua API e clique em **Next**.

![](fuseonline/37.png)

Veja que agora temos uma API com duas operações definidas **POST / GET** -- Na operação **GET** clique em **Create Flow**

![](fuseonline/38.png)

Clique no sinal de **+** no centro da tela para adicionar mais um passo a nossa API que será a conexão com o banco de dados.

![](fuseonline/39.png)

Selecione a conexão com o banco de dados que criamos anteriormente, no nosso caso será a opção **Database Customer_Order** 

![](fuseonline/40.png)

Logo em seguida seleciona e opção **Invoke SQL**

![](fuseonline/41.png)

Especifique a query abaixo e clique em **Next**. 

```sql
SELECT * FROM customer_order
``` 

Esta operação **GET** irá trazer os registros de transações do banco de dados. 

![](fuseonline/42.png)

Agora precisaremos mapear os campos recebidos em nossa API de acordo com o retorno da query do banco. Clique no ponto de exclamação e em seguida em **Add a data mapping step**

![](fuseonline/43.png)

Em seguida realize o mapeamento dos campos conforme a imagem abaixo em seguida clique em **Done**.

![](fuseonline/44.png)

Agora no canto esquerdo acima iremos selecionar a operação **POST**

![](fuseonline/45.png)

Clique no sinal de **+** para que possamos adicionar uma conexão ao banco de dados.

![](fuseonline/46.png)

Selecione novamente a opção **Database Customer_Order**

![](fuseonline/47.png)

Clique em **Invoke SQL** 

![](fuseonline/48.png)

Especique a query abaixo  e clique em **Next**.

```sql
INSERT INTO CUSTOMER_ORDER (transaction_id, status, system) values (:#transaction_id, :#status, :#system)
``` 

Veja que os campos que serão preenchidos pelos valores recebidos na API se iniciam com **:#**



![](fuseonline/49.png)

Agora clique em **Add a data mapping step**

![](fuseonline/50.png)

Em seguida realize o mapeamento dos campos conforme imagem a seguir e clique em **Done**

![](fuseonline/51.png)

Clique novamente em **Add a data mapping step** no passo final, que será a resposta da nossa query sql

![](fuseonline/52.png)

Faça o mapeamento dos campos conforme a seguir e clique em **Done**

![](fuseonline/53.png)

Clique em **Save** para salvar as alterações

![](fuseonline/54.png)

Defina um nome para sua API e clique em **Save and Publish**

![](fuseonline/55.png)

Agora acompanhe a publicação de sua API

![](fuseonline/56.png)

Ao final da publicação cliquem em **Disable Discovery** para que possamos copiar a URL de acesso a nossa API e aguarde a publicação ocorrer novamente

![](fuseonline/57.png)

Após copiar a URL de acesso a sua API, com uma ferramenta de testes ou usando comandos como o **curl** faça um teste chamando a API. **Não se esqueça de informar o path no final da url, no nosso caso é /customer_order**

**GET**
![](fuseonline/58.png)

**POST**
![](fuseonline/59.png)

Selecione sua API no Fuse Online e verifique em **Metrics** o total de mensagens recebidas

![](fuseonline/60.png)

Clique na aba **Activity** para mais detalhes

![](fuseonline/61.png)


### Utilizando Conditional Flows <a name="testdrive-step-4"></a>

Agora iremos utilizar [Simple Expression Language](https://camel.apache.org/components/latest/languages/simple-language.html) em conjunto com um **Conditional Flow** para inserir todas as mensagens que no campo **system** possuem **db** Já quando a mensagem possuir **others** no campo **system** a mesma será encaminhada para um tópico **Kafka**.

**Obs**: Antes de iniciar esta atividade, crie um cluster kafka com o Openshift Operators e logo em seguida crie uma conexão no Fuse Online para o cluster Kafka.

Do lado esquerdo clique em **Integrations**, localize a API que criamos anteriormente e clique em **Edit**

![](fuseonline/74.png)

Na transação **POST** clique em **Edit flow**

![](fuseonline/75.png)

Em seguida, exclua todos os **"steps"** da operação **POST** deixando conforme a imagem abaixo:

![](fuseonline/77.png)

Clique no sinal de **+** e selecione **Conditional Flows**

![](fuseonline/78.png)

Em seguida escolha a opção **Advanced expression builder**

![](fuseonline/79.png)

No campo **When** specifique a **simple expression** abaixo e marque a opção **Execute default flow**

```bash
${body.body.system} contains 'db'
``` 
**Obs:** Veja que nesta **Simple Expression** estamos utilizando duas vezes o body. Isto se deve ao fato do Fuse Online adicionar uma propriedade body quando algumas integrações iniciam com **API Provider, Webhook ou Custom REST API connection**

![](fuseonline/80.png)

Especifique o tipo de dado selecionando **JSON Instance** e clique em **Next**

![](fuseonline/81.png)

Agora teremos a opção de definir dois fluxos sendo eles **When** que será executado quando a variável **system** for **db** e o fluxo **Otherwise** que será executado caso a condição não seja atendida.

![](fuseonline/82.png)

Clique em **Open flow** no primeiro fluxo **(When)** e logo em seguida clique no sinal de **+**.

![](fuseonline/83.png)

Iremos selecionar a nossa conexão com o banco de dados **Database Customer Order**. Pois caso a variável **sytem** tenha seu valor como **db** e mensagem deverá ser gravada no banco de dados.

![](fuseonline/84.png)

Clique em Invoke SQL

![](fuseonline/85.png)

Especique a query abaixo e clique em Next.

```sql
INSERT INTO CUSTOMER_ORDER (transaction_id, status, system) values (:#transaction_id, :#status, :#system)
``` 
![](fuseonline/86.png)

Logo em seguida realize o mapeamento dos dados de entrada e execução da query conforme a seguir:

![](fuseonline/87.png)

![](fuseonline/88.png)

Em seguida clique em **Go to operation flow** para voltarmos ao fluxo principal.

![](fuseonline/89.png)

Agora iremos definir o fluxo **Otherwise** que será executado caso a variável **system** possua um valor diferente de **db**. Esta mensagem será encaminhada para um tópico kafka, e neste tópico outros sistemas poderão receber esta transação.

Clique em **Open flow**

![](fuseonline/90.png)

Logo em seguida clique em **+**

![](fuseonline/91.png)

Agora selecione a opção **Kafka Message Broker**. Lembrando que esta conexão foi definida previamente e consta como pré requisito desta atividade.

![](fuseonline/92.png)

Em seguida selecione o tópico Kafka que receberá esta mensagem

![](fuseonline/96.png)

Especifique o tipo de dado selecionando **JSON Instance** e clique em Next


```json
{
  "transaction_id": "131314",
  "status": "completed",
  "system": "others"
}
``` 

![](fuseonline/97.png)

Realize em seguida o mapeamento dos dados

![](fuseonline/98.png)

![](fuseonline/99.png)

Em seguida verifique se não existe nenhum erro e clique em **Go to operation flow**

![](fuseonline/101.png)

Pronto, agora é só clicar em **Publish** para publicar sua Integração

![](fuseonline/102.png)

Define um nome e selecione novamente **Save and Publish**

![](fuseonline/103.png)


Após publicar sua API, realize testes utilizando o **curl** ou outras ferramentas como o **postman**. Verifique nas métricas de sua rota se os fluxos estão sendo executados corretamente conforme o valor de **system**.






