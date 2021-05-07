+++
title = "Docker101"
date = 2020-04-08T17:08:07+03:00
author = "personinblack <berkay@tuta.io>"
cover = "programlama.docker101.jpeg"
tags = ["programlama"]
keywords = ["docker", "container", "cross-platform", "programlama"]
description = """
Docker, uygulamalarınızı konteynırlar içerisine alarak, her koşulda istenildiği gibi
çalışmalarını sağlayan bir araçtır. Peki Docker bunu nasıl sağlar? Docker ile nasıl
cross-platform bir uygulama geliştirebiliriz? Benimle kal, mevzu oldukça basit.
"""
+++

Docker, uygulamalarınızı konteynırlar içerisine alarak, her koşulda istenildiği gibi
çalışmalarını sağlayan bir araçtır. Peki Docker bunu nasıl sağlar? Docker ile nasıl
cross-platform bir uygulama geliştirebiliriz? Benimle kal, mevzu oldukça basit.

#### Giriş

Öncelikle https://docs.docker.com/get-docker/ sayfası üzerinden Docker'ı edinmemiz gerek.
Bu işlem tüm işletim sistemlerinde farklı olduğu için burada değinemiyorum fakat bize
**Docker Engine** veya **Docker Desktop** gerekli (*bu yazıda Docker Compose'dan
bahsetmeyeceğim, bu yüzden kendisine ihtiyacımız yok*). Kurulumu tamamladıktan sonra
sıradaki aşamaya geçebiliriz.

> *Not: `docker --version` komutu ile kurulumun başarılı olduğundan emin olabilirsin.*

#### Başlangıç

Bu blog yazısı için beraber basit bir Node.js projesi oluşturacağız, fakat yazıyı takip
edebilmek için Node.js veya JavaScript bilmen gerekmiyor.

Projemiz için birkaç dizin oluşturarak işe koyulalım: `mkdir ~/projects/dockertut && cd
~/projects/dockertut`

Ardından şu ufak Node.js kodunu *app.js* isminde bir dosya olarak kaydedelim. Bu bizim
basit uygulamamız olsun:

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const hostname = '127.0.0.1';
const port = 80;
const INDEX_FILE = 'data/index.txt';

fs.mkdirSync(path.dirname(INDEX_FILE), (err) => {});

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');

  // INDEX_FILE'dan indexi al
  let index = fs.existsSync(INDEX_FILE) ? fs.readFileSync(INDEX_FILE) : 0;

  // Mevcut indexi kullanıcıya göster
  res.end(`Index: ${index}`);

  // Indexe bir ekleyip INDEX_FILE'a kaydet
  fs.writeFile(INDEX_FILE, ++index, (_)=>{});
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

Docker konteynırları; *imajlar (image)*, *ağlar (network)* ve *volumelardan (saklama
alanları)* oluşurlar. İmajlar bir konteynırın hangi içerikle üretileceğini belirlememizi
sağlar. Örneğin biz bu örnekte kendi imajımızı oluştururken Node.js'in *Alpine Linux* ile
birlikte gelen Docker imajını kullanacağız. Bu imaj ile birlikte hazır bir Alpine Linux
kurulumu ve beraberinde gelen Node.js paketi kullanımımıza hazır olacak.

#### Dockerfile

İmajlar, *Dockerfile* isimli bir dosyadan aldıkları talimatlar doğrultusunda
oluşturulurlar. `touch Dockerfile` diyerek bir tane oluşturduktan sonra, kendisini metin
editörümüz ile düzenlemeye başlayalım.

Bizim imajımız başka bir imajın bir nevi uzantısı olacak. Dockerfile'ımızın ilk
satırında bu imajı belirtmemiz gerekiyor ki, sonraki adımlara geçtiğimizde elimizde
üzerinde çalışabileceğimiz bir sistem olsun. Bunun için de
[FROM](https://docs.docker.com/engine/reference/builder#from) direktifini kullanıyoruz:

```Dockerfile
FROM node:13-alpine
```

Sırada uygulamamızı konteynırın içerisine taşımak var. Kendisinin ebediyen barınacağı
dizine gittikten sonra, *app.js* dosyasını yerel makineden konteynıra aktaralım:

```Dockerfile
FROM node:13-alpine
WORKDIR /appdir
COPY app.js ./
```

[WORKDIR](https://docs.docker.com/engine/reference/builder#workdir) direktifi ile
mevcut çalışma dizinimizi değiştiriyoruz. */appdir* adında bir klasör var olmasa dahi
*WORKDIR* bizim için bu klasörü oluşturacak. Devamında
[COPY](https://docs.docker.com/engine/reference/builder#copy) ile dosyayı yerelden alıp
konteynırdaki çalışma dizinimize yerleştiriyoruz.

Uygulamamız *80* portu üzerinde çalışacak fakat bu port şu anda konteynırın dışından
erişilemez durumda. Portu dışarıdan erişilebilir hale getirmek için
[EXPOSE](https://docs.docker.com/engine/reference/builder#expose) direktifini ekleyelim:

```Dockerfile
FROM node:13-alpine
WORKDIR /appdir
COPY app.js ./
EXPOSE 80
```

Son olarak uygulamamızın konteynır içerisinde hangi komut ile çalışacağını
[ENTRYPOINT](https://docs.docker.com/engine/reference/builder#entrypoint) direktifi ile
tanımlayalım:

```Dockerfile
FROM node:13-alpine
WORKDIR /appdir
COPY app.js ./
EXPOSE 80
ENTRYPOINT [ "node", "app.js" ]
```

> *ENTRYPOINT* direktifi, imajımız *çalıştırıldığında* hangi komutun gerçekleştirileceğini
> belirtmemizi sağlar ve genellikle Dockerfile'ın sonunda bulunur. Eğer ki imajın
> *hazırlığı* sırasında komut girmek istersek
> [RUN](https://docs.docker.com/engine/reference/builder#run) direktifini kullanabiliriz.
>
> *Not: Aynı zamanda ENTRYPOINT direktifinin bir alternatifi de olan
> [CMD](https://docs.docker.com/engine/reference/builder#cmd) direktifi, ENTRYPOINT
> direktifine argümanlar belirtmek için kullanılabilir. Örneğin uygulamamız opsiyonlar
> kabul ediyor olsaydı şunu yapabilirdik:*
```Dockerfile
ENTRYPOINT [ "node", "app.js" ]
CMD [ "--option1=true", "--option2=false" ]

# veya (ENTRYPOINT yerine CMD kullanabiliriz)
ENTRYPOINT [ "node", "app.js", "--option1=true", "--option2=false" ]

# veya (tavsiye edilmez) (yine CMD yerine ENTRYPOINT kullanabiliriz)
CMD node app.js --option1=true --option2=false
```

#### Build && Run!

İmajımız builde hazır. `docker build -t blackness/docker101 .` komutu ile
build edebilir (sondaki nokta Dockerfile'ın bulunduğu dizin) ve
`docker run -dp 8080:80 blackness/docker101` komutu ile de çalıştırabiliriz.

`docker run` komutuna girdiğimiz
[ `-d`](https://docs.docker.com/engine/reference/run#detached--d) parametresi *detach*
anlamına gelir ve konteynırı arkaplanda çalıştırmamızı sağlar.
[ `-p yerel:konteynır`](https://docs.docker.com/engine/reference/#expose-incoming-ports)
parametresi ise konteynır ve yerel makine arasındaki 2 portu bağlamamız, konteynırın
portunu expose etmemiz için gerekli. Tüm parametreleri görmek istersen `docker run --help`
veya `man docker-run` komutlarından birini kullanabilirsin.

Bu aşamadan sonra tarayıcımız yardımı ile veya `curl` komutuyla "localhost:8080"
adresine bağlı uygulamamıza erişebilmemiz gerek. Ama erişemiyoruz? Peki neden? Çünkü
uygulamamız konteynır içerisindeki *127.0.0.1* adresini (localhost) dinliyor fakat
bu adres konteynır dışarısına yönlendirilmiyor.

Sorunu çözmek için *127.0.0.1* değerini *0.0.0.0* ile değiştirerek Docker'ın
konteynırımıza tanımladığı [arayüz](https://en.wikipedia.org/wiki/Network_interface) de
dahil tüm arayüzleri tek seferde dinleyebiliriz.

Pekala gerekli değişikliği yaptık ve imajımızı tekrar çalıştırmak istiyoruz. Bunun
için öncelikle `docker ps` komutuyla çalışan konteynırımızın ID'sini alalım. Ardından
`docker kill <Konteynır ID>` komutuyla kendisini ortadan kaldıralım. Yaptığımız
değişikliğin geçerli olabilmesi için imajımızı aynı komutlar ile yeniden build edelim ve
çalıştıralım.

Artık uygulamamız çalışıyor ve sayfayı her yenilediğimizde index değerinin arttığını
görebiliyoruz.

> *Not: İmajı yeniden build etmemizin sebebi "app.js" dosyasına yaptığımız
> değişikliklerin imaj içerisindeki "app.js" dosyasına yansımıyor olmasıdır. Dosyayı
> tekrardan imaj içerisine kopyalayarak güncellememiz gerekiyor.*

#### Konteynırlarda Kalıcı Veri ve Volumelar

Eğer ki önceki bölümde oluşturduğumuz imajımızı yeniden başlatırsak index değerinin
sıfırlandığını görebiliriz. Bunun sebebi konteynırı yokettiğimizde içerisindeki verilerin
de Docker'ın geçici saklama alanından siliniyor olmasıdır. Bu durumu düzeltebilmek için
"index.txt" dosyasını bir *volume* içerisinde kalıcı olarak saklamamız gerekiyor.

[VOLUME](https://docs.docker.com/engine/reference/builder#volume) direktifini kullanarak
indexi barındıran *data* klasörünü bir volume'a çevirelim:

```Dockerfile
FROM node:13-alpine
WORKDIR /appdir
COPY app.js ./

VOLUME [ "/appdir/data" ]

EXPOSE 80
ENTRYPOINT [ "node", "app.js" ]
```

*VOLUME* direktifimizin bir işe yaraması için volume'u bir yere
[mount](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)
etmemiz gerekiyor. Bunun için de imajımızı başlatmak için kullandığımız komutu şu şekilde
değiştirmeliyiz:

`docker run -dp 8080:80 -v myvol:/appdir/data blackness/docker101`

`-v` / `--volume` opsiyonu iki argümandan oluşmakta. Docker ilk argümanda belirttiğimiz
isim ile (*myvol*) aynı isimde bir Docker volume'u oluşturduktan sonra bu volume'u
ikinci argümanda ve *VOLUME* direktifinde belirtilen konuma mount ediyor. *VOLUME*
argümanından **önce** volume konumunda başka bir dosya oluşturmuşsak bu dosya silinmiyor
fakat üzerine mount işlemi gerçekleştirdiğimiz için erişilemez bir hale geliyor. Bundan
dolayı *VOLUME* direktifini önce belirtip üzerinde gerçekleşecek işlemleri sonrasında
yapmalıyız.

#### SON

Bu belki biraz uzun blog yazısında beraber küçük bir Docker imajı oluşturduk. Umarım
okudukların sana birşeyler katmış ve Docker kullanmaya teşvik etmiştir. Bir sonraki
yazımda belki Docker Compose'u anlatabilirim ama anlatmaya da bilirim. Bilemiyorum,
hayatta hiçbir şey kesin değil.
