---
---

Muitas vezes precisei estudar o código assembly gerado do meu código C/C++, inicialmente para aprender, depois por curiosidade, e até mesmo por acreditar que poderia optimizar alguma parte do código que escrevi. Bem o GCC já faz um excelente trabalho, ainda mais se vc ajuda-lo, já escrevi faz algum tempo sobre como escrever código em C otimizado para microcontroladores AVR. 

<!--more-->

O comando do GCC que me ajuda a obter o código C de forma legível é o `objdump`, se seu código foi gerado com símbolos de depuração, que é o caso do Arduino, você terá um resultado bastante interessante de fácil leitura e entendimento, pois as referências ao seu código estarão comandadas.

Para obter o help do `objdump`, digite o comando com a chave `--help` e terá o resultado similar abaixo (resumi a saida ao que nos interessa):

```bash
$ avr-objdump --help
[...]
Usage: avr-objdump.exe <option(s)> <file(s)>
 Display information from object <file(s)>.
 At least one of the following switches must be given:
[...]
  -d, --disassemble        Display assembler contents of executable sections
  -D, --disassemble-all    Display assembler contents of all sections
  -S, --source             Intermix source code with disassembly
[...]
  @<file>                  Read options from <file>
  -v, --version            Display this program's version number
  -i, --info               List object formats and architectures supported
  -H, --help               Display this information
[...]
  -z, --disassemble-zeroes       Do not skip blocks of zeroes when disassembling
[...]
```

Usaremos neste exemplo o código do exemplo *MoreComplexblinking.ino*, quando o arduino compila ele gera uma pasta temporária encontre esta pasta, no windows estará no diretório `%TEMP%\\arduino_build_996254`, o número muda a cada vez que abre a IDE do arduino e a cada sketch compilado.

Neste diretório você encontrar os seguintes aquivos após uma compilação bem sucedida:

```bash
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 12:35 .
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 12:40 ..
-rw-r--r-- 1 carlosdelfino 197121 1,6K jul 10 12:04 build.options.json
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 12:05 core
-rw-r--r-- 1 carlosdelfino 197121 2,2K jul 10 12:04 includes.cache
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 11:14 libraries
-rw-r--r-- 1 carlosdelfino 197121   13 jul 10 12:05 MoreComplexBlinking.ino.eep
-rw-r--r-- 1 carlosdelfino 197121  36K jul 10 12:05 MoreComplexBlinking.ino.elf
-rw-r--r-- 1 carlosdelfino 197121  13K jul 10 12:05 MoreComplexBlinking.ino.hex
-rw-r--r-- 1 carlosdelfino 197121  14K jul 10 12:05 MoreComplexBlinking.ino.with_bootloader.hex
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 11:14 preproc
drwxr-xr-x 1 carlosdelfino 197121    0 jul 10 12:04 sketch
```

O arquivo que é importante para nós é o arquivo de extensão **elf**, este arquivo conterá todo código já preparada para ser convertido no binário que será enviado ao Arduino, portanto ele pode ser convertido fácilmente em Assembly. o arquivo de extensão **.hex** já é o arquivo pronto para envio ao microcontrolador.

Vamos criar um arquivo com todo o código assembly, porém iremos exibir aqui apenas uma parte de nosso código, no caso o conteúdo da função que controla o LED vermelho.

```C
taskLoop(redLED)
{
  static unsigned char counter = 0;
  
  if (!greenLED_isOn)
  {
     if (counter >2)
       resumeTask(greenLED);
     counter++;
  }
  
  redLED_isOn = false;
  taskDelay(1000);
  redLED_isOn = true; 
  taskDelay(1000);
}
``` 

Resultado do Comando `$ avr-objdump -Sz MoreComplexBlinking.ino.elf > MoreComplexBlinking.ino.elf.S.dump` com toda o código que controla o LED vermelho, é um código longo, irei entrar em detalhes sobre o código com foco no DuinOS em outro artigo futuro:

```asm
taskLoop(redLED)
{
  static unsigned char counter = 0;
  
  if (!greenLED_isOn)
     ec2:	80 91 17 01 	lds	r24, 0x0117	; 0x800117 <greenLED_isOn>
     ec6:	81 11       	cpse	r24, r1
     ec8:	52 c0       	rjmp	.+164    	; 0xf6e <_Z11redLED_TaskPv+0xbe>
  {
     if (counter >2)
     eca:	f0 90 16 01 	lds	r15, 0x0116	; 0x800116 <__data_end>
     ece:	22 e0       	ldi	r18, 0x02	; 2
     ed0:	2f 15       	cp	r18, r15
     ed2:	08 f0       	brcs	.+2      	; 0xed6 <_Z11redLED_TaskPv+0x26>
     ed4:	49 c0       	rjmp	.+146    	; 0xf68 <_Z11redLED_TaskPv+0xb8>
       resumeTask(greenLED);
     ed6:	00 91 d5 05 	lds	r16, 0x05D5	; 0x8005d5 <greenLED>
     eda:	10 91 d6 05 	lds	r17, 0x05D6	; 0x8005d6 <greenLED+0x1>
		/* It does not make sense to resume the calling task. */
		configASSERT( xTaskToResume );

		/* The parameter cannot be NULL as it is impossible to resume the
		currently executing task. */
		if( ( pxTCB != pxCurrentTCB ) && ( pxTCB != NULL ) )
     ede:	80 91 29 06 	lds	r24, 0x0629	; 0x800629 <pxCurrentTCB>
     ee2:	90 91 2a 06 	lds	r25, 0x062A	; 0x80062a <pxCurrentTCB+0x1>
     ee6:	08 17       	cp	r16, r24
     ee8:	19 07       	cpc	r17, r25
     eea:	09 f4       	brne	.+2      	; 0xeee <_Z11redLED_TaskPv+0x3e>
     eec:	3d c0       	rjmp	.+122    	; 0xf68 <_Z11redLED_TaskPv+0xb8>
     eee:	01 15       	cp	r16, r1
     ef0:	11 05       	cpc	r17, r1
     ef2:	09 f4       	brne	.+2      	; 0xef6 <_Z11redLED_TaskPv+0x46>
     ef4:	39 c0       	rjmp	.+114    	; 0xf68 <_Z11redLED_TaskPv+0xb8>
		{
			taskENTER_CRITICAL();
     ef6:	0f b6       	in	r0, 0x3f	; 63
     ef8:	f8 94       	cli
     efa:	0f 92       	push	r0

		/* It does not make sense to check if the calling task is suspended. */
		configASSERT( xTask );

		/* Is the task being resumed actually in the suspended list? */
		if( listIS_CONTAINED_WITHIN( &xSuspendedTaskList, &( pxTCB->xStateListItem ) ) != pdFALSE )
     efc:	d8 01       	movw	r26, r16
     efe:	1a 96       	adiw	r26, 0x0a	; 10
     f00:	8d 91       	ld	r24, X+
     f02:	9c 91       	ld	r25, X
     f04:	87 5e       	subi	r24, 0xE7	; 231
     f06:	95 40       	sbci	r25, 0x05	; 5
     f08:	69 f5       	brne	.+90     	; 0xf64 <_Z11redLED_TaskPv+0xb4>
		{
			/* Has the task already been resumed from within an ISR? */
			if( listIS_CONTAINED_WITHIN( &xPendingReadyList, &( pxTCB->xEventListItem ) ) == pdFALSE )
     f0a:	f8 01       	movw	r30, r16
     f0c:	84 89       	ldd	r24, Z+20	; 0x14
     f0e:	95 89       	ldd	r25, Z+21	; 0x15
     f10:	f5 e0       	ldi	r31, 0x05	; 5
     f12:	80 3f       	cpi	r24, 0xF0	; 240
     f14:	9f 07       	cpc	r25, r31
     f16:	31 f1       	breq	.+76     	; 0xf64 <_Z11redLED_TaskPv+0xb4>
			{
				/* Is it in the suspended list because it is in the	Suspended
				state, or because is is blocked with no timeout? */
				if( listIS_CONTAINED_WITHIN( NULL, &( pxTCB->xEventListItem ) ) != pdFALSE ) /*lint !e961.  The cast is only redundant when NULL is used. */
     f18:	89 2b       	or	r24, r25
     f1a:	21 f5       	brne	.+72     	; 0xf64 <_Z11redLED_TaskPv+0xb4>
				{
					traceTASK_RESUME( pxTCB );

					/* The ready list can be accessed even if the scheduler is
					suspended because this is inside a critical section. */
					( void ) uxListRemove(  &( pxTCB->xStateListItem ) );
     f1c:	68 01       	movw	r12, r16
     f1e:	22 e0       	ldi	r18, 0x02	; 2
     f20:	c2 0e       	add	r12, r18
     f22:	d1 1c       	adc	r13, r1
     f24:	c6 01       	movw	r24, r12
     f26:	0e 94 a8 01 	call	0x350	; 0x350 <uxListRemove>
					prvAddTaskToReadyList( pxTCB );
     f2a:	d8 01       	movw	r26, r16
     f2c:	56 96       	adiw	r26, 0x16	; 22
     f2e:	8c 91       	ld	r24, X
     f30:	90 91 26 06 	lds	r25, 0x0626	; 0x800626 <uxTopReadyPriority>
     f34:	98 17       	cp	r25, r24
     f36:	10 f4       	brcc	.+4      	; 0xf3c <_Z11redLED_TaskPv+0x8c>
     f38:	80 93 26 06 	sts	0x0626, r24	; 0x800626 <uxTopReadyPriority>
     f3c:	8b 9d       	mul	r24, r11
     f3e:	c0 01       	movw	r24, r0
     f40:	11 24       	eor	r1, r1
     f42:	b6 01       	movw	r22, r12
     f44:	85 5f       	subi	r24, 0xF5	; 245
     f46:	99 4f       	sbci	r25, 0xF9	; 249
     f48:	0e 94 02 02 	call	0x404	; 0x404 <vListInsertEnd>

					/* A higher priority task may have just been resumed. */
					if( pxTCB->uxPriority >= pxCurrentTCB->uxPriority )
     f4c:	e0 91 29 06 	lds	r30, 0x0629	; 0x800629 <pxCurrentTCB>
     f50:	f0 91 2a 06 	lds	r31, 0x062A	; 0x80062a <pxCurrentTCB+0x1>
     f54:	d8 01       	movw	r26, r16
     f56:	56 96       	adiw	r26, 0x16	; 22
     f58:	9c 91       	ld	r25, X
     f5a:	86 89       	ldd	r24, Z+22	; 0x16
     f5c:	98 17       	cp	r25, r24
     f5e:	10 f0       	brcs	.+4      	; 0xf64 <_Z11redLED_TaskPv+0xb4>
					{
						/* This yield may not cause the task just resumed to run,
						but will leave the lists in the correct state for the
						next yield. */
						taskYIELD_IF_USING_PREEMPTION();
     f60:	0e 94 45 01 	call	0x28a	; 0x28a <vPortYield>
				else
				{
					mtCOVERAGE_TEST_MARKER();
				}
			}
			taskEXIT_CRITICAL();
     f64:	0f 90       	pop	r0
     f66:	0f be       	out	0x3f, r0	; 63
     counter++;
     f68:	f3 94       	inc	r15
     f6a:	f0 92 16 01 	sts	0x0116, r15	; 0x800116 <__data_end>
 * @param ticks 
 */
inline void taskDelay(const portTickType ticks) __attribute__((__always_inline__));
inline void taskDelay(const portTickType ticks)
{
	portTickType xLastWakeTime = xTaskGetTickCount();
     f6e:	0e 94 9e 01 	call	0x33c	; 0x33c <xTaskGetTickCount>
     f72:	9a 83       	std	Y+2, r25	; 0x02
     f74:	89 83       	std	Y+1, r24	; 0x01

	//Better than vTaskDelay:
	vTaskDelayUntil( &xLastWakeTime, ticks);
     f76:	68 ee       	ldi	r22, 0xE8	; 232
     f78:	73 e0       	ldi	r23, 0x03	; 3
     f7a:	ce 01       	movw	r24, r28
     f7c:	01 96       	adiw	r24, 0x01	; 1
     f7e:	0e 94 5e 06 	call	0xcbc	; 0xcbc <vTaskDelayUntil>
  }
  
  redLED_isOn = false;
  taskDelay(1000);
  redLED_isOn = true; 
     f82:	e0 92 d9 05 	sts	0x05D9, r14	; 0x8005d9 <redLED_isOn>
 * @param ticks 
 */
inline void taskDelay(const portTickType ticks) __attribute__((__always_inline__));
inline void taskDelay(const portTickType ticks)
{
	portTickType xLastWakeTime = xTaskGetTickCount();
     f86:	0e 94 9e 01 	call	0x33c	; 0x33c <xTaskGetTickCount>
     f8a:	9a 83       	std	Y+2, r25	; 0x02
     f8c:	89 83       	std	Y+1, r24	; 0x01

	//Better than vTaskDelay:
	vTaskDelayUntil( &xLastWakeTime, ticks);
     f8e:	68 ee       	ldi	r22, 0xE8	; 232
     f90:	73 e0       	ldi	r23, 0x03	; 3
     f92:	ce 01       	movw	r24, r28
     f94:	01 96       	adiw	r24, 0x01	; 1
     f96:	0e 94 5e 06 	call	0xcbc	; 0xcbc <vTaskDelayUntil>
     f9a:	93 cf       	rjmp	.-218    	; 0xec2 <_Z11redLED_TaskPv+0x12>

00000f9c <__vector_16>:
#if defined(TIM0_OVF_vect)
ISR(TIM0_OVF_vect)
#else
ISR(TIMER0_OVF_vect)
#endif
{
     f9c:	1f 92       	push	r1
     f9e:	0f 92       	push	r0
     fa0:	0f b6       	in	r0, 0x3f	; 63
     fa2:	0f 92       	push	r0
     fa4:	11 24       	eor	r1, r1
     fa6:	2f 93       	push	r18
     fa8:	3f 93       	push	r19
     faa:	8f 93       	push	r24
     fac:	9f 93       	push	r25
     fae:	af 93       	push	r26
     fb0:	bf 93       	push	r27
	// copy these to local variables so they can be stored in registers
	// (volatile variables must be read from memory on every access)
	unsigned long m = timer0_millis;
     fb2:	80 91 df 05 	lds	r24, 0x05DF	; 0x8005df <timer0_millis>
     fb6:	90 91 e0 05 	lds	r25, 0x05E0	; 0x8005e0 <timer0_millis+0x1>
     fba:	a0 91 e1 05 	lds	r26, 0x05E1	; 0x8005e1 <timer0_millis+0x2>
     fbe:	b0 91 e2 05 	lds	r27, 0x05E2	; 0x8005e2 <timer0_millis+0x3>
	unsigned char f = timer0_fract;
     fc2:	30 91 de 05 	lds	r19, 0x05DE	; 0x8005de <timer0_fract>

	m += MILLIS_INC;
	f += FRACT_INC;
     fc6:	23 e0       	ldi	r18, 0x03	; 3
     fc8:	23 0f       	add	r18, r19
	if (f >= FRACT_MAX) {
     fca:	2d 37       	cpi	r18, 0x7D	; 125
     fcc:	58 f5       	brcc	.+86     	; 0x1024 <__vector_16+0x88>
	// copy these to local variables so they can be stored in registers
	// (volatile variables must be read from memory on every access)
	unsigned long m = timer0_millis;
	unsigned char f = timer0_fract;

	m += MILLIS_INC;
     fce:	01 96       	adiw	r24, 0x01	; 1
     fd0:	a1 1d       	adc	r26, r1
     fd2:	b1 1d       	adc	r27, r1
	if (f >= FRACT_MAX) {
		f -= FRACT_MAX;
		m += 1;
	}

	timer0_fract = f;
     fd4:	20 93 de 05 	sts	0x05DE, r18	; 0x8005de <timer0_fract>
	timer0_millis = m;
     fd8:	80 93 df 05 	sts	0x05DF, r24	; 0x8005df <timer0_millis>
     fdc:	90 93 e0 05 	sts	0x05E0, r25	; 0x8005e0 <timer0_millis+0x1>
     fe0:	a0 93 e1 05 	sts	0x05E1, r26	; 0x8005e1 <timer0_millis+0x2>
     fe4:	b0 93 e2 05 	sts	0x05E2, r27	; 0x8005e2 <timer0_millis+0x3>
	timer0_overflow_count++;
     fe8:	80 91 da 05 	lds	r24, 0x05DA	; 0x8005da <timer0_overflow_count>
     fec:	90 91 db 05 	lds	r25, 0x05DB	; 0x8005db <timer0_overflow_count+0x1>
     ff0:	a0 91 dc 05 	lds	r26, 0x05DC	; 0x8005dc <timer0_overflow_count+0x2>
     ff4:	b0 91 dd 05 	lds	r27, 0x05DD	; 0x8005dd <timer0_overflow_count+0x3>
     ff8:	01 96       	adiw	r24, 0x01	; 1
     ffa:	a1 1d       	adc	r26, r1
     ffc:	b1 1d       	adc	r27, r1
     ffe:	80 93 da 05 	sts	0x05DA, r24	; 0x8005da <timer0_overflow_count>
    1002:	90 93 db 05 	sts	0x05DB, r25	; 0x8005db <timer0_overflow_count+0x1>
    1006:	a0 93 dc 05 	sts	0x05DC, r26	; 0x8005dc <timer0_overflow_count+0x2>
    100a:	b0 93 dd 05 	sts	0x05DD, r27	; 0x8005dd <timer0_overflow_count+0x3>
}
    100e:	bf 91       	pop	r27
    1010:	af 91       	pop	r26
    1012:	9f 91       	pop	r25
    1014:	8f 91       	pop	r24
    1016:	3f 91       	pop	r19
    1018:	2f 91       	pop	r18
    101a:	0f 90       	pop	r0
    101c:	0f be       	out	0x3f, r0	; 63
    101e:	0f 90       	pop	r0
    1020:	1f 90       	pop	r1
    1022:	18 95       	reti
```

É importante observar que o DuinOS tem várias funções inline, o que leva a uma execução mais rápida das threads, porém aumenta o tamanho do código, já que cada função inline é replicada completamente onde é chamada, evitando o uso do stack para chamada de blocos de códigos.