# Postfix - Para quem tem pressa

A maioria das distribuições linux já disponibiliza um servidor de e-mails pré-instalado e configurado para enviar e receber e-mails localmente. Isso é importante para que o sysadmin seja comunicado a respeito de eventos importantes que ocorrem na máquina local.

No entanto, em pequenas redes internas é comum que sysadmins centralizem o envio e recebimento de e-mails em um único servidor, possibilitando que os demais servidores da rede o utilizem quando precisam alertar o sysadmin à respeito de algum evento importante na rede.

Resumindo, iremos abordar o cenário intermediário dentre os três a seguir:

- **Somente local**: é o que vem pré-configurado na maioria das distribuições Linux para servidores.
- **Interno**: envio/recebimento somente interno, entre máquinas da sua rede interna.
- **Internet**: envio/recebimento entre máquinas/clientes internos/conhecidos e externos/desconhecidos. Nesse caso você não deveria ter pressa.


## Arquivos de Configuração

Por padrão ficam em `/etc/postfix` e os dois mais importantes são `main.cf` e `master.cf`.

- Não use aspas (`"`) nos arquivos de configuração.
- Você pode utilizar variáveis (por ex.: `$xpto`) mesmo antes de defini-las.

É possível verificar o valor atual de uma configuração, mesmo que ela não esteja no `main.cf` através do comando `postconf`:
```bash
postconf inet_interfaces
```
Ou até mesmo atribuir um novo valor:
```bash
postconf -e inet_interfaces=all
```
Sempre que uma configuração for modificada o Postfix deve ser recarregado:
```bash
postfix reload

# ou

systemctl reload postfix
```


## Cenário

Vamos imaginar que estamos configurando o Postfix em uma [VPC](https://aws.amazon.com/vpc) na Amazon Web Services. Essa VPC possui várias subnets e queremos que o servidor de e-mails seja utilizado por qualquer um dos servidores em qualquer uma dessas subnets:

```
endereço da VPC         = 172.30.0.0/16
endereço das subnets    = 172.30.X.0/24
domínio interno         = umbrella.corp
```

### Configurações para um servidor de e-mails somente interno

A seguir estão os parâmetros fundamentais do Postfix, que são utilizados como base para outros parâmetros de configuração.

- **[myorigin](http://www.postfix.org/BASIC_CONFIGURATION_README.html#myorigin)**:
Domínio utilizado nos e-mails enviados.
- **[mydestination](http://www.postfix.org/BASIC_CONFIGURATION_README.html#mydestination)**:
Domínios pelos quais o servidor será responsável por receber e-mail.
- **[relay_from](http://www.postfix.org/BASIC_CONFIGURATION_README.html#relay_from)**:
Para quais clientes o servidor fará relay de e-mails.
- **[relay_to](http://www.postfix.org/BASIC_CONFIGURATION_README.html#relay_to)**:
Para quais destinos o servidor fará relay de e-mails.
- **[relayhost](http://www.postfix.org/BASIC_CONFIGURATION_README.html#relayhost)**:
Método de entrega - direto ou indireto.


Caso o Postifx seja executado em uma interface de rede virtual ou outros sistemas de e-mail estejam rodando em interfaces virtuais, será necessário definir os seguintes parâmetros:

- [myhostname](http://www.postfix.org/BASIC_CONFIGURATION_README.html#myhostname)
- [mydomain](http://www.postfix.org/BASIC_CONFIGURATION_README.html#mydomain)
- [inet_interfaces](http://www.postfix.org/BASIC_CONFIGURATION_README.html#inet_interfaces)

```
myhostname = mail.umbrella.corp
mydomain = umbrella.corp
myorigin = $mydomain
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain, 
    mail.$mydomain, www.$mydomain, ftp.$mydomain
inet_interfaces = all
mynetworks = 172.30.0.0/16
```

---

Um host remoto que se conecte a porta 25 e tente enviar e-mail forjando remetente irá conseguir?
Se o SG/firewall permitir conexão externa na porta 25. Apesar da instancia só ter interface eth0 com IP da subnet que estiver. Os pacotes chegam nela com src ip do host de origem (ip publico). Sendo assim o parametro mynetworks seria suficiente para barrar conexão que não tenham origem na sua rede de servidores.

relay_host, relay_to e relay_from não encontrados.

>Ao enviar e-mail para "root", como é um rcpt incomplete o servidor de e-mails irá completar com o myorigin, myhostname/mydomain. Entregando para o usuário local.

> Por outro lado se o e-mail for enviado para root@umbrella.corp, esse domínio não faz parte de `mydestination`, então o servidor envia para fora via `relayhost.`

**!!** Dessa forma, todos os servidores deve ter a opção `relayhost` definida para o servidor de e-mails interno.

## TL;DR

Quando estiver configurando um servidor de e-mails, estas são as principais perguntas que você deve ter em mente e os parâmetros de configuração relacionados.

### Qual domínio utilizar ao enviar e-mails?

O parâmetro [myorigin](http://www.postfix.org/BASIC_CONFIGURATION_README.html#myorigin) especifica o domínio que irá aparecer nos e-mails enviados pelo servidor. O padrão é o _hostname_ do servidor. A não ser que esteja rodando um ambiente realmente pequeno, você vai querer alterar esse parâmetro para `$mydomain`, que por padrão armazena o domínio do servidor.

Por questões de consistência entre endereços de remetente e destinatário, `myorigin` também especifica o nome de domínio que será anexado em caso de endereços incompletos (_unqualified recipient address_).

```
/etc/postfix/main.cf:
    myorigin = $myhostname (default: envia e-amil como "user@$myhostname")
    myorigin = $mydomain   (geralmente desejável: "user@$mydomain")
```


### Por quais domínios receber e-mails?

O padrâmetro [mydestination](http://www.postfix.org/BASIC_CONFIGURATION_README.html#mydestination) especifica para quais domínios o servidor será responsável por entregar e-mails localmente, ao invés de repassar a outro servidor. O padrão é receber e-mails endereçados a própria máquina somente. 

Esse parâmetro aceita múltiplos formatos:
- `example.com` - nomes de domínios
- `/file/name` - arquivo
- `type:table` - consulta em tabelas hash, btree, nis ldap, mysql,...

**IMPORTANTE**: se este servidor de e-mails for responsável pelo envio e recebimento de e-mails do domínio e não somente local, a variável `$mydomain` deve estar presente nessa opção.

Exemplo 1: padrão
```
/etc/postfix/main.cf:
    mydestination = $myhostname localhost.$mydomain localhost
```

Exemplo 2: domínio
```
/etc/postfix/main.cf:
    mydestination = $myhostname localhost.$mydomain localhost $mydomain
```

Exemplo 3: caso o servidor tenha múltiplos nomes (registros `DNS A`)
```
/etc/postfix/main.cf:
    mydestination = $myhostname localhost.$mydomain localhost 
        www.$mydomain ftp.$mydomain
```

**Cuidado**: para evitar loops, você deve listar todos os hostnames do servidor, incluindo `$myhostname` e `localhost.$mydomain`


### Para quais clientes fazer relay?

Por padrão o Postfix irá enviar e-mails de clientes dos endereços de rede autorizados para qualquer destino. As redes autorizadas são definidas através do parâmetro [mynetworks](http://www.postfix.org/postconf.5.html#mynetworks). O padrão é autorizar somente a máquina local. Antes do Postfix 3, o padrão era autorizar todos os clientes que estiverem na mesma subnet do servidor.

O Postfix também pode ser configurado para enviar e-mails de clientes "mobile", clientes autorizados que estão fora das redes autorizadas. Isso é explicado em [SASL README](http://www.postfix.org/SASL_README.html) e [TLS README](http://www.postfix.org/TLS_README.html).

**IMPORTANTE**: caso seu servidor tenha uma interface de rede conectada diretamente a uma WAN, o padrão para `mynetworks` pode ser amigável demais.

Exemplos (use somente um dos seguintes):
```
/etc/postfix/main.cf:
    mynetworks_style = subnet  (default: authorize subnetworks)
    mynetworks_style = host    (safe: authorize local machine only)
    mynetworks = 127.0.0.0/8   (safe: authorize local machine only)
    mynetworks = 127.0.0.0/8 168.100.189.2/32 (authorize local machine)
```

Caso `mynetworks` seja utilizado, o Postfix irá ignorar o parâmetro `mynetworks_style`.


### Para quais destinos fazer relay?

Por padrão o Postfix somente repassa e-mails de clientes fora das redes autorizadas para clientes remotos autorizados. Clientes remotos autorizados são definidos em `relay_domains`. O padrão é autorizar todos os domínios e subdomínios dos que estiverem listados em `mydestination`.

Exemplo (use somente um dos seguintes):
```
/etc/postfix/main.cf:
    relay_domains = $mydestination (default)
    relay_domains =           (safe: never forward mail from strangers)
    relay_domains = $mydomain (forward mail to my domain and subdomains)
```

## Método de entrega - direto ou indireto

Por padrão o Postfix tenta entregar o e-mail diretamente. Mas dependendo das condições locais do seu servidor talvez isso não seja possível ou desejável. Seu servidor talvez tenha que ser desligado fora de expediente, pode estar atrás de um firewall ou pode estar conectado a um provedor que não permite o envio de e-mails para a Internet. Nesses casos você precisa configurar o Postfix para enviar e-mails indieretamente através de um [relay host]().

Exemplos (escolha um):
```
/etc/postfix/main.cf:
    relayhost =                   (default: direct delivery to Internet)
    relayhost = $mydomain         (deliver via local mailhub)
    relayhost = [mail.$mydomain]  (deliver via local mailhub)
    relayhost = [mail.isp.tld]    (deliver via provider mailhub)
```

O uso de `[]` elimina consultas `DNS MX`.


### Quais problemas reportar ao postmaster?

É recomendável redirecionar os e-mails enviados ao postmaster para a conta do sysadmin, assim como os e-mails do usuário root.
Para fazer isso basta editar o `/etc/aliases`:

```
/etc/aliases:
    postmaster: you
    root: you
```

Sempre execute `newaliases` após editar esse arquivo.
Pode ser que seu arquivo de alias esteja localizado em outro lugar. Use o comando `postconf alias_maps` para verificar.

O Postfix reporta seus problemas ao alias postmaster. Mas talvez você não esteja interessado em todos os tipos de relatórios. Por padrão apenas problemas graves `resource` e `software` são reportados:

```
/etc/postfix/main.cf:
    notify_classes = resource, software
```

Você pode encontrar mais informações sobre outras classes em [notify classes](http://www.postfix.org/postconf.5.html#notify_classes).


### O que você precisa saber sobre os logs do Postfix

O daemon do Postfix roda em background e registra tanto os problemas como as atividades normais. O `rsyslogd` (usado no CentOS 7/Redhat 7) registra esses eventos em arquivos de log, organizando por classe e severidade. As configurações do rsyslogd ficam em `/etc/rsyslog.conf` ou `/etc/rsyslog.d/`. Por padrão todos os logs da classe `mail` são enviados para o arquivo `/var/log/maillog`:

```
/etc/rsyslog.conf:
    # Log all the mail messages in one place.
    mail.*                                                  -/var/log/maillog
```

**IMPORTANTE**: no linux você deve colocar um `-` (hífen) antes do nome do arquivo para desativar a chamada do sistema a função `sync()` após cada linha ser registrada no arquivo. Do contrário, o rsyslog irá consumir mais recursos que o próprio Postfix.

