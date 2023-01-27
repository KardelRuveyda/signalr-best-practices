
# SignalR ile Run Time Uygulama Geliştirme- Server ve Client Uygulamalarının Geliştirilmesi

Merhabalar,
Yakın zamanda Doğuş Teknoloji bünyesindeki Chatbot(Lugado) ekibimizde geliştirdiğimiz uygulamamız için SignalR ile çalışma yapma imkanı bulduk. Buna istinaden bu teknolojiyi daha detaylı öğrenebilmek adına çalışmalara başladım. Bu noktada en Gençay Yıldız'ın SignalR ile Run Time Uygulama Geliştirme eğitim serisi(https://www.youtube.com/playlist?list=PLQVXoXFVVtp3RSycdru4WpnfPEOFxONiX) bana çok fayda sağladı. SignalR ile ilgili çalışma yapmak isteyen herkese de tavsiye ederim. Bu eğitimi izlerken aldığım notları da yazı halinde sizlerle paylaşmaya karar verdim. Bugünkü yazımızda Server ve Client Uygulamaların Geliştirilmesi konusuna değineceğiz. Herkese faydalı olması dileklerimle.

# SignalR' a neden ihtiyaç duyuyoruz? Request Response mantığında ilerlersek de işlerimizi halledemez miyiz?
Client-Server arasındaki klasik haberleşme yöntemi ile yapılan requeste karşılık verilen response ilişkisi üzerinden eşzamanlı olarak sağlanmaktadır.
Bu durum hepimizin yıllarca deneyimlediği gibi bir bekleme süreci yahut sayfanın gidip gelmesiyle sonuçlanmakta ve böyle kullanıcı açısından zamansal maliyetle birlikte deneyim açısından günümüze yakışmayan ilkelliğe neden olabilmektedir.
Client server klasik ilişkide bildiğimiz üzere klasik request ve response ilişkisi vardır. Client server tarafına bir request gönderir. Server gelen request neticesinde sonucu üretir. Üretilen sonucu response olarak geri client tarafına döndürür. Günümüzde ise bu durum pek yeterli olmayabilir. Yetersizlik durumlarını ortaya koyduğumuzda SignalR gibi Grpc gibi yapılandırmalar işe yarayabilir.

Web yaklaşımında ilerlenirse request response mantığında ilerlendiğinde gene bir sürekli yenime süreci olacaktır ve bu da kullanıcının tercih edeceği bir durum olmayabilir.
Özetle, Günümüz ihtiyaçlarını değerlendirirsek klasik web yaklaşımının tek başına pek yeterli olmadığı ve çözüm olarak farklı kütüphanelere hatta protokoller ihtiyaç olunduğunu kaçınılmaz. Örneğin, verilen örneklerde real time hizmet verebilecek bir teknolojiye ihtiyaç olduğu ve HTTP'den farklı olarak TCP protokolünü benimseyen WebSocket altyapılı sistemlerin kullanılması gerektiği ortadır.

# SignalR Nedir?
SignalR uygulamalarına,WebSocket teknolojisini kullanarak real time fonksiyonellik kazandıran open source bir kütüphanedir.

# SignalR temelinde hangi teknolojiyi barındırır?

SignalR altında yatan teknoloji WebSocket'tir Özünde RPC(Remote Procedure Call) mekanizmasını benimsemektedir. RPC sayesinde server, client tarafında bulunan herhangi bir metotun tetiklenmesini ve veri transferini sağlayabilmektedir. Böylece uygualmalar serverdan sayfa yenilemeksizin data transferini sağlamış olacak ve gerçek zamanlı uygulama davranışı sergilemiş olacaktır. Uygulamanın gerçek zamanı olması client ile server'ın anlık olarak karşılıklı haberleşmesi anlamına gelir.

# Peki ne zaman geliştirildi?

Microsoft tarafından 2011 yılında geliştirilmiştir. 2013 yılında ASP.NET mimarisine entegre edilmiştir. Günümüzde Asp.NET 6.0 mimarilerinde rahatlıkla kullanılabilmektedir. O yıllarda tüm browserların WebSocket protokolünü desteklememesi üzerine SignalR'ın kendi altyapısıyla gelerek client ile server arasındaki arasındaki haberleşmeyi real time olarak gerçekleştirebiliyor olması bir anda popüler olmasını sağlamıştır.

# Çalışma mantığı nasıl ?
SignalR 'Hub' ismi verilen merkezi bir yapı üzerinden şekillenmektedir. 'Hub' özünde bir class'tır ve içerisinde tanımlanan bir metota subscribe olan tüm clientlar 'Hub' üzerinden iletilen mesajları alırlar.

# DEMO

SignalR'ı daha yakından anlayabilmek için ufak bir uygulama gerçekleştirelim. Bir sonraki yazıılarda bu uygulamaya izlemiş olduğum eğitim dahilinde de eklemeler yapacağız. Haydi başlayalım.
# SignalR'da Server Uygulaması Geliştirme
Clientların iletişime geçmesi gereken yapıyı oluştururken öncelikle server yapısını kurmak önemidir. Hub clientlar ile server arasındaki ilişkiyi sağlamaktadır. Yani TCP ilişkisini sağlar. Clienttan gelen istekler, mesajlar hub üzerindeki tarafik ile sağlanır. Proje içerisinde birden fazla hub sınıfımız olabilir.
# Projenin Oluşturulması
Proje olarak boş bir .Net Core projesi seçebiliriz. Ek olarak farklı .Net Mvc ya da başka bir proje template seçerek de işlemlerinize devam edebilirsiniz. Tercih size kalmış. Bu proje kapsamında .Net Core 6.0 seçimlenerek bir server yapısı oluşturulmuştur.

# Hub Sınıfının oluşturulması
Server projesini .Net Core 6.0 ile oluşturmuş olduk. Bu noktada proje içerisinde bir klasör açmamız gerekir. Bu klasörde hublar tutulacaktır. Projenizde sadece bir tane değil birden fazla hub olabilir. O nedenle klasör adını "Hubs" şeklinde tutmamız doğru olacaktır. 
Hub sınıfından da türetmek gerekiyor. Bu örnekte de Hubs klasörünün altına bir adet test amaçlı "MyHub" sınıfı açılmıştır. Buradaki sınıfları açarken gene hub temelli olduğunun belirtilmesi için sonuna "Hub" öbeğini eklememiz yazım açısından doğru oalcaktır. Tıpkı controller oluşturduğumuzda sonuna "Controller" öbeğini eklememiz gibi düşünebilirsiniz. Eğer eklemezseniz tabii ki projeniz çalışmaz diyemem ama ekleseniz daha iyi olur sanki. :)

MyHub sınıfını oluşturmak tek başına yeterli olmayacaktır tabii. Bu sınıfı "Hub" dan türetmek de gerekir. Bu yapıyı kullanabilmek için herhangi bir nuget package yüklemenize gerek yoktur. SignalR .Net Core çekirdeğinin dahilinde gelen bir kütüphanedir. Direkt projenize dahil edebilirsiniz.


```c#
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace SignalRServerExample.Hubs
{
    public class MyHub : Hub
    {

    }
}
```

Bu senaryonun devamı için şu şekilde düşünebiliriz. Oluşturduğumuz bir server var ve MyHub'a bağlanan "n" adet clientımız olacak. Hub bağlı olabilmesi için benim burada bir fonksiyon oluşturmam gerekmektedir. Ne zaman ki o fonksiyon tetiklenir, ilgili clientlar da RPC sistemi sayesinde gerekli fonksiyonlar ile tetiklenmiş olur. Bu nedenle dışarıya public olan asenkron bir fonksiyon oluşturulabilir. Fonksiyonun ismi SendMessageAsync() ismini verdim. Bu metot cleintların subscribe olduğu metottur. Bizim declient tarafından gelecek olan mesajı parametreden almamız gerekmektedir. Bu kısımda aslında senaryo basittir. Client Hub'a bir mesaj gönderir. Server da bu mesajı alır. Server mesajı aldıktan sonra gerekli operasyonu Hub üzerinden diğer clientlara gönderir.
Whatsapp'tan yola çıkalım. Bir grubumuz var bu grupta mesaj yazıldığında mesaj sizin dışınızda herkese gider. Bu mimaride de gelen mesaj hub ile karşılanır. Bu grubu temsil eden fonksiyona subscribe olan diğer kullanıcıların her biri mesajı anında alır. Bu işlem de aslında bizim yazacağımız SendMessageAsync ile sağlanmaktadır.

ReceiveMessage isimli client tarafından bir fonksiyon bekliyorum. O fonksiyonu tetikle, tetiklerken de Client'ın gönderdiğ mesajı diğerlerine gönder. Ana mantık bu şekildedir. Bu işlemi de gerçekleştirdikten sonra hub hazır hale gelecektir. Ayrıca aşağıdaki örnekte tüm clientlara mesaj gönderilmesi istendiği için "Clients.All" " yapısı kullanılmıştır.



```c#
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace SignalRServerExample.Hubs
{
    public class MyHub : Hub
    {
        public async Task SendMessageAsync(string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", message);
        }
    }
}
```


# Program.cs üzerinde yapılması gereken ayarlamalar

Core mekanizmasına WebSocket protokolünü kullanacağını bildirmek gerekmektedir. Bunun için AddSignalR() fonksiyonunu kullanmanız gerekmektedir. .Net 6.0 ile geliştirme yapıyorsanız bileceğiniz üzere Startup.cs dosyası bulunmuyor işlemlerimizi Program.cs üzerinde yapmamız gerekmektedir. Aşağıdaki gibi eklemeler yaparsanız Server üzerindeki Program.cs ayarlarını başarıyla tamamlamış olacaksınız. Ek olarak endpoint tarafına da değinmek isterim. Aşağıdaki kod bloğunda da bulunan kısım aslında client tarafının server tarafını tetiklerken gideceği url linkini gösterir. Yani {domain}/myhub dendiğinde oluşturduğumuz MyHub sınıfınındaki SendMessageAsync metotunu tetiklemiş oluruz.

```c#
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Authorization;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;
using SignalRServerExample.Hubs;
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSignalR();
builder.Services.AddCors();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseWebSockets();

app.UseHttpsRedirection();

app.UseRouting();

app.UseRouting();

// global cors policy
app.UseCors(x => x
    .AllowAnyMethod()
    .AllowAnyHeader()
    .SetIsOriginAllowed(origin => true) // allow any origin
    .AllowCredentials()); // allow credentials            

app.UseAuthorization();

app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<MyHub>("/myhub");
});

app.Run();
```


# SignalR'da Client Uygulaması Geliştirme

Client uygulamasını istediğiniz teknoloji ile gerçekleştirebilirsiniz. Bu örnekte basit bir HTML üzerinden işlemlerimizi gerçekleştireceğiz. HTML üzerinden server tarafını tetikleyecek bir uygulama hazırlayabilmek için iki adet kütüphaneye ihtiyacım var. JQuery ile işlem gerçekleştirileceği için Jquery , SignalR tetiklemelerini gerçekleştirebilmek için ise microsoft/signalr kütüphanesini yüklemek gerekmektedir.

```c#
npm i jquery @microsoft/signalr
```
Projeleri yükledikten sonra node_modules tarafında ihtiyaç duyacağım kısmlar signalr.min.js ve jquery.min.js olacaktır. Bu dosyaları dosya dizinine taşıyıp diğer paketleri silebilirsiniz.

HTML içerisinde bu kütüphaneleri dahil ederken öncelikle signalr.min.js, sonrasında jquery.min.js refarans alınmalıdır.


```html
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="signalr.min.js"></script>
    <script src="jquery.min.js"></script>
    <title>Document</title>
</head>
```

SignalR'ın tetikleneceği js kodlarını da <head> içerisinde <script> etiketi arasında tanımlayacağız.

```html
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="signalr.min.js"></script>
    <script src="jquery.min.js"></script>
    <title>Document</title>

    <script>

    </script>
</head>
```

Script içerisinde ready fonksiyonuyla client tabanlı yapılacak kodlar tanımlanacaktır. Burada SignalR'ın HubConnectionBuilder() sınıfından yararlanarak hub adresi eklenip derlenme işlemi gerçekleştirilir.


```html
<script>
        $(document).ready(() => {
            const connection = new signalR.HubConnectionBuilder().
            withUrl("https://localhost:5001/myhub").build();
        })
    </script>
```
Bu işlem gerçekleştirildikten sonra bağlantının gerçekleşebilmesi için "connection.start()" kullanılır.


```html
<script>
        $(document).ready(() => {
            const connection = new signalR.HubConnectionBuilder().
            withUrl("https://localhost:5001/myhub").build();

            connection.start();
        })
    </script>
```

Client tarafının server tarafını tetikleyebilmesi için oluşturulan bu demo uygulamada daha gerçekçi bir test olabilmesi adına bir buton bir de input alan belirlenir.

```html
<body>
    <input type="text" id="txtMessage">
    <button id="btnGonder"> Gönder </button>

    <div></div>
</body>
```

Belirlenen input alanına yazı girilip göndere basıldığında hub tarafındaki "SendMessageAsync" metotunun tetiklenebilmesi için "connection.invoke" 'dan yararlanılır. Buradaki alanda eğer herhangi bir hata alınırsa HubConnectionBuilder sınıfının sağladığı "catch" hata yakalama özelliği kullanılarak hatalar konsola basılır.

```html
<script>
        $(document).ready(() => {
            const connection = new signalR.HubConnectionBuilder().
            withUrl("https://localhost:5001/myhub").build();

            connection.start();

            $("#btnGonder").click(()=>{
                let message = $("#txtMessage").val();
                connection.invoke("SendMessageAsync",message)
     .catch(error => console.log(`Mesaj gönderilirken hata oluştu. ${error}`))
            })
        })
    </script>
```

Şu anda client ile server arasındaki bağlantı sağlandı ve mesajlarımız server tarafına ulaşıyor. Ancak client tarafından server tarafına giden mesajların bir de geri dönüş olarak client tarafına gidebilmesi için client tarafının bu sonucuyu dinlemesi gerekemektedir. Bunun için de "connection.on" yapısından yararlanılır.


```html
<script>
        $(document).ready(() => {
            const connection = new signalR.HubConnectionBuilder().
            withUrl("https://localhost:5001/myhub").build();

            connection.start();

            $("#btnGonder").click(()=>{
                let message = $("#txtMessage").val();
                connection.invoke("SendMessageAsync",message).
  catch(error => console.log(`Mesaj gönderilirken hata oluştu. ${error}`))
            })

            connection.on("ReceiveMessage", message => {
                console.log(message);
                $("div").append(message + "<br>");
            })
        })
    </script>
```

Veee uygulama hazır! Mesajların server tarafına gönderildiği gibi server tarafından da client tarafına mesajların ulaştığı ve bu bağlantı gerçekleştikçe yeni bir div atarak bu mesajların gösterildiğini api projesinde deneyimleyebilirsiniz.

