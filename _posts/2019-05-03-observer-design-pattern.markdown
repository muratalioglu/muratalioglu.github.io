---
layout: post
title:  "Observer Design Pattern"
date:   2019-05-03
categories: design-patterns
---
Bir nesne ile bu nesneyi gözlemleyen nesneler arasındaki ilişkiyi açıklarken Observer Pattern kullanılabilir. Kabaca, Observer pattern yapısında bir gözlenen ve bir veya birden çok gözlemci bulunur. Gözlenen, Observable veya Subject olarak; gözlemciler ise Observer olarak adlandırılabilir.

Bu yapıya verilebilecek en basit örneklerden biri gazete/dergi aboneliğidir. Bir dergi aboneliği durumunda derginin kendisi Subject, abone olacak taraf ise Observer olur. Derginin basım/yayımını gerçekleştiren kurum, abonelerinin listesini tutar. Yeni sayı basıldıktan sonra, bu sayının dağıtımını gerçekleştirerek abonelerini bilgilendirir. Görüldüğü gibi bu yapıda, abonelerin yeni bir sayı basımından habersiz olması mümkün değildir. Aboneler, isteğe bağlı olarak aboneliklerini sonlandırarak kendilerine yeni sayıların gönderilmesini durdurabilirler.

![Observer Pattern diagram](/assets/images/observer_pattern.svg)

Yukarıdaki şemada dergi aboneliğini açıklayacak olursak Subject dergi basım/yayımını gerçekleştiren kurum; ConcreteObserverA ve ConcreteObserverB ise, bu derginin herhangi iki abonesidir. Görüldüğü gibi Subject, abonelerin kaydını observerCollection listesinde tutmaktadır. Bu listeye yeni abone ekleme işlemi registerObserver, abonelik durdurma işlemi unregisterObserver metoduyla gerçekleştirilir ve notifyObservers metoduyla da abonelere yeni sayının bildirilmesi/ulaştırılması sağlanır.

Observer Pattern kullanımına örnek olarak bir radyo istasyonunu da örnek verebiliriz. Bir radyo istasyonumuz ve bu istasyonun yayınını dinleyen abonelerimiz olsun. Radyonun yapacağı yayını RadioData sınıfı altında takip edeceğiz. RadioData sınıfı bir Subject olduğu için, implement edeceği interface aşağıdaki gibi olacak. 

{% highlight java %}
public interface Subject {
    public void addObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}
{% endhighlight %}

Ve RadioData sınıfımız da aşağıdaki gibi olacak.

{% highlight java %}
import java.util.ArrayList;

public class RadioData implements Subject {
    private ArrayList<Observer> clients = new ArrayList<>();
    private String nowPlaying;
	
    @Override
    public void addObserver(Observer client) {
        clients.add(client);
    }

    @Override
    public void removeObserver(Observer client) {
        clients.remove(client);
    }

    @Override
    public void notifyObservers() {
        for (Observer client : clients) {
            client.update(nowPlaying);
        }
    }
	
    public void setNowPlaying(String nowPlaying) {
        this.nowPlaying = nowPlaying;
        nowPlayingChanged();
    }
	
    public void nowPlayingChanged() {
        notifyObservers();
    }
}
{% endhighlight %}

Yukarıda görüldüğü üzere `clients` adlı ArrayList, radyo dinleyicilerinin kaydını tutuyor.
`nowPlaying` adlı String ise, o an çalmakta olan şarkı adını tutacak.
`addObserver` ve `removeObserver` metotlarıyla radyo dinleyicilerinin eklenmesi ve silinmesi işlemi, `notifyObservers` metoduyla da şarkı değiştirildiğinde bunun radyolara bildirilmesi işlemi gerçekleştiriliyor.

Şimdi radyo sınıfımızı görelim. Radyonun implement edeceği interface olan Observer görüldüğü gibi bir adet metot içeriyor, `update`.

{% highlight java %}
public interface Observer {
    public void update(String status);
}
{% endhighlight %}

Radio sınıfı da aşadağı gibi olacak.

{% highlight java %}
public class Radio implements Observer {

    private String nowPlaying;
    private RadioData radioData;	
	
    public Radio(RadioData radioData) {
        this.radioData = radioData;
        radioData.addObserver(this);
    }
	
    @Override
    public void update(String status) {
        this.nowPlaying = status;
        display();
    }
	
    public void display() {
        System.out.println("Now playing: " + nowPlaying);
    }
}
{% endhighlight %}

Son olarak da radyo istasyonunu temsil eden RadioStation sınıfımızı yazalım.

{% highlight java %}
public class RadioStation {
    public static void main(String[] args) {		
        RadioData radioData = new RadioData();
		
        Radio radioA = new Radio(radioData);
        Radio radioB = new Radio(radioData);
		
        radioData.setNowPlaying("Italobrothers - Stamp On The Ground");
		
        radioData.removeObserver(radioA);
		
        radioData.setNowPlaying("ItaloBrothers - Moonlight Shadow");		
    }
}
{% endhighlight %}

RadioStation sınıfında, radyo yayını bilgisini tutacak bir RadioData nesnesi oluşturuluyor. Ve bu radyo yayınını dinleyecek olan iki adet Radio nesnesi oluşturuluyor. Çalınan şarkının ismi `setNowPlaying` metoduyla değiştiriliyor. `RadioData` sınıfında bulunan bu metot, şarkının değiştiğini belirten `nowPlayingChanged` metodunu çağırıyor ve bu metot da notifyObservers ile bu bilgiyi radyolara iletiyor. `notifyObservers` metodu, client listesinde bulunan tüm radyolarda bulunan `update` metotlarına ulaşarak çalmakta olan şarkının adını iletiyor.

RadioStation sınıfında görüleceği üzere radioA, removeObserver metoduyla yayını durdurulduktan sonra sıradaki şarkının adını görüntülemeyecektir.