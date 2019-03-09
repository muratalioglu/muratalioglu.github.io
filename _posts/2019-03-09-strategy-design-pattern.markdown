---
layout: post
title:  "Strategy Design Pattern"
date:   2019-03-09
categories: design-patterns
---
Strategy Pattern, çalışma esnasında, iş mantığına uygun algoritmanın seçilebilmesine olanak sağlayan davranışsal (behavioral) bir tasarım desenidir/kalıbıdır.

Doğrudan, kullanılacak algoritmayı içeren sınıfları değil; bu sınıfların implement ettiği arayüzü (interface) kullanır. Bu sayede çalışma zamanında, kullanılan senaryoya göre strateji değiştirilebilmesi mümkün olur.

Aşağıdaki görselden anlaşılacağı üzere, Context (ana program) doğrudan Strategy1 ya da Strategy2 sınıfını kullanmak yerine bunların implement ettiği Strategy arayüzünü kullanıyor. `algorithm()` metodunu bu arayüz aracılığıyla çağırıyor.

![Strategy Pattern diagram](/assets/strategy_pattern.png)

Bir örnekle anlatalım. Oldukça basit, ülkelerarası bir kargo hizmeti düşünelim. Müşterinin ihtiyacına göre standart ya da yüksek hızlı kargo iletimi yapıyor. Söz konusu iki hizmetin de implement edeceği bir interface oluşturalım.

ShipmentStrategy.java
{% highlight java %}
public interface ShipmentStrategy {
    void deliver();
}
{% endhighlight %}


Şimdi, ulaştırma hizmetini standart hızda gerçekleştirecek algoritmayı sunan sınıfı yazalım. (Örnek amacıyla yazıldıkları için, bu iki algoritma sınıfının birer satır mesaj yazdırması yeterli.)

{% highlight java %}
public class StandardShipment implements ShipmentStrategy {
    @Override
    public void deliver() {
        System.out.println("The order will be delivered in 28 days.");
    }
}
{% endhighlight %}

Ve yüksek hızda ulaştırma yapacak olan sınıfımızı yazalım.

{% highlight java %}
public class ExpressShipment implements ShipmentStrategy {
    @Override
    public void deliver() {
        System.out.println("The order will be delivered in 7 days.");
    }
}
{% endhighlight %}

Şimdi ise StandardShipment ve ExpressShipment sınıflarının implement ettiği ShipmentStrategy arayüzünü kullanan Shipment sınıfını yazalım.

{% highlight java %}
public class Shipment {
    private ShipmentStrategy shipmentStrategy;
	
    public Shipment() {
    }
	
    public Shipment(ShipmentStrategy shipmentStrategy) {
        this.shipmentStrategy = shipmentStrategy;
    }
	
    public void setStrategy(ShipmentStrategy shipmentStrategy) {
        this.shipmentStrategy = shipmentStrategy;
    }
	
    public void run() {
        shipmentStrategy.deliver();
    }
}
{% endhighlight %}

Shipment sınıfının sahip olduğu tek field shipmentStrategy, bildiğimiz gibi bir interface. Yani onu implement eden iki sınıfa ait nesneleri buna bağlayabiliriz.

shipmentStrategy ister yeni bir Shipment nesnesi oluşturulurken constructor yoluyla, ister Shipment nesnesi oluşturulduktan sonra setter yoluyla atanabilir. Setter metodu olan `setStrategy()` ile, çalışma esnasındaki senaryoya göre strateji değiştirebiliriz.

Örneği kullanmak için, ürünleri depo eden bir Warehouse sınıfı yazalım.

{% highlight java %}
import java.util.Random;

public class Warehouse {
    public static void main(String[] args) {
    Shipment shipment = new Shipment();
		
    boolean isUrgent = new Random().nextBoolean();
		
    if (isUrgent) {
        shipment.setStrategy(new ExpressShipment());
    } else {
        shipment.setStrategy(new StandardShipment());
    }		
    shipment.run();
    }
}
{% endhighlight %}

Bu kodda karmaşık bir senaryo yok. Kargonun acele olup olmaması bir Random() sonucuna bırakılıyor. Acele ise ExpressShipment, değilse StandardShipment stratejisi seçiliyor.