Installation Steps -
=================
Download and install XAMPP
------------------------------------------------------------------------------------
https://www.apachefriends.org/download.html 

Enable PHP extensions & Configure php.ini [PHP] 
------------------------------------------------------------------------------------
Enable below extensions in the xampp/php/php.ini file:
;extension=gd 
;extension=intl 
;extension=soap 
;extension=xsl 
;extension=sockets 
;extension=sodium
Change the limit of below keys in the xampp/php/php.ini file:
max_execution_time=3000 
max_input_time=2000 
memory_limit=3G 

Restart XAMPP 
------------------------------------------------------------------------------------

Download and install 
------------------------------------------------------------------------------------
From this website - https://getcomposer.org/download/ 

Download and run Elasticsearch 
------------------------------------------------------------------------------------
https://www.elastic.co/elasticsearch/
 
Server configuration (Create virtual host) 
------------------------------------------------------------------------------------
C:\xampp\apache\conf\extra\httpd-vhosts.conf 
Add the below lines in the above file.
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs/mage2rock/pub"
    ServerName mage2rock.magento.com
</VirtualHost>
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs"
    ServerName localhost
</VirtualHost>

Then open the C:\Windows\System32\drivers\etc\hosts file in Notepad and add the below line at the bottom of the file.
127.0.0.1 mage2rock.magento.com

Restart XAMPP.

Download Magento
------------------------------------------------------------------------------------
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition mage2rock 
Public Key(username):  (Copy from Magento website)
Private Key(password): Copy from Magento website

Create Database in phpmyadmin
------------------------------------------------------------------------------------
Add the below line in this file - C:\xampp\htdocs\elasticsearch-8.7.0\config\elasticsearch.yml

xpack.security.enabled: false

Restart elastic-search.

Check elastic-search is working properly - curl -X GET 'http://localhost:9200'

Go to project root directory - cd mage2rock
------------------------------------------------------------------------------------
Install Magento  
php bin/magento setup:install --base-url=http://mage2rock.magento.com --db-host=localhost --db-name=magento --db-user=root --db-password=  --admin-firstname=admin  --admin-lastname=admin  --admin-email=admin@admin.com  --admin-user=admin  --admin-password=admin123  --language=en_US  --currency=USD  --timezone=America/Chicago --use-rewrites=1 --search-engine=elasticsearch8 --elasticsearch-host="localhost" --elasticsearch-port=9200 --elasticsearch-enable-auth=true

Find the validateURLScheme function in
------------------------------------------------------------------------------------
vendor\magento\framework\Image\Adapter\Gd2.php file. at line 70. 
------------------------------------------------------------------------------------
Replace the function with below code: 
------------------------------------------------------------------------------------
private function validateURLScheme(string $filename) : bool
{
          $allowed_schemes = ['ftp', 'ftps', 'http', 'https'];
          $url = parse_url($filename);
          if ($url && isset($url['scheme']) && !in_array($url['scheme'], $allowed_schemes) && !file_exists($filename)) {
              return false;
          }

          return true;   
}

Go to: C:\xampp\htdocs\magento2.4\vendor\magento\framework\View\Element\Template\File - > Edit Validator.php using a text editor and find this line:
------------------------------------------------------------------------------------
Find this line
strpos($realPath, $directory) 
Replace
strpos($path, $directory)

 Then, Open up app/etc/di.xml in the editor, find the below line, and replace:
------------------------------------------------------------------------------------
 “Magento\Framework\App\View\Asset\MaterializationStrategy\Symlink” replace
 “Magento\Framework\App\View\Asset\MaterializationStrategy\Copy” 

Now Execute Below Command:
------------------------------------------------------------------------------------
php bin/magento indexer:reindex 
php bin/magento setup:upgrade 
php bin/magento setup:static-content:deploy -f 
php bin/magento cache:flush 
php bin/magento module:disable Magento_TwoFactorAuth 

In \xampp\apache\conf\extra\httpd-xampp.conf
------------------------------------------------------------------------------------
Find:
LoadFile "/xampp/php/php7ts.dll"
LoadFile "/xampp/php/libpq.dll"
LoadModule php7_module "/xampp/php/php7apache2_4.dll"
Add:
LoadFile "/xampp/php/libsodium.dll"
