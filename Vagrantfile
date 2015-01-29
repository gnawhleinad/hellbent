# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "hellbent"

  config.vm.provider "virtualbox" do |v, override|
    v.gui = false
    v.customize ["modifyvm", :id, "--memory", 2048]
    v.customize ["modifyvm", :id, "--cpus", 1]
  end

  $script = <<SCRIPT
sudo debconf-set-selections <<< \
'mysql-server mysql-server/root_password password vagrant' && \
sudo debconf-set-selections <<< \
'mysql-server mysql-server/root_password_again password vagrant' && \
sudo apt-get update && \
sudo apt-get install -qq -y git apache2 mysql-server libappconfig-perl \
libdate-calc-perl libtemplate-perl libmime-perl build-essential \
libdatetime-timezone-perl libdatetime-perl libemail-sender-perl \
libemail-mime-perl libemail-mime-modifier-perl libdbi-perl libdbd-mysql-perl \
libcgi-pm-perl libmath-random-isaac-perl libmath-random-isaac-xs-perl \
apache2-mpm-prefork libapache2-mod-perl2 libapache2-mod-perl2-dev \
libchart-perl libxml-perl libxml-twig-perl perlmagick libgd-graph-perl \
libtemplate-plugin-gd-perl libsoap-lite-perl libhtml-scrubber-perl \
libjson-rpc-perl libdaemon-generic-perl libtheschwartz-perl \
libtest-taint-perl libauthen-radius-perl libfile-slurp-perl \
libencode-detect-perl libmodule-build-perl libnet-ldap-perl \
libauthen-sasl-perl libtemplate-perl-doc libfile-mimeinfo-perl \
libhtml-formattext-withlinks-perl libgd-dev lynx-cur python-sphinx

(cd /var/www && rm -rf html && \
git clone -b bugzilla-4.4-stable https://git.mozilla.org/bugzilla/bugzilla html) 

mysql -uroot -pvagrant \
-e "GRANT ALL PRIVILEGES ON bugs.* TO bugs@localhost IDENTIFIED BY 'bugs'" && \
service mysql restart

cat > /etc/apache2/sites-available/bugzilla.conf << APACHE
ServerName localhost

<Directory /var/www/html>
  AddHandler cgi-script .cgi
  Options +ExecCGI
  DirectoryIndex index.cgi index.html
  AllowOverride Limit FileInfo Indexes Options
</Directory>
APACHE
a2ensite bugzilla && \
a2enmod cgi headers expires && \
service apache2 restart

(cd /var/www/html && \
./install-module.pl --all && \
./checksetup.pl && \
sed -i 's/^\\(\\$webservergroup\\s*=\\s*\\).*\$/\\1'\\''www-data'\\'';/' localconfig && \
sed -i 's/^\\(\\$db_pass\\s*=\\s*\\).*\$/\\1'\\''bugs'\\'';/' localconfig)
cat > answers << ANSWERS
\\$answer{'ADMIN_EMAIL'} = 'bugs@hellbent.local';
\\$answer{'ADMIN_PASSWORD'} = 'hellbent';
\\$answer{'ADMIN_REALNAME'} = 'hellbent';
\\$answer{'NO_PAUSE'} = 1;
ANSWERS
(cd /var/www/html && ./checksetup.pl answers)
SCRIPT

  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.provision "shell", inline: $script
end

