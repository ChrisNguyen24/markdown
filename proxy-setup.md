# setup proxy for ubuntu
1. *đảm bảo đã cài đặt apache*
2. *start apache*

### shellscript for set up proxy
> `__port `: setup port tương ứng với container muốn setup proxy
> `__domain` : setup domain  tương ứng với host muốn map

- ví dụ: `__port`: 8282, `__domain`: downstream.localhost.com
![[Pasted image 20230416100241.png]]
```
#! /usr/bin/env bash  
  
# the auto scrip to create a proxy file and  
# send it to sites-availables then create a symlink  
# then attempt to restart server  
# Note: this is only tested on ubuntu 20 and apache  
  
__port=8282;  
__domain=jar-portal-v2.localhost.com  
  
__v_file="${__domain}.conf";  
__the_dir=$(dirname $(realpath $0));  
__the_v_file="${__the_dir}/${__v_file}";  
#__config_path=/etc/apache2/sites-available;  
#__enable_path=/etc/apache2/sites-enabled;  
  
if [ ! -a $__the_v_file ]; then  
echo "File ${__the_v_file} not found -> Create one.";  
{  
echo "" > $__the_v_file;  
} || { echo "failed to create file. Pls try later."; exit; }  
fi;  
  
  
echo "File exists: ${__the_v_file}";  
echo ">>> Writing to file";  
echo "  
<VirtualHost *:80>  
ServerName ${__domain}  
ProxyPreserveHost On  
ProxyPass / http://localhost:$__port/  
ProxyPassReverse / http://localhost:${__port}/  
</VirtualHost>  
" > $__the_v_file;  
  
  
echo "Attemping to save this file to sites-availables";  
__has_config=$(ls $__config_path | grep $__domain | wc -l);  
  
if [ $__has_config -gt 0 ]; then  
echo "Virtual Host file for domain ${__domain} was found. ";  
exit;  
fi;  
  
echo "Host file not found. Copy this file";  
sudo cp $__the_v_file $__config_path/$__v_file;  
  
echo "Enable host file";  
sudo ln -s $__config_path/$__v_file $__enable_path/${__v_file};  
  
echo "Check host at /etc/hosts";  
if [ $(cat /etc/hosts | grep $__domain | wc -l) -eq 0 ]; then  
echo "Host not found --> adding ";  
echo "127.0.0.1 $__domain" | sudo tee -a /etc/hosts;  
fi;  
  
echo "Restart server";  
sudo systemctl restart apache2;
```


## For mac Os

```
#! /usr/bin/env bash  
  
# the auto scrip to create a proxy file and  
# send it to extra  
# then attempt to restart server  
# Note: this is only tested on macOs Montery and apache  
  
# downstream_report  
__port=8383;  
__domain=downstream-report.localhost.com  
  
__v_file="${__domain}.conf";  
__the_dir=$(dirname $(realpath $0));  
__the_v_file="${__the_dir}/${__v_file}";  
__config_path=/etc/apache2/extra;  
__vhost_file=$__config_path/httpd-vhosts.conf  
  
if [ ! -a $__the_v_file ]; then  
echo "File ${__the_v_file} not found -> Create one.";  
{  
echo "" > $__the_v_file;  
} || { echo "failed to create file. Pls try later."; exit; }  
fi;  
  
  
echo "File exists: ${__the_v_file}";  
echo ">>> Writing to file";  
echo "  
<VirtualHost *:80>  
ServerName ${__domain}  
ProxyPreserveHost On  
ProxyPass / http://localhost:$__port/  
ProxyPassReverse / http://localhost:${__port}/  
</VirtualHost>  
" > $__the_v_file;  
  
if [ $(cat $__vhost_file | grep $__domain | wc -l) -eq 0 ]; then  
echo "Virtual Host adding for domain ${__domain} . ";  
cat $__the_v_file | sudo tee -a $__vhost_file  
exit;  
fi;  
  
echo "Check host at /etc/hosts";  
if [ $(cat /etc/hosts | grep $__domain | wc -l) -eq 0 ]; then  
echo "Host not found --> adding ";  
echo "127.0.0.1 $__domain" | sudo tee -a /etc/hosts;  
fi;  
  
  
echo "Restart server";  
sudo apachectl restart;
```