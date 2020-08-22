---
description: Tasarım desenlerine giriş ve "Strategy" deseninin incelemesi.
---

# C1 - Intro To Design Patterns

"**Duck**", soyut\(abstract\) sınıfımız var. Tüm ördekler yüzebilir\(swim\) ve vaklayabilir\(quack\) ve aynı şekilde yapar. `Display()` metodumuz soyut, çünkü tüm alt tipler\(subclasses\) farklı görünüme sahip olacak.

![](.gitbook/assets/image%20%2842%29.png)

Daha sonra bu ördeklerin uçmasını istedik. Sorun çok basit gözüküyor. Sadece "Duck" sınıfına `fly()` metodunu ekleyebiliriz:

![](.gitbook/assets/image%20%288%29.png)

Bu bir hata olabilir, plastik ördek yani uçamayan bir ördek eklemek istersek, uçamadığı halde uçmak için bir metodu bulunacaktır. Geçici bir çözüm olarak `fly()` metodu hiçbir şey yapmayacak şekilde _override_ edilebilir. Ancak vaklamayan ve uçamayan bir ördek türü eklemek istersek yine bunları da _override_ etmek gerekecektir. Ve gelecekte eklenecek her ördek türü için bu işlemler yapılması gerekecektir. Burada anlıyoruz ki "~~kalıtım~~" bizim çözümümüz değil!

#### Interface Kullanmak?

![](.gitbook/assets/image%20%2823%29.png)

Bu şekilde, uçmak ve vaklamak için bir interface oluşturmak sorunun sadece bir kısmını çözmektedir \(uçabilen plastik ördek olmaması\). Eğer örneğin uçma davranışını değiştirmek istersek, bu davranışı kullanan bütün alt sınıfları kontrol etmek ve değiştirmek gerekebilir.  Ek olarak, bu davranışlar için kodun yeniden kullanılabilmesini engellemekte ve başka problemleri doğurmaktadır. Ayrıca birden fazla "uçma" davranışı da olabilir. Burada şu tasarım prensibini uygulayabiliriz:

{% hint style="info" %}
**Tasarım Prensibi:** Uygulamanızın değişen ve aynı kalan kısımlarını birbirinden ayırın!
{% endhint %}

![](.gitbook/assets/image%20%2843%29.png)

Biliyoruz ki, uçma ve vaklama davranışı bizim uygulamamızın değişebilen kısımları. Bu nedenle bu kısımları ördek sınıfından ayırıyoruz. Böylece "Duck" sınıfı, bu davranış sınıflarının ve implementasyonları hakkında bilgiye sahip olmayacak.

{% hint style="info" %}
**Tasarım Prensibi:** Bir interface'e \(bir süper tipe: interface, abstract class vb.\) programlayın, implementasyona değil.
{% endhint %}

Bu presibin anlamı; değişkenlerin tanımlanan türü genellikle soyut bir sınıf ya da interface olmak üzere bir süper tip olmalıdır, böylece bu değişkenlere atanan nesneler; bu süper tipin somut varlığı olabilir. Yani süper tip referansı alt tipleri gösterecektir. Böylece bu değişkeni tanımlayan sınıf, nesnenin tipini bilmek zorunda olmaz! Böylece, çalışma zamanında \(run-time\) nesne koda sıkıştırılmaz ve polimorfizmden \(çok biçimlilik\) faydalanılmış olur.

Her bir davranışı temsil etmek için, ve tek amacı bu davranışı temsil etmek olan sınıflar yaratacağız:

![](.gitbook/assets/image%20%281%29.png)

Bu tasarımla; "Duck" sınıfına bağımlı olmaksızın, davranışlar yeniden kullanılabilir ve farklı alanları düzenlemeye gerek kalmaksızın yeni davranışlar ekleyebiliriz.

#### Davranışları Entegre Etmek

Öncelikle iki sınıf değişkeni ekleyeceğiz. Bunlar; uçma davranışını temsil eden "_flyBehaviour_", vaklama davranışını temsil eden "_quackBehaviour_". Her "Duck" nesnesi, bu referansları kullanarak ve polimorfizmden faydalanarak çalışma zamanında spesifik uçma veya vaklama davranışını belirleyecektir. Ayrıca daha önce tanımladığımız `fly()` ve `quack()` metodlarını, `performFly()` ve `performQuack()` olarak değiştirdik:

![](.gitbook/assets/image%20%2839%29.png)

Şimdi bu metodları dolduralım:

```java
public class Duck{
    /* Her ördeğin, QuackBehaviour Interface'ini implement eden bir 
    referansı var. */
    QuackBehaviour quackBehaviour;
    
    public void performQuack(){
        /* Vaklama davranışı ile kendisi ilgilenmek yerine quackBehaviour
        ile gösterilen nesneyi kullanıyor. Hangi tip nesne olduğunu 
        bilmesine gerek yok */
        quackBehaviour.quack();
    }
}
```

Şimdi örnek olarak bir ördek sınıfının davranışlarını ayarlayalım:

```java
public class MallardDuck extends Duck{

    public MallardDuck(){
        quackBehaviour = new Quack();
        flyBeaviour = new FlyWithWings();
    }
    
    public void display(){
        System.out.println("I'm a Mallard Duck");
    }
}
```

**Soru:** Üstteki örnekte constructor içinde, Quack sınıfının somut bir nesnesini yaratıyoruz. Bu daha önce bahsettiğimiz implementasyon yerine üst tipe kodlamamızı söyleyen prensibe aykırı değil mi?

**Cevap:** Evet. Şimdilik bunu böyle yapıyoruz, ancak ileride bunu düzeltecek tasarımlar öğreneceğiz. Yine de, somut nesneyi yine bir üst tip ile referans gösterdiğimiz için \(`quackBehaviour = new Quack()`\) bunu çalışma zamanında kolayca değiştirme imkanına sahibiz. Bu da bize esneklik sağlamaktadır. Çalışma zamanında kolayca bir ördeğin davranışlarını değiştirebiliriz.

**Sonuç**

**HAS-A**, **IS-A** ilişkisinden daha iyi olabilir. Bu şekilde iki sınıfı bir araya getirmeye **composition** denir. Ördekler, davranışı kalıtım yoluyla almak yerine, doğru davranış nesnesi ile bir araya getiriliyor.

{% hint style="info" %}
**Tasarım Prensibi:** Kompozisyonu \(compositon\), kalıtıma \(inheritance\) tercih edin.
{% endhint %}

**Tebrikler!** Yukarıda uygulanan tasarım deseninin adı **Strategy Pattern.** 

**Strategy Pattern:** Bir algoritma ailesi tanımlar, her birini kapsül içine alır \(encapsulate\) ve bunları değiştirilebilir hale getirir. Strateji deseni, algoritmanın onu kullanan istemciden \(client\) bağımsız olarak değişmesine izin verir.

![](.gitbook/assets/image%20%2831%29.png)

