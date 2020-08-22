# Chapter Bullets

**Chapter 1**

* OO temellerini öğrenmek sizi iyi bir OO tasarımcısı yapmaz.
* İyi OO tasarımları, tekrar kullanılabilir, genişletilebilir ve sürdürülebilirdir.
* Tasarım desenleri, OO tasarım kalitesi sunarak nasıl iyi sistemler inşa edebileceğiniz gösterir.
* Tasarım desenleri, kanıtlanmış OO tecrübeleridir.
* Desenler size kodu vermez, tasarım problemlerine karşı genel çözümleri sunar. Siz kendi spesifik uygulamanızı onu uygularsınız.
* Desenler icat edilmemişlerdir, keşfedilmişlerdir.
* Çoğu tasarım deseni ve prensip yazılımdaki **değişime** odaklanır.
* Çoğu tasarım deseni sistemin bir kısmını diğer kısmından bağımsız olarak çalışmasını izin verir.
* Çoğunlukla sistemde değişen kısmı alıp onu kapsülleriz. \(encapsulate\)
* Tasarım desenleri diğer yazılımcılarla olan iletişim kalitenizi artıracak ortak bir dil sağlar.

**Chapter 2**

* Observer deseni, nesneler arası bire-çok \(one-to-many\) ilişki tanımlar.
* Subject'ler ya da Observable'lar, ortak bir interface kullanarak Observar'ları günceller.
* Observer'lar gevşek bağlıdır, ki Observable Observer interface'ini implement ettikleri dışında haklarında bir şey bilmez.
* Deseni kullanırken, Observable'dan datayı "push" ya "pull" olarak alabiliriz. "pull" un daha doğru olduğu varsayılır.
* Observer'ların için belirli bir bildirim sırasına bağımlı olmayın.
* Observer deseninin, birçok farklı implementasyonu bulunmaktadır. Java'nın dahili genel amaçlı deseni bunlardan biridir.
* java.util.Observable problemli yanlarına dikkat edin.
* Gerektiğinde kendi implementasyonunuzu yazmaktan korkmayın.
* Swing Observer desenini çok sık bir şekilde kullanmaktadır, diğer pek çok GUI framework'ünde olduğu gibi.
* Bu deseni ayrıca birçok yerde bulabilirsiniz: JavaBeans, RMI gibi.

**Chapter 3**

* Kalıtım, genişletmenin\(extension\)  bir formudur ama her zaman esneklik sağlamaz, en iyi çözüm değildir.
* Tasarımlarımızda, var olan kodu değiştirmeden davranışın genişletilmesine izin vermeliyiz.
* Kompozisyon ve delegasyon, çalışma zamanında davranış eklemek için kullanılabilir.
* Decorator deseni, somut bileşenleri sarmalamak için kullanılan dekoratör sınıflardan oluşur.
* Decorator sınıflar, sarmaladıkları bileşenlerle aynı tipe sahiptirler. \(Kalıtım ya da interface implementasyonu yoluyla\)
* Dekoratörler, nesneyi iletmeden önce ve/veya sonra yeni bir işlev eklemektedir. 
* Bir bileşeni, istediğiniz sayıda dekoratörle sarmalayabilirsiniz.
* Dekoratörler tipik olarak component'in istemcisi için saydamdırlar, ancak istemci component'in somut tipine ihtiyaç duyuyorsa bu geçerli değildir.
* Dekoratörler, tasarımımızda birçok küçük nesnenin var olmasına sebep olabilir ve fazla kullanılması karmaşıklığı artırır.

**Chapter 4**

* Tüm fabrikalar nesne yaratımını kapsüller.
* "Simple Factory", gerçek bir tasarım deseni değil, sadece istemcileri somut sınıflardan ayrıştırmak için bir yoldur.
* "Factory Method" kalıtıma dayanır; nesne yaratımı, alt sınıflar tarafından factory method'u doldurarak yapılır.
* "Abstract Factory" nesne kompozisyonuna dayanır; nesne yaratımı, fabrika interface'inde belirtilen metodlarda uygulanır. 
* Tüm fabrika desenleri, uygulamanızın somut sınıflara bağımlılığını azaltarak gevşek bağlı bir tasarım uygular.
* Factory method'un niyeti, nesne yaratım işini alt sınıflara ertelemektir.
* Abstract Factory'nin niyeti somut sınıflara bağımlı olmaksızın ilişkili nesne aileleri yaratmaktır.
* Dependency Inversion prensibi, somut tiplere bağımlılığı azaltmamız ve soyutlamalaya çabalamamız için bir rehberdir.
* Fabrikalar, somut sınıflara değil, soyutlamalara kodlamak için güçlü bir tekniktir.

**Chapter 5**

* Singleton, uygulamanızda bir sınıfın en fazla bir nesnesi olmasını garantiler.
* Aynı zamanda bu nesneye, global erişim noktası sunar.
* Bu desenin Java'da uygulaması; private constructor ve statik değişkenle kombine statik bir metod şeklindedir.
* Multithreaded uygulamalar için doğru Singleton modelini seçmek için performans ve kaynak sınırlılıklarınızı gözden geçirin. \(Ayrıca, tüm uygulamaların multithreaded olduğunu varsaymalıyız.\)
* Eğer birden fazla "class loader" varsa bu Singleton mantığını bozabilir.

**Chapter 6**

* Command deseni, bir eylemi nasıl gerçekleştireceğini bilen nesne ile onu gerçekleştirmeyi isteyen nesneyi ayrıştırır.
* Bir Command nesnesi, ayrıştırmanın merkezinde; bir Receiver'ı bir eylemle birlikte kapsüller.
* Bir Invoker, Command nesnesinin execute\(\) metodunu çağırarak, Receiver'daki eylemleri harekete geçirir.
* Invoker'lar çalışma zamanında dinamik olarak bile Command'larla parametrelendirilebilir.
* Command'lar son execute çağrısından önceki durumu tutarak undo işlemini destekler.
* Macro Command'lar birden çok komutu çalıştıran Command'ın basit bir uzantısıdır. Aynı şekilde makro komutlar da kolayca undo destekleyebilir.
* Pratikte, bir isteği Receiver'a vekalet etmek yerine kendi gerçekleştiren "akıllı" komut nesneleri nadir bir durum değildir.
* Command'lar logging ve transaction uygulamalarında da kullanılabilir.

**Chapter 7**

* Var olan bir sınıfı kullanmak istediğinde ancak arayüzü\(interface\) istenilen biçimde olmadığında, adaptör kullan.
* Büyük bir interface'i ya da interface grubunu basitleştirmek ve birleştirmek için facade kullan.
* Bir adaptör, bir interface'i istemcinin beklediği biçime dönüştürür.
* Facade, karmaşık bir alt sistemi istemciden ayırır.
* Hedef interface'in büyüklüğüyle orantılı olarak, bir adaptör inşa etmenin yükü değişir.
* Bir Facade'i uygulamak için; Facade'i alt sistemlerle kompoze ederiz ve delegasyon / vekalet yöntemiyle facade istenilen işi gerçekleştirir.
* İki tür adaptör bulunur: Sınıf ve nesne adaptörü. Sınıf adaptörü çoklu kalıtım gerektirir.
* Bir alt sistem için birden fazla facade kullanılabilir.
* Bir adaptör interface'ini değiştirmek için nesneyi sarmalarken, dekoratör yeni sorumluluk veya davranış ekler, facade ise daha basit bir arayüz sunmak için sarmalar.

