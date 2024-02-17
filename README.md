# GLPI-Install
## Script de Instalação do GLPI

### Video Referencia:

[GLPI - Instalação passo a passo | MasterMindTI](https://youtu.be/_OjuhFMcXfA)

## Vamos para o Script ?
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/bash/bash-original.svg" width="40" height="40"/> <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/linux/linux-original.svg" width="40" height="40"/> <img src="https://raw.githubusercontent.com/glpi-project/glpi/master/pics/logos/logo-GLPI-250-black.png" width="60" height="60"/>
                

####################################################################
## 1. Pré-Requisitos
####################################################################
- 1.1. Servidor Ubuntu 22.04 LTS Instalado
- 1.2. Acesso a internet para download e instalação de pacotes
- 1.3. Acesso de root ao servidor

####################################################################
## 2. Instalação
####################################################################
- 2.1. Atualizar
```
sudo apt update && sudo apt dist-upgrade -y
```

- 2.2. Reconfigurar timezone
```
sudo dpkg-reconfigure tzdata
```
- 2.3. Reiniciar
```
sudo reboot
```

- 2.4. Instalar Apache, PHP e MySQL
```
sudo apt install -y \
	apache2 \
	mariadb-server \
	mariadb-client \
	libapache2-mod-php \
	php-dom \
	php-fileinfo   \
	php-json \
	php-simplexml \
	php-xmlreader \
	php-xmlwriter \
	php-curl \
	php-gd \
	php-intl \
	php-mysqli   \
	php-bz2  \
	php-zip \
	php-exif \
	php-ldap  \
	php-opcache \
	php-mbstring
```

- 2.5. Criar banco de dados do glpi
```
sudo mysql -e "CREATE DATABASE glpi"
sudo mysql -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost' IDENTIFIED BY 'P4ssw0rd'"
sudo mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost'"
sudo mysql -e "FLUSH PRIVILEGES"
```
- 2.6. Carregar timezones no MySQL
```
mysql_tzinfo_to_sql /usr/share/zoneinfo | sudo mysql -u root mysql
```

- 2.7. Desabilitar o site padrão do apache2
```
sudo a2dissite 000-default.conf
```

- 2.8. Habilita session.cookie_httponly
```
sudo sed -i 's/^session.cookie_httponly =/session.cookie_httponly = on/' /etc/php/8.1/apache2/php.ini && \
	sudo sed -i 's/^;date.timezone =/date.timezone = America\/Sao_Paulo/' /etc/php/8.1/apache2/php.ini
```	

- NOTA: Caso de algum erro, as vezes pode ser que a versão do PHP no caso 8.1 pode ter sido mudada, ai para poder localizar a pasta do php.ini, execute o comando abaixo:

```	
sudo find /etc/php -type f -name "php.ini"
```	


- 2.9. Criar o virtualhost do glpi
```
cat << EOF | sudo tee /etc/apache2/sites-available/glpi.conf
<VirtualHost *:80>
	ServerName glpi.ninjapfsense.com.br
	DocumentRoot /var/www/glpi/public
	<Directory /var/www/glpi/public>
		Require all granted
		RewriteEngine On
		- Redirect all requests to GLPI router, unless file exists.
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteRule ^(.*)$ index.php [QSA,L]
	</Directory>
</VirtualHost>
EOF
```

- 2.10. Habilitar o virtualhost
```
sudo a2ensite glpi.conf
```

- 2.11. Habilitar módulos do Apache necessários
```
sudo a2enmod rewrite
```

- 2.12. Reinicia o apache para entrar em vigor
```
sudo systemctl restart apache2
```

- 2.13. Download do glpi
```
wget -q https://github.com/glpi-project/glpi/releases/download/10.0.7/glpi-10.0.7.tgz
```

- 2.14. Descompactar a pasta do GLPI
```
tar -zxf glpi-*
```

- 2.15. Mover a pasta do GLPI para a pasta htdocs
```
sudo mv glpi /var/www/glpi
```

- 2.16. Configura a permissão na pasta www/glpi
```
sudo chown -R www-data:www-data /var/www/glpi/
```

- 2.17. Finalizar setup do glpi pela linha de comando
```
sudo php /var/www/glpi/bin/console db:install \
	--default-language=pt_BR \
	--db-host=localhost \
	--db-port=3306 \
	--db-name=glpi \
	--db-user=glpi \
	--db-password=P4ssw0rd
```


####################################################################
## 3. Ajustes de Segurança
####################################################################
- 3.1. Remover o arquivo de instalação
```
sudo rm /var/www/glpi/install/install.php
```

- 3.2. Mover pastas do GLPI de forma segura 
```
sudo mv /var/www/glpi/files /var/lib/glpi
sudo mv /var/www/glpi/config /etc/glpi
sudo mkdir /var/log/glpi && sudo chown -R www-data:www-data /var/log/glpi
```

- 3.3. Mover pastas do GLPI de forma segura | conf-dir
```
cat << EOF | sudo tee /var/www/glpi/inc/downstream.php
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
   require_once GLPI_CONFIG_DIR . '/local_define.php';
}
EOF
```

- 3.4. Mover pastas do GLPI de forma segura | data dir
```
cat << EOF | sudo tee /etc/glpi/local_define.php
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
EOF
```

####################################################################
## 4. Primeiros Passos
####################################################################
- 4.1. Acessar o GLPI via web browser
- 4.2. Criar um novo usuário com perfil super-admin
- 4.3. Remover os usuários glpi, normal, post-only, tech.
- 4.3.1. Enviar os usuários para a lixeira
- 4.3.2. Remover permanentemente
- 4.3.4. Configurar a url de acesso ao sistema em: Configurar -> Geral -> Configuração Geral -> URL da aplicação.

####################################################################
## Informações Adicionais:
#### Citra IT - Excelência em TI
#### Procedimento: INSTALAÇÃO GLPI
#### @Responsável: luciano@citrait.com.br
#### @Data: 13/06/2023
#### @Versão: 1.0
#### @Homologado: Ubuntu 22.04
####################################################################

### Video Referencia:

[GLPI - Instalação passo a passo | MasterMindTI](https://youtu.be/_OjuhFMcXfA)

### Qualquer dúvida, entre em contato.

e-mail: contato@mastermindti.com.br

## Contatos:

<div>
<a href="https://www.youtube.com/@mastermindti" target="_blank"><img src="https://img.shields.io/badge/YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white" target="_blank"></a>
<a href = "mailto:contato@mastermindti.com.br"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white" target="_blank"></a>
<a href="https://www.linkedin.com/in/luciano-rodrigues-676691159/" target="_blank"><img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white" target="_blank"></a>   
</div>

## Estatisticas

<div>
<a href="https://github.com/MasterMindTI">
<img height="180em" src="https://github-readme-stats.vercel.app/api?username=MasterMindTI&show_icons=true&theme=dracula&include_all_commits=true&count_private=true"/>
</div>

