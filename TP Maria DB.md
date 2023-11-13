---


---

<h1 id="tp-maria-bd">TP Maria BD</h1>
<ol start="2">
<li>Desactiver firewall AlmaLinux : <code>sudo systemctl stop firewalld</code></li>
<li>
<ul>
<li>Installer le paquet mariadb-server: <code>sudo yum install mariadb-server</code></li>
<li>Installer le paquet mariadb : <code>sudo yum install mariadb</code><br>
Avec la commande <code>repoquery --list mariadb</code>, on voit le contenu du paquet :</li>
</ul>
</li>
</ol>
<blockquote>
<p>AlmaLinux 9 - AppStream                                                                                                                                                                                1.6 kB/s | 4.1 kB     00:02<br>
AlmaLinux 9 - BaseOS                                                                                                                                                                                   7.0 kB/s | 3.8 kB     00:00<br>
AlmaLinux 9 - Extras                                                                                                                                                                                   7.2 kB/s | 3.8 kB     00:00<br>
Extra Packages for Enterprise Linux 9 - x86_64                                                                                                                                                          63 kB/s |  33 kB     00:00<br>
Extra Packages for Enterprise Linux 9 - x86_64                                                                                                                                                         6.9 MB/s |  19 MB     00:02<br>
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                                                                                                                                   1.6 kB/s | 2.5 kB     00:01</p>
</blockquote>
<ul>
<li>Ce paquet est un <strong>SGBDD</strong> qui permet d’effectuer des opérations sur une base de données MariaDB. La différence avec mySQL est que MariaDB est beaucoup plus rapide et performant.</li>
</ul>
<ol start="4">
<li>
<ul>
<li>C’est le fichier <code>/etc/my.cnf</code></li>
<li>C’est un format <strong>ASCII text</strong> avec <code>file my.cnf</code></li>
<li>La particularité est que ce fichier permet de spécifier des paramètres liés à la configuration du serveur : taille de la mémoire tampon, la taille du cache, le nombre maximum de connexions, etc.</li>
<li>Les BDD sont stockées dans le répertoire : <code>/var/lib/mysql/</code></li>
<li>Cet emplacement n’est pas pertinent car le répertoire <strong>/var/</strong> est dans le même répertoire que dans le système donc si le système est touché, les BDD le seront aussi.</li>
<li>Nous devons utiliser RAID ou LVM pour une configuration de production. avec cela, les processus de MariaDB sont forkés et cela permet d’avoir une redondance au niveau des processus et une tolérance aux pannes.</li>
</ul>
</li>
</ol>
<hr>
<ol start="5">
<li>
<ul>
<li>Démarrer le serveur mariadb : <code>sudo systemctl start mariadb</code> puis on s’authentifie avec un compte utilisateur</li>
<li>On vérifie que le port est bien à l’écoute avec sur le port <strong>3306</strong> <code>sudo ss -tulnp</code></li>
<li>Il écoute sur l’adresse <strong>0.0.0.0</strong></li>
</ul>
</li>
<li>
<ul>
<li>Joindre le serveur MariaDB : <code>sudo mariadb</code> ou <code>sudo mysql</code></li>
<li>Cela permet de choisir le protocole de connexion : (<em>{TCP|SOCKET|PIPE|MEMORY}</em>)</li>
<li>On peut se connecter : sois en <strong>SOCKET</strong>, donc depuis la machine et par défaut avec le compte root de mysql <code>sudo mariadb [-u root]</code> ou alors en <strong>TCP/IP</strong> si on est sur une machine distante ou même depuis la machine du serveur sur l’IP loopback. Par défaut, l’user <em>root</em> est rejeté à part si on joint la BDD directement sur la loopback : <code>sudo mariadb [-u root] -h localhost</code>. La <strong>limite</strong> est que l’utilisateur <em>root</em> est bloquée par défaut.</li>
<li>C’est le protocole <strong>SOCKET</strong>, on le prouve car on peut se connecter avec l’user <em>root</em> à la BDD</li>
</ul>
</li>
</ol>
<hr>
<ol start="7">
<li>
<ul>
<li>Le compte administrateur par défaut est <em>root</em></li>
<li>On essaye de se connecter avec <code>sudo mariadb -h 172.18.169.255</code> mais on a l’erreur <strong>ERROR 2002 (HY000): Can’t connect to MySQL server on ‘172.18.169.255’ (115)</strong></li>
<li>
<ul>
<li>Avec l’adresse : <code>sudo mariadb -h 127.0.0.1</code> et on obtient l’erreur <strong>ERROR 1698 (28000): Access denied for user ‘root’@‘localhost’</strong>. Mais avec la commande <code>sudo mysql -h localhost -u root</code> cela fonctionne.
<ul>
<li>On utilise la commande <code>status</code> pour vérifier</li>
<li><code>sudo mysql -h localhost -u root</code></li>
</ul>
</li>
</ul>
</li>
<li>Cela fonctionne pas avec 127.0.0.1 mais avec localhost oui</li>
<li>La sécurité repose sur les <strong>permissions</strong> des utilisateurs. Il y a un risque qu’un utilisateur arrive à <strong>usurper</strong> un compte avec des <strong>privilèges admin</strong> ou alors qu’un pirate arrive à faire une <strong>élévation de privilège</strong> via une attaque informatique type injection SQL</li>
<li><code>sudo mysql_secure_installation</code></li>
</ul>
</li>
</ol>
<hr>
<ol start="8">
<li>
<ul>
<li>
<p><code>&gt; CREATE DATABASE worlddb;</code></p>
</li>
<li>
<p><code>&gt; SHOW DATABASES;</code></p>
</li>
<li>
<p>Après avoir fait un glisser déposer dans MobaXterm : <code>$ sudo mysql -p worlddb &lt; /home/User/WORLDDB-FINAL-UTF8.sql</code></p>
</li>
<li>
<p>Ensuite, pour vérifier que le script a bien fonctionner, on fait <code>sudo mariadb</code>, on rentre dans la base de donnée avec <code>USE worlddb</code> et on fait <code>SHOW TABLES;</code>. les tables présentes sont : *<em>city, country, countrylanguage</em></p>
</li>
<li>
<p><code>&gt; SELECT count(*) FROM country;</code> et on trouve <strong>239 entrées</strong></p>
</li>
<li>
<p><code>&gt; SHOW CREATE DATABASE nomDeTable;</code> et on fait cela pour les 3 tables</p>
</li>
<li>
<p>Après avoir glisser deposer le dossier dans la VM, on rentre dans <strong>mysql</strong>, on <strong>créer</strong> une nouvelle BDD <code>CREATE DATABASE sakila;</code>, puis on <strong>utilise</strong> cette BDD <code>USE sakila;</code>, on créer le <strong>schéma</strong> de la BDD <code>source /home/User/sakila-db/sakila-schema.sql</code> et on y affecte les <strong>données</strong> <code>source /home/User/sakila-db/sakila-data.sql</code></p>
</li>
<li>
<p>On fait la commande : <code>SELECT * FROM actor LIMIT 5;</code> et on obtient :</p>
<p>|actor_id | first_name | last_name    | last_update         |<br>
±---------±-----------±-------------±--------------------+ |        1 | PENELOPE   | 				GUINESS      | 2006-02-15 04:34:33 | |        2 | NICK       | WAHLBERG     | 2006-02-15 04:34:33 | |        3 | ED      | CHASE        | 2006-02-15 04:34:33 | |        4 | JENNIFER   | DAVIS | 2006-02-15 04:34:33 | |        5 | JOHNNY     | LOLLOBRIGIDA | 2006-02-15 04:34:33 ||  |</p>
</li>
<li></li>
</ul>
</li>
</ol>

