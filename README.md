# Bootcamp DIO Cybersegurança 
## Desafio 1

 ### Simulação de Auditoria com testes de invasão utilizando Máquinas Virtuais e sistemas Kali Linux e Metasploitable 2.  
 Testes Simulados:
 - Brute Force explorando serviço FTP;
 - Brute Force explorando serviço HTTP;
 - [Password Spraying em lista de usuários de servidor](#ataque-em-cadeia-de-usuarios-de-servidor-simulado-com-password-spraying);

Ferramentas utilizadas:
- NMAP;
- Enum4linux;
- Medusa;
- Hydra;

1.1 - Configuração do laboratório para realização dos testes:
- Utilizei o Gerenciador da Oracle de Máquinas Virtuais;
- Criação da Máquina Virtual da Kali Linux, para utilizar as ferramendas de auditoria e Metasploitable 2 para simular servidor antigo;
- Ambas as máquinas foram conectas com adaptador de rede ligado à host-only, ou seja, em ambiente isolado.
- Criação de "Snapshot" do sistema Metasploitable 2 em ```Máquina > Criar Snapshot``` para restaurar o estado original do sistema, garantindo que estamos iniciando um teste de invasão com máquina limpa;
- 
[![oracle-vms.png](https://i.postimg.cc/RhRXWwCM/oracle-vms.png)](https://postimg.cc/q6N8YhCY)

1.2 - Início da Comunicação entre as Máquinas Virtuais:
- Execução do comando ```ip a``` em cada máquina para identificar seus respectivos endereços ip;
- Na máquina utilizando o Metasploitable 2 foi identificado o ip ```192.168.56.101``` ;
- Abrindo o terminal com o Kali Linux foi executado o comando ```ping -c 3 192.168.56.101``` que envia 3 pacotes de 64 bytes para o Metasploitable 2, testando a comunicação entre as máquinas;

 ### Brute Force

2.1 - Primeiro teste de Auditoria Com Brute Force:
- Foco no serviço FTP;
- Teste de invasão simulando um servidor STP antigo (legado);
- O processo de Auditoria iniciou com a Enumeração utilizando o NMAP, utilizando o seguindo comando via terminal: ```nmap -sV -p 21,22,80,445,139 192.168.56.101```
- O teste retornou o estado aberto dessas portas;

[![nmap-result.png](https://i.postimg.cc/gk73Ng7L/nmap-result.png)](https://postimg.cc/gnvw0HmG)

2.2 - Explorando Serviço FTP:
- Foi executado o comando:
  ```ftp 192.168.56.101```
  - Funcionando este comando valida que o serviço FTP está aceitando conexões;

[![ftp-test.png](https://i.postimg.cc/qM2Cxp2W/ftp-test.png)](https://postimg.cc/3dJRK5SZ)
 
3.1 - Criando Arquivos de chaves de Usuários e Senhas Comuns para Simulação de Ataque:
- Para a criação do arquivo de chaves de usuários e senhas respectivamente foram executados os comandos:
  ```echo -e "user\nmsfadmin\nadmin\nroot" > users.txt```
  ```echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt```

3.2 - Comando de Ataque utilizando o Medusa no terminal Kali Linux:
- ```medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6```
  - No comando após o -h inserimos o ip do alvo, após o -U e o -P são os arquivo contendo o arquivo de texto criado para tentativas de combinação de acesso que o medusa irá fazer, após o -M indicamos o serviço que estamos atacando (no caso FTP), após o -t indica a quantidade de threads utilizadas pelo Medusa

[![medusa-1.png](https://i.postimg.cc/SNczfmcw/medusa-1.png)](https://postimg.cc/hfPjSWyr)

 ### Simulação de Ataques de Força Bruta em Formulários de Login Web
4.1 - Verificação do sistema de Login do Servidor:
- Pelo navegador web do Kali Linux acessar : ```192.168.56.101/dvwa/login.php```
- Através da ferramenda do desenvolvedor, atalho F12, verificar parâmetros do login, testar como o sistema se comporta com um falha;
    - Clicar na requisição e verificar parâmetros utilizados em "Request"

4.2 - Criação de wordlists:
- Criação das listas de nome de login de usuários e senhas possíveis em arquivos .txt, representando uma tentativa;
- Utilizar a princípio mesmos comandos utilizados no teste anterior para a criação dos arquivos .txt
- Atacantes reais utilizam listas com milhares e até milhões de entradas;

4.3 - Problemas de Falso Positivo e suport HTTP com o Medusa:
- Ao utilizar o seguinte comando abaixo, utilizando o Medusa para o ataque:
  ```
  medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
  -m PAGE: '/dvwa/login.php' \
  -m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
  -m 'FAIL=Login failed' -t 6
  ```
  - O Medusa retornava um falso positivo, acusando sucesso em credenciais que não funcionavam na realidade;
  - Isso é devido à uma limitação do Medusa que não acompanha o redirecionamento que o serviço HTTP realiza e o gerenciamento de cookies de sessão que o php utiliza
 
[![medusa-erro-1.png](https://i.postimg.cc/1zVSD3qQ/medusa-erro-1.png)](https://postimg.cc/NytZcYDJ)

[![medusa-erro-2.png](https://i.postimg.cc/vmxy5ZVG/medusa-erro-2.png)](https://postimg.cc/nsxNJZ2W)

4.4 - Testes com a ferramenta Hydra:
- Por recomendação de colegas de bootcamp comecei testes com a ferramenta Hydra, tentando o seguinte comando abaixo para a mesma finalidade do teste no Formulário dvwa;
  ```
  hydra 'http-post-form://192.168.56.101/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed' \
  -L /home/users.txt \
  -P /home/pass.txt -t 5 > hydra_dvwa.txt
  ```
- Após alguns ajustes de sintaxe a ajuste no comando correto da ferramenta obtive o resultado do teste sem falsos positivos:

[![sucesso-hydra.png](https://i.postimg.cc/CKn0bxq0/sucesso-hydra.png)](https://postimg.cc/bGP4h8gV)

4.5 - Conclusões do teste de invasão de formulários de login web:
- Ferramenta Medusa possui limitações para esse teste e resulta em falsos positivos, o que é de certa forma pior do que uma mensagem de erro. Você precisa validar se a credencial que ele acusa como correta é correta mesmo, o que pode comprometer seu teste.
- Ferramenta Hydra foi efetiva, porém tive dificuldade inicial com sua sintaxe. Parece uma ferramenta mais adequada para o Brute Force em serviços HTTP.

 ### Ataque em Cadeia de Usuários de Servidor Simulado com Password Spraying

5.1 - Enumeração de Usuários com Enum4linux:
- Enumeração SMB (Server Message Block)
  - Protocolo da Microsoft para compartilhar arquivos, pastas, autenticação de pessoas e comunicação de máquinas windows e linux via Samba
  - Porta de Entrada para compartilhar recursos em redes internas
- Comando para enumeração de usuários com enum4linux: ```enum4linux -a 192.168.56.101 | tee enum4_output.txt```
  - Nesse comando acima o enum4linux vai enumerar diversas informações do sistema, entre elas a lista de usuários do servidor simulado; O comando vai armazenar o resultado da enumeração no arquivo "enum4_output.txt";
- Ao consultar o arquivo txt, conferimos a lista de usuários acomapnhada de seus códigos RID, relativo ao sistema remoto;
  - Nomes de usuários genéricos são bons candidatos para ataque de password spraying;

[![enum4-output.png](https://i.postimg.cc/KvjH1sT6/enum4-output.png)](https://postimg.cc/XpM8hLQL)
 
5.2 - Ataque de Password Spraying Utilizando o Medusa:
- O spraying é uma técnica furtiva de ataque a senhas;
- Vai testar poucas senhas mas em muitos usuários diferentes;
- Primeiro são criados as wordlists;
- Em seguida para o password spraying pela Medusa utilizei o seguinte comando:
  ```medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50```
  - Após o -h segue o ip do nosso alvo, após -U e -P são as wordlists, -M é o módulo atacado (smb), -t 2 indica duas threads simultâneas, -T até 50 usuários com testes de credenciais
  - O spraying vai testar combinações de credenciais por usuários, fazendo checagens e acusando alguma "Account Found"

[![medusa-spray.png](https://i.postimg.cc/Yq0svygc/medusa-spray.png)](https://postimg.cc/8j8H34tw)
 
5.3 - Validando o acesso obtido com o Password Spraying:
- Executar o comando:
  ```smbclient -L //192.168.56.101 -U msfadmin```
  - Inserir as credenciais obtidas no terminal, validando o acesso

 5.4 - Conclusões do teste:
 - Muito eficaz para sistemas com login sem multifator e usuários genéricos;
 - Parece eficaz principalmente para credenciais com senhas fracas e usuários inativos dentro de um servidor;

