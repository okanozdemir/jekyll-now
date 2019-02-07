---
layout: post
title: systemd İle Kapanışta Script Çalıştırmak
---

Bu blog yazısı, **systemd** ile kapanışta bir script çalıştırmanızı sağlayacak **systemd unit**'i oluşturmayı anlatmaktadır.

Sistem kapanırken bir script çalıştırmak istediğim için servis ayağa kalktığında bir işlem yapmayıp, sistem kapatılırken sıra bizim servisimizin kapanmasına geldiğinde belirttiğim scriptin çalışmasını istiyorum.

*Unit*'inizi **/etc/systemd/system** dizinin altında konumlandırmalısınız. Servis dosyalarının bulunduğu tek konum burası değil. Bu konu hakkında detaylı bilgi için [1]'i inceleyebilirsiniz.

### Unit dosyasının oluşturulması

Unit dosyasının alabildiği tüm parametreler hakkında bilgiye **man systemd.unit** üzerinden ulaşabilirsiniz. Oluşturduğum **/etc/systemd/system/before-shutdown.service** unit dosyasının içeriği:
```
[Unit]
Description=Script for Shutdown         
After=network.target
Before=shutdown.target

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/bin/true
ExecStop=/home/oozdemir/sync-data.sh
TimeoutSec=15

[Install]
WantedBy=multi-user.target
```
**After** ve **Before** parametreleri ile systemd servislerinin çalıştırılma ve durdurulma sırası ayarlanmış oluyor yani bunlar servisimizin bağımlılıkları gibi. Gibi diyorum çünkü zorunlu bağımlılıklar değil bunlar. Bir şekilde burada belirttiğimiz servisler çalışmazsa bizim servisimiz yine de çalıştırılıyor. Bağımlılıklarının çalışmaması durumunda servisi de çalıştırmak istemiyorsanız bunun için de parametreler mevcut.

**After** parametresinde belirttiğimiz servisler bizim servisimizden önce başlatılacak servisleri belirttiğimiz parametre. Servisler sonlandırılırken de buraya yazdığımız servisler bizden önce sonlandırılmıyor. **Before** parametresinde belirttiğimiz servisler tam tersi bizim servisimizden sonra çalıştırılacak servisler. Bu parametrelerde olarak birden fazla servis belirtmek istersek bunları boşlukla ayırarak yazıyoruz.

**ExecStart** parametresi ile servis çalıştırılırken çalışmasını istediğimiz komutu yada scripti belirtiyoruz. **ExecStop** da ise tahmin edebileceğiniz üzere servis durdurulurken çalışacak komutu yada scripti belirtiyoruz.

Tabiki sistem açılırken ve kapanırken çalıştırılacak olan bu scriptlerin çalışma süreleri varsayılan olarak sınırlandırılmış durumda(90 saniye). Eğer siz bu değeri değiştirmek isterseniz **TimeoutSec** parametresi ile yeni bir değer tanımlayabiliyorsunuz. Bu süre bitene kadar işleminiz sonlandırılmazsa sistem tarafından çalışan işlem öldürülüyor.

### Kaynaklar

**GitHub Deposu** : [Before-Shutdown]()

**Easy Systemd Startup and Shutdown Scripts** : [userscripts4systemd](http://userscripts4systemd.blogspot.com/2016/07/easy-systemd-startup-and-shutdown.html)

**DigitalOcean** : [Understanding Systemd Units and Unit Files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

> Geri bildirimleriniz ve sorularınız için okn.ozdemir@gmail.com mail adresimden ulaşabilirsiniz.
