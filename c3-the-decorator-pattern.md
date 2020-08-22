# C3 - The Decorator Pattern

Starbuzz hızla büyüyen bir kahve dükkanı ve içecek sipariş sistemlerini güncellemek istiyorlar ve ilk tasarladıkları sınıflar aşağıdaki gibi:

![](.gitbook/assets/image%20%2821%29.png)

_Beverage_ abstract\(soyut\) bir sınıf ve tüm içecekler bu sınıfı miras alıyorlar ve kendi `cost()` metodlarını doldurarak içeceğin ücretini hesaplıyorlar. Daha sonra kahve çeşitlerine ek olarak, çeşitli maddeler\(süt, çikolata vb.\) ekleyeceklerini fark ettiler. İlk tasarım tercihleri şu şekilde oldu:

![](.gitbook/assets/image%20%2834%29.png)

Bu karışık yapının yanında, eğer yeni bir madde, süsleme eklemek istedikleri zaman ya da sütün fiyatı değiştiği zaman birçok yerde değişiklik yapmak zorunda kalacaklarını fark ettiler. Bunun saçma olduğunu fark ettiler; bu maddeleri değişken olarak tutan bir süper sınıfı miras alarak bu maddeleri takip etmeyi düşündüler:

![](.gitbook/assets/image%20%2846%29.png)

Buradaki düşünce: Her ek madde için boolean değerler var. Bu sefer cost\(\) metodunu soyut bırakmak yerine _Beverage_ sınıfında doldurduk, böylece bu maddelere bağlı olarak değişen maliyet bu sınıfta hesaplanacak. Alt sınıflar hala bu metodu override edecek ve kendi maliyetleri üzerine süper metoddan gelecek ek madde maliyetini ekleyecek. Böylece sınıf sayısını azaltmış olduk. Ancak ileride tasarım değişmesi gerekirse problemler doğuracaktır.

{% hint style="info" %}
**Tasarım Prensibi:** Sınıflar genişletmeye açık, düzenlemeye kapalı olmalıdır!
{% endhint %}

Sınıfa yeni davranışlar ekleyebilmeli, istediğimiz özellikleri ekleyerek genişletebilmeliyiz. Ancak var olan kodu değiştirmeden bunu yapabilmeliyiz. Bir sınıfın değiştirilmeye kapalı fakat genişletilmeye açık olması çelişkili gözükebilir ancak bunu yapmamızı sağlayan teknikler bulunmaktadır.

 Genişletilmesi gereken kod kısmını seçerken dikkatli olun! **Açık-Kapalı** prensibini her yere uygulamanın israflı, gereksiz olmasının yanı sıra, kodun okunması zor ve karışık olmasına yol açar.

### The Decorator Pattern

Yukarıdaki yapılan tasarımların yerine; bir içecek ile başlayıp çalışma zamanlı olarak onu çeşnilerle decorate \(süslemek, donatmak\) edeceğiz.

![](.gitbook/assets/image%20%2815%29.png)

Burada DarkRoast içeceği ile başladık. Daha sonra müşteri mocha istedi ve Mocha nesnesi burada bir dekoratör oldu. Aynı şekilde Whip de sarmaladı. Buradaki tüm tipler Beverage tipinde ve cost metoduna sahip. Fiyatı hesaplamak için en dıştaki cost metodunu çağırıyoruz. Her cost metodu sarmaladığı nesnenin cost metodunu çağırarak, ekleye ekleye fiyatı hesaplıyor.

Buradan şunu çıkardık:

* Dekoratörler, sarmaladıkları nesne ile aynı süper tipe sahip olmalıdır.
* Bir nesneyi sarmalamak için bir veya daha fazla dekoratör kullanabilirsin.
* Dekoratör, nesneyi iletmeden önce ya da sonra kendi davranışını eklemektedir. \(Anahtar nokta!\)
* Nesneler herhangi bir zamanda dekore edilebilirler. Böylece çalışma zamanında dinamik olarak nesneleri istediğimiz kadar sarmalayabiliriz.

**Decorator Pattern**, nesneye dinamik olarak ek sorumluluklar yükler. İşlevselliği genişletmek için, alt sınıf yaratmaya göre esnek bir alternatiftir.

Sınıf Diyagramı:

![](.gitbook/assets/image%20%2830%29.png)

Bu diyagrama bakarak Starbuzz projesine tasarımı uygularsak:

![](.gitbook/assets/image%20%2810%29.png)

Burada bize ilk tasarımdan itibaren soyut sınıflar verildiği için bu şekilde devam ediliyor. İstenirse interface'ler de kullanılabilir.

```java
public abstract class Beverage{
    String description = "Unknown beverage";
    
    public String getDescription(){ return description; }
    public abstract double cost();
}
```

```java
public abstract class CondimentDecorator extends Beverage{
    public abstract String getDescription();
}
```

```java
public class Espresso extends Beverage{
    public Espresso(){
        description = "Espresso";
    }
    
    public double cost(){
        return 1.99;
    }
}
```

```java
public class HouseBlend extends Beverage{
    public HouseBlend(){
        description = "HouseBlend";
    }
    
    public double cost(){
        return .89;
    }
}
```

```java
public class Mocha extends CondimentDecorator{
    Beverage beverage;
    
    public Mocha(Beverage beverage){
        this.beverage = beverage;
    }
    
    public String getDescription(){
        return beverage.getDescription() + ", Mocha";
    }
    
    public double cost(){
        return .20 + beverage.cost();
    }
}
```

```java
public class StarbuzzCoffee{
    public static void main(String[] args){
        // Sade Espresso
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $" + beverage.cost());
        
        // DarkRoast, Double Mocha, Whip
        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " $" + beverage2.cost());
        
        // HouseBlend, Soy, Mocha, Whip
        Beverage beverage3 = new HouseBlend();
        beverage3 = new Soy(beverage3);
        beverage3 = new Mocha(beverage3);
        beverage3 = new Whip(beverage3);
        System.out.println(beverage3.getDescription() + " $" + beverage3.cost());
    }
}
```

**Soru:** Örneğin; HouseBlend gibi belirli bir somut bileşende bir işlem yapmak istiyoruz. Mesela indirim yapacağız. Ancak HouseBlend'i dekoratörlerle sarmaladığımız için bu işlemi yapamayacağız.

**Cevap:** Evet, doğru. Eğer somut bileşenin tipine dayalı bir kodunuz varsa, dekoratörler kodu bozar. Soyut bileşenin tipinde kod yazmaya devam ettiğiniz sürece dekoratörler, problem yaratmayacaktır. Aksi takdirde dekoratörleri kullanma fikrini ve tasarımınızı tekrar gözden geçirmelisiniz.

### Real World Decorators: Java I/O

java.io paketine baktığınız zaman çok fazla sınıf olduğunu görürsünüz. İlk başta korkutucu gelse de, dekoratör desenini bildikten sonra bu paket daha anlamlı hale gelecektir. Çünkü bu pakette ağırlıklı olarak bu desen kullanılmıştır.

![](.gitbook/assets/image%20%2813%29.png)

Burada FileInputStream, StringBufferInputStream, ByteArrayInputStream gibi bileşenler bize byte okumak için temel bir bileşen verir. BufferedInputStream somut bir dekoratör sınıfıdır. Bu dekoratör, bufferlayarak performansı artırır ve interface'e karakter bazlı ve satır satır okuyan `readline()` metodunu ekler.  LineNumberInputStream de somut bir dekoratördür. Okuma yaparken satır numaralarını sayma yeteneğini ekler. Bu 2 dekoratör de, soyut dekoratör sınıfı olarak davranan FilterInputStream'den extend edilmiştir.

![](.gitbook/assets/image%20%2838%29.png)

Aynı şekilde output streamlerinin de aynı tasarıma sahip olduğunu görebilirsiniz. Bu desenin bir negatif sonucu olarak, çok sayıda küçük sınıflardan oluşması yazılımcıyı bunaltabilir.

### Kendi Java I/O Dekoratörünüzü Yazmak

Input Stream'deki tüm büyük harf karakterleri küçük harfe çeviren bir dekoratör yazalım:

```java
public class LowerCaseInputStream extends FilterInputStream{
    public LowerCaseInputStream(InputStream in){
        super(in);
    }
    
    public int read() throws IOException{
        int c = super.read();
        return (c == -1 ? c : Character.toLowerCase((char) c));    
    }
    
    public int read(byte[] b, int offset, int len) throws IOException{
        int result = super.read(b, offset, len);
        for(int i = offset; i < offset + result; i++){
            b[i] = (byte) Character.toLowerCase((char) b[i]);
        }
        return result;
    }
}
```

```java
public class InputTest{
    public static void main(String[] args){
        int c;
        try{
            InputStream in = 
                new LowerCaseInputStream(
                    new BufferedInputStream(
                        new FileInputStream("test.txt")));
                        
            while((c = in.read()) >= 0){
                System.out.println((char) c);
            }
            
            in.close();
        } catch(IOException exc){
            exc.printStackTrace();
        }
    }
}
```

