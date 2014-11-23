ELF 1
=====

Analisando o código, verificando o que continha no endereço usado para checar
o password, podemos notar o seguinte:

```
=> 0x4007cf <xxxxx+85>:	mov    $0x400f40,%eax
(gdb) x/s 0x400f40
0x400f40:	 "samir"
```

Visto no código que ele chama um base64 decoder no input fornecido, basta então
passar tal string em base64.

```
$ ./binary
*********************************
  welcome to cracking challenge
*********************************
Enter you password: c2FtaXI=
Flag: ping-pong@
```

A flag no final é apenas **ping-pong**.
