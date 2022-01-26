---
title: 'Configurando acentos para teclados em inglês no linux'
date: 2018-12-04 01:00:00 +0300
categories: [linux]
tags: [teclado, linux, configurando]
image:
  src: '/v1643156145/blog/configurar-acentos-teclado-linux/teclado-antigo_l4umef.jpg'
  width: 1000
  height: 400
---

Instalei o linux no meu notebook com teclado em Inglês, porém, mesmo ajustando o teclado para o idioma correto ele ainda não consigo fazer os acentos em Português, como arrumar?

## Um pouco de história

Antes de mais nada, precisamos saber o motivo de alguns teclados ainda serem em inglês, e obviamente o mais certo aqui é o fato de que os computadores
tem origem nos EUA, claro.

Mas por que ainda temos no Brasil muitos teclados em inglês? Bom, pq a maioria é importado e lá fora, o mais comum de encontrarmos é os teclados ditos 
como Inglês Internacional (como esse da foto acima).

## Configurando o teclado

Você pode fazer isso de duas formas, pela janelinha de configurações do teclado no painel de controle do linux ou via linha de comando. Aqui vou mostrar 
a segunda opção.

Caso seu teclado seja como na foto acima (talvez ele ainda tenha as teclas do windows, o que não muda nada) você pode configurá-lo corretamente usando o 
seguinte comando no console:

```shell
setxkbmap -model abnt -layout us -variant intl
```

E caso seu teclado seja em português tupiniquim com a famosa cedilha (ABNT 2), use:

```shell
setxkbmap -model abnt2 -layout br -variant abnt2
```

## Deixando a configuração fixa

Simples né? Porém ainda temos um problema, caso vc reinicie seu PC a configuração irá se perder. Como podemos fazer para deixá-la fixa? 

Você pode editar o seu arquivo ```bash_rc``` ou ```bash_profile``` e adicionar o comando ao final dele, assim cada vez que fizer logon, o comando será executado e seu teclado ficará configurado.