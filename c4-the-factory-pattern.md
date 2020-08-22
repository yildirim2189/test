# C4 - The Factory Pattern

Önceki bölümlerde bahsettiğimiz prensibe göre, implementasyonlara değil soyut, süper tiplere programlamamız gerekiyor. Ancak **new** kelimesini kullandığımız her yerde aksini yapıyoruz, çünkü **new** ile somut bir sınıfın nesnesini yaratıyoruz. Ne kadar somut sınıflara kodlama yaparsak o kadar değişmesi, büyümesi zorlaşan kod yaratıyoruz.

Diyelim ki bir pizza dükkanı işletiyoruz:

```java
Pizza orderPizza(){
    Pizza pizza = new Pizza();
    /* Burada soyut bir sınıf ya interface kullanmak istiyoruz ancak,
    bunların örneğini oluşturmamız mümkün değil */
    
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

Ancak tek bir pizza tipinden daha fazlasına ihtiyacımız var ve derleme zamanında bunu bilmiyoruz:

```java
Pizza orderPizza(String type){
    Pizza pizza;
    
    // Kodumuzun sürekli değişen kısmı
    if(type.equals("cheese")){
        pizza = new CheesePizza();
    } else if(type.equals("greek")){
        pizza = new GreekPizza();
    } else if(type.equals("pepperoni")){
        pizza = new PepperoniPizza();
    }
    
    // Kodumuzun değişmeyen kısmı
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

Ancak daha sonra yeni pizza çeşitleri eklemek istiyoruz ya da "Greek" tipi pizza satış yapmadığı için menüden kaldırmak istiyoruz. Görüyoruz ki, var olan kodu sürekli değiştirmek zorundayız. Kodumuzun değişen ve değişmeyen kısımlarını biliyoruz. Bu yüzden, değişen nesne yaratma kısmını farklı bir yere taşımak ve kapsüllemek mantıklı olacaktır. 

```java
Pizza orderPizza(String type){
    Pizza pizza;
    
    /*
    Peki buraya ne gelecek? Değişen kısmı kaldırdık. Buraya sadece pizza nesneleri
    yaratmaya odaklanan bir nesne koyacağız. Eğer herhangi bir nesne pizza 
    nesnesine ihtiyaç duyarsa bu nesneye gelecek.
    Bu nesneye "Factory" / "Fabrika" diyoruz.
    */
    
    // Kodumuzun değişmeyen kısmı
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

**Fabrikalar** \(factories\), nesne yaratmanın detaylarıyla ilgilenir. Basit bir pizza fabrikasına sahip olduktan sonra `orderpizza()` metodu, sadece fabrikamıza bir pizza yapması için istekte bulunacaktır. Pizza çeşitleri ya da aralarındaki farklar gibi bilgilerden haberdar olmayacaktır, ki bizim istediğiniz şey de bu.

#### Simple Factory \(Basit Fabrika\) İnşa Etmek

```java
public class SimplePizzaFactory{
    public Pizza createPizza(String type){
        if(type.equals("cheese")){
            pizza = new CheesePizza();
        } else if(type.equals("greek")){
            pizza = new GreekPizza();
        } else if(type.equals("pepperoni")){
            pizza = new PepperoniPizza();
        }
        return pizza;
    }
}
```

Burada sadece problemi başka bir yere taşımız gibi görünüyoruz. Ancak fabrikanın birçok müşterisi olabilir. Pizza yaratmayı kapsüllediğimiz için bir değişiklik olduğunda düzenleyeceğimiz tek bir yer var. Ayrıca istemci kodundan somut kodlamayı kaldırmış olacağız. Bazı yerlerde fabrika metodunun statik olduğunu görebilirsiniz, bu yaygın bir yöntemdir. Eğer sadece bir metodu kullanmak istiyorsanız, o sınıfın nesnesini yaratmaya gerek olmadan bunu yapmak mantıklıdır. Ancak bunun dezavantajı, miras alamazsınız ve metodun davranışını değiştiremezsiniz.

```java
public class PizzaStore{
    SimplePizzaFactory factory;
    
    public PizzaStore(SimplePizzaFactory factory){
        this.factory = factory;
    }
    
    public Pizza orderPizza(String type){
        Pizza pizza;
        // Artık burada somut kodlama yok! Üst tipe kodluyoruz.
        pizza = factory.createPizza(type);
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

**Simple Factory** aslında bir desen değil, daha çok bir deyim diyebiliriz ve sık kullanılmaktadır. Bu fabrika mantığını anlamak için bir ısınma turuydu diyebiliriz çünkü birazdan iki tane ağır sorumluluğa sahip fabrika tipi tasarım deseni göreceğiz.

#### Franchising

Pizza dükkanı çok popüler oldu ve herkes kendi mahallesinde bir dükkan istiyor. Ayrıca her şube, kendi yerel pizzasını yapmak istiyor. Örneğin New York şubesi, kendi New York tipi pizza üreten, Chicago kendi tipinde pizza üreten bir fabrika istiyor. Bir yaklaşım, farklı fabrikalar oluşturup, bu pizza şubelerine doğru pizza fabrikasıyla kompoze edebiliriz.

```java
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.order("Pepperoni");
```

Bu bir çözüm olabilir ancak biraz daha kalite kontrol iyi olabilir. Bu dükkanlar bizim fabrikamızı kullanabilir ancak kendi prosedürlerini uygulayabilirler; örneğin, farklı pişirebilirler ya da pizza kesmeyi unutabilirler. Problemi tekrar düşündüğümüzde, dükkan ile pizza üretimini birbirine bağlayan ancak yine bazı şeyleri esnek biçimde bırakan bir framework \(çerçeve\)'e ihtiyacımız olduğunu görüyoruz.

Pizza yapma aktivitelerini PizzaStore sınıfına özelleştiren ve hala şubelere kendi yerel tarzlarını ekleme esnekliği sağlayan bir yol var. 

```java
public abstract class PizzaStore{
    public Pizza orderPizza(String type){
        Pizza pizza;
        
        pizza = createPizza(type);
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    
    protected abstract Pizza createPizza(String type);
}
```

Her şube createPizza\(\) metodunu override ederek kendi tarzında pizza pişirebilir. 

![](.gitbook/assets/image%20%2811%29.png)

```java
public class NYPizzaStore extends PizzaStore{
    Pizza createPizza(String item){
        if(item.equals("cheese")){
            return new NYStyleCheesePizza();
        }else if(item.equals("veggie")){
            return new NYStyleVeggiePizza();
        }else if(item.equals("clam")){
            return new NYStyleClamPizza();
        }else if(item.equals("pepperoni")){
            return new NYStylePepperoniPizza();
        }else{
            return null;
        }
    }
}
```

Üst sınıfta bulunan, `orderPizza()` metodunun hangi tip pizza pişirdiğimizden haberi yok. O sadece hazırlama, kesme, kutulama vb. işlemleriyle ilgileniyor. Pizza nesnesini yaratma sorumluluğu, fabrika gibi davranan bir metoda taşındı. Bir fabrika metodu, nesne yaratımıyla ilgilenir ve onu alt bir sınıfta kapsüller. Bu üst sınıftaki istemci koduyla, alt sınıftaki nesne yaratılan kodu birbirinden ayrıştırır.

Pizza sınıfını ekleyelim:

```java
public abstract class Pizza{
    String name;
    String dough;
    String sauce;
    ArrayList toppings = new ArrayList();
    
    void prepare(){
        System.out.println("Preparing " + name);
        System.out.println("Tossing dough...");
        System.out.println("Adding sauce...");
        System.out.println("Adding toppings: ");
        for(int i = 0; i < toppings.size(); i++){
            System.out.println("   " + toppings.get(i));
        }
    }
    
    void bake(){
        System.out.println("Bake for 25 min at 350");
    }
    
    void cut(){
        System.out.println("Cutting the pizza into diagonal slices.");
    }
    
    void box(){
        System.out.println("Place pizza in official PizzaStore box.");
    }
    
    public String getName(){ return name; }
}
```

```java
public class NYStyleCheesePizza extends Pizza{
    public NYStyleCheesePizza(){
        name = "NY Style Sauce and Cheese Pizza";
        dough = "Thin Crust Dough";
        sauce = "Marinara Sauce";
        
        toppings.add("Grated Reggiano Cheese");
    }
}
```

```java
public class PizzaTestDrive{
    public static void main(String[] args){
        PizzaStore nyStore = new NYPizzaStore();
        Pizza pizza = nyStore.orderPizza("cheese");
        System.out.println("Ordered pizza: " + pizza.getName());
    }
}
```

Tüm fabrika tipi desenler nesne yaratımını kapsüller. **"Factory Method Pattern"** ise alt sınıfların hangi objelerin yaratılacağına karar vermesiyle, nesne yaratımını kapsüller.

![](.gitbook/assets/image%20%2818%29.png)

**Factory Method Pattern:** Nesne yaratmak için bir interface tanımlar, ancak hangi sınıftan yaratılacağını alt sınıflar karar verir. Factory Method, bir sınıfın nesne yaratımını alt sınıflara bırakmasına izin verir.

![](.gitbook/assets/image%20%2825%29.png)

**Soru:** Factory metodu ve Creator sınıfı her zaman abstract olmak zorunda mı?

**C:** Hayır, isterseniz default factory metodu tanımlayarak Creator sınıfının alt sınıfı olmasa bile somut Product nesneleri üretebilirsiniz.

**Soru:** Simple Factory ile Factory Metod arasındaki fark nedir? Pizza nesnesini alt sınıfta döndüren Factory metodu hariç benzer gözüküyorlar.

**C:** Benzer gözüküyorlar fakat, Simple Factory'i tek seferlik bir işlem gibi düşünebilirsin. Çünkü aksine Factory Method ile hangi implementasyonun kullanacağına alt sınıfların karar verdiği bir framework yaratıyorsun. Simple Factory sadece nesne yaratımını kapsüllüyor ve yaratılan ürünleri çeşitlendirmenin bir yolu yok.

#### Bağımlı Bir Pizza Dükkanı

```java
public class DependentPizzaStore{
    public Pizza createPizza(String style, String type){
        Pizza pizza = null;
        if(style.equals("NY")){
            if(type.equals("cheese")){
                pizza = new NYStyleCheesePizza();
            } else if(type.equals("veggie")) {
                pizza = new NYStyleVeggiePizza();
            } else if(type.equals("pepperoni")){
                pizza = new NYStylePepperoniPizza();
            }    
        } else if(style.equals("Chicago")){
            if(type.equals("cheese")){
                pizza = new ChicagoStyleCheesePizza();
            } else if(type.equals("veggie")) {
                pizza = new ChicagoStyleVeggiePizza();
            } else if(type.equals("pepperoni")){
                pizza = new ChicagoStylePepperoniPizza();
            }    
        } else{
            System.out.println("Error: Invalid Pizza Type.");
            return null;
        }
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

Pizza dükkanının bu versiyonu tüm pizza nesnelerine bağımlı, pizzaların üretimini başka bir yere vekalet vermek yerine. Her yeni pizza türü burada pizza dükkanımıza bağımlılık yaratıyor.

### The Dependency Inversion Principle

{% hint style="info" %}
**Tasarım Prensibi:** Somut sınıflara değil, soyutlamalara bağımlı olun.
{% endhint %}

Bu prensip, üst tipe kodlamamız gerektiğini söyleyen önceki prensibe benziyor. Ancak bu prensip soyutlamaya daha fazla vurgu yapıyor: Yüksek-seviye bileşenler, düşük-seviye bileşenlere bağımlı değil; ikisi de soyutlamalara bağımlı olmalıdır. \(Yüksek-seviye bileşen: davranışı diğer düşük-seviye bileşenlerle tanımlanan bileşenlerdir.\) Burada yüksek-seviye bileşenimiz "PizzaStore", pizza implementasyonları düşük-seviye ve şu an açıkça PizzaStore somut pizza sınıflarına bağımlı durumda. Bu prensip bu noktada diyor ki yüksek ve düşük tüm bileşenler soyutlamalara bağımlı olmalı. 

Öğrendiğimiz Factory Method desenini uyguladıktan sonra farkedeceksiniz ki; üst-seviye bileşen PizzaStore ve alt-seviye bileşen pizzalar; ikisi de Pizza soyutlamasına bağımlı. Factory Method, Dependency Inversion prensibini uygulamanın tek yolu değil, ancak en güçlülerinden.

Prensibi uygulamak için ayrıca şu kuralları izleyebiliriz:

* Hiçbir değişken somut bir sınıfa referans tutmamalı. \(Eğer **new** kullanırsan, somut sınıfa referans tutuyorsun demektir, bu  yüzden factory kullan\)
* Hiçbir sınıf, somut bir sınıftan türetilmemeli. \(Eğer somut bir sınıftan miras alırsan somut sınıfa bağımlı olursun, bu yüzden interface ya da abstract class kullan\)
* Hiçbir metod, üst sınıflarında doldurulmuş bir metodu override etmemelidir.

Bu kuralların hepsini tamamen uygulamak imkansızdır, ancak kod yazarken bu kurallara dikkat etmek gerekir.

Pizza dükkanımıza dönersek; her şube kendi standartlarını uyguladığı için kalite düşüyor. Bu nedenle tüm malzemeleri biz yollamaya karar verdik. Ancak farkettik ki her şube kendi yerel pizzalarında farklı malzemeler kullanabiliyor. Her malzeme ailesini ve malzemeyi yaratacak bir fabrika inşa etmek istiyoruz:

```java
public interface PizzaIngredientFactory{
    public Dough createDough();
    public Sauce createSauce();
    public Cheese createCheese();
    public Veggies[] createVeggies();
    public Pepperoni createPepperoni();
    public Clams createClam();
}
```

```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory{
    public Dough createDough(){
        return new ThinCrustDough();
    }
    
    public Sauce createSauce(){
        return new MarinaraSauce();
    }
    
    public Cheese createCheese(){
        return new ReggianoCheese();
    }
    
    public Veggies[] createVeggies(){
        Veggies veggies[] = {new Garlic(), new Onion(), new Mushroom(),
            new RedPepper()};
        return veggies;
    }
    
    public Pepperoni createPepperoni(){
        return new SlicedPepperoni();
    }
    
    public Clams createClam(){
        return new FreshClams();
    }
}
```

Pizza sınıfını düzenleyelim:

```java
public abstract class Pizza{
    String name;
    Dough dough;
    Sauce sauce;
    Veggies veggies[];
    Cheese cheese;
    Pepperoni pepperoni;
    Clams clam;
    
    // prepare metodunu abstract yaptık.
    abstract void prepare();
    
    void bake(){
        System.out.println("Bake for 25 mins at 350");
    }
    
    void cut(){
        System.out.println("Cutting the pizza into diagonal slices.");
    }
    
    void box(){
        System.out.println("Place the pizza into official box");
    }
    
    void setName(String name){
        this.name = name;
    }
    
    String getName(){ return name; }
    
}
```

```java
public class CheesePizza extends Pizza{
    PizzaIngredientFactory ingredientFactory;
    
    public CheesePizza(PizzaIngredientFactory ingredientFactory){
        this.ingredientFactory = ingredientFactory;
    }
    
    void prepare(){
        System.out.println("Preparing " + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
    }
}
```

Malzemeler, hangi malzeme fabrikasını seçtiğimize göre değişiyor. Pizza sınıfı umrunda değil, sadece pizzayı hazırlıyor. Bölgesel farklılıklar bu şekilde ayrıştırıldığı için, yeni bölgelere şube açıldığında yeniden kullanılabilir koda sahibiz.

```java
public class NYPizzaStore extends PizzaStore{
    protected Pizza createPizza(String item){
        Pizza pizza = null;
        PizzaIngredientFactory ingredientFactory =
            new NYPizzaIngredientFactory();
        
        if(item.equals("cheese")){
            pizza = new CheesePizza(ingredientFactory);
            pizza.setName("New York Style Cheese Pizza");
        } else if(item.equals("veggie")){
            pizza = new VeggiePizza(ingredientFactory);
            pizza.setName("New York Style Veggie Pizza");
        } else if(item.equals("clam")){
            pizza = new ClamPizza(ingredientFactory);
            pizza.setName("New York Style Clam Pizza");
        } else if(item.equals("pepperoni")){
            pizza = new PepperoniPizza(ingredientFactory);
            pizza.setName("New York Style Pepperoni Pizza");
        }
        return pizza;
    }
}
```

### Ne Yaptık?

**"Abstract Factory",** bir ürünler ailesi yaratmak için bir arayüz sunar. Bu interface'i kullanarak; kodumuzu, gerçekten ürünü üreten fabrikadan ayrıştırıyoruz. Bu, bize farklı içeriklerde, farklı çeşitliliklerde fabrika oluşturmamıza yardımcı oluyor. \(farklı bölgeler, farklı işletim sistemleri, farklı görünüm ve davranışlar\) Kodumuz ayrıştığı için, farklı davranışları elde etmek için farklı fabrikaları kullanabiliyoruz.

Görüyoruz ki sipariş sürecimiz değişmedi:

```java
public class PizzaTestDrive{
    public static void main(String[] args){
        PizzaStore nyStore = new NYPizzaStore();
        Pizza pizza = nyStore.orderPizza("cheese");
        System.out.println("Ordered pizza: " + pizza.getName());
    }
}
```

**Abstract Factory,** somut sınıflarını belirtmeden, bağımlı ya da ilişkili nesneler ailesi yaratmamızı sağlayan bir interface sunar.

![](.gitbook/assets/image%20%2819%29.png)

![](.gitbook/assets/image%20%2841%29.png)

**Soru:** Farkettim ki, Abstract Factory'deki her metod **Factory Method** gibi gözüküyor \(createDough\(\), createSauce\(\) vb.\) Her metod soyut\(abstract\) tanımlanmış ve alt sınıflar nesne yaratmak için onu override ediyor. Bu Factory Method değil mi?

**Cevap:** Evet! Genellikle Abstract Factory metodları factory method olarak tasarlanır. Mantıklı değil mi? Abstract Factory'nin işi; ürünler grubu için bir interface tanımlamak. İnterface'deki her metod, somut bir ürün yaratmakla sorumlu ve biz AbstractFactory'den miras alarak bu metodları dolduruyoruz. Yani, factory metodlar, abstract factory'deki ürün metodlarını doldurmak için doğal bir yöntem.

### Factory Method Vs Abstract Factory

* İkisinin de işi nesne yaratmaktır. Factory Metod bunu kalıtım yoluyla yaparken, Abstract Factory bunu composition yoluyla yapar. Yani Factory Method kullanarak nesne yaratmak için bir sınıfı "extend" etmek ve bir fabrika metodunu "override" etmek gerekiyor. Daha sonra nesne yaratma işini alt sınıflara bırakır. Bu şekilde; istemciler sadece kullandıkları soyut tipten haberdardırlar, somut tipten alt sınıflar sorumludur. Diğer bir deyişle; istemcileri somut tiplerden ayrıştırır.
* Abstract Factory ise bunu farklı bir yolla yapar. Bir ürün ailesi yaratmak için soyut bir tip sunar. Bu tipin alt sınıfları ürünlerin nasıl üretileceğini tanımlar. Fabrikayı kullanmak için; bir tane yaratır ve onu soyut bir tipte gösterirsiniz. Yani aynı Factory Method'da olduğu gibi, istemciler kullandıkları somut ürünlerden ayrıştırılmıştır. Diğer bir avantajı ise, ilişkili ürünleri gruplar. Bu ilişkili ürün ailesini genişletmek isterseniz, interface'te değişiklik yapmanız gerekir. Ve genellikle, factory method kullanarak somut nesneleri yaratır. Alt sınıflar soyut fabrikadan aldığı factory method'u doldururlar. 
* Sonuç olarak ikisi de; nesne yaratımını kapsülleyerek gevşek bağlı ve implementasyonlara daha az bağımlı uygulamalar yaratmanızı sağlar. Bir ürün ailesi yaratmanız gerektiğinde ve istemcilerin birbiriyle ilişkili ürünler yaratması gerektiğinde Abstract Factory kullanılabilir. İstemci kodu, yaratmanız gereken somut sınıflardan ayrıştırmak istiyorsanız veya ihtiyacınız olacak tüm somut sınıfları önceden bilmiyorsanız Factory Method kullanılabilir. 

![](.gitbook/assets/image%20%2828%29.png)

![](.gitbook/assets/image%20%2817%29.png)

