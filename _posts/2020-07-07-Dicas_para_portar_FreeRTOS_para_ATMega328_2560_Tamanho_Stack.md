---
layout: post
title: Tamanho do Stack - Dicas para portar o FreeRTOS para ATMega328 e ATMega2560
description: Dicas para portar o FreeRTOS para a família ATMega328 e ATMega2560 (e suas variántes)
image: assets/images/avr/avr-logo.png
tags: ATMega, 328, 2560, AVR, Microchip
category : avr
---

O port mais simples do Arduino é para os chips da familia AVR, em especial ATMega328 e ATMega2560, Arduino UNO e Arduino Mega respectivamente, mas isso não quer dizer que não seja preciso tomar alguns cuidados.

<!--more-->

Vamos là.

## Tamanho do Stack

O Stack no ATMega2560 é limitado da mesma forma que que no ATMega328, e qualquer outro AVR (salvo que seja especificado ao contrário em algum datasheet), não utiliza mais que dois bytes normalmente para endereçar sua alocação, já que o ATmega2560 tem uma mémoria de 8KBs e o ATMega328 tem 2KBs, (0x2000 e 0x0800 respectivamente). 

Mesmo o ATMega640 ao ATMega2561 tendo a possibilidade de expandir sua memória com chips externos e com registradores especiais para tal manejo, sempre usaremos nesta versão do DuinOS o stack na mémoria interna. Tais chips tem um contador de memória com 3 bytes, diferentemente do demais ATMegas.

Portanto o tamanho do Stack estará limitado ao tamanho da memória interna, tipicamente metade desta memória, já que o restante é usado para HEAP (que crescendo no sentido oposto ao Stack, podendo vir a colidir).

Veja a figura abaixo para entender como a memória é alocada tipicamente (não se está considerando o uso de memória externa);

![Mapeamento Padrão para alocação de memória]({{site.url}}{{site.baseurl}}/assets/images/gcc/malloc-std.png)

Porém quando se está trabalhando com endereços de retorno de funções, gerados com instruções RET/CALL ou RETI/CALLI, na maioria das implementações AVR usa dois bytes no stack para armazenar o endereço de retorno, mas no caso do ATmega2560/2561 usa 3 bytes, tal necessidade, é identificada pela presença da macro de compilação `__AVR_HAVE_RAMPZ__` ou `__AVR_3_BYTE_PC__`

## Referências

* https://www.nongnu.org/avr-libc/user-manual/malloc.html
* https://embeddedgurus.com/stack-overflow/category/efficient-cc/
* Datasheets for ATmega/AVR cores
* Appnotes for ATmega/AVR Cores