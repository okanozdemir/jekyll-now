---
layout: post
title: DNS Sunucusunun Yapılandırılması
---

**DNS** (Domain Name Service), alan adları ve bu alan adlarını barındıran makinelerin IP adreslerini birbirleriyle eşleştiren bir servistir. Böylece bir sunucuya ulaşmak istediğimizde IP adreslerini hatırlamak zorunda kalmayız.

Bu yazıda yapılandırmayı anlatmak için fiziksel makinemi bir DNS sunucusu(ad sunucusu) haline getirip, gelen alan adı sorgularına yanıt vermesini sağlayacağım. Örnek bir alan adı için sunucumuza DNS kayıtları girip daha sonra bu alan adını ve kaydetmediğimiz alan adlarını sorgulayacağız. Bu sorguları dilerseniz aynı fiziksel makinenizden, dilerseniz de aynı ağda bulunduğunuz farklı herhangi bir makineden yapabilirsiniz. DNS sunucu yazılımı olarak **BIND 9** kullanacağım.

### BIND 9'un Kurulum ve Yapılandırılması

Aşağıdaki ilk komutla BIND 9'un kurulumunu yapalım.

```bash
sudo apt-get install bind9
```
Kurulum tamamlandıktan sonra BIND 9'un yapılandırma dosyalarının bulunduğu **/etc/bind** dizinine girelim. Burada bulunan yapılandırma dosyalarından daha detaylı bahsedeceğiz ancak kısaca anlatmak gerekirse:

* **named.conf.options** dosyası BIND 9'un genel ayarlarını yapılandırdığımız dosya. Biz bu dosyada pek değişiklik yapmayacağız.

* **db.** ile başlayan dosyalar yanıt vermekte yetkili olduğumuzu belirttiğimiz alan adlarına ait DNS kayıtlarını sakladığımız dosyalar.

* **named.conf.local** dosyası sorgularına yanıt vermekte yetkili olduğumuzu söylediğimiz alan adlarını belirttiğimiz dosya. Her alan adı için DNS kayıtlarını farklı dosyalara kaydedip burada bu dosyaların yollarını belirtiyoruz. ("db." ile başlayarak isimlendirilmiş dosyalar.)

Öncelikle DNS kayıtlarını bizim tutmadığımız alan adları için gelen sorgulara da cevap verebilmemiz için sorguları yönlendirebileceğimiz DNS sunucularının IP bilgisini **named.conf.options** dosyasına girelim. "forwarders" alanının yorum satırlarını kaldırarak aşağıdaki gibi değiştirebilirsiniz. Ben Google'a ait DNS sunucu adreslerini kullandım.

```
forwarders {
        8.8.8.8;
        8.8.4.4;
};
```

> Eğer bulunduğunuz ağda, ağ dışında bulunan DNS sunucularına sorgu yapılmasına izin verilmiyorsa (Örn: ÇOMÜ ağı) bu kısmı o ağa dahilken uygun adreslerle değiştirmelisiniz.

Şimdi DNS sorgularına cevap vermek istediğim alan adının DNS kayıtlarını oluşturalım. Bunun için aşağıdaki komutla "db.local" dosyasının istediğiniz isimde bir kopyasını oluşturabilirsiniz.

```bash
sudo cp db.local db.okanozdemir.me
sudo nano db.okanozdemir.me
```

Buradaki **A** kaydını istediğiniz herhangi bir IP adresiyle değiştirebilirsiniz. Ben fiziksel makinemin sanal makineyle haberleştiği ağ arayüzünün IP adresi olarak değiştiriyorum.

> Bu dosyayı her değiştirdiğinizde **Serial** numarasını değiştirmeniz gerekmektedir. Kolaylık olması açısından **Serial** numarasını değişikliği yaptığınız tarih olarak belirleyebilirsiniz.

![okanozdemir.me için DNS kayıtları]({{ site.baseurl }}/images/okanozdemir.me-DNS.png "okanozdemir.me için DNS kayıtları")

Daha sonra **named.conf.local** dosyasına aşağıdaki gibi bir kayıt ekleyerek bu alan adına ait bilgilerin tutulduğu dosyayı gösteriyorum.

```
zone "okanozdemir.me" {
     type master;
     file "/etc/bind/db.okanozdemir.me";
};
```

![named.conf.local yapılandırılması]({{ site.baseurl }}/images/named.conf.local.png "named.conf.local yapılandırılması")

Aşağıdaki komutla DNS servisimi yeniden başlatıyorum.

```bash
sudo service bind9 restart
```

### DNS Sunucusuna Sorgu Gönderme

Şimdi yapılandırmış olduğumuz sunucuya sorgular gönderek aldığımız cevapları test edebiliriz. Sunucuya sorgu göndermek için **dig** komutunu kullanacağım.

> Sunucuya bir kez sorgu gönderdiğinizde gelen cevap bir süre için bilgisayarınızda saklanacak ve tekrar sorgu yapmak istediğinizde bilgisayarınız sunucuya sormak yerine daha önceden aldığı cevabı size döndürecektir. Bunu engellemek için DNS önbelleğinizi temizlemeniz gerekir. Bunun içinse **service network-manager restart** komutunu kullanabilirsiniz.

```bash
dig +short @192.168.42.1 okanozdemir.me
```

Yukarıdaki komutla _192.168.42.1_ IP adresli DNS sunucusuna _okanozdemir.me_ alan adına karşılık gelen IP adresini sormuş oldum. Tüm DNS kaydını istiyorsanız _+short_ argümanını kaldırabilirsiniz.

### Hatalar

Aldığınız yanıt belirlediğiniz IP adresi değil de o alan adına ait doğru IP adresiyse o alan adına ait dosyalarda bir isim yanlışlığı yapmış olabilirsiniz.

DNS sunucunuzdan yanıt alamıyorsanız **service bind9 status** komutuyla servisinizin çalışıp çalışmadığını kontrol etmelisiniz. Servisiniz çalışmıyorsa yapılandırma dosyalarında sözdizimi hatası yapmış olmanız çok muhtemel, örnek yapılandırma dosyalarını dikkatlice tekrar incelemelisiniz.

### Kaynaklar

Daha ayrıntılı yapılandırma bilgisini aşağıdaki bağlantılardan edinebilirsiniz.

**Ubuntu Documentation** : [BIND9ServerHowto](https://help.ubuntu.com/community/BIND9ServerHowto)

**DigitalOcean** : [Bind as a Caching or Forwarding DNS Server](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-14-04)

**DigitalOcean** : [Bind as an Authoritative-Only DNS Server](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-an-authoritative-only-dns-server-on-ubuntu-14-04)

**DigitalOcean** : [BIND as a Private Network DNS Server](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-14-04)

> Geri bildirimleriniz ve sorularınız için okn.ozdemir@gmail.com mail adresimden ulaşabilirsiniz.
