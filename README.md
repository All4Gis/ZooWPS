# ZooWPS
All the instructions for the installation of a Zoo project WPS in Ubuntu with NGINX Serving CGI Scripts.
 
##Install all the dependencies (Ubuntu 14.04 LTS):

```
sudo apt-get install flex bison libfcgi-dev libxml2 libxml2-dev 
sudo apt-get install curl openssl autoconf  python-software-properties subversion python-dev build-essential
sudo apt-get install gdal-bin python-gdal libgdal1-dev libproj-dev libmozjs185-dev
sudo apt-get install libxslt1-dev python-libxslt1
sudo apt-get install libcgic2
sudo add-apt-repository ppa:ubuntugis/ppa

sudo apt-get update

sudo apt-get install libgdal1-dev
```

##Download ZOO-Project:

```
svn checkout http://svn.zoo-project.org/svn/trunk zoo-project
```
 
Before proceeding with the installation you must configure Nginx Serving CGI Scripts

Add this to:  /etc/nginx/sites-available/<project_name> this:

```
   location /cgi-bin/ {
     # Disable gzip (it makes scripts feel slower since they have to complete
     # before getting gzipped)
     gzip off;
     # Set the root to /usr/lib (inside this location this means that we are
     # giving access to the files under /usr/lib/cgi-bin)
     root /srv/www/<project_name>/public_html;
     # Fastcgi socket
     fastcgi_pass  unix:/var/run/fcgiwrap.socket;
     # Fastcgi parameters, include the standard ones
     include /etc/nginx/fastcgi_params;
     # Adjust non standard parameters (SCRIPT_FILENAME)
     fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
   }
```

Restart the NGINX service:

```
sudo service nginx restart
``` 
 
##Test the NGINX service:
```
sudo chmod 777 /usr/lib/cgi-bin
nano /usr/lib/cgi-bin/hello.py
```
Create the file:
```
#!/usr/bin/env python
print "Content-type: text/html\n\n"
print "<h1>Hello World</h1>"
```

Give permission to this file:
```
sudo chmod +x /usr/lib/cgi-bin/hello.py
```

###Test it:
```
http://localhost/cgi-bin/hello.py 
```
 
Install the cgic library from packages using the following command:
``` 
cd zoo-project/thirds/cgic206/
sed "s:lib64:lib:g" -i Makefile 
make
```

Head to the ZOO-Kernel directory:
```
cd ../../zoo-project/zoo-kernel/
```
Create a configure file as follow:
```
autoconf
```
Run configure with the desired options, for example with the following command:
```
./configure ./configure --with-js --with-python --with-xsltconfig=/usr/bin/xslt-config --with-gdal-config=/usr/bin/gdal-config --with-proj 
```

Compile ZOO-Kernel as follow:
```
make
```
Install the libzoo_service.so.1.5 by using the following command:
```
sudo make install
```
Copy the necessary files to the cgi-bin directory (as administrator user):
```
sudo cp main.cfg /srv/www/<project_name>/public_html/cgi-bin
sudo cp zoo_loader.cgi /srv/www/<project_name>/public_html/cgi-bin
```
Install ZOO ServiceProviders, for example the basic Python service (as administrator user):
```
cp ../zoo-services/hello-py/cgi-env/*.zcfg /srv/www/<project_name>/public_html/cgi-bin/
cp ../zoo-services/hello-py/cgi-env/test_service.py /srv/www/<project_name>/public_html/cgi-bin/    (this step is not correct in the official documentation)
cd ../zoo-services/ogr/base-vect-ops  (this step is not correct in the official documentation)
```

Make directories:
```
mkdir -p /srv/datos/zoo/data/tmp
sudo chmod 777 /srv/datos
```

Edit the main.cfg file as follow:
```
nano /srv/www/<project_name>/public_html/lib/cgi-bin/main.cfg
- serverAddress = http://127.0.0.1
 
dataPath=/srv/datos/zoo/data
tmpPath=/srv/datos/zoo/data/tmp
```
Restart the NGINX service:
```
sudo service nginx restart
```
 
Test the ZOO-Kernel installation with the following requests:
```

http://localhost:2222/cgi-bin/zoo_loader.cgi?ServiceProvider=&metapath=&Service=WPS&Request=GetCapabilities&Version=1.0.0
```

HelloPy
```
http://localhost:2222/cgi-bin/zoo_loader.cgi?request=Execute&service=WPS&version=1.0.0&Identifier=HelloPy&DataInputs=a=Djay
```
 
###Compilation of the Gdal Elevations profile (Gdal):
```
cd /home/<ubuntu_user>/zoo-project/zoo-project/zoo-services/gdal/profile  
make

sudo cp /home/<ubuntu_user>/zoo-project/zoo-project/zoo-services/gdal/profile/cgi-env/* /srv/www/<project_name>/public_html/cgi-bin

```
Copy your *.tiff in the dataPath:
```
dataPath=/srv/datos/zoo/data
```
 
###Test it
```
 
http://localhost:2222/cgi-bin/zoo_loader.cgi?request=Execute&service=WPS&version=1.0.0&Identifier=GdalExtractProfile&DataInputs=RasterFile=madrid.tif@dataType=string;Geometry={%22type%22:%22LineString%22,%22coordinates%22:[[400545.964382,4484372.9664],[427201.757392,4501863.54839]]}@dataType=string
```

