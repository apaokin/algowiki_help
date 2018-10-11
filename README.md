Инструкция по установке Algowiki на Ubuntu 18. Далее полагаю,что у Вас есть пользователь на сервере и доступ по ssh. Хост прописан в .ssh/config и назван algo.  

Для mediawiki необходим веб-сервер: nginx или apache. На боевом сервере поставлен apache, в инструкции ставить будем nginx(c ним проблем не будет гарантированно).

Скопируем себе в домашнюю директорию один из последних бекапов на сервере(они делаются раз в неделю). Присмотрите себе бекап посвежее, скопируйте себе в директорию и заархивируйте сам движок медиавики с дополнения:

	sudo su - root
	cd wiki_backup
	cp -r wiki_backup/2018_10_08-1538949661/ ~
	tar -cvzf ~/srv.tar.gz  /srv/mediawiki/
	exit
	exit

Далее скачиваем. Можно воспользоваться sftp:

	sftp algo
	get -r 2018_10_08-1538949661
	get -r srv.tar.gz
	exit

Распаковываем:

	sudo tar -xvf wiki.tar.gz -C  /
	mkdir /srv/mediawiki
	get -r 2018_10_08-1538949661
	sudo tar -xvf srv.tar.gz  -C  /
	/var/www/algowiki/ru/set-symlinks.sh 	
	/var/www/algowiki/en/set-symlinks.sh 	

Ставим нужные пакеты:

	sudo apt-get install nginx mysql-server php php-fpm php-apcu php-mysql

Правим  nginx(/etc/nginx/sites-available/default), добавляя блок server из файла в этом репозитории.

Важно прописать актуальный сокет из пакета php-fpm здесь:

	fastcgi_pass unix:/run/php/php7.0-fpm.sock;

Чтобы изменения вступили в силу, перезапустим nginx:
	sudo service nginx restart

Создаем в mysql пользователя algo и даём ему пароль(здесь он будет your_password):

	sudo mysql
	CREATE USER 'algo'@'localhost' IDENTIFIED BY 'your_password';
	GRANT ALL PRIVILEGES ON * . * TO 'algo'@'localhost';
	FLUSH PRIVILEGES;
	exit;

Входим под новым пользователем:

	mysql -u algo -p # далее вводим пароль
    Create database algowiki_ru;
	Create database algowiki_en;
	Create database algowiki_pool;

Далее из каталога с бекапом(wiki_backup/2018_10_08-1538949661/):

	gunzip algowiki_en.sql.gz &&  gunzip algowiki_ru.sql.gz &&  gunzip algowiki_pool.sql.gz  
	mysql -u algo  -p algowiki_ru < algowiki_ru.sql
	mysql -u algo  -p algowiki_pool < algowiki_pool.sql
	mysql -u algo  -p algowiki_en < algowiki_en.sql

Дальше в файлах /var/www/algowiki/ru/LocalSettings.php и /var/www/algowiki/en/LocalSettings.php изменяем значения переменных:

	$wgScriptPath       = "/ru"; и $wgScriptPath       = "/en";
	$wgDBuser           = "algo"; и $wgDBuser           = "algo";
	$wgDBpassword       = "your_password"; и $wgDBpassword       = "your_password";
	$wgMainCacheType = CACHE_NONE; и $wgMainCacheType = CACHE_NONE;


Коммиты и Issues reports приветcтвуются.
