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

### Instalando PEAR

Conjunto de funcionalidades PHP.

```
sudo apt-get install php-pear
```

### Instalando PHP5-curl

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

Você pode utilizar um arquivo já default ou criar um novo, o imporantante é que todos tenham a mesma estrutura e fique no mesmo diretório.

No exemplo vou editar o arquivo:

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

Pronto agora basta reiniciar o apache:

```
sudo service apache2 restart
```

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

* HOST: ftp.IP-DO-SEU-DROPLET
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

É isso pessoal, espero que eu possa ter ajudado em algo.

Este repositório está aberto a contribuições ;)
