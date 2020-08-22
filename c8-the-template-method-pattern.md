---
description: Algoritmaları Kapsüllemek (Encapsulating Algorithms)
---

# C8 - The Template Method Pattern

Bazı insanlar kahve düşkünüyken bazı insanlar çay düşkünüdür. İkisinin de kafein gibi ortak özellikleri ya da hazırlanırken ortak adımları vardır.

{% tabs %}
{% tab title="Coffee" %}
```java
public class Coffee{
    void prepareRecipe(){
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugarAndMilk();
    }
    
    /* Her bir metod bir algoritmanın bir adımını temsil ediyor. */
    public void boilWater(){
        System.out.println("Boiling water..");
    }
    
    public void brewCoffeeGrinds(){
        System.out.println("Dripping coffee through filter..");
    }
    
    public void pourInCup(){
        System.out.println("Pouring into cup..");
    }
    
    public void addSugarAndMilk(){
        System.out.println("Adding sugar and milk..");
    }
}
```
{% endtab %}

{% tab title="Tea" %}
```java
public class Tea{
    void prepareRecipe(){
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }
    
    /* Her bir metod bir algoritmanın bir adımını temsil ediyor.
    boilWater() ve pourInCup() metodları kahve ile ortak.
    */
    public void boilWater(){
        System.out.println("Boiling water..");
    }
    
    public void steepTeaBag(){
        System.out.println("Steeping the tea..");
    }
    
    public void pourInCup(){
        System.out.println("Pouring into cup..");
    }
    
    public void addLemon(){
        System.out.println("Adding lemon..");
    }
}
```
{% endtab %}
{% endtabs %}

Coffee ve Tea sınıflarına baktığımızda ortak metodların bulunduğunu görüyoruz. Kod tekrarını engellemek istiyoruz. Aklımıza ilk gelen tasarım şu şekilde olabilir:

![](.gitbook/assets/image%20%289%29.png)

Tamamen aynı olan boilWater\(\) ve pourInCup\(\) metodlarını soyutladık. prepareRecipe\(\) metodunu ise alt sınıflarda dolduruyoruz. Diğer metodları soyutlamadık ama temel olarak aynı şeyi yapıyorlar. Sıcak sudan kahve veya çay çekiyoruz ya da bu içeceklere bazı maddeler ekliyoruz. Bu yüzden prepareRecipe\(\) metodunu da soyutlamayı deneyelim:

{% tabs %}
{% tab title="CaffeineBeverage" %}
```java
public abstract class CaffeineBeverage{
    
    // Bu metodun override edilmesini istemiyoruz. Bu yüzden "final".
    final void prepareRecipe(){
        boilWater(); // su kaynat.
        brew(); // çay veya kahveyi demle.
        pourInCup(); // bardağa dök.
        addCondiments(); // çeşni ekle.
    }
    
    /* Kahve ve çay bu işlemleri farklı yaptıkları için, bu metodlar
    "abstract". Kahve ve çay sınıfları kendileri ilgilenecekler. */
    abstract void brew();
    abstract void addCondiments();
    
    void boilWater(){
        System.out.println("Boiling water..");
    }
    
    void pourInCup(){
        System.out.println("Pouring into cup..");
    }
}
```
{% endtab %}

{% tab title="Coffee" %}
```java
public class Coffee extends CaffeineBeverage{
    public void brew(){
        System.out.println("Dripping the coffee through filter..");
    }
    
    public void addCondiments(){
        System.out.println("Adding sugar and milk..");
    }
}
```
{% endtab %}

{% tab title="Tea" %}
```java
public class Tea extends CaffeineBeverage{
    public void brew(){
        System.out.println("Steeping the tea..");
    }
    
    public void addCondiments(){
        System.out.println("Adding lemon..");
    }
}
```
{% endtab %}
{% endtabs %}

### Template Method \(Şablon Metod\) Deseni

Template / Şablon deseni, bir algoritmanın adımlarını tanımlar ve alt sınıflara bir veya daha fazla adımı kendinin doldurmasına izin verir.

**Template Method Pattern,** bir metoddaki algoritmanın iskeletini tanımlar ve bazı adımları alt sınıfların insiyatifine bırakır. Bu desen; alt sınıflara, bir algoritmanın yapısını değiştirmeden sadece belirli adımlarını tekrar tanımlama imkanı sağlar.

| Desensiz Tea ve Coffee Uygulaması | Yeni, Template Method Uygulaması |
| :--- | :--- |
| Coffee ve Tea algoritmayı ayrı ayrı kontrol ediyor. | CaffeineBeverage algoritmayı tutuyor ve koruyor. |
| Coffee ve Tea üzerinde kod tekrarı var. | Tekrar kullanım maksimize edildi. |
| Algoritmanın değişmesi, alt sınıflarda ve birden fazla değişikliği gerektiriyor. | Algoritma sadece bir yerde yaşıyor ve değişim sadece burada gerçekleşiyor. |
| Yeni bir kafeinli içeçek eklenmesi durumunda fazla iş yükü gerektiren sınıf organizasyonu. | Yeni içeceklerin sadece belirli metodları doldurması gerekiyor. Yeni içecekler için bir çerçeve sağlıyor. |
| Algoritmanın bilgisi ve uygulaması farklı sınıflara dağılmış durumda. | CaffeineBeverage sınıfı algoritma bilgisine sahip ve tek bir yerden sınıflara ulaşıyor. |

![Template Method Pattern - S&#x131;n&#x131;f Diyagram&#x131;](.gitbook/assets/image%20%2829%29.png)

Burada soyut sınıfa "hook" metod denilen abstract olmayan fakat hiçbir şey yapmayan metodlar eklenebilir. Bu metodlar alt sınıflar tarafından doldurulması isteğe bağlıdır. "Hook" bulunan örneği inceleyelim:

{% tabs %}
{% tab title="CaffeineBeverageWithHook" %}
```java
public abstract class CaffeineBeverageWithHook{
    final void prepare(){
        boilWater();
        brew();
        pourInCup();
        if(customerWantsCondiments){
            addCondiments();
        }
    }
    
    abstract void brew();
    abstract void addCondiments();
    
    void boilWater(){
        System.out.println("Boiling water..");
    }
    
    void pourInCup(){
        System.out.println("Pouring into cup..");
    }
    
    /* "Hook" metod. İsteğe bağlı olarak alt sınıflar tarafından
    override edilebilir. Varsayılan olarak çeşnileri ekliyor. */
    boolean customerWantsCondiments(){
        return true;
    }
}
```
{% endtab %}

{% tab title="CoffeeWithHook" %}
```java
public CoffeeWithHook extends CaffeineBeverageWithHook{
    public void brew(){
        System.out.println("Dripping the coffee through filter..");
    }
    
    public void addCondiments(){
        System.out.println("Adding sugar and milk..");
    }
    
    // İsteğe bağlı olarak override ediyoruz.
    public boolean customerWantsCondiments(){
        String answer = getUserInput();
        
        if(answer.toLowerCase().startsWith("y")){
            return true;
        } else{
            return false;
        }
    }
    
    public String getUserInput(){
        String answer = null;
        System.out.println("Kahvenizde şeker ya da süt " 
        + "ister misiniz ? (y/n)");
        BufferedReader in = 
            new BufferedReader(new InputStreamReader(System.in));
        
        try{
            answer = in.readLine();
        } catch(IOException exc){
            System.err.println("IO hatası!");
        }
        
        if(answer == null){
            return "no";
        }
        return answer;
    }
}
```
{% endtab %}

{% tab title="Demo" %}
```java
public class Demo{
    public static void main(String[] args){
        TeaWithHook teaHook = new TeaWithHook();
        CoffeeWithHook coffeeHook = new CoffeeWithHook();
        
        System.out.println("Çay hazırlanıyor...\n");
        teaHook.prepareRecipe();
        
        System.out.println("Kahve hazırlanıyor...\n");
        coffeeHook.prepareRecipe();
    }
}
```
{% endtab %}
{% endtabs %}

**Soru:** Bir template metod yaratırken, ne zaman abstract ne zaman hook metod kullanmalıyım?

**Cevap:** Algoritmanın adımı seçeneğe bağlı ise "hook", alt sınıflar algoritmanın bir adımını doldurmak zorunda ise "abstract" metod kullanılmalı.

### Hollywood Prensibi

{% hint style="info" %}
**Tasarım Prensibi:** Bizi arama, biz sizi ararız.
{% endhint %}

Bu prensip; yüksek seviye bileşenlerin alt seviye bileşenlere bağımlı olduğu, onların da yüksek seviye bileşenlere bağımlı olduğu, onların yan bileşenlere bağımlı olduğu, onların da düşük seviye bileşenlere bağımlı olduğu... diye süren ve kimsenin sistemin nasıl tasarlandığını anlamadığı yapılardan kaçınmayı amaçlar. Hollywood prensibinde; alt seviye bileşenler sisteme "takılırlar\("hook"\). Onların kullanıp kullanılmayacağına üst seviye bileşenler karar verir. Kısacası üst seviye bileşenler, alt seviye bileşenlere "İhtiyacımız olursa, biz sizi ararız." davranışını sunar. Template Method deseni ile alt sınıflara bunu yapıyoruz:

![Hollywood Prensibi](.gitbook/assets/image%20%285%29.png)

### Template Method ile Sıralama

Java'da dizileri sıralamak için kullanışlı bir şablon metod sunduğunu görüyoruz:

```java
public static void sort(Object[] a){
    Object aux[] = (Object[])a.clone();
    mergeSort(aux, a, 0, a.lenght, 0);
}
```

