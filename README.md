# anchore ile docker container scanning

docker-compose ile componentler kurulacağı için lokalinizde veya remote bir serverda docker ve docker-compose kurulu olması gerekmektedir.

iki uygulamayı da yüklemek için scriptimden faydalanabilirsiniz.

```
curl https://raw.githubusercontent.com/alperen-selcuk/docker-install/main/ubuntu-1804.sh | bash - 
```

## anchore-engine install

öncelikle anchore engine çalışacağı bir volume yaratalım bu volume u mount edeceğiz.

```
mkdir ~/aevolume
cd aevolume
curl -O https://engine.anchore.io/docs/quickstart/docker-compose.yaml
```

docker compose ile ayağa kaldıracağız.

```
docker-compose pull
docker-compose up -d
```

tüm containerlar ayağa kalktıktan sonra api containerında system status a bakabiliriz hepsini UP görmemiz gerek.

```
docker-compose exec api anchore-cli system status
```

son adım olarak anchore kullanabilmek için anchore-cli yükleyeceğiz.

https://github.com/anchore/anchore-cli

github üzerinden kurabilirsiniz ben ubuntu için aşağıdaki şekilde devam edicem

```
apt-get update
apt-get install python3-pip
pip3 install anchorecli
```

burda .local e atıyor path olarak eklemeniz gerekiyor. export PATH="$HOME/.local/bin/:$PATH"

daha sonra 3 tane variable set edeceğiz bunlarla anchore apiye bağlanacağız.

```
export ANCHORE_CLI_URL=http://<IP adress>:8228/v1
export ANCHORE_CLI_USER=admin
export ANCHORE_CLI_PASS=foobar
```

(docker-compose içinden admin password ü değiştirebilirsiniz default olarak "foobar" olmaktadır)

## anchore engine kullanım

 anchore il kullanımda vuln scan eden catalog u güncellememiz gerekiyor.  "anchore-cli system feeds sync" komutu ile.

 anchore-cli image add <image:tag> 
 
 şeklinde kullanılmaktadır, default olarak docker.io da aramaktadır imageleri. eğer bir private repoda bulunan imagei scan etmek istiyorsanız. registry eklemeniz gerekiyor.
 ```
 anchore-cli registry add REGISTRY USERNAME PASSWORD
```

image tarama durumunu "anchore-cli image list" diyerek taranan imagedeki açıklara bakmak için ise "anchore-cli image vuln <image:tag> all" şeklinde kullanıyoruz.

!!!
PS: anchore ilk kullanım için “anchore-cli system feeds list” diyerek mevcut vulnerability referanslarının olduğunu kontrol ediniz eğer liste komple pending durumda ise, “anchore-cli system feeds sync” komutu ile sync hale getirmeniz gerekiyor bu işlem biraz uzun sürebilir.
!!!
