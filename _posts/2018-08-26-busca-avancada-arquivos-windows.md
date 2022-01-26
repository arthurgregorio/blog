---
title: 'Busca avançada de arquivos no windows'
date: 2018-08-26 01:00:00 +0300
categories: [windows, powershell]
tags: [powershell, windows]
comments: false
---

Ai um dia você precisa buscar todos os arquivos que não contenham um determinado texto, no linux é fácil, mas e no windows? 

## PowerShell ao resgate

Você já deve ter usado ele para algo mas o que vou mostrar aqui vai além do uso do PowerShell (ou PS para os íntimos) para ver
ou acessar algum diretório.

Pra começar, usei a ferramenta de construção de scripts que o próprio windows tem, o *Windows PowerShell ISE* assim construí um
script poderoso para fazer uma busca em todos os arquivos, recursivamente, dentro de um conjunto de pastas limitando somente à 
aqueles que possuíam extensão ```.java```.

O comando ficou assim:

```powershell
Get-ChildItem -include *.java -recurse | ForEach-Object { 
     if( !( select-string -pattern "Arthur Gregorio, AG.Software" -path $_.FullName) ) {
          $_.FullName
     }
}
``` 

A saída do comando foi a lista de arquivos que não tinham o texto *Arthur Gregorio, AG.Software* em seu conteúdo. Simples né?

Vale a pena dar uma olhada em outros comandos que o PS entrega pra nós, da pra fazer muito automatização de maneira muito simples
apenas usando o PS ISE.