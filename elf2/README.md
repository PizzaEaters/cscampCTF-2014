ELF 2
=====

Analisando o binário, e fazendo os devidos `jmp` para bypassar o anti-debugging do binário,
chegamos a seguinte parte onde é impressa a flag:

```
(gdb) x/s $ecx
0x400878:	 "Flag: %s"
(gdb) u
0x00000000004006d2 in xxyyzz ()
1: x/i $pc
=> 0x4006d2 <xxyyzz+59>:	mov    $0x400881,%edx
(gdb) x/s $edx
0x400881 <__FUNCTION__.3311>:	 "xxyyzz"
```

Ai está a flag!
