# Applications Devops
Essa configuração de infraestrutura feita em arquivos de docker-compose permite a disponibilização de aplicações seguindo as melhores práticas de infraestrutura e devops.
A base dessa configuração é um proxy reverso utilizando o traefik, um registry para armazenamento e distribuição de imagens docker e o portainer que permite o fácil gerenciamento de containers.

Uma dessas praticas é a utilização de um proxy como camada de entrada da internet. Essa é a responsabilidade do traefik. Ele recebe todas as requisições da internet e delega para os container respectivos normalmente seguindo regras de url.
Além disso o traefik também possui loadbalance, service discovery para os container e um configuração que gerar certificados ssl atravez do letsencrypt de forma automática.

Já o portainer é uma interface visual para gerências containers do docker, o qual permite visualizar os container e várias informações sobre eles.

## Start para fazer um teste local
Se você desejar fazer um teste local em sua maquina é necessário fazer algumas configurações.
Essa teste ira ajudar você a entender o fluxo e funcionamento dessa configuração de infraestrutura.

1. Adicione a seguinte regra no seu arquivo de hosts(no linux é o /etc/hosts)
  * 127.0.0.1	registry.localhost portainer.localhost meuapp.localhost
2. Rode o comando 'docker-compose -f infra/docker-compose.yml up -d' para iniciar a stack de infra
3. Acesse o localhost:8080 para ver a interface gráfica do traefik.
  * Você ira ver 2 backends e 2 frontends, basicamente é a regra e entrada e pra qual serviço ela redireciona.
4. Acesse o portainer.localhost para acessar o portainer com usuário 'admin' e senha 'test'.
  * Vai aparecer uma paginá com erro de segurança. Isso é porque o traefik está redirecionando para https mas não consegue gerar certificado local.
5. Faça login no registry local com o serguinte comando: docker login registry.localhost
  * Usuário: testuser Senha: testpassword
6. Rode o comando 'docker-compose -f app/docker-compose.yml up -d' para iniciar a stack de apps
7. Veja o app na interface do traefik(localhost:8080) e acesse ele no meuapp.localhost


## Configurando autenticação do registry
Rode o seguinte comando na raiz deste diretório para criar um novo usuário e senha para acessar seu registry.
```
docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```

## Configurando autenticação do portainer
Para gerar um novo password para o portainer rode o seguinte comando substituindo o <password> por sua senha:
```
docker run --rm httpd:2.4-alpine htpasswd -nbB admin <password> | cut -d ":" -f 2
```
Abra o docker-compose de infra, no serviço do portainer altere o parâmetro --admin-password.


Lembre-se de escapar todos os $ com outro $ no password gerado. Ex: $2y$05 vai ficar $$2y$$05
