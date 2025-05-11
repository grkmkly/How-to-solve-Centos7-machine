# How-to-solve-Centos7-machine

## Makineyi İmport Etme ve Virtual Box Ayarları
- Öncelikle Virtual Box üzerinden *File-> Import Appliance* yaparak .*ova* uzantılı dosyamızı seçiyoruz ve İçe aktarılmasını bekliyoruz. İçe Aktarıldıktan sonra Özgür Yazılım A.Ş yazan makineye tıklıyoruz ve ayarlarına giriyoruz. 
- Burada bulunan Network ayarları kısmından Adapter 1 için bağlı olan ayarı Bridged Adaptor yaparak internete bağlanmasını yeni IP alacak şekilde sağlıyoruz. Ayrıca Name kısmında yazan kısımda kendi Wi-Fi kartımızı seçmeye özen gösteriyoruz. Sonrasında gelen Centos makinesi için default olarak Adapter 2 açık olarak geliyor biz bu Network'u bu sistem için kullanmayacağız için Adaptor 2'yi tıklayıp *Enable Network Adapter* tickini kapatıyoruz. Onayladıktan sonra makineyi başlatıyoruz.

## Makinenin Şifresini Sıfırlama
- Makine açılırken bir GRUB menüsü geliyor. GRUB menüsü genellikle Linux işletim sistemlerinde bulunan bir önyükleyici programıdır. Bu program işletim sistemlerini seçmeyi ve bu işletim sistemlerinin çekirdekleri parametreleriyle RAM'e yükleyen programdır. GRUB BIOS/UEFI sistemleriyle birlikte çalışır. BIOS önyükleme sırasında MBR ve GPT gibi Sektör bilgilerini getirir ve bu bilgilere göre GRUB menüsü bize işletim sistemlerini seçme olasılığı sunar. Ayrıca *E* tuşuna basarak konfigürasyon yapmamıza da olanak sağlar. Biz burada konfigürasyon yaparak Centos 7 makinemizde diski mount işlemi yapmadan açmasını sağlayacağız.
- İlk aşama olarak gelen CentOs Linux (3.10.0-1062.e17.x86_64) 7 (Core) kısmı üstüne gelerek *E* tuşuna basıyoruz. Burası GRUB menüsü konfigürasyonu yapacağımız bölümdür. Burada OK tuşları ile aşağı ve yukarı hareket edebiliriz. Burada aşağı inerek 
```plaintext
linux16 /vmlinuz-3.10.0-1062.e17.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm. lv=centos/swap rghb quiet LANG=en_US.UTF-8 
```
kısmını buluyoruz. Bu kısım Linux çekirdeğinin yolunu ve parametrelerini ayarlayan kısımdır. Burayı açıklamak için çekirdek ne demek sorusu da aklımıza gelmektedir. Çekirdek İşletim Sistemi üzerindeki uygulamaların ve donanıımm arasında bir köprü görevi gören ve sistem kaynaklarını düzenleyen yazılım bileşenidir. Şimdi bu kısmı değiştireceğiz.
- Bu kısımda bulunan ro yazan kısmı siliyoruz. Bu kök dosya sistemini başlangıçta sadece okunabilir(readonly) olarak bağlamaktadır. Bu bizim işlem yapamayacağımız anlamına gelmektedir. Biz işlem yapacağımız için bu kısımı sildikten sonra o kısma aşağıda bulunan kısım eklenecektir.
```plaintext
rw init=/sysroot/bin/sh
```
- Bu kısım bizim kök dosyalar üzerinde yazılabilir işlem yapılabilmesini sağlar(rw) ve init=/sysroot/bin/sh parametresi ise tüm servislerin çalışmasını atlayarak direkt kabuk üzerinden sisteme bağlantı sağlayacaktır. Konunun anlaşılması için Shell(Kabuk) kısmını da açıklamak gerekir. Shell(Kabuk) kullanıcı ve işletim sisteminin çekirdeği arasında bağlantı kuran bir arayüz programıdır. Bu kısmı da yazdıktan sonra *CTRL+X* kombinasyonu da yaparak sistemi bu konfigürasyon işlemiyle çalıştırıyoruz.
- Bu işlemden sonra shell kısmı gelmektedir. Buraya 
```bash
chroot /sysroot 
```
komutunu yazarak bundan sonra yapılan her işlemi kök dizinini *sysroot* ayarlar. *sysroot* sistemin kök dizini anlamına gelmektedir. Bu işlemden sonra ise root şifresini değiştireceğiz. Bunun için
```bash
passwd
```
komutunu kullanıyoruz ve oluşturulacak yeni şifremizi 2 kez girerek root şifresini değiştiriyoruz fakat bu şifreyi kaydetmek için
```bash
touch /.autorelabel
```
komutunu kullanarak chroot sonrası dosya işlemlerini kaydetmemizi sağlar. Bu komuttan sonra artık makineyi yeniden başlatalım.
```bash
mount -a
```
komutunu kullanalım. (AÇIKLAMA)
Sonrasında ise
```bash 
vi /etc/ftsab
```
komutunu kullanalım. (AÇIKLAMA)
sonrasında
```bash
reboot 
```
yaparak sistemi yeniden başlatalım. Artık kullanıcı adı kısmına root yazarak ve oluşturduğumuz şifremizi girerek giriş yapabiliriz.

## Makine Analizi ve Apache Çözümü
- Artık makine içindeyiz. Buradaki README dosyasını okuyalım.
```bash
cat README
```
yazılan bu README'yi yaparak öncelikle problemlerimize bakalım.
```bash
sudo systemctl status httpd
```
komutunu yazarak komutu ile öncelikle durumu kontrol edelim. Herhangi bir çalışma yok. Öncelikle 
```bash
sudo systemctl restart httpd
```
komutunu kullanarak başlatıyoruz fakat bize failed veriyor. Bunu düzeltmek için öncelikle configtest'i başlatmamız gerek.
```bash
sudo httpd -t
```
yaparak config testi başlattık. Bu httpd bulunan config dosyasında hata var mı onu gösteriyor. Burada hata olarak gösterilen Cannot accces directory '/etc/httpd/logs' diyerek bize bu dosyaya ulaşılamadığını gösteriyor. Bunun için bu klasöre erişeceğiz. 
```bash
cd /etc/httpd/
```
yazarak o klasöre erişiyoruz. Sonrasında ls -al diyerek bakıyoruz *logs* klasörü kırmızı yanmakta bu bir hata olduğunun göstergesi ve bu *logs* klasörü sembolik olarak bağlanmış durumda. Bu sembolik olarak bağlandığı kısımda ne hatası var oraya bakmamız gerekiyor.
```bash
ls -ld ../../var/log
```
yaparak bakıyoruz bize bu klasör veya dosyanın olmadığını söylüyor. bizde teker teker bu klasöre girmeyi deneyelim.
```bash
cd ..
cd ..
cd var
cd log
cd httpd
```
bunları yaptıktan sonra gördüğümüz üzere httpd klasörü bulunmamakta buraya bir httpd klasörü oluşturalım.
```bash
mdkir httpd
```
yapalım. Sonrasında config testimizi tekrar yapalım.
```bash
sudo httpd -t
```
şimdi *SYNTAX OK* olarak bir bildirim verdi. Şimdi tekrar httpd serviceni tekrardan başlatalım.
```bash
sudoo systemctl restart httpd
```
bu komut herhangi bir hata vermedi. Tekrar baktığımızda bu servisin active(running) olduğunu görüyoruz.

## İnternetten DHCP ile IP Alma ve Sorun Çözümleri

- Şimdi konfigürasyon ayarı ile kendi bilgisayarımızdan açmayı deneyeceğiz. Bunun için öncelikle hangi IP'yi almışız ona bakmamız gerekiyor.
```bash
ip a
```
yaparak hangi IP adresimizi görmemiz gerekiyor. Burada bulunan enp0s3 isimli arayüz bizim kullanacağımız ağ arayüzüdür. Ağ arayüzü bilgisayarımızın fiziksel veya mantıksal olarak bağlanmasını sağlayan noktadır. Bize tahsis edilen IP adresi 192.168.56/24 olarak gözüküyor. Şimdi Windows bilgisayarımızdan da IP'yi doğru verdiğini kontrol edelim. Bu yüzden Windows üzerinden kontrol edelim. Bunun için Windows terminali üzerinden bu kısma bakalım.
```bash
ipconfig 
```
yaparak ağ arayüzümüzü açalım. Bu arayüz üzerinden Wi-Fi kartı üzerindeki IP'mizin bundan tamamen farklı olduğunu gördük. Burada bir hata sezdik. Şimdi ağ arayüz ayarlarına bakalım ve sorunlarına bakalım. Ağ ayarlarına bakmak için
```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
yaparak config ayarlarına gelmiş bulunmaktayız burada bulunan özelliklere bakarak buradaki ağ arayüzüne verilen IP static olarak atanmış durumdadır. Burada bulunan 
```plaintext
BOOTPROTO=static
```
yazan kısım bunu belirtiyor. Ayrıca en altta atanan
```plaintext
IPADDR=192.168.56.52
NETMASK=255.255.255.0
```
kısmı da bunu destekler nitelikte. Biz *IPADDR kısmını ve NETMASK kısmını silip* ve 
```plaintext
BOOTPROTO=dhcp
```
yaparak DHCP sunucusundan IP almasını sağlayacağız. DHCP sunucusu internete bağlanmamız için bizim aygıtlarımızı otomatikleştiren ve bize uygun olan IP'yi atayan protokoldür. Şuan dhcp'ye bağlanacak fakat ilk olarak buraya bağlanmamız için öncelikle bağlantıyı kesip tekrar bağlanmamız gerekli. Bu yüzden 
```bash
sudo dhclient -r enp0s3
```
komutunu kullanarak yönetilen tüm IP'leri serbest bırakalım sonrasında ise bu ağ arayüzü için DHCP sunucusuna istek atarak yeni IP'mizi alalım. Bunun için de
```bash
sudo dhclient enp0s3
```
komutunu kullanalım ve yeni IP'mizi DHCP sunucusundan almış olduk. Şimdi tekrar 
```bash
ip a
```
komutunu kullanalım ve yeni IP'mizi görmüş olduk. IP'mizin çalıştığını denemek için 
```bash
ping -c 5 google.com
```
yaparak ping gönderelim. Eğer 5 paket için kaybımız yoksa bu durumda çalışıyor demektir. 
- *NOT = Makine elime geldiğinde DNS nameserver de bozuktu fakat bu işlemi yaptıktan sonra DNS otomatik olarak geldi bu sebeple tekrar bir şey yapmayacağız. Onu da kısa bir anlatımla özet geçelim.*
```bash
cat /etc/resolv.conf
```
komutunu kullanarak bu dosya DNS sunucularını bir dosya olarak tutan kısımdır. DNS, alan adlarını(domain) IP'ye dönüştüren protokoldür. Bu işlem bizim IP'leri aklımızda tutmamıza gerek kalmadan istediğimiz siteye ulaşabilmemizi sağlar. Yukarı bulunan komut ile
```plaintext
nameserver 8.8.8.7 
```
çıktısını aldık. Bu adres geçerli bir DNS adresi değildir. Genelde kullanılan Google'a sahip olan 8.8.8.8 DNS sunucusunu kullanacağız. Bu dosyaya nano ile yazı yazılmaya çalışıldı fakat bu dosya immutable(değiştirilemez) belirlenmiş
```bash
lsattr /etc/resolv.conf
``` 
komutunu kullanarak dosya özniteliklerine ulaşabiliriz. Bu dosyayı bu durumdan kurtarmamız gerekiyor. Bunun için 
```bash
chattr -i /etc/resolv.conf
```
yaparak bu özniteliği kaldırarak artık üzerinde değişiklik yapabileceğiz. Şimdi tekrar nano ile
```bash
sudo nano /etc/resolv.conf
``` 
yaparak düzenleyeceğiz. Bu dosyanın içeriğini
```plaintext
nameserver 8.8.8.8
```
olarak değiştiriyoruz ve sonrasında *CTRL + X* kombosu ile kaydediyoruz. Artık DNS serveri düzelttik.

## Apache Konfigürasyonları 

- Bu bölüm için Apache konfigürasyonlarını düzenleyeceğiz. Konfigürasyon dosyalarına erişelim. Bundan önce bağlanacağımız DHCP sunucusundan aldığımız IP'yi kullanarak bağlanacağız bu yüzden
```bash
ip a
```
komutu DHCP sunucusu ile aldığımız IP adresini aklımızı kaydedelim. Sonrasında konfigürasyon dosyalarını açalım.
```bash
sudo nano /etc/httpd/conf/httpd.conf
```
dosyasını açalım ve burada bulunan
```plaintext
#LISTEN 12.34.56.78:80 
```
kısmını IP'miz ile port olacak şekilde (80) olarak yazalım ve # işaretini kaldıralım. Burada bulunan
```plaintext 
Listen 80
``` 
kısmının başına # koyalım. Bununla yorum satırına almış olduk. Sonrasında aşağıya doğru indiğimizde
```plaintext
ServerName www.example.com:80
```
yazan kısıma 
```plaintext
www.ozguryazilim.com:80
```
yazalım. Bu isteğe bağlıdır. (Başka herhangi bir konfigürasyon çarpmadı gözüme). Burada işimiz tamamdır. Buradaki işlemi yapıp kaydedelim. Konfigürasyon dosyası üzerinde hata yapmadığımızı kontrol etmek için
```bash
sudo httpd -t 
```
komutunu kullanalım. Eğer *SYNTAX ON* bildirimi gelirse bir hata yapılmamıştır. Şimdi ise firewall üzerinden 80 portlu yani HTTP portunu kontrol edip açacağız. Bunun için öncelikle durumlara bakalım ve sonrasında ekleyelim.
```bash
sudo firewall-cmd --list-all
```
yaparak listeye bakalım. Burada services kısmında *http* gözükmemektedir. Bu demek oluyor ki http service'i için 80 nolu portumuz açık değil. Açmak için
```bash
sudo firewall-cmd --permanent --add-service=http
```
komutunu kullanıyoruz. Bu komut firewall üzerinde bulunan http service'nin portundan alışverişe izin verir. Bu port genel olarak 80 portu olduğu için buna göre ayarlar.
- Burada ilk olarak verdiğimiz IP üzerinden kullandığımız tarayıcı ile giriş yapalım. Tarayıcının kısmına 80 portu ile birlikte yazalım. Aşağıda bir örnek verilmektedir. Buna benzer yapıyoruz.
```plaintext
192.168.1.2:80
```
yazalım. Herhangi bir çıktı vermedi. Halbuki herhangi bir hata alınmamıştı. Şimdi ise *index.html* dosyasının özelliklerine bakalım. Bunun için 
```bash
sudo ls -ld /var/www/html/index.html
```
komutunu yazalım. Bu komut ile bu dosya hakkında bilgileri aldık ve gördüğümüz üzere herhangi bir çalıştırma izni verilmemiş. Bu dosyaya genel olarak  verilen izinleri verelim.
```bash
sudo chmod 644 /var/www/html/index.html
```
Bunu yaptıktan sonra izinleri vermiş olduk. Artık sistem *index.html* dosyasını çalıştırabilir. Bu işlemi de yaptıktan sonra Apache sunucumuzu tekrar başlatalım.
```bash
sudo systemctl restart httpd
```
Bu işlem ile Apache sunucumuzu yeniden başlattık. Sonrasında durumuna bakalım.
```bash
sudo systemctl status httpd
```
Görüldüğü üzere *active* kısmında çalıştığını söylüyor ve yeşil yanıyor. BAŞARDIK!!! 
- Şimdi tarayıcımız üzerinden sunucumuzun IP'si ile giriş yapalım ve gördüğümüz üzere 
```plaintext
Apache çalışıyor :)
Fakat Apache sürümü eski, güncellemek lazım :(
```
sayfası açıldı. Şimdi Apache sürümünü güncelleyeceğiz.

## Yum Üzerinden Apache Güncellemesi

- yum update httpd yaptığımızda indirmede bir problem olduğunu gördük. Bizi sürekli oyalıyor ve paketleri doğru adreste bulamıyor. Bunun için öncelikle CentOS 7 için resmi depolarına eriştiği dosyaya bakacağız. Bunun için
```bash
sudo nano /etc/yuum.repos.d/cenOS-Base.repo
```
yaparak bu dosyaya bakıyoruz. İçinde bulunan depo linklerinin artık kullanılmadığını gördük. Bunun için burayı aşağıdaki kısım ile güncelliyoruz.
```plaintext
[base]
name=CentOS-$releasever - Base
baseurl=https://vault.centos.org/7.9.2009/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=1

[updates]
name=CentOS-$releasever - Updates
baseurl=https://vault.centos.org/7.9.2009/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=1

[extras]
name=CentOS-$releasever - Extras
baseurl=https://vault.centos.org/7.9.2009/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=1

[centosplus]
name=CentOS-$releasever - CentOSPlus
baseurl=https://vault.centos.org/7.9.2009/centosplus/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0
```
- Centos 7 için bılunan yeni çalışan depo(repository)'ler bu linklerde mevcut. Yukarıda bulunan kısımları bu dosyanın içine yapıştıralım.
*NOT: Bu kısımda kopyalama ve yapıştırma işlemi için ssh bağlantısı kullanılmıştır.*

- Sonrasında ilk olarak repolar içindeki cacheleri temizleyelim ve sonrasında update işlemini yapalım. Cache temizlemek için
```bash
sudo yum clean all
```
komutunu kullanalım. Sonrasında da Apache'yi güncellemek için
```bash
sudo yum update httpd
```
yapalım ve gelen kısımda indirmek istediğinize emin misiniz sorusuna y diyelim ve indirelim. Update işlemi yaptık. Şimdi tarayıcımızda yeni bir sekme açalım.(Cache olduğu için) ve makinemizin IP'sini yazalım. Gördüğümüz üzere artık sayfada bir gülen yüz var. BAŞARDIK!
```plaintext
Apache çalışıyor hem de son sürümde :) 
```
- yazısı ile karşılaştık ve görevimizi tamamladık.
