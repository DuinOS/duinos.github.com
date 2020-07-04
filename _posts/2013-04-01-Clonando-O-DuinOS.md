---
layout: post
title: Clonando ou baixando o DuinOS
description: Como clonar ou baixar o DuinOS
image: assets/images/pic06.jpg
tags: DuinOS, Zip, Git
category : Tutoriais
---

Se você quer usar o pacote DuinOS em seu Arduino original você deve baixar este pacote.

Há duas formas de se baixar o pacote, uma delas é usando a ferramenta GIT e a segunda 
opção é o pacote zipado.

## Clonando o DuinOS

Você pode fazer um Fork do projeto, mas não irei entrar neste detalhe, para fazer seu 
clone em sua area de trabalho e depois instalar no seu pacote Arduino.

O link para clonagem é o seguinte: git@github.com:DuinOS/DuinOS.git

Você pode usar qualquer ferramenta GIT que desejar, aqui irei mostrar como fazer na linha
de comando:

```
$ mkdir seu-workspace/DuinOS
$ cd seu-workspace/DuinOS
$ git clone git@github.com:DuinOS/DuinOS.git DuinOS
$ cd seu-workspace/DuinOS
```


Este pacote já vem pronto para usar com Arduino 1.0.x ou 1.5.x sendo que você deve usar o 
Branch Master, os outros Branchs são para desenvolvimento.

Pronto agora vc já pode instalar/adaptar o DuinOS para seu Arduino Original

## Baixando o pacote zipado

Para baixar o pacote basta clicar no link: https://github.com/DuinOS/DuinOS/archive/master.zip

Em seguida basta descompactar o pacote em seu workspace e fazer a instalação/adaptação 
do DuinOS para seu Arduino.

Em novo artigo eu irei ensinar como fazer esta adaptação tanto para o Arduino 1.0.x e para o 1.5.x

Para Suporte use o menu [Issues](https://github.com/DuinOS/DuinOS/issues) ou envie e-mail para duinos@carlosdelfino.eti.br
