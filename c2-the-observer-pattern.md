# C2 - The Observer Pattern

Şirket bizden, bize gönderilecek "WeatherData" nesnesi üzerine inşa edilecek bir uygulama istedi. Bu uygulama, genişletilebilir bir uygulama olacak, güncel  hava durumunu takip edecek \(sıcaklık, nem, basınç\). Bunun yanında 3 farklı gösterim \(display\) ekranı bulunacak: 

* Güncel Hava Durumu \(Sıcaklık, Nem, Basınç\)
* Hava İstatistikleri
* Basit Hava Tahmini

![](.gitbook/assets/image%20%2824%29.png)

Şirket tarafından bize "WeatherData" gönderildi:

![](.gitbook/assets/image%20%2814%29.png)

Bildiklerimiz:

* Sıcaklık, nem ve basınç değerleri için _get_ metodları bulunuyor.
* `measurementsChanged()` metodu yeni bir hava değeri olduğunda çağrılıyor. \(Nasıl çağrıldığı bizi ilgilendirmiyor, sadece biliyoruz\)
* Hava verilerini kullanarak 3 adet ekran göstermemiz gerekiyor ve bu ekranların her yeni veri ulaştığında güncellenmesi gerekiyor.
* Uygulamanın genişletilebilir olması gerekiyor. Diğer yazılımcılar yeni bir ekran yaratabilir ve kullanıcılar istedikleri kadar yeni ekran ekleyebilir veya çıkartabilir. Şimdilik sadece bu 3 ekranın varlığını biliyoruz.

#### İlk Tasarım Denemesi

```java
public class WeatherData{
    // sınıf değişkenkeri...
    public void measurementsChanged(){
        float temp = getTemparature();
        float humidity = getHumidity();
        float pressure = getPressure();
        
        currentConditionsDisplay.update(temp, humidity, pressure);
        statisticsDisplay.update(temp, humidity, pressure);
        forecastDisplay.update(temp, humidity, pressure);
    }
    
    // diğer WeatherData metodları...
}
```

* Burada görüntüleri güncellediğimiz alan\(8,9,10. satırlar\) değişen bir alan olduğu için kapsüllememiz gerekiyor.
* Bir süper tip yerine, somut referanslara kodladığımız için programı değiştirmeden yeni görüntüler eklememiz mümkün değil.
* Görüntü elementleri ile ortak parametrelerle mesajlaşıyoruz: "temp, humidity, pressure". Hepsinin bu parametreleri alan `update()` metodu bulunuyor.

Buradaki problemi çözmeden önce **Observer Pattern'i** inceleyelim.

### Observer Pattern

* JDK'da en çok kullanılan desenlerden biridir.

**Observer Pattern:** Nesneler arası "One-to-Many" bir bağımlılık ilişkisi tanımlar. Burada state'i \(durum\) elinde bulunduran \(One\) "subject" nesnesi ona bağımlı olan \(Many\) "observer" nesneleri, "state" değiştikçe bilgilendirir.

Gazete yayıncısı ile aboneleri arasındaki ilişki örnek verilebilir. Yeni bir gazete çıktığında abonelere dağıtılır haber verilir. İsteyen abone olabilir ya da abonelikten çıkabilir. Abone olduğu sürece gazete almaya devam edecektir. Gazete yayıncısı burada **subject,** gazete aboneleri ise **observer'**dır.

Observer desenini uygulamanın birden çok yolu vardır ancak çoğu "Subject" ve "Observer" interfacelerini içeren bir yapıdadır:

![](.gitbook/assets/image%20%2827%29.png)

İki nesne gevşek bağlı \(**loose coupling**\) olduğu zaman iletişime geçebilirler fakat birbirleri hakkında az bilgiye sahiptirler. Burada Subject ve Observer  gevşek bağlıdır.

Burada Subject'in Observer hakkında bildiği tek şey, Observer interface'ini implement ettiğidir. \(somut sınıfı hakkında bilgisi yoktur\) Yeni Observer'lar herhangi bir zaman eklenebilir. Yeni Observer eklemek için Subject'i modifiye etmeye gerek yoktur. Subject'leri veya Observerleri birbirinden bağımsız olarak tekrar kullanabiliriz. \(**loose coupling**\) Subject'te veya herhangi bir Observer'da gerçekleşecek bir değişiklik diğerini etkilemez.

{% hint style="info" %}
**Tasarım Prensibi:** Etkileşen nesneler arasında gevşek bağlı tasarımlar için çaba gösterin.
{% endhint %}

Bu deseni öğrendiğimize göre uygulamamıza ekleyelim:

![](.gitbook/assets/image%20%287%29.png)

Java'nın bu desen için dahili desteği bulunmaktadır. Bazı durumlarda onu kullanabilirsiniz, ancak kendinizinkini yazmak esneklik sağlayacaktır, hem de yazması o kadar da zor değil:

```java
public interface Subject{
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}
```

```java
public interface Observer{
    public void update(float temp, float humidity, float pressure);
}
```

```java
public interface DisplayElement{
    public void display();
}
```

```java
public class WeatherData implements Subject{
    private ArrayList observers;
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherData(){
        observers = new ArrayList();
    }
    
    public void registerObserver(Observer o){
        observers.add(o);
    }
    
    public void removeObserver(Observer o){
        if(observers.indexOf(o) >= 0){
            observers.remove(o);
        }
    }
    
    public void notifyObservers(){
        for(int i = 0; i < observers.size(); i++){
            Observer observer = (Observer) observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public measurementsChanged(){
        notifyObservers();
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure){
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
    
    // Diğer metodlar...
}
```

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement{
    private float temperature;
    private float humidity;
    private Subject weatherData;
    
    public CurrentConditionsDisplay(Subject weatherData){
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
    
    public void update(float temperature, float humidity, float pressure){
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }
    
    public void display(){
        System.out.println("Current conditions: " + temperature 
        + "F degrees and " + humidity + "% humidity");
    }
    
}
```

```java
public class WeatherStation{
    public static void main(String[] args){
        WeatherData weatherData = new WeatherData();
        
        CurrentConditionsDisplay currentDisplay = 
            new CurrentConditionsDisplay(weatherData);
            
        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(85, 55, 32.4f);
        weatherData.setMeasurements(90, 65, 25.4f);
    
    }
}
```

Yukarıdaki örnekte "push" yani Subject, observer'lara bir güncelleme oldukça bildirim yapma şeklinde kullanıldı. Bunun yerine "pull" durumu da yani, isteyen observer istediği state'i istediği zaman sorabilir. Hem "pull" hem "push" u destekleyen Java'nın kendi Observer desenini de kullanabiliriz.

#### Java'nın Dahili Observer Deseni

![](.gitbook/assets/image%20%2840%29.png)

Gördüğünüz üzere temel fark; Subject sınıfımız \(WeatherData\) Observable sınıfından extend ediliyor ve metodlarını miras alıyor.

Bir Observable'ın bildirim gönderebilmesi için`;`

* `setChanged()` metodunu çağırarak, nesnenizde state'in değiştiğini belirtmeniz gerekiyor.
* Daha sonra iki versiyonu bulunan _notifyObservers_ metodundan birini çağırmanız gerekiyor.
  * `notifyObservers()` 
  * `notifyObservers(Object arg)`

Bir Observer'in bildirim alabilmesi için;

`update()` metodunu doldurur ancak metodun imzası farklıdır: `update(Observable o, Object arg)`

Bildirimi gönderen "Subject" ilk argüman olarak, notifyObservers'a iletilecek veri nesnesi ikinci argüman olarak kullanılır.

Eğer veriyi "push" lamak istiyorsanız, veri nesnesini notifyObservers\(arg\) metoduyla iletin. Yoksa, Observer Observable nesnesinden veriyi "pull"lamak zorunda kalacaktır.

Neden `setChanged()` metoduna ihtiyaç duyduk? Daha önce yoktu.

`setChanged()` metodu, state'in değiştiğini ve `notifyObservers()`metodu çağrıldığında observerlar'ını güncellemesi gerektiğini belirtir. Önce bu metod çağrılmadan notifyObservers çağrılırsa, observerlar bildirim almaz! Arka planda şu şekilde çalışır:

![](.gitbook/assets/image%20%284%29.png)

Peki bu neden gerekli? setChanged\(\) metodu; size bildirimleri optimize etme şansı vererek, observerları güncellemede esneklik sağlar. Örneğin; diyelim ki bizim hava uygulamamızda ölçümler o kadar hassas ki okunan sıcaklık değerleri derecenin onda birinde sürekli dalgalanıyor. Bu durum WeatherData nesnesinin sürekli bildirim göndermesine sebep olur. Burada, sadece yarım dereceden fazla değişiklik olursa setChanged\(\) metodunu çağırarak bildirim gönderilmesini sağlayabiliriz.

Bu işlev çok sık kullanılmasa da, bazı durumlarda faydalı olabilir. Bu metodun yanında _changed_ durumunu _false_ yapan clearChanged\(\) metodu ve _changed_ durumunu belirten _hasChanged\(\)_ metodu bulunmaktadır.

#### Java'nın Dahili Observer Deseni Uygulamamızda Kullanmak

```java
import java.util.Observable;
import java.util.Observer;

public class WeatherData extends Observable{
    // Bu defa Observable sınıfından miras aldık.
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherData(){}
    
    /* Observer'ların durumunu takip etmek ya da kayıtlarını yapmak gibi işlemleri 
    bu sefer yapmıyoruz. Üst sınıf ilgileniyor. Bu nedenle register, add, notify
    kod kısımları kaldırıldı. */
    
    public void measurementsChanged(){
        setChanged();
        notifyObservers();
        // Boş parametreli olanı kullandık. "Pull" yapıyoruz.
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure){
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
    }
    
    public float getTemperature(){ return temperature; }
    public float getHumidity(){ return humidity; }
    public float getPressure(){ return pressure; }
    
}
```

```java
import java.util.Observable;
import java.util.Observer;

public class CurrentConditionsDisplay implements Observer, DisplayElement{
    // Bu defa java.util.Observer'ı implement ediyoruz.
    Observable observable;
    private float temperature;
    private float humidity;
    
    public CurrentConditionsDisplay(Observable observable){
        this.observable = observable;
        observable.addObserver(this);
    }
    
    public void update(Observable obs, Object arg){
        if(obs instanceof WeatherData){
            WeatherData weatherData = (WeatherData) obs;
            this.temperature = weatherData.getTemperature();
            this.humidity = weeatherData.getHumidity();
            display();
        }
    }
    
    public void display(){
        System.out.println("Current conditions: " + temperature 
        + "F degrees and " + humidity + "% humidity");
    }
}
```

#### Sonuç

Java'nın kendi Observable deseni, daha önce belirttiğimiz implementasyona değil bir interface'e programlamamız gerektiği prensibine aykırı gözüküyor. İlk problem _Observable_ bir sınıf ve miras almamız gerekiyor. Bu, hali hazırda başka bir sınıftan miras alan \(extend\) bir sınıfa Observable davranışını ekleme imkanımız olmadığı anlamına geliyor. Bu da, tekrar kullanılabilirlik potansiyelini sınırlandırıyor. Zaten bu nedenle tasarım desenlerini kullanıyoruz. İkinci olarak, bir Observable interface'i bulunmadığı için Java'nın Observable deseni ile uyumlu çalışacak kendi implementasyonlarımızı yapamıyoruz.

Eğer setChanged\(\) metoduna bakarsanız protected olduğunu görürsünüz. Bu demektir ki, bu metodu çağırmak için miras almanız gerekiyor. Observable sınıfının bir örneğini oluşturmak ve kendi nesnelerimizle birleştirmek \(compose\) için miras almanız gerekiyor. Bu da başka bir tasarım prensibimizi ihlal ediyor: "Kompozisyonu, kalıtıma tercih edin."

**Observable** sınıfı eğer miras alabiliyorsanız ihtiyaçlarınızı karşılayabilir. Aksi takdirde, bölümün başında yaptığımız gibi kendi implementasyonumuzu yapmamız gerekiyor.

