# Microsserviços com Docker
Considerando 3 máquinas virtuais vamos configurar microsserviço de banco de dados MySQL

### Banco de dados
Instalar MySQL via comando:
```
apt install mysql-server-8.0 -y
```

Acessar MySQL com o comando:
```
mysql -u root -p
```

Criar banco de dados
```
create database "nome do banco de dados";
```

Criar tabela
```
use "nome do banco de dados";
create table "nome da tabela"("campos da tabela");
```

### Microsserviço (página em php)
Criamos um arquivo index.php no diretório
```
'/var/lib.docker/volumes/app/_data'
```
Esse arquivo será o responsável por enviar os dados para o banco de dados

Para rodar o código php no servidor, subimos um container com linux e apache montados com o código dentro.

Exemplo:
```
docker run --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/webdevops/php-apache:alpine-php7
```
onde web-server é o nome do microsserviço gerado
Para matar esse microsserviço utilizamos o comando 
```
docker rm --force web-server
```

### Cluster swarm
Para replicar o microsserviço(escalonar) criamos um Docker swarm
```
docker swarm init
```
Após criar o swarm aparecerá na tela uma mensagem sobre como adicionar outra máquina(worker).

Exemplo:
```
docker swarm join --token SWMTKN-1-4mndsakdas4askporiugjhc43hkjdkvsidfsuy7t5hsdskajkadhs88nkdfbmqvzopx4312zbxbxshqdsbkjasc39 172.31.0.127:2377
```
Esse comando deve ser executado nas outras máquinas que servirão de workers
Para visualizar todos os nodes doswarm utilizamos o comando
```
docker node ls
```

Para replicar os containers em 3 máquinas utilizamos o comando
```
docker service create --name web-server --replicas 3 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7
```

### NFS-Server
Para replicar o volume contido no diretório da aplicação instalamos o nfs-server na máquina líder e instalamos nfs-common nas outras máquinas
Na máquina líder editamos o arquivo de configuração do nfs-server via editor de texto, adicionando a linha
```
/var/lib/docker/volumes/app/_data "IPs das máquinas que serão replicadas"(rw,sync,subtree_check)
```
após o comando
```
exportfs -ar
```

Nas máquinas workers criamos e montamos o diretório exportado
```
mkdir /var/lib/Docker/volumes/app/_data
cd /var/lib/Docker/volumes/app/_data
mount -o v3 172.31.0.127:/var/lib/Docker/volumes/app/_data /var/lib/Docker/volumes/app/_data
```

### Proxy
Criamos um proxy para replicar automaticamente 
```
cd /
mkdir proxy
cd proxy
```
Dentro desse diretório vai o arquivo nginx.config

Após criamos um dockerfile
```
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```
E subimos o container
```
docker build -t proxy-app .
```
```
docker container run --name my-proxy-app -dti -p 4500:4500 proxy-app
```