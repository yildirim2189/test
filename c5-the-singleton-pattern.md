# C5 - The Singleton Pattern

Sadece bir nesne olmasını gerektiren birçok durum vardır: Thread havuzları, cache, dialog kutuları, tercihleri yöneten nesneler, loglama için kullanılan nesneler vb. Eğer bu tip nesnelerden birden fazla olursa, yanlış program davranışı, kaynakların israfı, tutarsız sonuçlar gibi hatalar meydana gelmektedir. 

**Singleton deseni**, bize bir nesneden sadece bir tane yaratılmasını güvence altına alan bir desendir. Bunun yanında, global değişkenler gibi ancak eksi yönleri olmadan global bir erişim noktası sunar. Bu eksiler şu şekilde olabilir; eğer bir global değişkene bir nesne atar isek bu nesne program çalıştığında yaratılabilir \(eager\). Eğer bu nesne çok kaynak tüketen bir nesne ise kötü sonuçlar doğurabilir. Singleton deseninde, nesne biz istediğimiz zaman yaratılır.

```java
public class Singleton{
    
    // Tek nesnemizi tutacak statik değişken
    private static Singleton uniqueInstance;
    
    /* Constructor private, böylece sadece sınıf içinden yapılacak 
    çağrı ile bu sınıfın nesnesi yaratılabilir. */
    private Singleton(){}
    
    /* Bu sınıfın nesnesini yaratmadan erişebilmek için statik metod.
    Eğer değişkenimiz null ise yeni nesne yaratıyoruz. Eğer null değil 
    ise daha önce yaratılmıştır. */
    public static Singleton getInstance(){
        if(uniqueInstance == null){
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

![](.gitbook/assets/image%20%2836%29.png)

**Problem:** Yukarıdaki gibi Singleton deseni kullanan sınıfımızın birden fazla nesneye sahip olduğunu ve programımızın bozulduğunu farkettik. Bunun sebebi ise birden fazla thread'in aynı anda metoda erişerek nesne yaratması olabilir mi?

**Multithreading**

```java
public class Singleton{
    private static Singleton uniqueInstance;
    
    private Singleton(){}
    
    public static synchronized Singleton getInstance(){
        if(uniqueInstance == null){
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

**synchronized** anahtar kelimesini ekleyerek, aynı anda birden fazla thread'in metoda girmesini engelledik. Çok basit bir şekilde problemi çözmüş olduk. Ancak "synchronization" biraz pahalı bir yöntem. Çünkü sadece nesne ilk yaratıldığı zaman işe yarıyor, yaratıldıktan sonra yapılacak tüm getInstance\(\) çağrıları için gereksiz olacak. Peki ne yapabiliriz:

1. getInstance\(\) performansı, uygulama için kritik değilse hiçbir şey yapmayın. Senkronize etmek sorunu çözdü ve etkili. Ancak metod üzerinde yüksek bir trafik varsa uygulamanın hızı önemli ölçüde düşecektir.
2. Eğer nesne uygulanın başında ya da sürekli yaratılıyorsa ve yaratım külfetli değilse, "Lazy" nesne yaratımı yerine, "eager" nesne yaratımını tercih edebiliriz. Bu şekilde JVM sınıf yüklendiği anda, herhangi bir thread'in erişiminden önce nesneyi yaratacaktır.

```java
public class Singleton{
    private static Singleton uniqueInstance = new Singleton();
    
    private Singleton(){}
    
    public static synchronized Singleton getInstance(){
        return uniqueInstance;
    }
}
```

3. "Double-checked Locking" \(Çift kontrol kilitleme\) kullanarak getInstance senkronizasyonu azaltılabilir. Bu şekilde önce nesne yaratılmış mı kontrol ederiz, yaratılmamışsa senkronize ederiz. Bu şekilde sadece ilk yaratımda senkronize etmiş oluruz:

```java
public class Singleton{
    
    /* volatile anahtar kelimesi, uniqueInstance değerinin farklı 
    threadler tarafından doğru okunmasını sağlayacaktır. */
    private volatile static Singleton uniqueInstance;
    
    private Singleton(){}
    
    public static Singleton getInstance(){
        
        /* Sadece ilk yaratımda senkronize bloka girecek. İlk if blokuna
        aynı anda erişim ihtimaline karşın ikinci kontrol yapılarak
        garanti altına alınıyor.
        */
        if(uniqueInstance == null){
            synchronized(Singleton.class){
                if(uniqueInstance == null){
                    uniqueInstance = new Singleton();
                }
            }
        }
        
        return uniqueInstance;
    }
}
```

**Soru:** Eğer tüm metod ve değişkenleri statik olan bir sınıf yaratsaydım, bu Singleton ile aynı olmaz mıydı?

**Cevap:** Evet, sınıfınız bağımsızsa ve karmaşık yüklemeye bağlı değilse. Bununla birlikte, Java'da statik yüklemelerin yapısı nedeniyle, bu durum, özellikle birden fazla sınıf söz konusu olduğunda çok karmaşık olabilir. Genellikle bu senaryo, yüklenme sırasını içeren ince, bulunması zor hatalarla sonuçlanabilir. "Singleton",u bu şekilde uygulamaya zorlayan bir ihtiyaç olmadıkça, nesne dünyasında kalmak çok daha iyi olacaktır.

Bunun yanında eğer birden fazla "class loader" ve "singleton" kullanıyorsanız dikkatli olmanız gerekir. Eğer bir sınıf, farklı "class loader" tarafından birden fazla yüklenirse birden fazla "Singleton" nesnesine sahip olabilirsiniz.



