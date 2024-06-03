# terraform
Deploy com Terraform, AWS, Docker e Spring Boot.

## O que é o Terraform IaC (Infrastructure as Code)

<p>O Terraform é uma ferramenta de código aberto desenvolvido pela HashiCorp que permite criar, alterar e versionar infraestrutura de forma segura e 
eficiente. Ele é uma ferramenta de orquestração de infraestrutura que permite a criação e gerenciamento de recursos de infraestrutura de forma 
declarativa.</p>

<p>Com o Terraform, é possível criar e gerenciar infraestrutura de forma eficiente, segura e escalável. Ele permite que você defina sua infraestrutura 
na AWS, Azure, Google Cloud Platform, entre outros, usando uma linguagem de configuração simples e fácil de entender.</p>

![Screenshot 2024-05-31 at 3.29.50 PM.png](img%2FScreenshot%202024-05-31%20at%203.29.50%20PM.png)
![Screenshot 2024-05-31 at 3.30.17 PM.png](img%2FScreenshot%202024-05-31%20at%203.30.17%20PM.png)

## Como funciona o Terraform

![Screenshot 2024-05-31 at 3.33.21 PM.png](img%2FScreenshot%202024-05-31%20at%203.33.21%20PM.png)
![Screenshot 2024-05-31 at 3.37.13 PM.png](img%2FScreenshot%202024-05-31%20at%203.37.13%20PM.png)
![Screenshot 2024-05-31 at 3.37.46 PM.png](img%2FScreenshot%202024-05-31%20at%203.37.46%20PM.png)
![Screenshot 2024-05-31 at 3.38.05 PM.png](img%2FScreenshot%202024-05-31%20at%203.38.05%20PM.png)

## Como instalar o Terraform

<p>Para instalar o Terraform, basta acessar o site oficial da ferramenta.</p>

<a href="https://developer.hashicorp.com/terraform/install?product_intent=terraform">Download Terraform</a>

* No MacOS, você pode instalar o Terraform usando o Homebrew:

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

* Confirmar a instalação:

```shell
terraform --version
```

## Criar uma conta na AWS

<p>Para usar o Terraform com a AWS, é necessário criar uma conta na AWS.</p>

<a href="https://aws.amazon.com/pt/console/">Criar conta na AWS</a>

* Deixar selecionado a região como Norte da Virginia (us-east-1).

## Criar uma aplicação Java/Spring Boot

<p>Para criar uma aplicação Java/Spring Boot, basta acessar o site oficial do Spring Initializr.</p>

<a href="https://start.spring.io/">Spring Initializr</a>

* Crie um projeto Maven com as seguintes dependências:

    * Spring Web
    * Java 21
  
* O nome do projeto é deploy (app)

## Publicar a aplicação no Docker Hub (na raiz do projeto)

<p>Para publicar a aplicação no Docker Hub, é necessário criar uma conta no site oficial.</p>

<a href="hhttps://hub.docker.com/">Docker Hub</a>

* Crie um arquivo na raiz do projeto chamado Dockerfile:

* Gere um jar do projeto:

```shell
mvn clean package
```

* Configurando o Dockerfile:

```shell
FROM openjdk:21
WORKDIR app
COPY target/deploy-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* Criar a imagem do Docker:

```shell
docker build . -t mulhermaav/public-api:latest
```

* Faça o login no Docker Hub:

```shell
docker login
```

* Publicar a imagem no Docker Hub:

```shell
docker push mulhermaav/public-api:latest
```

* Executar a imagem do Docker:

```shell
docker run -p 8080:8080 mulhermaav/public-api:latest
```

## Configurar o Terraform

<p>Para configurar o Terraform, é necessário criar um arquivo chamado main.tf na raiz da pasta <b>terraform</b>.</p>

* Configurando o main.tf:

```tf
provider "aws" {
  region = "us-east-1"
}
```

* Inicializar o Terraform:

```shell
cd infra
terraform init
```

<p>É criado uma pasta chamada <b>.terraform</b>, com as configurações do provider AWS.</p>

## Criar usuário com acesso programático na AWS

* Acesse o console da AWS e vá para o serviço IAM.
* Selecione criar usuário (escolha um nome).
* Selecione uma permission policy (AdministratorAccess).
* Após a criação, na tela do usuário, clique em <b>Criar chave de acesso</b>.
* Selecione o caso de uso local code (Programático).
* Após criar a chave de acesso, copie a chave de acesso e a chave secreta (salve em local seguro).
* Instale a AWS CLI:
  * <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">Instalar AWS CLI</a>
    * No MacOS, você pode instalar a AWS CLI usando o Homebrew:

    ```shell
    brew install awscli
    ```
    
    * Confirmar a instalação:

    ```shell
    aws --version
    ```

* Configurar o acesso programático:

```shell
aws configure
```

  * Informe a chave de acesso e a chave secreta.
  * Informe a região (us-east-1).
  * Informe o formato de saída (json).
  * Verificar a configuração:

```shell
aws sts get-caller-identity
```

<p>É criado um arquivo dentro da pasta do usuário chamada <b>.aws</b> e dentro da pasta contém as credenciais.</p>

## Criar uma instância EC2 na AWS

* Configurando o main.tf:

<a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance">Como criar uma instância AWS</a>

* Como saber a AMI (Amazon Machine Image):
  * Acesse o console da AWS e vá para o serviço EC2.
  * Selecione Catálogo de AMIs.
  * Escolha a AMI e copie o ID.

* Configurar o security group (firewall) para permitir acesso via HTTP (saída e entrada).

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "securitygroup" {
    name = "securitygroup"
    description = "Permitir acesso HTTP e acesso a internet"
  
    ingress {
        from_port = 80
        to_port = 80
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"] //defini os ranges de IPs que podem acessar a instância
    }
  
    egress {
      from_port   = 0
      to_port     = 65535
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
}
resource "aws_instance" "servidor" {
  ami = "ami-0632bbd74ce561b38"
  instance_type = "t2.nano"
  vpc_security_group_ids = [aws_security_group.securitygroup.id]
}
```

## Configurar o script de inicialização

<p>Dentro do EC2 sempre que ele é inicializado, ele executa um arquivo de script chamado <b>user_data.sh</b> dentro da pasta infra.</p>

* Cria um arquivo chamado user_data.sh dentro do diretório infra:

```bash
#!/bin/bash
```

* Configurando o main.tf:

```tf
resource "aws_instance" "servidor" {
  ami = "ami-0632bbd74ce561b38"
  instance_type = "t2.nano"
  user_data = file("user_data.sh")
  vpc_security_group_ids = [aws_security_group.securitygroup.id]
}
```

<p>O script do user_data.sh, irá instalar o Docker, baixar a imagem da aplicação e rodar a imagem.</p>

```bash
#!/bin/bash

sudo su
yum update -y
yum install -y docker
service docker start
usermod -a -G docker ec2-user

docker run -p 80:8080 mulhermarav/public-api:latest
```

## Realizar a criação da instância EC2 via Terraform
  
* Dentro da pasta infra:

  * Identifica o arquivo main.tf e informa o que foi detectado e o que ele irá criar:

  ````shell
  terraform plan
  ````

  * Informa as alterações que irá fazer e pede permissão para prosseguir com as aplicações:

  ```shell
  terraform apply
  ```
  
<p>Após aplicação do main.tf, é gerado um arquivo no diretório de infra chamado <b>terraform.tfstate</b>, 
que armazena todo estado dos recursos e faz versionamento dos recursos.</p>
<p>O arquivo <b>terraform.tfstate</b> armazena esses dados para saber o que foi feito, o que não foi feito, para saber o que pode fazer.</p>

* Teste a aplicação acessando o console da AWS e vá para o serviço EC2.
* Selecione a instância criada e copie o IP público.
* Acesse o navegador e cole o IP público da instância.