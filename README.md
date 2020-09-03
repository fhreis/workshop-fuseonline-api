# Workshop Fuse Online API

Demonstração de como criar uma API no Fuse Online para inserir e obter registros de um banco de dados.

> Nehum código é necessário para este workshop.


## Pre-requisitos

### Integrately Environment

Para completar esta atividade é necessário instalar e configurar o Fuse Online no Openshift.

### Criando um banco de dados no Openshift

Realize o login no Openshift selecionando o identity provider 'testing-idp'

![](fuseonline/01.png)

Informe seu usuário e senha:

![](fuseonline/02.png)

Clique em 'Create Project' e crie um projeto para o seu banco de dados conforme os dados a seguir:

![](fuseonline/04.png)

Clique no projeto que foi criado e do lado esquerdo acima mude para 'DEVELOPER':

![](fuseonline/06.png)
`

### Creating a Database Connection on Fuse Online

Open your tutorial page: https://tutorial-web-app-webapp.apps.latam-3a88.openshiftworkshop.com

> Update this url `https://tutorial-web-app-webapp.apps.latam-3a88.openshiftworkshop.com` according to your environment

Open Fuse Online

![](imgs/01.png)

Click on `Connections`

![](imgs/02.png)

Click on `Create Connection`

![](imgs/03.png)

Then, select `Database`

![](imgs/04.png)

Fill the database configuration with the following values:

```properties
url: jdbc:postgresql://postgresql.fuse-demo:5432/sampledb
user: redhat
password: redhat
```

![](imgs/05.png)

Now, click on `Validate` to make sure everything is working as expected. If it is all good, click on `Next`.

![](imgs/09.png)

The Connection Name is: `Users Database`. Then, click on `Create`

![](imgs/06.png)

Now you should see connection `Users Database` listed in the connections page.

![](imgs/10.png)

We are good to go for our API creation demo.

## Demo

### Create an API from Scratch

Back to our `Home` page, click on `Create Integration`

![](imgs/11.png)

Then select `API Provider` from the connections listed.

![](imgs/12.png)

Choose `Create from scratch`

![](imgs/13.png)

Click on `Add a data type`

![](imgs/14.png)

Give it a name like: `User`

![](imgs/15.png)

Paste the following json example and choose `REST Resource`. Then, click `Save`.

```json
{
    "id": 0,
    "name": "Rodrigo Ramalho",
    "phone": "11 95474-8099",
    "age": 30
}
```

Click `Save` again.

![](imgs/16.png)

Now, click on `Next`

![](imgs/17.png)

And give a name for our integration: `Users API`. Click on `Save and continue`

![](imgs/18.png)

#### Creating an API for `Get All Users` (GET)

Create now a flow for the GET Method that list all users:

![](imgs/19.png)

Add a step in our flow clicking on `+`:

![](imgs/20.png)

Now choose our `Users Database` connection created previously.

![](imgs/21.png)

Click on `Invoke SQL to obtain, store, update or delete data`:

![](imgs/22.png)

Fill the `SQL Statement` with: `select * from users` and then click `Next`

![](imgs/23.png)

Add a log step in our flow. Click again on the `+`:

![](imgs/24.png)

Then choose `Log`

![](imgs/25.png)

In the `Custom Text`, write `Loading users from database` and click `Done`.

![](imgs/26.png)

Now, let's add a data mapping to our flow. In the last step, click in the yellow icon and then go to `Add a data mapping step`.

![](imgs/27.png)

Expand both panel clicking on the arrows:

![](imgs/28.png)

Now, drag and drop the source fields matching with the target fields and then click on `Done`.

![](imgs/29.png)

Click now on `Save`.

![](imgs/30.png)

#### Creating API for `Create a users` (POST)

From the combobox `Operations`, choose `Create a users`:

![](imgs/31.png)

Repeat the same steps you did when `Creating an API for Get All Users (GET)`

When adding the Users Database, you need to click on `Invoke SQL to obtain, store, update or delete data` and add `INSERT INTO USERS(NAME,PHONE,AGE) VALUES(:#NAME,:#PHONE,:#AGE);` in the field `SQL statement`.

![](imgs/32.png)

Also, during the data mapping you won't need to associate the `id` field because it will be already generate by the postgres database.

![](imgs/33.png)

In the end, you should have something like:

![](imgs/34.png)

Now, click on `Save` and then on `Publish`

![](imgs/35.png)

Now, we need to wait Openshift build our container. When done, you should see `Published version 1` on the top of the page.

If you go to the `Home` page, we have 1 integration running.

![](imgs/37.png)

Our last step is to expose our integration on Openshift using `Route`s.

```bash
oc create route edge i-users-api --service=i-users-api -n fuse
```

### Testing your integration

You can check if your integration is working properly running:

```bash
curl https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```

Or you can try with [httpie](https://httpie.org/):

```bash
http https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```

### Exposing your API using 3Scale

#### Importing API from Openshift

First, let's import our API from Openshift. To do that, just click on `NEW API`.

![](imgs/38.png)

Select `Import from Openshift`. Then choose `fuse` for the `Namespace` combobox and `i-users-api` for the `Name` field. Click on `Create Service`.

![](imgs/39.png)

Now you should see your new api on the 3scale dashboard.

![](imgs/40.png)

#### Creating an application plan for our API

We need to create an application plan for our users api. Click on `Dashboard` menu and then on `i-users-api`

![](imgs/41.png)

Now, click on `Create Application Plan`.

![](imgs/42.png)

For the `Name` field use: `Basic Plan`. And for the `System name`: `basic-plan`. Now click on `Create Application Plan`.

![](imgs/43.png)

We need to publish our application plan. To do that, click on `Publish`

![](imgs/44.png)

#### Creating an application for our API


