# GPCRMD_VIRTUAL_MACHINE_SERVER

## HOW TO CREATE A VIRTUAL MACHINE TO INSTALL GPCRMD SERVER:

Download the latest version of [Virtualbox](https://www.virtualbox.org/wiki/Downloads)

Open virtualbox:
```
virtualbox
```

Create a CentOS7 machine in virtualbox. In this [link](https://www.saltos.org/portal/en/wiki/view/358/installing-centos-7-in-virtualbox) you have a tutorial that you can follow. 
Give at least 50GB of memory to your virtual machine.

Start your virtual machine:
give internet connexion and change the keyboard to a convenient one in the installation step. In addition, create a user with all permisions. 
 
I also recommend to activate the GNOME graphical interface.

Once we have started the virtual machine the mouse will be stuck to the virtual machine. Using ctrl (the right one on the keyboard we will be able to return the mouse to our graphical interface). In addition using ctrl (right one) + f we will be able to use full screen.

We install guest additions. Here you have a [tutorial](https://linuxconfig.org/how-to-install-virtualbox-guest-additions-on-centos-7-linux) on how to install and run it.

To make sure that we have properly installed the CentOS7 machine in virtualbox just type in terminal:

```
cat /etc/redhat-release 
```
It should indicate that you have CentOS Linux release 7.*.* (Core)

### MINICONDA ENVIRONMENT

Download the miniconda environment that you have to use for the GPCRmd server from [here](https://drive.google.com/file/d/1CfuIF9nuCafYJLvtOtyh4gKxyhyvoOJt/view?usp=sharing)

Uncompress the file and copy the miniconda3 folder to /opt:
```
cp miniconda3 /opt/miniconda3
```
And activate the environment:

```
export PATH=/opt/miniconda3/bin:$PATH
source activate
```
## APACHE CONFIGURATION 

We install apache:

```
yum install httpd httpd-devel -y
systemctl enable httpd.service
```
and we add the configuration of apache (explined how to do in the file: [apache_conf.txt](https://github.com/OriolCanal/GPCRmd_server_VirtualMachine/blob/main/apache_conf.txt))

## PACKAGES INSTALLATION
Install the following packages using yum:
```

sudo yum install epel-release centos-release-scl -y

sudo yum -y install cmake sqlite sqlite-devel tcl-devel tk-devel readline-devel bzip2-devel libtiff-devel freetype-devel libwebp-devel lcms2-devel cairo mod_xsendfile openblas-threads git openbabel expect htop wget clustal-omega perl-Archive-Tar perl-Digest-MD5 perl-List-MoreUtils argtable argtable-devel java-1.8.0-openjdk java-1.8.0-openjdk-devel logrotate curl PyYAML libyaml libyaml-devel libffi libffi-devel libjpeg-turbo-devel zlib zlib-devel libicu libicu-devel perl cmake sqlite sqlite-devel tcl-devel tk-devel readline-devel bzip2-devel libtiff-devel freetype-devel libwebp-devel lcms2-devel cairo
```
## NCBI BLAST

```

wget  https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.11.0/ncbi-blast-2.11.0+-1.x86_64.rpm
sudo yum localinstall ncbi-blast-2.11.0+-1.x86_64.rpm -y
```

## POSTGRESQL INSTALLATION FROM POSTGRES WEBPAGE
Here you have a [link to go to the postgresql webpage](https://www.postgresql.org/download/linux/redhat) where you will find all the instructions to create a postgreSQL Yum repository. Install version 10 of postgresql. 

Here I put the instructions from the link to Install version 10:

```
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql14-server
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14

```

## Install boost >1.58:
```
conda install boost==1.61.0
```

## RDkit

RDKit is a collection of cheminformatics and machine-learning software written in C++ and Python. We can install it executing these commands. We need to have the miniconda environment loaded:

```
wget https://github.com/rdkit/rdkit/archive/Release_2016_03_1.tar.gz
tar -zxvf Release_2016_03_1.tar.gz
cd rdkit-Release_2016_03_1/
export RDBASE=$(pwd)
mkdir build
cd build
source activate 
cmake -D RDK_BUILD_SWIG_WRAPPERS=OFF -D PYTHON_EXECUTABLE=/opt/miniconda3/bin/python -D PYTHON_LIBRARY=/opt/miniconda3/lib/libpython3.so -D PYTHON_INCLUDE_DIR=/opt/miniconda3/include/python3.4m -D RDK_BUILD_AVALON_SUPPORT=ON -D RDK_BUILD_INCHI_SUPPORT=ON -D BOOST_ROOT=/opt/miniconda3 -D PYTHON_INSTDIR=/opt/miniconda3/lib/python3.4/site-packages -DRDK_INSTALL_INTREE=OFF -DCMAKE_INSTALL_PREFIX=/usr/local/rdkit ..
make -j2
sudo make install -j2
```


## GENERATING CERTIFICATES

Create a directory where the certificates will be stored:

```
mkdir certificats
cd certificats
```

```
openssl genrsa -des3 -out server.key 1024
```

We will be asked to enter a pass phrase.



```
openssl req -new -key server.key -out server.csr
```
We enter pass phrase (whatever)
Contry name : ES
State or province name : Bcn
Locality name: Bcn
Organization name: GPCR Lab
Organizational Unit name: GPCR
Common name: gpcr.test
Email adress: xxx@gpcr.test
challange password: whatever2022

```
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
```
Again we will be asked to enter a pass phrase

```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
We have generate the certificate : gpcr.crt
and the key: gpcr.key

We copy these files to the conf of apache:

```
cp server.crt /etc/pki/tls/certs/gpcr.crt
cp server.key /etc/pki/tls/private/gpcr.key
```

Ìnstall some packages:
```
yum install openssl-devel
yum install openssl -y
yum install httpd24-mod_ssl
yum install mod_ssl
```

Now in /etc/pki/tls/certs should appear the ssl.conf file

Edit the ssl.conf file using a text editor and include where the certificate (SSLCertificateFile /etc/pki/tls/certs/gpcr.crt) and the key (SSLCertificateKeyFile /etc/pki/tls/private/gpcr.key) can be found. You just have to replace the 2 paragraphs that you have below in the ssl.conf file:

#Server Certificate:  
#Point SSLCertificateFile at a PEM encoded certificate. If     
#the certificate is encrypted, then you will be prompted for a  
#pass phrase.  Note that a kill -HUP will prompt again.  A new  
#certificate can be generated using the genkey(1) command.  
SSLCertificateFile /etc/pki/tls/certs/gpcr.crt  


  
  
    
    
#Server Private Key:  
#If the key is not combined with the certificate, use this  
#directive to point at the key file.  Keep in mind that if  
#you've both a RSA and a DSA private key you can configure  
#both in parallel (to also allow the use of DSA ciphers, etc.)  
SSLCertificateKeyFile /etc/pki/tls/private/gpcr.key  


Finally, you just have to save the file.




## TRANSFER OF GPCRMD FILES

We will transfer a subset of simulations as the virtual machine won't be able to support the whole dataset.

We can download the gpcrmd server from [here](https://github.com/GPCRmd/gpcrdb).

You will need permissions to see and download the contents of the previous github repository.

You download all the data from the github repository.


Create the following directories:

/protwis/sites/protwis

and give permissions to edit their content:

```
chmod 777 /protwis/
```


Unizp the github repository compressed file And store all the unziped files into the /protwis/sites/protwis/ directory:

```
unzip gpcrdb-dev.zip
cd gpcrdb-dev
mv -r * /protwis/sites/protwis/
```

Create a  directory into /protiwis/sites named files where we will store a subset of the GPCRmd database files:
```
cd /protwis/sites
mkdir files
cd files
```

Now we have to download files from ori using scp (to do this step you need to have a user in ori). You will need to enter the ori user password in every command you execute:

```
scp -r username@ori:/protwis/sites/files/Covid19Data .
scp -r username@ori:/protwis/sites/files/Model .
scp -r username@ori:/protwis/sites/files/Molecule .
scp -r username@ori:/protwis/sites/files/Molecule_sc .
scp -r username@ori:/protwis/sites/files/Summary .
scp -r username@ori:/protwis/sites/files/test .
scp -r username@ori:/protwis/sites/files/Covid19Dynamics .
```

In addition, we also have to copy a subset of dynamics simulations. So we create a Dynamics folder and copy a subset of simulations into it:

``` 
mkdir Dynamics
cd Dynamics
sudo scp -r username@ori:/protwis/sites/files/Dynamics/*dyn_688* .
sudo scp -r username@ori:/protwis/sites/files/Dynamics/*trj_688* .
sudo scp -r username@ori:/protwis/sites/files/Dynamics/*dyn_700* .
sudo scp -r username@ori:/protwis/sites/files/Dynamics/*trj_700* .
```


I also recommend to activate the GNOME graphical interface

Download miniconda3 from: (miniconda with python version 3.4 which is required from Ismael):
https://repo.anaconda.com/miniconda/Miniconda3-3.10.1-Linux-x86_64.sh

#If we click crtl right one + f it will allow us to use a full screen mode.


#we give permissions to create a  directory in /opt/:

sudo chmod 777 /opt/

#And install miniconda in /opt/miniconda3:

sh ./Downloads/Miniconda3-3.10.1-Linux-x86_64.sh 

#Accept the licencse term and install in the path:

/opt/miniconda3

#create a conda environment with python=3.4:
conda create -n myenv python=3.4
source activate myenv




#INSTALL SOME PACKAGES USING YUM

sudo yum install epel-release centos-release-scl -y

sudo yum -y install cmake sqlite sqlite-devel tcl-devel tk-devel readline-devel bzip2-devel libtiff-devel freetype-devel libwebp-devel lcms2-devel cairo mod_xsendfile openblas-threads git openbabel expect htop wget clustal-omega perl-Archive-Tar perl-Digest-MD5 perl-List-MoreUtils argtable argtable-devel java-1.8.0-openjdk java-1.8.0-openjdk-devel logrotate curl PyYAML libyaml libyaml-devel libffi libffi-devel libjpeg-turbo-devel zlib zlib-devel libicu libicu-devel perl cmake sqlite sqlite-devel tcl-devel tk-devel readline-devel bzip2-devel libtiff-devel freetype-devel libwebp-devel lcms2-devel cairo

#NCBI BLAST

wget  https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.11.0/ncbi-blast-2.11.0+-1.x86_64.rpm
sudo yum localinstall ncbi-blast-2.11.0+-1.x86_64.rpm -y


#POSTGRESQL INSTALLATION postgres webpage: (https://www.postgresql.org/download/linux/redhat) and we follow the instructions to create the postgreSQL Yum repository:


sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql14-server
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14


#install boost >1.58:
conda install boost==1.61.0







ERRRORS:

PACKAGES PROBLEMS WITH BIOPYTHON
when installing bokeh it appears:
ERROR: PyYAML requires Python '>=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*' but the running Python is 3.4.5



pip install conda==4.3.30:   ERROR: Could not find a version that satisfies the requirement conda==4.3.30 (from versions: 3.0.6, 3.5.0, 3.7.0, 3.17.0, 4.0.0, 4.0.1, 4.0.2, 4.0.3, 4.0.4, 4.0.5, 4.0.7, 4.0.8, 4.0.9, 4.1.2, 4.1.6, 4.2.6, 4.2.7, 4.3.13, 4.3.16)
ERROR: No matching distribution found for conda==4.3.30

pip install cryptography==1.8.1



ERROR: PyYAML requires Python '>=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*' but the running Python is 3.4.5


pip install matplotlib==2.2.5


pip install MDsrv==0.3.5.post1


 pip install mdtraj==1.9.4

pip install mod-wsgi==4.7.1

pip install numpy==1.16.6

pip install pandas==0.22.0

pip install psycopg2==2.6

pip install pycosat==0.6.2

pip install scipy==1.2.3




ln -s /var/solr/data/collection_gpcrmd/conf /protwis/sites/protwis/solr/collection_gpcrmd/conf/
service solr restart









AQUÍ S'ACABEN ELS ERRORS: Comandes:
pip install alabaster==0.7.12
pip install appdirs==1.4.4
pip install asn1crypto==0.22.0
pip install astor==0.8.1
pip install Babel==2.9.0
pip install backcall==0.2.0
pip install backports-abc==0.5
pip install biopython==1.67
pip install bokeh==1.2.0
pip install cairocffi==2020.11.8
pip install cairocffi==0.9.0
pip install certifi==2020.11.8
pip install cffi==1.10.0
pip install Click==7.0
pip install conda==4.3.30
pip install cryptography==1.8.1
pip install ipython==6.5.0

pip install cycler==0.10.0
pip install Cython==0.29.21
pip install decorator==4.4.2
pip install defusedxml==0.5.0
pip install distlib==0.3.1
pip install Django==1.9
pip install django-debug-toolbar==1.9
pip install django-graphos==0.3.41
pip install django-haystack==2.5.0
pip install django-rest-swagger==0.3.10
pip install django-revproxy==0.10.0
pip install django-sendfile==0.3.11
pip install djangorestframework==3.9.4
pip install docutils==0.15.2
pip install filelock==3.0.12
pip install Flask==1.0.4
pip install idna==2.6
pip install imagesize==1.2.0
pip install importlib-metadata==1.1.3
pip install importlib-resources==1.0.2
pip install ipython==6.5.0
pip install ipython-genutils==0.2.0
pip install itsdangerous==1.1.0
pip install jedi==0.16.0
pip install Jinja2==2.10.3
pip install kiwisolver==1.1.0
pip install MarkupSafe==1.1.1
pip install matplotlib==2.2.5
pip install MDsrv==0.3.5.post1
pip install mdtraj==1.9.4
pip install mod-wsgi==4.7.1
pip install numpy==1.16.6
pip install mod-wsgi==4.7.1
pip install packaging==16.8
pip install pandas==0.22.0
pip install parso==0.7.1
pip install pathlib2==2.3.5
pip install pexpect==4.8.0
pip install pickleshare==0.7.5
pip install Pillow==5.4.1
pip install pip==9.0.1
pip install prompt-toolkit1.0.18
pip install prompt-toolkit==1.0.18
pip install psycopg2==2.6
pip install cryptography==1.8.1
pip install ptyprocess==0.6.0
pip install pycosat==0.6.2
pip install pycparser==2.18
pip install Pygments==2.3.1
pip install pyOpenSSL==17.0.0
pip install pyparsing==2.2.0
pip install pysolr==3.6.0
pip install python-dateutil==2.8.1
pip install pytz==2020.4
pip install PyYAML==3.12
pip install requests==2.11.0
pip install scandir==1.10.0
pip install scipy==1.2.3
pip install setuptools==27.2.0
pip install simplegeneric==0.8.1
pip install six==1.10.0
pip install snowballstemmer==2.0.0
pip install Sphinx==1.8.5
pip install sphinxcontrib-websupport==1.1.2
pip install sqlparse==0.3.1
pip install tornado==5.1.1
pip install traitlets==4.3.3
pip install typing==3.7.4.3
pip install urllib3==1.25.7
pip install virtualenv==20.1.0
pip install wcwidth==0.2.5
pip install Werkzeug==0.16.1
pip install wheel==0.29.0
pip install xlrd==1.2.0
pip install XlsxWriter==1.3.7
pip install zipp==1.2.0

