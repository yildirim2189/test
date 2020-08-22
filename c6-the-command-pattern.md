# C6 - The Command Pattern

Bizden bir akıllı ev kumandası uygulaması istendi. Kumandada, her cihaz için bir açma, kapatma tuşu ve global bir geri al\(undo\) tuşu olması istendi.

![](.gitbook/assets/image%20%2845%29.png)

Ve şirketten bize verilen cihaz sınıfları şu şekilde:

![](.gitbook/assets/image%20%283%29.png)

Görüyoruz ki, ortak bir interface bulunmuyor. Bazı sınıfların on\(\), off\(\) metodları varken bazı sınıflarda setTemperature\(\) vb. gibi tamamen farklı metodlar bulunuyor. Bunun yanında bize gelecekte farklı metodlarda farklı sınıflar geleceği söylendi. İlk düşüncemiz, kumandanın bu sınıflar hakkında fazla bilgiye sahip olmadan, buton tuşlarıyla isteklerde bulunması. Örneğin bir cihazı nasıl kapatacağının bilgisi kumandada bulunmamalı. Ve, eğer TV ise şunu yap, eğer küvet ise bunu yap şeklinde if ifadelerinin kumandada bulunmasının kötü tasarım olduğunu biliyoruz.

**Command Pattern**, faydalı olabileceğini duyduk. Çünkü, Command deseni, bir eylemi gerçekten gerçekleştiren nesne ile o eylemin istemcisini ayrıştırır. Bunu uygulamak için ise, tasarımımıza **Command** nesneleri ekleyeceğiz. Bir command nesnesi belirli bir nesne üzerindeki belirli bir eylemi kapsülleyecek. Eğer her butonda bu command nesnelerini saklarsak, butona bastığımızda command nesnesine bir şey yapmasını emredeceğiz. Kumanda işin ne olduğunu bilmeyecek, command nesnesi doğru cihazla konuşarak işi halledecek. Böylece kumanda cihazdan ayrıştırılmış olacak.

Bu desenin mantığını anlamak için bir restoranın işleyişini düşünelim. 

* Müşteri\(**Client**\) garsona\(**Invoker**\) sipariş\(**Command**\) verir. 
* Garson siparişi fişe ekler.\(`setCommand()`\) 
* Garson, siparişi aşçıya\(**Receiver**\) bildirir.\(`execute()`\)

 Aşçı siparişe bakarak siparişi hazırlar. Burada sipariş fişini bir nesne olarak düşünelim. Yemeğin hazırlanma isteğini içermekte. Garsonun sipariş hakkında bilgi sahibi olmasına ya da siparişi kimin  hazırladığına dair bilgisi olmasına gerek yoktur. O sadece siparişi aşçıya iletir. Aynı şekilde aşçı garsonla iletişime geçmek zorunda değildir. O sipariş fişine bakarak siparişi hazırlar:

![](.gitbook/assets/image%20%2822%29.png)

Client, bir Command nesnesi yaratır. Client, `setCommand()` metodunu kullanarak Invoker'da komutu saklar. Daha sonra Client Invoker'a komutu çalıştırması gerektiğini söyler. Invoker'a yüklenen komut, kullanılabilir ya da kullanılmayabilir.

**Command Interface**

Öncelike tüm **Command** nesneleri bir interface'i implement etmeli. 

```java
public interface Command{
    public void execute();
}
```

Şimdi bir komutu implement edelim:

```java
public class LightOnCommand implements Command{
    // Cihaz sınıfına referans
    Light light;
    
    public LightOnCommand(Light light){
        this.light = light;
    }

    public void execute(){
        light.on();
    }
}
```

```java
public class SimpleRemoteControl{
    Command slot;
    
    public SimpleRemoteControl(){}
    
    public void setCommand(Command command){
        slot = command;
    }
    
    public void buttonWasPressed(){
        /* Butona basıldığında bu metod çalışıyor. Kumandaya gönderilen
        command nesnesinin execute metodunu çalıştırıyor. Ne yaptığı
        hakkında bilgisi yok. */
        slot.execute();
    }
}
```

```java
public class RemoteControlTest{
    public static void main(String[] args){
        SimpleRemoteControl remote = new SimpleRemoteControl();
        Light light = new Light();
        LightOnCommand lightOn = new LightOnCommand(light);
        
        remote.setCommand(lightOn);
        remote.buttonWasPressed();
    }
}
```

**Command Pattern,** bir isteği nesne olarak kapsüller, böylece diğer nesneleri farklı isteklerle parametrelendirmenize, istekleri sıraya veya kayıt altına almanızı ve geri alınabilen işlemleri desteklemenize imkan sağlar.

![](.gitbook/assets/image%20%2835%29.png)

Daha önce yaptığımız gibi, komutları slotlara atamanın bir yolunu bulmalıyız:

```java
public class RemoteControl{
    Commands[] onCommands;
    Commands[] offCommands;
    
    public RemoteControl(){
        onCommands = new Command[7];
        offCommands = new Command[7];
        
        Command noCommand = new NoCommand();
        for(int i = 0; i < 7; i++){
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
    }
    
    public void setCommand(int slot, Command onCommand, Command offCommand){
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    
    public void onButtonWasPushed(int slot){
        onCommands[slot].execute();
    }
    
    public void offButtonWasPushed(int slot){
        offCommands[slot].execute();
    }
    
    public String toString(){
        StringBuffer stringBuff = new StringBuffer();
        stringBuff.append("\n----- Remote Control ------\n");
        for(int i = 0; i < onCommands.lenght; i++){
            stringBuff.append("[slot " + i + "] "
            + onCommands[i].getClass().getName() + "    " 
            + offCommands[i].getClass().getName() + "\n");
        }
        return stringBuff.toString();
    }
}
```

```java
public class LightOffCommand implements Command{
    Light light;
    
    public LightOffCommand(Light light){
        this.light = light;
    }
    
    public void execute(){
        light.off();
    }
}
```

```java
public class StereoOnWithCdCommand implements Command{
    Stereo stereo;
    
    public StereoOnWithCdCommand(Stereo stereo){
        this.stereo = stereo;
    }
    
    public void execute(){
        stereo.on();
        stereo.setCD();
        stereo.setVolume(11);
    }
}
```

Tekrar kumandamızı test edelim:

```java
public class RemoteLoader{
    public static void main(String[] args){
        RemoteControl remoteControl = new RemoteControl();
        
        // Cihazları yarattık.
        Light livingRoomLight = new Light("Living Room");
        Light kitchenLight = new Light("Kitchen");
        CeilingFan ceilingFan = new CeilingFan("Living Room");
        GarageDoor garageDoor = new GarageDoor("");
        Stereo stereo = new Stereo("Living Room");
        
        // Light komutları
        LightOnCommand livingRoomLightOn = 
            new LightOnCommand(livingRoomLight);
        LightOffCommand livingRoomLightOff =
            new LightOffCommand(livingRoomLight);
        LightOnCommand kitchenLightOn =
            new LightOnCommand(kitchenLight);
        LightOffCommand kitchenLightOff =
            new LightOffCommand(kitchenLight);
            
        // CeilingFan
        CeilingFanOnCommand ceilingFanOn =
            new CeilingFanOnCommand(ceilingFan);
        CeilingFanOffCommand ceilingFanOff =
            new CeilingFanOffCommand(ceilingFan);
            
        // GarageDoor
        GarageDoorUpCommand garageDoorUp =
            new GarageDoorUpCommand(garageDoor);
        GarageDoorDownCommand garageDoorDown =
            new GarageDoorDownCommand(garageDoor);
            
        // Stereo
        StereoOnWithCDCommand stereoOnWithCD =
            new StereoOnWithCdCommand(stereo);
        StereoOffWithCDCommand stereoOffWithCD =
            new StereoOffWithCDCommand(stereo);
        
        
        // set Commands
        remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
        remoteControl.setCommand(1, kitchenLightOn, kitchenLightOff);
        remoteControl.setCommand(2, ceilingFanOn, ceilingFanOff);
        remoteControl.setCommand(3, stereoOnWithCD, stereoOffWithCD);
        
        System.out.println(remoteControl);
        
        remoteControl.onButtonWasPushed(0);
        remoteControl.offButtonWasPushed(0);
        remoteControl.onButtonWasPushed(1);
        remoteControl.offButtonWasPushed(1);
        remoteControl.onButtonWasPushed(2);
        remoteControl.offButtonWasPushed(2);
        remoteControl.onButtonWasPushed(3);
        remoteControl.offButtonWasPushed(3);
        
    }
}
```

Şu an ilk 4 slota komut atadık. Ancak diğer 3 slotta "NoCommand" nesneleri bulunmakta. NoCommand nesnesi "null" nesne örneğidir. Null nesne, döndürecek anlamlı bir nesne olmadığında ve istemcinin "null" kontrolüyle uğraşmaması istendiğinde kullanışlıdır. Tasarım desenlerinde "null nesne" örneklerini görebilirsiniz.

Kumanda tasarımımızı genel olarak tamamladık. Ancak "undo" / "geri al" butonunu eklemeyi unuttuk:

```java
public interface Command{
    public void execute();
    // undo metodunu ekledik.
    public void undo();
}
```

```java
public class LightOnCommand implements Command{
    // Cihaz sınıfına referans
    Light light;
    
    public LightOnCommand(Light light){
        this.light = light;
    }

    public void execute(){
        light.on();
    }
    
    public void undo(){
        /* execute ışığı açarken, undo kapatıyor. Aynı şekilde diğer 
        sınıflara da metodu ekleyelim. */
        light.off();
    }
}
```

```java
public class RemoteControlWithUndo{
    Commands[] onCommands;
    Commands[] offCommands;
    Command undoCommmand;
    
    public RemoteControl(){
        onCommands = new Command[7];
        offCommands = new Command[7];
        
        Command noCommand = new NoCommand();
        for(int i = 0; i < 7; i++){
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }
    
    public void setCommand(int slot, Command onCommand, Command offCommand){
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    
    public void onButtonWasPushed(int slot){
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }
    
    public void offButtonWasPushed(int slot){
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }
    
    public void undoButtonWasPushed(){
        /* Her butona basıldığında en son basılan tuşu sakladığımız için
        sakladığımız command referansını kullanarak geri al metodunu
        çağırıyoruz. */
        undoCommand.undo();
    }
    
    public String toString(){
        // ...
    }
}
```

Evet, undo butonumuzu eklemiş olduk. Ancak biraz kolay oldu. Daha fazla "state" barındıran CeilingFan için de ekleyelim:

```java
public class CeilingFan{
    public static final int HIGH = 3;
    public static final int MEDIUM = 2;
    public static final int LOW = 1;
    public static final int OFF = 0;
    String location;
    int speed;
    
    public CeilingFan(String location){
        this.location = location;
        speed = OFF;
    }
    
    public void high(){
        speed = HIGH;
    }
    
    public void medium(){
        speed = MEDIUM;
    }
    
    public void low(){
        speed = LOW;
    }
    
    public void off(){
        speed = OFF;
    }
    
    public int getSpeed(){ return speed; }
    
}
```

CeilingFan sınıfını inceledik ve önceki hızı hafızada tutmamız gerektiğine karar verdik:

```java
public class CeilingFanHighCommand implements Command{
    CeilingFan ceilingFan;
    int prevSpeed;
    
    public CeilingFanHighCommand(CeilingFan ceilingFan){
        this.ceilingFan = ceilingFan;
    }
    
    public void execute(){
        prevSpeed = ceilingFan.getSpeed();
        ceilingFan.high();
    }
    
    public void undo(){
        if(prevSpeed == CeilingFan.HIGH){
            ceilingFan.high();
        } else if(prevSpeed == CeilingFan.MEDIUM){
            ceilingFan.medium();
        } else if(prevSpeed == CeilingFan.LOW){
            ceilingFan.low();
        } else if(prevSpeed == CeilingFan.OFF){
            ceilingFan.off();
        }
    }
}
```

Sonraki adımda, bizden cihazların hepsini tek bir tuşla açan bir makro tuş istendi. Var olan kodumuzu değiştirmeye gerek yok:

```java
public class MacroCommand implements Command{
    Command[] commands;
    
    public MacroCommand(Command[] commands){
        this.commands = commands;
    }
    
    public void execute(){
        for(int i = 0; i < commands.lenght; i++){
            commands[i].execute();
        }
    }
}
```

```java
public class RemoteControlMacroTest{
    public static void main(String[] args){
        Light light = new Light("Living Room");
        TV tv = new TV("Living Room");
        Stereo stereo = new Stereo("Living Room");
        Hottub hottub = new Hottub();
        
        LightOnCommand lightOn = new LightOnCommand(light);
        TVOnCommand tvOn = new TVOnCommand(tv);
        StereoOnCommand stereoOn = new StereoOnCommand(stereo);
        HottubOnCommand hottubOn = new HuttonOnCommand(hottub);

        LightOffCommand lightOff = new LightOffCommand(light);
        TVOffCommand tvOff = new TVOffCommand(tv);
        StereoOffCommand stereoOff = new StereoOffCommand(stereo);
        HottubOffCommand hottubOff = new HuttonOffCommand(hottub);
        
        Command partyOn[] = {lightOn, tvOn, stereoOn, hottubOn};
        Command partyOff[] = {lightOff, tvOff, stereOff, hottubOff};
        
        MacroCommand partyOnMacro = new MacroCommand(partyOn);
        MacroCommand partyOffMacro = new MacroCommand(partyOff);
        
        remoteControl.setCommand(0, partyOnMacro, partyOffMacro);
        
        // Hepsini açalım
        remoteControl.onButtonWasPushed(0);
        // Kapatalım
        remoteControl.offButtonWasPushed(0);
    }
}
```

**Soru:** Her zaman bir "receiver"a ihtiyacım var mı? Neden Command nesnesi execute\(\) metodunun detaylarını içeremiyor?

**Cevap:** Genellikle sadece receiver'daki eylemi harekete geçiren daha "aptal" Command nesneleri oluşturmaya çalışırız. Ancak "akıllı" Command nesnelerinin olduğu birçok örnek de vardır. Bunu yapabilirsiniz tabi, ancak receiver\(aşçı\) ile invoker\(garson\) arasındaki ayrıştırma\(gevşek bağ\) aynı seviyede olmayacaktır, ve Receiver'larla komutlarınızı parametrelendirme imkanınız bulunmayacak.

**Soru:** "Geri al" işlemini birden fazla yapmak istiyorum. Bunun geçmişini nasıl tutabilirim?

**Cevap:** Çok basit. Önceki operasyona referans tutmak yerine, önceki komutları tutan bir "stack" tanımlayabilirsiniz. Böylece her geri al butonuna basıldığında en son basılan tuş stack'ten çıkarılacaktır. \(pop\)

### Command Deseninin Diğer İşlevi: Queuing Requests

Command, bize bir hesaplamayı \(Receiver ve eylemleri\), paketleyip bir nesne olarak iletmemize olanak sağlar. Buradaki hesaplamanın kendisi, command nesnesi oluşturulduktan çok daha sonra yapılabilir. Kumandanın butonuna çok daha sonra basabileceğiz ya da hiç basmayabileceğimiz gibi. Bu hesaplama farklı bir thread tarafından bile gerçekleştirilebilir. Biz bu senaryoyu alıp birçok faydalı uygulamada kullanıyoruz: Zamanlayıcılar, thread havuzları, görev kuyrukları...

Bir görev kuyruğunu düşünelim. Bir tarafta command nesneleri diğer tarafta threadler bekliyor. Thread bir command nesnesini kuyruktan çıkartıp, onun execute\(\) metodunu çağırıyor, bitmesini bekliyor ve nesneyi bırakarak başka  bir nesneyi alıyor. Görev kuyruğu sınıfları, hesaplamayı yapan nesnelerden ayrılmış oluyor. Bir thread, bir anda matematiksel hesaplamayı yapan bir nesneyi işlerken, diğer bir anda network bağlantısı yapan bir nesneyi işleyebiliyor. Nesnenin ne yaptığının onun için bir önemi olmuyor. 

### Command Deseninin Diğer İşlevi: Logging Requests

Bazı uygulamaların mantığı gereği, bazı eylemleri loglayarak \(kayıt altına alarak\), bir çökme yaşandığında bu eylemleri tekrar çalıştırarak kurtarma yapılabilir. Command deseni iki metod ekleyerek bu özelliği destekleyebilir: store\(\) ve load\(\). Java'da bu metodları uygulamak için serileştirmeyi \(serialization\) kullanabiliriz. Komutları çalıştırırken bunların geçmişini diske kaydedebiliriz. Bir çökme yaşandığında, diskten tekrar command nesnelerinin execute metodlarını sırasıyla çağırarak kurtarma yapabiliriz.

Bu mantık kumanda uygulaması için anlamsız olabilir. Ancak büyük veriler üzerinde işlem yapan ve kaydetme işlemini hızlıca gerçekleştiremeyen birçok uygulama için önemli bir yöntemdir. Excel gibi bir uygulama düşünelim. Her değişiklik olduğunda dosyanın kopyasını diske yazmak  yerine son kayıt noktasından sonra yapılan operasyonların kaydını tutmak isteyebiliriz. Daha gelişmiş uygulamarda bu işlemler gruplanabilir. Ya bütün operasyonlar başarıyla gerçekleşir ya da hiçbiri uygulanmaz.





