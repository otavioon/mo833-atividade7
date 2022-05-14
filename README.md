# Atividade 7: Semana 17/05

## Objetivo

O objetivo desta atividade é aprender a implantar uma aplicação e gerenciá-la, de forma programática, na nuvem computacional da AWS. 

## Descrição

Inicialmente, você deve ler o capítulo de livro intitulado "Deploying and Configuring Infrastructure". 

Nesta atividade, você deve desenvolver um *playbook* Ansible que faça a implantação e execução da [DCGAN](https://github.com/otavioon/Distributed-DCGAN.git) em uma máquina virtual da nuvem computacional. 
Em suma, o *playbook* Ansible desenvolvido deverá realizar as seguintes operações:
1. Instalar as aplicações necessárias para execução da [DCGAN](https://github.com/otavioon/Distributed-DCGAN.git);
2. Clonar o repositório da [DCGAN](https://github.com/otavioon/Distributed-DCGAN.git);
3. Executar a DCGAN por 1 época, utilizando os dados de teste do MNIST, salvando a saída (padrão) emitida pela aplicação em um arquivo de saída ``output.txt`` e;
4. Copiar o arquivo de saída da máquina remota para a máquina local.

Para execução do seu *playbook* Ansible uma infraestrutura básica é disponibilizada e os detalhes são descritos na seção seguinte.

## Implantação da Aplicação em uma Máquina Virtual da AWS

Esta seção descreve os requisitos e detalhes da infraestrutura para execução do seu *playbook* Ansible. 
Utilizaremos o termo "máquina local" para designar a máquina que será utilizada para realização da atividade e execução do Ansible e o termo "máquina remota" para designar a máquina virtual na nuvem computacional na qual a aplicação será executada. 

### Requisitos

Para execução da atividade, você precisará de:
1. Instanciar máquina virtual na AWS utilizando o AWS *Web Console*, semelhantemente a atividade anterior;
2. Seu identificador da chave de acesso e identificador da shave de acesso secreta (`Access Key ID` e `Secret access key`, respectivamente). Estes foram enviados por e-mail;
3. Um par de chaves (publica e privada) registrados na AWS, bem quanto o nome deste par de chaves registrado (`KEYPAIR_NAME`). Este par de chaves é o mesmo utilizado para acessar uma máquina virtual por SSH.;
4. A aplicação Docker instalada na máquina local para execução do Ansible.

A máquina virtual na AWS (item 1) deve ser instanciada através do *AWS Web Console* com a seguinte configuração:

```
Tipo de instancia: t2.medium
Keypair: O KEYPAIR_NAME indicado no item 3
Volume: EBS, de 16 Gb, montado em /dev/sda1
Imagem: Ubuntu 20.04 Server, com AMI-ID: ami-0c4f7023847b90238
```

Uma vez instanciada, IP público (`public_ip_address`) ou DNS da máquina virtual, deve ser anotado. Este pode ser encontrado na descrição da máquina virtual, no AWS  *Web Console*, no campo `Auto-assigned IP address` ou `Public IPv4 DNS`.

### Infraestrutura para Execução do *Playbook*

O presente repositório deve ser clonado e executado apenas na máquina local. 
Este conta com os seguintes arquivos para execução do seu *playbook* Ansible:

- ``build_docker.sh``: *Script* para gerar a imagem Docker, contendo Ansible e boto (cliente da AWS), na máquina local. A imagem gerada é chamada `mo833-ansible`. Este arquivo não deve ser alterado;
- ``configure.yaml``: *Playbook* Ansible que deve ser preenchido com *tasks* necessárias para completar a atividade;
- ``Dockerfile``: Arquivo de descrição da imagem Docker, utilizado pelo script `build_docker.sh`. Este arquivo não deve ser alterado;
- ``inventory.yaml``: Arquivo de inventorio contendo a máquina remota que será gerenciada pelo Ansible;
- ``run_configure.sh``: *Script* para executar seu playbook. Este arquivo não deve ser alterado;
- ``vars.sh``: *Script* conténdo váriaveis globais de ambiente utilizadas para diversas finalidades. Nota: as alterações neste *script* **NÃO** devem ser submetidas ao git.

Primeiramente, para preparação do ambiente, os seguintes passos devem ser realizados:

1. Instale o docker na máquina local e gere a imagem Docker executando o *script* `build_docker.sh`. Este passo pode ser realizado apenas uma vez.

2. Altere o arquivo `inventory.yaml` e, no campo indicado (`ansible_host`), substitua o marcador de exemplo (`XXXXX`) pelo IP público (`public_ip_address`) ou DNS da máquina remota, recém anotado. Note que, caso a máquina desligue, este IP pode ser alterado e deve ser modificado. Além disso, substitua apenas os campos indicados.

3. Substitua os valores dos campos indicados no arquivo `vars.sh` com seus devidos valores. Os campos a serem substituidos são: `AWS_ACCESS_KEY_ID`, com seu respectivo *Access key ID*, enviado a si por e-mail; `AWS_SECRET_ACCESS_KEY`, com seu respectivo *Secret access key*, enviado a si por e-mail e; `KEYPAIR_NAME`, com o nome do par de chaves, registrado na AWS. Além disso, substitua apenas os campos indicados.

4. Copie sua chave privada, utilizada para acessar as máquinas da AWS, para este diretório e renomei-a para `key.pem`.

Após realizado os passos acima, o *script* `run_configure.sh` pode ser executado. 
Este *script* executa o *playbook* Ansible `configure.yaml` nas máquinas descritas no arquivo de inventório `inventory.yaml`.

### Execução da DCGAN

Para esta atividade, você deve alterar o arquivo `configure.yaml` e, no local indicado, adicionar *tasks* para:

1. Instalar as aplicações necessárias para execução da [DCGAN](https://github.com/otavioon/Distributed-DCGAN.git), como os pacotes `docker.io` e `python3-docker`;
2. Clonar o repositório da [DCGAN](https://github.com/otavioon/Distributed-DCGAN.git);
3. Criar a imagem Docker da DCGAN (`dist_dcgan`), conforme o manual de uso da mesma (`docker build`);
4. Executar a DCGAN por 1 época, utilizando os dados de teste do MNIST, e salvando a saída (padrão) emitida pela aplicação em um arquivo de saída ``output.txt``. A linha de comando utilizada para execução da aplicação e redirecionamento da saída para o arquivo `output.txt` é mostrada a seguir (pode ser alterada caso deseje):
```
docker run --rm -e HOMEDIR=$(pwd) -w $(pwd) -v=$(pwd):$(pwd) dist_dcgan:latest python -m torch.distributed.launch dist_dcgan.py --dataset mnist --dataroot data --download --image_size 64 --batch_size 128 --out_folder output --test_data --num_epochs 1 --max_workers 1 >> output.tx
```
4. Copiar o arquivo de saída produzido na máquina remota (`output.txt`) para a máquina local, dentro de um direrório chamado `resultados` (podem haver mais subdiretórios dentro deste, caso deseje).

## Entrega e Avaliação

Para entrega, apenas o arquivo `configure.yaml` deve ser submetido ao repositório. Para tanto, quando estiver pronta a submissão, utilize os comandos abaixo, para fazer submissão dos apenas deste arquivo no git:
```
git add configure.yaml
git commit -m "Submissão #1"
git push
```

As alterações no repositório devem ser realizadas até o dia 24/05/2021 às 13h59min, horário de Brasilia.

## Dicas e Observações
- **NÃO** submeta nenhum arquivo adicional.

- Altere apenas os campos indicados nos arquivos, uma vez que a avaliação será realizada re-executando os passos descritos nas seções anteriores, entretanto, com o seu arquvo `configure.yaml`. Além disso, é essencial que seus arquivos não dependa de nenhum outro arquivo externo.

- Você pode ir montando e executando seu *playbook* aos poucos e testar, sempre que possível.

- Não esqueça de testar antes de submeter!

- Você pode, paralelamente, efetuar *login* na máquina remota para observar os efeitos realizados pelas *tasks* executadas.

- Você pode utilizar qualquer módulo embutido na instalação do Ansible, mas não deve instalar nenhum módulo adicional.

- Algun modulos do Ansible úteis para esta atividade podem ser: 
    - [Módulo `apt`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html) para instalação e atualização de pacotes.
    - [Módulo `git`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/git_module.html) para utilizar git.
    - [Módulo `docker_image`](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html) para construção de imagens Docker.
    - [Módulo `shell`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html) para execução de comandos shell.
    - [Módulo `fetch`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fetch_module.html) para copia de arquivos da máquna remota para a máquina local.

- Antes de instalar pacotes, lembre-se de atualizar a lista de pacotes (veja a opção `update_cache` do módulo `apt`)

- A chave `bacome` com valor `True` (*e.g.* `become: true`) pode ser adicionada a uma *task* (ou *play*) para que esta seja executada como super usuário (*e.g.* *root*). Isto é util para *tasks* que necessitam de permissões elevadas (como instalação de pacotes, por exemplo).

- Para longas tarefas (execução da aplicação e construção da imagem Docker), o Ansible fornece as chaves [`pool`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html#asynchronous-playbook-tasks) e [`async`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html#asynchronous-playbook-tasks) para evitar desconexões SSH ao executar as tarefas. Valores como `pool: 30` e `async: 600` podem ser razoáveis para estas tarefas, caso necessite.