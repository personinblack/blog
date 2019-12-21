+++
title = "Android ve youtube-dl"
date = 2019-12-20T19:20:07+03:00
author = "personinblack <berkay@tuta.io>"
cover = "android.youtube-dl.jpeg"
tags = ["android"]
keywords = ["android", "youtube", "youtube-dl", "last.fm", "müzik", "video", "playlist", "indir"]
description = """
youtube-dl, YouTube ve benzeri bir çok siteden video indirebilen bir cli uygulamasıdır.
Peki bu uygulamayı Android cihazlarımızda nasıl efektif olarak kullanabiliriz? Benimle
kal, mevzu oldukça basit.
"""
+++

    Fotoğraf Cedric Letsch

# Android ve youtube-dl

[youtube-dl](https://youtube-dl.org/), YouTube ve benzeri bir çok siteden video
indirebilen bir cli uygulamasıdır. Peki bu uygulamayı Android cihazlarımızda nasıl
efektif olarak kullanabiliriz? Benimle kal, mevzu oldukça basit.

#### Giriş

Şimdi siz soracaksınız: `Ben neden Spotify kullanmak varken bununla uğraşayım ki?`.

Şöyle ki bu sistem;

- size Spotify'ın aksine, her yerde kullanabileceğiniz, istediğiniz uzantıda ses
dosyaları verir.
- YouTube üzerinde dinlemekte olduğunuz herhangi bir parçayı iki tık ile indirebilmenizi
sağlar.
- Spotify'dan daha geniş bir arşive sahiptir. YouTube, Vimeo, [vb. platformlarda]
(https://github.com/ytdl-org/youtube-dl/blob/master/docs/supportedsites.md) bulunan tüm
parçaları önünüze serer.
- **last.fm** ile birleştirilerek Spotify'dan daha güçlü bir parça öneri mekanizması elde
edilebilir. (Aynı zamanda dinleme alışkanlıklarınızı takip etmenize de olanak sağlar)
- ebediyen ücretsizdir.

İkna olduysanız, siz Spotify üyeliğinizi iptal ettikten sonra kuruluma başlayabiliriz.

#### Kurulum

Öncelikle **Termux** isimli terminal emülatörünü yüklemeliyiz. Kendisi [F-Droid]
(https://f-droid.org/en/packages/com.termux/) ve [Play Store]
(https://play.google.com/store/apps/details?id=com.termux)'da mevcut; oralardan
edinebilirsiniz.

Termux sizden istemez ama sizin kendisine *depolama yetkisi* vermeniz gerek. Aksi
durumda, youtube-dl ile indirdiklerinizi sizin rahatlıkla erişebileceğiniz *storage*
ortamına kaydetmesi mümkün olmayacaktır. Ardından Termux'ı başlatıp `$ apt update` ile
paket veritabanımızı güncelleyebiliriz.

youtube-dl'in iki adet gereksinimi mevcut: **Python** ve **ffmpeg**. Bunları
`$ apt install python ffmpeg` ile yükleyelim. Python'ın paket yöneticisi
**pip** ile de `$ pip install youtube-dl` diyerek youtube-dl'i kurduktan sonra 
konfigürasyon'a hazırız.

#### Konfigürasyon

Termux'ın home klasöründe youtube-dl'in istediği konfigürasyon klasörünü oluşturalım:
`$ mkdir -p ~/.config/youtube-dl/`. Devamında config dosyasını oluşturup düzenlemeliyiz:
`$ vi ~/.config/youtube-dl/config` (Burada ben Termux ile beraber geldiği için **vi**
editörünü kullanıyorum).

*~/.config/youtube-dl/config*:
```
# İndirilenlerin kaydedileceği yer ve dosya isimlerinin belirlenmesi
-o /storage/emulated/0/music/%(title)s.%(ext)s

# Video içerisindeki sesin maksimum kalitede ve mp3 formatında çıkartılması
--extract-audio --audio-quality 0 --audio-format mp3
```

> *Not: Yukarıdaki örneği kendi kafanıza göre düzenlemek isterseniz `$ man youtube-dl`
içerisinde bulunan 'OUTPUT TEMPLATE' ve 'OPTIONS' bölümlerine göz atmayı unutmayın.*

Pekala, youtube-dl konfigürasyonumuz hazır. Fakat her video indirmek istediğimizde Termux'ı
başlatıp `youtube-dl https://youtube.com/video` demek olmaz; bunu otomatikleştirmemiz
gerek. İşte burada *termux-url-opener* devreye giriyor:

`$ mkdir -p ~/bin/ && vi ~/bin/termux-url-opener`

*~/bin/termux-url-opener*:
```bash
#!/bin/bash
youtube-dl $1
```

Scripti çalıştırılabilir hale getirmeyi de unutmayalım tabi:
`$ chmod +x ~/bin/termux-url-opener`.

Artık YouTube'a girip herhangi bir videonun (veya public playlistin) paylaşma butonuna
bastıktan sonra Termux'ı seçerseniz; videonun adresi yukarıdaki scripte yönlendirilecek
ve scriptimiz de kendisine yöneltilen bu adresi youtube-dl'e vererek videoyu mp3
formatında indirmemizi sağlayacak.

> *Not: youtube-dl bir süre kullanımdan sonra hata vermeye başlayabilir. Bu genellikle
YouTube'un API'ının güncellenmesinden kaynaklanır. Böyle durumlarda youtube-dl'i
`pip install --upgrade youtube-dl` komutuyla güncellemeyi deneyebilirsiniz. Eğer
youtube-dl, YouTube'un API değişikliklerine uygun olarak güncellenmişse sorun ortadan
kalkacaktır. Aksi durumda güncellemeleri takip etmelisiniz. Zaten güncellenmesi çok da
uzun sürmez.*

Muhteşem! Artık YouTube'dan iki tık ile müzik indirebiliyoruz. Ancak bir eksiğimiz var...
İndirdiğimiz müziklerin tagleri düzgün değil ve kapak fotoğrafları yok. Bu sebeple
müzik çalar uygulamamızda müziklerimizi yönetmemiz ve arama yapmamız hiç de kolay değil.

#### Tag Otomasyonu

Bir kaç ay öncesinde bu işi çok iyi yapan bir uygulama keşfettim; ismi **AutomaTag**.
Kendisi [Play Store]
(https://play.google.com/store/apps/details?id=com.fillobotto.mp3tagger)'da
barınıyor.

Bu uygulama cihazınızın medya veritabanını tarayıp, bulduğu parçaları size listeliyor.
Siz de ardından bu parçaları seçip tagleyebiliyorsunuz. Fakat bu işin güzel yanı;
uygulamanın, dosya ismine bakarak size tag önerisinde bulunabiliyor olması. Eğer öneri
doğru değilse veya uygulama herhangi bir tahminde bulunamadıysa; elle bir iki bilgi
(sanatçı ve parça adı gibi) girdikten sonra tekrardan arama yapabilirsiniz. Ayrıca
parça listesindeyken filtre bölümüne girip *Exclude songs with tag* seçeneğini aktif
ederseniz zaten tagli olan parçalar gizlenir.

#### Kapanış

youtube-dl, kolay yoldan müzik elde edebilmenin en iyi yollarından biri. Bu yazımda
youtube-dl'i Android cihazlarınızda nasıl daha etkili kullanabileceğinizi anlatmaya
çalıştım. Seneye (aklıma ne zaman yeni bir konu gelirse o zaman) bir başka yazıda
görüşmek üzere. Kendinize iyi bakın.
