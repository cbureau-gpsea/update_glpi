# Update GLPI on Debian

This README.md is an operating mode to correctly update GLPI without loss of data. Just in case, make a backup of your server.

## Enable maintenance

To start, enable the maintenance mode. Go to the GLPI folder, generally (*/var/www/html/glpi*) :
```bash
sudo -i

cd /var/www/html/glpi

sudo php bin/console glpi:maintenance:enable
```
You can still access your site by adding at the end of your link :
```bash
?skipMaintenance=1
```

## Save data

Now, save the database (you need mysql root password) :
```bash
mkdir /var/www/backup_glpi/

mysqldump -u root -p --databases [name_of_your_database] > /var/www/backup_glpi/backup_db_glpi.sql
```

And save the configuration files :
```bash
cp -Rf /var/www/html/glpi/files /var/www/backup_glpi/

cp -Rf /var/www/html/glpi/plugins /var/www/backup_glpi/

cp -Rf /var/www/html/glpi/config /var/www/backup_glpi/

cp -Rf /var/www/html/glpi/marketplace /var/www/backup_glpi/

cp -Rf /var/www/html/glpi/pics /var/www/backup_glpi/
```

## Download GLPI

Before downloading the new version, check compatibility, if you have errors, download the missing extensions : 
```bash
cd /var/www/html/glpi

php bin/console glpi:system:check_requirements
```

After that, delete the GLPI folder :
```bash
rm -Rf /var/www/html/glpi
```

And install the latest version of GLPI :
```bash
cd /tmp

wget https://github.com/glpi-project/glpi/releases/download/10.0.X/glpi-10.0.X.tgz #replace 10.0.X with the latest version

tar -xzvf glpi-10.0.X.tgz #replace 10.0.X with the latest version
```

Move the new folder of GLPI in */var/www/html* :
```bash
mv glpi /var/www/html
```

## Reinject configuration

Put the configuration files back into new GLPI :
```bash
cp -Rf /var/www/backup_glpi/files /var/www/html/glpi

cp -Rf /var/www/backup_glpi/plugins /var/www/html/glpi 

cp -Rf /var/www/backup_glpi/config  /var/www/html/glpi

cp -Rf /var/www/backup_glpi/marketplace /var/www/html/glpi

cp -Rf /var/www/backup_glpi/pics /var/www/html/glpi

chown -R www-data:www-data /var/www/html/glpi/
```

## Update database

Now, check the prerequisites for updating the database :
```bash
cd /var/www/html/glpi

php bin/console glpi:system:check_requirements

php bin/console db:update
```

## Check your plugins

Check if your plugins work, after activating them with this link (*https://[your_link]/front/plugin.php*). To reinstall SSO, here is the procedure :
```bash
wget https://github.com/derricksmith/phpsaml/archive/refs/tags/1.2.1.tar.gz

tar -xzvf 1.2.1.tar.gz

mv phpsaml-1.2.1 phpsaml

nano /var/www/html/glpi/plugins/phpsaml/phpsaml.xml #change the next line by your version: <compatibility>~10.0.X</compatibility>
```
## Disable maintenance

Now, you can disable the maintenance mode, the update has been carried out :
```bash
cd /var/www/glpi

php bin/console glpi:maintenance:disable
```
