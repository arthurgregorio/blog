---
title: 'AWS + Spring na prática: SNS, SQS e Localstack'
date: 2023-09-18 22:00:00 +0300
categories: [Programação]
tags: [programacao, java, kotlin, aws, microserviços, spring]
image:
  path: '/v1695090930/blog/aws-spring-sqs-sns-localstack/aws-spring-sqs-sns-localstack.png'
  width: 1000
  height: 400
---

Hoje vamos ver algo muito comum na vida de um desenvolvedor Java/Kotlin que utiliza AWS: comunicação via mensageria
utilizando SNS e SQS, e ainda, como testar tudo isso localmente no localstack.

## O problema

Imagine um ecossistema (sim, mais de um serviço) corporativo responsável por gerenciar o ciclo de vida de um pedido.
Este pedido que aqui vamos chamar carinhosamente de `Order` é criado por alguém via API REST (um e-commerce? Um sistema
de pedidos mobile? Não importa) e depois precisa ser distribuído entre outros serviços com atribuições específicas, por
exemplo: separar os itens do pedido, alocar a coleta, faturar... Entre outros.

Para facilitar esse entendimento, deixou um diagrama simples do fluxo:

![pub-sub-order](/v1695091358/blog/aws-spring-sqs-sns-localstack/pub-sub_np6ivt.jpg)

## Solução

Dado o contexto, nossa ideia aqui é criar o serviço que irá publicar o "aviso" (SNS) que temos uma nova `Order` e um dos
serviços que irá consumir essa mensagem por uma fila (SQS) especificamente criada para este fim.

{% include youtube.html id="d_M-zDT44U8" %}

Para isso, fiz um vídeo explicando cada etapa do processo [neste projeto](https://github.com/arthurgregorio/aws-pub-sub) que disponibilizei como exemplo em meu
github.

## Stack utilizada

Para dar um diversificada, resolvi usar duas linguagens diferentes, porém parecidas para trabalhar o código da solução:
Kotlin e Java. No mais, temos o bom e velho Spring Boot para tratar dos outros detalhes:

- Spring Boot v3
- Java para o SNS producer
- Kotlin para o SQS consumer
- Localstack para testar local
- MongoDB para persistência no SNS producer

Gostou? Compartilhe esse post com seus amigos que também querem aprender um pouco mais sobre comunicação entre
micro-serviços, AWS ou Spring.
