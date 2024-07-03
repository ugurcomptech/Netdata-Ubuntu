# Netdata Kurulumu

Netdata, gerçek zamanlı sağlık ve performans izleme için geliştirilmiş açık kaynaklı bir yazılımdır. Kullanıcıların sistemleri ve uygulamaları hakkında ayrıntılı veri toplayarak, bu verileri anlık olarak görselleştirir ve analiz eder. Netdata, CPU, RAM, disk kullanımı, ağ trafiği, veritabanları, uygulamalar gibi pek çok bileşenin performansını izleyebilir.

Aşağıdaki komut ile linux sunucunuza indirebilirsiniz.

```
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

İndirtikten sonra aşaığdaki komutları sırayla yazalım

```
systemctl enable netdata
systemctl start netdata
systemctl status netdata
```

Gerekli adımları sağladıktan sonra tarayıcnıza <Sunucu_IP_Adresiniz:19999> yazarak erişebilirsiniz. Ben güvenlik açısından ilgili portu değiştiridm. `nano /opt/netdata/etc/netdata/netdata.conf ` dosyasını açtıktan sonra `default_port=19999` seçeneğini istemiş olduğunuz port numarasını yazarak değişim sağlayabilirsiniz. Gerekli değişikliği sağladıktan sonra netdata servisini yeniden başlatmayı unutmayınız.

## Güvenlik Önlemleri

Netdata'yı indirdiğiniz zaman sisteme giriş sağladıktan sonra sizlere herhangi bir kullanıcı adı veya şifre bilgisi sormaz. Kullanmış olduğunuz Web servisi ile isterseniz bir doğrulama sistemi oluşturabilir veya ilgili port numarasını dışarıya kapatıp sadece sizlere ait IP adreslerine erişim izni verebilirsiniz.

### NGİNX ile Doğrulama 

Genelde doğrulama sistemi NGİNX üzerinden yapılmaktadır. Aşağıdaki adımları sırayla izleyelim.

Aşağıdaki komutları çalıştırarak NGINX ve apache2-utils'i kurabilirsiniz:

```
sudo apt install nginx apache2-utils
```


Kullanıcı adı-şifre çifti oluşturmak için bu komutu çalıştırın:

```
sudo htpasswd -c /etc/nginx/.htpasswd usertest
```

Enter tuşuna basın ve istemlere usertest'in parolasını yazın.

Kullanıcı adı-şifre çiftinin oluşturulduğunu aşağıdaki komutu çalıştırarak doğrulayın:

```
cat /etc/nginx/.htpasswd
```

NGINX yapılandırma dosyanızı (nginx.conf) açın ve http bloğunu bulun. (Nginx.conf dosyanız genellikle /usr/local/nginx/conf, /etc/nginx veya /usr/local/etc/nginx konumunda bulunur)


http bloğunuza aşağıdaki satırları ekleyin:

```
upstream backend {
   server 127.0.0.1:19999;
   keepalive 64;
}

server {
   listen <10.0.0.1>:80;
   server_name <example.com>;

   auth_basic "Protected";
   auth_basic_user_file /etc/nginx/.htpasswd;

   location / {
     proxy_set_header X-Forwarded-Host $host;
     proxy_set_header X-Forwarded-Server $host;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_pass http://backend;
     proxy_http_version 1.1;
     proxy_pass_request_headers on;
     proxy_set_header Connection "keep-alive";
     proxy_store off;
   }
}
```

-  <10.0.0.1>'i genel IP Adresinizle değiştirin.
- <example.com> alanını kendi alan adınızla değiştirin.


Tarayıcınızı açın ve <10.0.0.1> veya <example.com> adresine gidin. Web arayüzüne erişmek için kullanıcı adı-şifre çiftinizi kullanın.


## Firewall İle Güvenlik Sağlama

İlk öncellikle kullanmış olduğunuz Netdata portunu dışarıya gelen isteklere karşı kapatmamız gerekiyor.


**UFW**

```
sudo ufw deny 19999
sudo ufw reload
```


**IPTables**

```
sudo iptables -A INPUT -p tcp --dport 19999 -j REJECT
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```


İlgili komutu yazdıktan sonra :19999 portu dışarıya kapatılmıştır. Şimdi sadece bizim belirlemiş olduğumuz IP adresleri için giriş izni sağlayacağız.


**UFW**

```
sudo ufw allow from 88.221.74.126 to any port 19999
sudo ufw reload
```

**IPTables**

```
sudo iptables -A INPUT -p tcp -s 88.221.74.126 --dport 19999 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```



88.221.74.126 IP adresi için :19999 portuna gelen isteklere izin vermiş bulunmaktayız. Bu IP adresi harici kimse ilgili sisteme erişemeyecektir.



# Teşekkürler

Okuduğunuz için teşekkürler. Sizlerle işbirliği yapmaktan mutluluk duyarım.






