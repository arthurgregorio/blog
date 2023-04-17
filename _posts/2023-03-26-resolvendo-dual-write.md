---
title: 'Resolvendo o problema do Dual Write'
date: 2023-03-26 22:00:00 +0300
categories: [Programação]
tags: [programacao, java, spring, microsservicos]
image:
  path: '/v1678455714/blog/dual-writing-microsservicos/microservices_dvbsdd.jpg'
  width: 1000
  height: 400
---

Anteriormente [neste artigo](https://arthurgregorio.eti.br/posts/dual-write-microservicos/) falei sobre o problema do
_Dual Write_ que comumente atinge nossos sistemas distribuídos, popularmente conhecidos como microsserviços, agora
vamos aprofundar um pouco mais esse assunto em um vídeo prático sobre o tema.

{% include youtube.html id="MEcNFY6S8OE" %}

[Aqui](https://github.com/arthurgregorio/servico-pedidos) você encontra o projeto de exemplo.

> Ah Arthur, mas você não vai mostrar como fazer o serviço de envio dos eventos?

Fica para um próximo post! Mas para que você não fique na mão, deixo abaixo duas ótimas referências de implementação do
outbox com Debezium e outra com Postgres + Hibernate:

- [Microservices & Data – Implementing the Outbox Pattern with Hibernate](https://thorben-janssen.com/outbox-pattern-hibernate/)
- [Implementing the Outbox Pattern with CDC using Debezium](https://thorben-janssen.com/outbox-pattern-with-cdc-and-debezium)
- [Avoiding dual writes in event-driven applications](https://developers.redhat.com/articles/2021/07/30/avoiding-dual-writes-event-driven-applications)
