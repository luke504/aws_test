# Schema Architettura Infrastruttura per un applicazione Wordpress

## Introduzione
In questa guida, spiegherò come distribuire un nuovo sito Web WordPress sul cloud utilizzando Amazon Web Services (AWS). Utilizzeremo AWS Elastic Beanstalk per creare un ambiente scalabile e tollerante ai guasti per il nostro sito WordPress.

### Diagramma Architettura
                  +-----------------+
                  | Elastic Load    |
                  | Balancer        |
                  |                 |
                  +-----------------+
                             |
                             |
                             |
    +----------------+   +----------------+   +----------------+
    | EC2 instance 1 |   | EC2 instance 2 |   | EC2 instance 3 |
    |                |   |                |   |                |
    +----------------+   +----------------+   +----------------+
          |                    |                   |
          |                    |                   |
          |                    |                   |
    +------------------------------------+     +----------------+
    |            Auto Scaling group       |     |     RDS        |
    |                                    |     |  (MySQL database)|
    +------------------------------------+     +----------------+
                    |
                    |
                    |
           +------------------+
           | Elastic Beanstalk|
           |                  |
           +------------------+

In questo diagramma:
- l'Elastic Load Balancer distribuisce il traffico tra le istanze EC2, che ospitano il sito WordPress (istanza 1 e 2).
    - L'Auto Scaling Group monitora la domanda di traffico e aumenta o diminuisce il numero di istanze EC2 in base ad essa. 
    - Elastic Beanstalk gestisce la configurazione e il rilascio delle istanze EC2, l'aggiornamento del software e il bilanciamento del carico.
- Il database MySQL è fornito da Amazon RDS (istanza 3). 


#### Prerequisiti
- AWS Account (se non ne hai uno, crea un nuovo account)
- AWS CLI (installa AWS CLI sulla tua macchina locale)
- Git (installa Git sulla tua macchina locale)

#### Passi:
<ol>
<li>Clona la repository WordPress <br> 
    <code>git clone https://github.com/WordPress/WordPress.git</code>
</li>
<li>Crea un nuovo ambiente Elastic Beanstalk <br>
    <code>aws elasticbeanstalk create-environment --application-name WordPress --environment-name wordpress-env --solution-stack-name "64bit Amazon Linux 2 v5.4.7 running PHP 7.4" --option-settings Namespace=aws:autoscaling:launchconfiguration,InstanceType=t3.small,MinSize=2,MaxSize=4 --option-settings Namespace=aws:ec2:vpc,Subnets=your-subnet-id --option-settings Namespace=aws:ec2:vpc,VPCId=your-vpc-id --option-settings Namespace=aws:elasticbeanstalk:environment,EnvironmentType=LoadBalanced
</code>
Qui stiamo creando il nuovo ambiente dove poter deployare il nostro sito web. Stiamo utilizzando PHP version 7.4 e abbiamo impostato altri parametri come l'auto scalabilità in modo da renderlo più veloce ed efficiente.
</li>
<li>Crea una nuova instanza RDS <br>
    <code>aws rds create-db-instance --db-instance-identifier wordpress-db --db-instance-class db.t2.micro --engine MySQL --master-username admin --master-user-password password --allocated-storage 10
    </code><br>
    Abbiamo creato l'istanza RDS per avere il nostro database relazionale.
</li>
4. Oltre ad aver creato l'istanza, creiamo il nuovo database sull'istanza: <br>
    <code>aws rds create-db-instance --db-instance-identifier wordpress-db --db-instance-class db.t2.micro --engine MySQL --master-username admin --master-user-password password --allocated-storage 10</code>
    <br> Il driver che stiamo utilizzando è MySQL.
<li>Aggiorniamo il file di configurazione di WordPress (wp-config.php) con l'endpoint RDS, il nome del database, il nome utente e la password. <br>
<code>define('DB_NAME', 'your_database_name_here');<br>
    define('DB_USER', 'your_username_here');<br>
    define('DB_PASSWORD', 'your_password_here');<br>
    define('DB_HOST', 'your_rds_endpoint_here');
</code>
</li>
<li>Esegui il commit delle modifiche ed esegui il push del codice nell'ambiente Elastic Beanstalk.</li>
<code>git add . <br>
git commit -m "Deploy WordPress on AWS Elastic Beanstalk" <br>
git aws.push</code>
<li>Attendiamo che l'ambiente Elastic Beanstalk faccia il deploy dell'applicazione.
</li>
<li>Accediamo al sito WordPress utilizzando l'URL dell'ambiente Elastic Beanstalk.
</li>
</ol>

### Conclusione
In questa guida, abbiamo distribuito con successo un sito WordPress su AWS Elastic Beanstalk utilizzando un'istanza RDS per il database. La soluzione è scalabile, tollerante ai guasti, sicura e adattabile al carico medio. Si può personalizzare ulteriormente l'ambiente aggiungendo certificati SSL, CDN e altre funzionalità per migliorare le prestazioni e la sicurezza del sito web.
