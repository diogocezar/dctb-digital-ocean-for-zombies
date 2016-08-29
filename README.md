# Por que isso?

Nas minhas andanças pelas web, vejo muita gente reclamando de servidores que acabam caindo, não respondem como o esperado e tudo mais.

Mesmo no brasil, a média de um servidor para hospedar sua página é de R$ 20,00. Um tanto quanto salgado né?

Eu já utilizo o Digital Ocean a dois anos, para hospedar os meus projetos pessoais e demonstrações dos meus freelas, e não tenho do que reclamar.

Claro, existem algumas desvantagens, mas vou procurar explicar aqui tudo certinho ;)

# O que vou precisar?

A princípio basta ter um cartão de crédito internacional, pois como o serviço é gringo, é uma forma bacana de efetuar os pagamentos, além disso eles também disponibilizam o pagamento via PayPal ;)

Bom, não tem muito segredo basta criar uma conta e pronto!

# Qual droplet devo escolher?

Minha recomendação é começar com calma, escolha um ubuntu com uma versão estável e faça os testes iniciais para projetos pequenos.

A escolha do Droplet na versão mais barata, te dará uma maquina de 512MB de memória e 20GB de HD. Acho que é o suficiente para começar.

# Como faço para acessar?

Logo que você cadastrar, você receberá em seu email a senha de root e o ip do seu Droplet.

Para acessar estando no windows, recomendo o programa Putty:

http://www.putty.org/

Se estiver no linux basta fazer um acesso por SSH (se vc está no linux imagino que saiba fazer isso)

Acessando o seu droplet pela primeira vez ele pedirá para alterar a senha de ROOT. E pois bem, faça isso pois é importante lembrar.

Outra recomendação é anotar todas as senhas que serão pedidas nos próximos passos :D

# Configuração Inicial

Não se assute meu amigo, a vida de programador é 80% olhando a tela preta do terminal, se você não está fazendo isso reveja seus conceitos :P

Mas vamos lá, recomendo que você crie um usuário que será o usuário de acesso, não é bacana ficar acessando tudo como root.

Então:

```
adduser zombie
```

E depois altere as permissões deste usuário no sistema (vou assumir que você sabe usar o vi ou o nano)

```
visudo
```

Adicione a linha:

```
zombie    ALL=(ALL:ALL) ALL
```

Agora é bacana que vc deslogue do root e logue com o novo usuário e senha que acabou de criar.

# Instalando o Lamp

Para quem ainda não sabe, Lamp é Linux + Apache + MySQL + PHP ;)

Instalar cada um deles separado pode dar um trabalho bacana, mas se quiser se aventurar tudo bem.

Aqui vou utilizar o tasksel que é um assistente bem bacana.

Para instalá-lo:

```
sudo apt-get install tasksel
```

depois basta executar:

```
sudo tasksel
```

Ele possui um assistente gráfico, você não terá dificuldades para terminar a instalação do LAMP.

Depois disso você já poderá acessar o seu IP e verá que seu servidor está funcionando ;)

## Configurações adicionais

Antes de seguir vamos fazer algumas configurações adicionais que podem ser opcionais dependendo do seu projeto

### Habilitando o MOD_REWRITE

Permite que você reescreva e reaponte as urls do seu projeto, por exemplo apontar de www.seusite.com.br/home -> www.seusite.com.br/index.php

```
sudo a2enmod rewrite
```

### Instalando PHPMyAdmin

Permite que você acesse e controle seus bancos de dados MySQL:

```
sudo apt-get install phpmyadmin
```

### Instalando PEAR (Opcional)

Conjunto de funcionalidades PHP.

```
sudo apt-get install php-pear
```

### Instalando PHP5-curl (Opcional)

Permite o acesso a urls externas através do comando curl no PHP.

```
sudo apt-get install php5-curl
```

# Configurando o apache

Agora que já temos o ambiente pronto, vamos entender um pouco como funciona o apache.

O servidor web apache será o responsável por fazer as rotas de DNS que chegarão ao seu servidor.

Então é nele que precisamos dizer que o site www.zombie1.com.br irá apontar para a sua pastar /var/www/zombie1 e o site www.zombie2.com.br irá apontar para /var/www/zombie2

Antes de tudo vamos fazer uma configuração geral para permitir que os nomes dos arquivos possam ser sobrescritos pelo MOD Rewrite:

```
sudo nano /etc/apache2/apache2.conf
```

Procure por

```
<Directory /var/www/>
```

E altere:

```
AllowOverride All
```

Ficará algo como:

````
<Directory /var/www/>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
```

Pronto, agora podemos editar o arquivo que irá fazer todo o nosso roteamento.

Aqui temos 2 formas de se fazer isso, da forma rápida e fácil e da forma correta.

## Forma Rápida e Fácil:

```
sudo nano /etc/apache2/sites-available/000-default.conf
```

É também nesse aquivo que você poderá mudar a rota padrão do seu ip, quando nenhum domínio estiver apontado para ela.

Mas bem, vamos ao que precisamos incluir neste arquivo.

No final dele inclua algo como:

```
<VirtualHost *:80>
  ServerName zombie1.com.br
  ServerAlias zombie1.com.br www.zombie1.com.br
  DocumentRoot /var/www/zombie1
</VirtualHost>

<VirtualHost *:80>
  ServerName zombie2.com.br
  ServerAlias zombie2.com.br www.zombie2.com.br
  DocumentRoot /var/www/zombie1
</VirtualHost>
```

Desta forma, você terá configurado 2 sites em seu servidor.

Sempre que houver a necessidade de adicionar algum novo site, basta incluir alguém novo alí ;)

## Forma Correta:

Crie o primeiro arquivo de Virtual Host

Começe copiando o arquivo para o primeiro domínio:

```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/zombie1.com.conf
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/zombie2.com.conf
```

Abra o novo arquivo em seu editor com privilégios de root:

```
sudo nano /etc/apache2/sites-available/zombie1.com.conf
```

O arquivo será algo parecido com isso (eu removi os comentários aqui para tornar o arquivo mais acessível):

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Deixe com algo parecido com:

```
<VirtualHost *:80>
  ServerName zombie1.com.br
  ServerAlias zombie1.com.br www.zombie1.com.br
  DocumentRoot /var/www/zombie1
</VirtualHost>
```

Copie o primeiro Virtual Host e personalize-o para o Segundo Domínio (zombie2).

### Ative os novos arquivos de Virtual Host

```
sudo a2ensite www.zombie1.com.br.conf
sudo a2ensite www.zombie2.com.br.conf
```

Quando terminar, você precisará reiniciar o Apache para fazer com que estas alterações tenham efeito:

```
sudo service apache2 restart
```

# Alterando o timezone

Como o servidor não está no Brasil, precisamos informar que nosso servidor deve seguir o nosso horário, para isso:

```
sudo dpkg-reconfigure tzdata
```

Selecione ```América / São Paulo```

E estamos quase terminando, basta fazer as configurações no Digital Ocean.

# Configurando os DNS's

No seu painel do Digital Ocean, irá achar uma seção na qual você pode incluir regras de DNS.

Em networking vá até Domains.

Inclua seus novos domínios:

```
zombie1.com.br -> ip do seu droplet
zombie2.com.br -> ip do seu droplet
```

Depois disso inclua 2 regras de CNAME para cada um deles

```
Name : *
Hostname zombie1.com.br.
```

```
Name : www
Hostname zombie1.com.br.
```

Faça isso para todos os sites que você deseja apontar para o seu servidor.

Agora basta que você aponte o DNS do seu site para o digital ocean.

No site que você registrou zombie1.com.br e zombie2.com.br

Aponte os dns para:

```
ns1.digitalocean.com.
ns2.digitalocean.com.
ns2.digitalocean.com.
```

# Como faço para enviar os arquivos?

Bom, vou ensinar aqui 2 maneira de fazer isso:

## Configurando o FTP

Para isso vamos instalar e configurar o proftpd:

```
sudo apt-get install proftpd
sudo nano /etc/proftpd/proftpd.conf
```

Procure, descomente e altere para:

```
DefaultRoot /var/www/
```

```
sudo adduser zombie www-data
sudo chown -R www-data:www-data /var/www
sudo chmod -R g+rw /var/www
sudo service proftpd restart
```

Com isso você já conseguirá acessar por ftp o seu servidor na pasta /www

As informações são:

* HOST: IP-DO-SEU-DROPLET
* USER: zombie
* PASS: SUA-SENHA

## Instalando o GIT

É a opção que eu mais acho bacana, com ela você consegue ter a versão em produção do seu site direto no seu servidor.

Para isso basta instalar o GIT

```
sudo apt-get install git
```

E ai vou assumir que você sabe usar o git, ou caso contrário, envie por FTP ;)

---

Pronto! Seu novo servidor está configurado, com apenas U$ 5,00 por mês!

Não é um caminho fácil, mas garanto que vale muito a pena.

# Nem tudo são 1.000 maravilhas

Eu tive muita dificuldade em tentar configurar um seridor de email, principalmente porque o Digital Ocean bloqueia algumas portas para o envio e tudo mais, sei que existem formas de se fazer, mas recomendo altamente que os emails sejam colocados fora do Digital Ocean se você está com uma instância basicona como esta do exemplo.

Mas aproveito para recomendar o Zoho Mail, um serviço que é gratis para até 10 usuários ;)

Vale bastante a pena:

https://www.zoho.com/mail/

Além disso, também não é tão simples enviar email por lá.

Então recomendo a utilização do SendGrid:

https://sendgrid.com/

Que também tem uma versão free e tem até plugin para o WordPress.

Se você quiser mesmo enviar emails pelo servidor, então recomendo o seguinte:

1. Abra um ticket pedindo para liberarem as portas de conexão a SMTP;
2. Você precisará enviar algumas informações pessoais, por causa da politica Anti-Spam do Digital Ocean;
3. Com as portas liberadas, acesse sua conta no Gmail e habilite a opção de acesso a aplicativos menos segudos.
4. Baixe um PHPMailer, e faça um exemplo de teste com as configurações para envio pelo GMAIL:

```
<?php
	require './vendor/autoload.php';

	$mail = new PHPMailer;

	$mail->SMTPDebug = 4;

	$mail->isSMTP();
	$mail->Host = 'smtp.gmail.com';
	$mail->SMTPAuth = true;
	$mail->Username = 'zombie@gmail.com';
	$mail->Password = '***';
	$mail->SMTPSecure = 'tls';
	$mail->Port = 587;

	$mail->setFrom('zombie@gmail.com', 'Zombie');
	$mail->addAddress('zombie@gmail.com', 'Zombie');

	$mail->isHTML(true);

	$mail->Subject = 'Here is the subject';
	$mail->Body    = 'This is the HTML message body <b>in bold!</b>';
	$mail->AltBody = 'This is the body in plain text for non-HTML mail clients';

	if(!$mail->send()) {
	    echo 'Message could not be sent.';
	    echo 'Mailer Error: ' . $mail->ErrorInfo;
	} else {
	    echo 'Message has been sent';
	}
?>
```

Quando você executar o script, o gmail irá lhe informar que estão com a sua senha, afinal o seu droplet não está no mesmo país que você. Permita a ação e então os próximos acessos estarão liberados.

# Precisa de mais memória? Habilite o Swap

https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04

Ai tem um tutorial bacana em inglês explicando tudo certinho, mas caso queira fazer da forma fácil e rápida basta seguir:

```
sudo swapon -s
```

Nada deverá aparecer. Ou algo como:

```
Filename                Type        Size    Used    Priority
```

```
free -m
```

```
             total       used       free     shared    buffers     cached
Mem:          3953        154       3799          0          8         83
-/+ buffers/cache:         62       3890
Swap:            0          0          0
```

Então siga com:

```
sudo fallocate -l 4G /swapfile
ls -lh /swapfile
```

```
-rw-r--r-- 1 root root 4.0G Apr 28 17:19 /swapfile
```

```
sudo chmod 600 /swapfile
ls -lh /swapfile
```

```
-rw------- 1 root root 4.0G Apr 28 17:19 /swapfile
```

```
sudo mkswap /swapfile
```

```
Setting up swapspace version 1, size = 4194300 KiB
no label, UUID=e2f1e9cf-c0a9-4ed4-b8ab-714b8a7d6944
```

```
sudo swapon /swapfile
sudo swapon -s
```

```
Filename                Type        Size    Used    Priority
/swapfile               file        4194300 0       -1
```

```
free -m
```

``` 
             total       used       free     shared    buffers     cached
Mem:          3953        101       3851          0          5         30
-/+ buffers/cache:         66       3887
Swap:         4095          0       4095
```

```
sudo nano /etc/fstab
```

Adicione ao final:

```
/swapfile   none    swap    sw    0   0
```

## Veio com PHP 7.0? Quer uma versão anterior? Deixe os 2 habilitados ;)

Caso queiram utilizar uma distribuição que venha com o PHP7.0 instalado por default, você pode deixar seu servidor dual PHP, deixando o 7 e também o 5.X.

Para isso:

```
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install php7.0 php5.6 php5.6-mysql php-gettext php5.6-mbstring php-xdebug libapache2-mod-php5.6 libapache2-mod-php7.0
```

Ai então para trocar de versão (Apache e PHP):

Do php5.6 para php7.0 :

Apache:

```
sudo a2dismod php5.6 ; sudo a2enmod php7.0 ; sudo service apache2 restart
sudo ln -sfn /usr/bin/php7.0 /etc/alternatives/php
```

Do php7.0 para php5.6 :

```
sudo a2dismod php7.0 ; sudo a2enmod php5.6 ; sudo service apache2 restart
sudo ln -sfn /usr/bin/php5.6 /etc/alternatives/php
```

Provavelmente você irá precisar reinstalar alguns módulos para o php 5.6, então:

```
sudo apt-get install php5.6-gd
sudo apt-get install php5.6-xml
sudo apt-get install php5.6-curl
```


É isso pessoal, espero que eu possa ter ajudado em algo.

Este repositório está aberto a contribuições ;)
