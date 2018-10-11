---
layout: post
title: DHCP Sunucusunun Yapılandırılması
---

**DHCP** (The Dynamic Host Configuration Protocol), bilgisayarların IP adresi, alt ağ bilgisi, DNS sunucusu bilgisi gibi ağ yapılandırmalarının elle yapılması yerine sunucu ve istemciler kullanılarak bu işlemin otomatikleştirilmesini sağlayan bir protokoldür.

Bu yazıda yapılandırmayı anlatmak için fiziksel makinemi bir DHCP sunucusu haline getirip sanal makinelere bu sunucu aracılığı ile ağ yapılandırması yapacağız. Sanallaştırma aracı olarak **VirtualBox 5.2**, DHCP sunucu yazılımı olarak da **ISC DHCP Server** kullanacağım.

### Sanal Ağ Arayüzünün Oluşturulması

Öncelikle fiziksel makinemizde sanal bir ağ arayüzü oluşturmamız gerekiyor. Bu arayüz sayesinde fiziksel makinemizle sanal makinemiz haberleşebilecek. Bu sanal ağ arayüzünü VirtualBox yazılımı üzerinden aşağıdaki gibi oluşturalım.
>_Aşağıda VirtualBox 5.2 sürümü için yapılandırmayı gösterdim. Daha eski sürümlerde aynı işlemi "Dosya -> Tercihler -> Ağ -> Sadece Anamakine Ağları" yolunu izleyerek yapabilirsiniz._

![Sanal ağ arayüzü oluşturma]({{ site.baseurl }}/images/create-vboxnet0.gif "Sanal ağ arayüzü oluşturma")

Burada oluşturduğum sanal ağ arayüzüne otomatik olarak **vboxnet0** adını verdi. IPv4 adresi kısmına yazdığım sabit IP adresi benim DHCP sunucusumun IP adresi olacak aynı zamanda sanal makine de fiziksel makineyi bu IP adresiyle tanıyacak çünkü yalnızca bu sanal ağ arayüzü ile haberleşebiliyor. İkinci sekmede görünen DHCP Sunucusu ayarları ise VirtualBox'ın kendi DHCP sunucusunu aktifleştirmeyi sağlıyor ancak biz bu işlemi fiziksel makinemizden kendimiz yapmak istediğimiz için bunu aktifleştirmiyoruz.

### ISC DHCP Sunucusunun Yapılandırılması

Şimdi DHCP sunucusu haline getirmek istediğimiz fiziksel makinemize **isc-dhcp-server** paketini kuruyoruz.

```bash
sudo apt-get install isc-dhcp-server
```
Kurulum işlemini tamamladıktan sonra **/etc/dhcp/** dizini altında bulunan **dhcpd.conf** yapılandırma dosyasını bir metin editörü kullanarak süper kullanıcı haklarıyla açmamız gerekiyor. Yapılandırma dosyasının içinde birkaç adet yapılandırma örneği mevcut, isterseniz yorum satırı işaretlerini kaldırarak bu komutları düzenleyebilirsiniz. İsterseniz de benim aşağıda yaptığım gibi yorum satırlarını bırakıp dosyanın en altına kendi yapılandırmanızı yazabilirsiniz. Böylece örnek komutları kaybetmemiş olursunuz.

DHCP bildiğiniz üzere birçok bilgiyi gönderebiliyor ancak bizim için şuan **alt ağ**, **ağ maskesi**, **dağıtılacak ip aralığı** yeterli.
```
subnet : oluşturmak istediğimiz alt ağ
netmask : ağ maskesi
range : dağıtmak istediğimiz ip adres aralığı
```

![DHCP sunucusunun yapılandırılması]({{ site.baseurl }}/images/dhcpd-conf.gif "DHCP sunucusunun yapılandırılması")

Son olarak da DHCP sunucumuza hangi ağ arayüzü üzerinden gelen DHCP isteklerini dinlemesi gerektiğini söylememiz gerekiyor. Bizim senaryomuzda bu **vboxnet0** isimli sanal ağ arayüzümüz oluyor.

```bash
sudo nano /etc/default/isc-dhcp-server
```
Komutuyla dosyayı açıp aşağıda bulunan **INTERFACESv4=""** satırını **INTERFACESv4="vboxnet0"** şeklinde değiştiriyoruz. Bu aşamadan sonra sunucu yapılandırmamız bitmiş oluyor.

### VirtualBox Sanal Makinelerinin Ağ Ayarları

DHCP sunucumuzda yapmamız gereken adımları tamamladıktan sonra VirtualBox'a geri dönüp oluşturduğumuz sanal makinelerin ağ arayüzlerini doğru biçimde yapılandırmamız gerekiyor.
>Bu aşamada NAT yapılandırmasını iptal edeceğimiz için sanal makinenizin internet bağlantısı kesilecektir. Kurmanız gereken paket varsa (ifconfig gibi) bu işlemden önce tamamlamanız daha iyi olur.

DHCP hizmeti vermek istediğimiz sanal makineyi seçip yukarıdaki menüden ayarlar butonuna basarak Ağ kısmına girip aşağıdaki gibi **Sadece Anamakine Ağ Arayüzü** seçeneğini ve daha önceden oluşturduğumuz **vboxnet0** sanal ağ arayüzünü seçmeliyiz. Burada yaptığımız işlem sanal makinemize tanımlanacak ağ arayüzünün, fiziksel makinemizdeki sanal ağ arayüzüyle bağdaştırılmasını sağlamak. Artık sanal makinemizi de çalıştırabiliriz.

![Sanal makinenin ağ yapılandırması]({{ site.baseurl }}/images/vm-network-options.png "Sanal makinenin ağ yapılandırması")

### DHCP Servisinin Çalıştırılması

Aşağıdaki komutları fiziksel makinemize girerek DHCP servisimizi yeniden başlatıp durumuna bakabiliriz.

```bash
sudo service isc-dhcp-server restart
sudo service isc-dhcp-server status
```
![DHCP sunucusunun durumu]({{ site.baseurl }}/images/dhcp-status.png "DHCP sunucusunun durumu")

Herhangi bir hata almazsak sanal makinemizde de **ifconfig** komutunu çalıştırarak **enp** ile başlayan varsayılan ağ arayüzünün aldığı IP adresinin bizim seçtiğimiz alt ağdan ve belirlediğimiz IP adres aralığından olduğunu görmeliyiz.

![Sanal sunucunun ağ arayüzleri]({{ site.baseurl }}/images/vm-ifconfig.png "Sanal sunucunun ağ arayüzleri")

### Hatalar

Eğer yukarıdaki gibi servisiniz sorunsuz çalışmıyorsa öncelikle **ifconfig** komutu ile fiziksel makinenizde bulunan ağ arayüzlerini görüntüleyip **vboxnet0**'ın aktif olup olmadığını kontrol etmelisiniz. Bazen bu sanal arayüz sadece sanal makine çalıştığında aktifleşebiliyor. Bu durumda sanal makineyi çalıştırıp **vboxnet0**'ın çalıştığını gördükten sonra servisi yeniden başlatıp durumuna bakabilirsiniz.

Bir diğer sık yapılan hata ise **vboxnet0**'a tanımladığınız sabit IP adresiyle, DHCP sunucunuzda tanımladığınız alt ağ adresinin yada IP adres aralığının uyuşmamasıdır.

### Kaynaklar

Daha ayrıntılı yapılandırma bilgisini aşağıdaki bağlantıdan bulabilirsiniz.

**Ubuntu Documentation** : [isc-dhcp-server](https://help.ubuntu.com/community/isc-dhcp-server)

> Geri bildirimleriniz ve sorularınız için okn.ozdemir@gmail.com mail adresimden ulaşabilirsiniz.
