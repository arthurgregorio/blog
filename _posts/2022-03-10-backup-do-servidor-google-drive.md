---
title: 'Fazendo backup do servidor para o Drive'
date: 2023-03-10 22:00:00 +0300
categories: [Devops]
tags: [infra, linux, devops, servidores]
image:
  path: '/v1664847657/blog/backup-servidor-gdrive/gdrive_d0kngt.webp'
  width: 1000
  height: 400
---

Desde que migrei minha instância do [webBudget](https://webbudget.com.br/) do Jelastic para o
[Vultr](https://www.vultr.com/?ref=8069460) precisei fazer algumas coisas manuais e uma delas foi automatizar o backup
do banco de dados. Aqui vou contar um pouco como fiz isso usando o google drive como local de armazenamento.

## Contexto

Depois que saí da minha instância do Jelastic, um ótimo cloud provider que te entrega tudo de maneira simples e com uma
interface de gerenciamento bem simples, acabei por migrar para um outro mais simples (e drasticamente mais barato!),
o [Vultr](https://www.vultr.com/?ref=8069460).

Lá as coisas são mais "by-hands", ou seja, nada de interfaces botõezinhos coloridos para clicar e ter as coisas
funcionando, como toda VPS você precisa acessar o servidor e fazer as instalações que precisa, por conta.

Não é um problema pra mim, subi rapidamente a infra do webBudget v3 com wildfly e postgres, tudo lindo! Até que lembrei
que um backup do banco de dados era algo importante. Mas onde eu ia armazenar isso?

Neste momento, havia 2 opções:

1. Deixar no servidor
2. Mandar para algum lugar

A primeira opção era a mais simples porém a que não me serviria de nada caso a minha instância morresse por causas
desconhecidas, sendo assim, fiquei com a segunda e o local onde escolhi jogar os arquivos foi o Google Drive.

## Cliente do GDrive

> o cliente usado aqui foi descontinuado, mas calma, tem um novo que funciona do mesmo jeito e pode ser encontrado
> [aqui](https://github.com/glotlabs/gdrive)

~~Utilizando [este cliente](https://github.com/prasmussen/gdrive/) tudo ficou muito fácil! Basicamente, você precisa fazer
o download do cliente e com um simples comando, configurar o mesmo para a sua conta e pronto, pode fazer upload,
download entre outras coisas direto do seu google drive.~~

### Configurando

> como mencionei anteriormente, o processo de configuração é quase o mesmo, recomendo ler a doc do novo cliente do
> gdrive [aqui](https://github.com/glotlabs/gdrive/blob/main/docs/create_google_api_credentials.md) esse guia é bem
> completo e vai te ajudar direitinho

~~Depois de deixar o seu cliente em um local adequado no seu servidor linux, execute o seguinte comando `gdrive about` e
isso irá gerar um link no seu console, copie ele e cole no seu browser, uma tela de autorização do OAuth vai aparecer e
lá basta autorizar com sua conta.~~

~~Após este processo, copie o código de ativação mostrado e cole novamente no console e pronto! Ah, você ainda pode usar
`gdrive help` para ver a lista de comandos disponíveis.~~

## O script

Depois do cliente instalado, para facilitar as coisas criei então um script, assim fica mais de boas rodar todo o
processo:

```bash
#!/bin/bash

BACKUP_DIR=/root/backups # onde vou gerar o backup local, no server
FOLDER_ID= # informe aqui o id da pasta de destino
DAYS_TO_KEEP=5 # dias para manter os backups por aqui no server?
FILE_SUFFIX=_pg_backup.sql # sufixo do arquivo de backup
DATABASE=webbudget # o banco pra fazer backup
USER=sa_webbudget # o usuário que pode fazer o backup
HOST=localhost # o host do banco

FILE=`date +"%Y%m%d%H%M"`${FILE_SUFFIX}

OUTPUT_FILE=${BACKUP_DIR}/${FILE}

# dump
pg_dump --file ${OUTPUT_FILE}
  --host ${HOST}
  --username ${USER}
  --format=p
  --no-owner
  --section=pre-data
  --section=data
  --section=post-data
  --no-privileges
  --no-tablespaces
  --no-unlogged-table-data
  --inserts ${DATABASE}

# gzipa o arquivo
gzip $OUTPUT_FILE

# mostra umas infos
echo "${OUTPUT_FILE}.gz was created:"
ls -l ${OUTPUT_FILE}.gz

# upload pro drive
/usr/local/bin/gdrive upload --parent ${FOLDER_ID} ${OUTPUT_FILE}.gz

# deleta os backups antigos
find $BACKUP_DIR -maxdepth 1 -mtime +$DAYS_TO_KEEP -name "*${FILE_SUFFIX}.gz" -exec rm -rf '{}' ';'
```

> Importante! você vai precisar do ID da pasta no Google Drive se quiser enviar os arquivos para ela. Para pegá-lo,
> basta abrir o drive e copiar no final da URL, exemplo: */folders/o-id-fica-aqui*. Cole ele ali na variável e sucesso!

> Estou tendo problemas com um erro 400 após um certo tempo, dicas? Sim, o token vence! Será preciso autenticar
> novamente, veja [esse link aqui](https://github.com/prasmussen/gdrive/issues/586).
