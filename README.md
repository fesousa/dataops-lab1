# DataOps - Laboratório 1

Laboratório para ambientação no console AWS.
Armazenamento com S3 e notificação com SNS.


## Objetivos

* Ambientação no ambiente e console AWS
* Criar um bucket S3 para armazenar arquivos
* Criar um Tópico SNS para enviar mensagens
* Criar uma assinatura de e-mail para o tópico SNS
* Configurar S3 para enviar uma mensagem para SNS quando incluir ou excluir arquivo

## Arquitetura da solução

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/lab1.png" height='330'/>


## Bucket S3 e notificações SNS

1.	No módulo Learner Lab - Associate Services clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img1.png" height='20'/> para iniciar o ambiente AWS, caso ainda não tenha feio

2.	Espere o status do ambiente ficar verde (<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img2.png" height='20'/> ) e clique nele para abrir o console AWS

    a.	As instruções dos Labs estarão em português. Para alterar o idioma, procure a opção na barra inferior.

3.	No console da AWS, procure pelo serviço S3 na barra pesquisa superior e clique para abrir o serviço

    a.	O Amazon S3 (Amazon Simple Storage Service) é um serviço de armazenamento de objetos com escalabilidade, disponibilidade, segurança e desempenho. A quantidade de dados que pode ser armazenada é virtualmente ilimitada

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img3.png" height='270'/> 

4.	O primeiro passo é criar um bucket para armazenar objetos. 

    a.	Um bucket é um contêiner de objetos. Um objeto é qualquer arquivo ou documento colocado no bucket. 
    b.	Clique no botão <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img4.png" height='22'/>  para criar um novo bucket
5.	Na tela de criação e configuração do novo bucket, preencha os seguintes campos:

&nbsp;&nbsp;&nbsp;&nbsp;    a.	"Nome do bucket": dataops-dados-nomesobrenome
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. Troque "nomesobrenome" pelo seu nome e sobrenome. O nome do bucket deve ser único em toda a AWS, independente da conta e região
   
&nbsp;&nbsp;&nbsp;&nbsp;    b. "Região da AWS": Leste dos EUA (Norte da Virgínia) us-east-1

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. Preste atenção na região. Sempre vamos utilizar essas nos labs

&nbsp;&nbsp;&nbsp;&nbsp;    c. Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img4.png" height='22'/>


6.	Você será redirecionado para a página inicial com a lista dos buckets, na qual verá o bucket criado 

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img5.png" height='70'/>

7.	O próximo passo é criar um tópico SNS

    a.	O Amazon SNS (Amazon Simple Notification Service) é um serviço de mensagens totalmente gerenciado para comunicação entre aplicações e comunicação entre apli-cações e pessoas

    b.	As mensagens são enviadas e armazenadas em um tópico, num sistema de muitos para muitos. Ou seja, uma mensagem pode ser enviada para diversos destinos

    c.	Mensagem são enviadas por um sistema de push para destinos que fazem a assina-tura de um tópico. Um tópico pode ter vários assinantes
    d.	Um assinante pode ser um e-mail, SMS, endpoint HTTP/S, função lambda, fila SQS, entre outros.

8.	No console da AWS, procure pelo serviço SNS na barra pesquisa superior e clique para abrir o serviço


<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img6.png" height='270'/>


9.	Abra o menu lateral clicando em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img7.png" height='22'/>  e escolha a opção <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img8.png" height='22'/>

10.	Clique no botão <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img9.png" height='22'/>

11.	Na tela de criar e configurar o tópico, configure os seguintes campos:

    a.	"Tipo": Padrão 

    b.	"Nome": Topico-Evento-Dados-S3

    c.	Abra a seção "Política de acesso"

    &nbsp;&nbsp;&nbsp;&nbsp; i.	Uma política de acesso define quais as permissões do tópico criado. A permissão padrão define que apenas o proprietário do tópico (conta que criou) pode publicar mensagens. É preciso dar a permissão para o S3 também publicar uma mensagem

    d.	Selecione a opção "Avançado"

    e.	Adicione o seguinte "Statement" antes do fechamento do último colchete (]), separan-do por uma vírgula (,). Os campos em negrito devem ser substituídos pelos valores da sua conta. Sempre que encontrar um campo em negrito nas instruções, lembre-se de trocar por uma configuração do seu ambiente


```json
{
  "Sid": "Statement-id",
  "Effect": "Allow",
  "Principal": { "Service": "s3.amazonaws.com" },
  "Action": "sns:Publish",
  "Resource": "arn:aws:sns:us-east-1:id-conta:Topico-Evento-Dados-S3",
  "Condition": {
    "ArnLike": {
      "aws:SourceArn": "arn:aws:s3:::dataops-dados-nomesobrenome"
    }
  }
}
```

    Troque o nome do bucket (dataops-dados-nomesobrenome) pelo nome do bucket que criou nos passos anteriores e id-conta pelo id da sua conta (disponível na barra superior, ao clicar no nome do usuário – voclabs/user..., campo Minha Conta)

    O documento completo da política deve ficar assim:

```json
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:Publish",
        "SNS:RemovePermission",
        "SNS:SetTopicAttributes",
        "SNS:DeleteTopic",
        "SNS:ListSubscriptionsByTopic",
        "SNS:GetTopicAttributes",
        "SNS:Receive",
        "SNS:AddPermission",
        "SNS:Subscribe"
      ],
      "Resource": "",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": " id-conta"
        }
      }
    },
    {
      "Sid": "Statement-id",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:id-conta:Topico-Evento-Dados-S3",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::dataops-dados-nomesobrenome"
        }
      }
    }
  ]
}

```



<div class="footer">
    &copy; 2022 Fernando Sousa
    <br/>
    
Last update: 2022-02-05 20:53:34
</div>