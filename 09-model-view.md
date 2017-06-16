  
p.p1 {margin: 0.0px 0.0px 0.0px 0.0px; font: 11.0px Helvetica; -webkit-text-stroke: \#000000}  
p.p2 {margin: 0.0px 0.0px 0.0px 0.0px; font: 11.0px Helvetica; -webkit-text-stroke: \#000000; min-height: 13.0px}  
span.s1 {font-kerning: none}  


我們現在要來自己寫個 view，把前面製作的 model 顯示出來。我們把這個 view 放在 /stores/，所以：

  


\# lunch/u r ls.py

  


from django.conf.u r ls import include, u r l

from django.contrib import admin

from stores.views import home, store\_list \# 記得 import

  


u r lpatterns = \[

 u r l\(r'^$', home, name='home'\),

 u r l\(r'^store/$', store\_list, name='store\_list'\),  \# 新增這一行

 u r l\(r'^admin/', include\(admin.site.u r ls\)\),

\]

接著是 store\_list function。為了把店家列表顯示在這個頁面上，我們要使用 Django 的 ORM query。Django 的 ORM query 模式如下：

  


Model class 的某些 attributes（預設是 objects）會回傳一個 model manager。

Model manager 提供 query methods，回傳 query 物件——這是一個 iterable，所以你可以對它使用 index 與 iterator 介面。

Query 物件同樣提供 query methods，可以產生新的 Query 物件。

當你實際需要 Query 物件中包含的 model 物件時，Django 才會真正從資料庫把資料拿出來。

以 store\_list 而言，我們需要 Store 的全部資料。所以我們可以使用下面的語法：

  


stores = Store.objects.all\(\)

其中 Store.objects 會回傳一個 model manager。這個 manager 提供了 all method，回傳一個 query 物件。所以 stores 包含的是一個 query 物件，其中包含一個 SQL 指令（但尚未執行），大致等同於下面的格式：

  


SELECT \* FROM "stores";

所以我們可以用上面的語法，建立列表頁面的 view function：

  


\# stores/views.py

  


from .models import Store

  


def store\_list\(request\):

 stores = Store.objects.all\(\)

 return render\(request, 'store\_list.html', {'stores': stores}\)

我們取出所有的 store objects 後，在 render 的最後面增加一個參數。這個參數為 render 提示了額外的 context data，可以在處理 template 時使用。

  


再來是 store\_list.html：

  


{\# stores/templates/store\_list.html \#}

  


&lt;!DOCTYPE html&gt;

&lt;html&gt;

&lt;head&gt;

&lt;title&gt;店家列表 \| 午餐系統&lt;/title&gt;

&lt;link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css"&gt;

&lt;/head&gt;

  


&lt;body&gt;

&lt;nav class="navbar navbar-default navbar-static-top" role="navigation"&gt;

&lt;div class="container"&gt;

&lt;div class="navbar-header"&gt;

&lt;a class="navbar-brand" href="{% u r l 'home' %}"&gt;午餐系統&lt;/a&gt;

&lt;/div&gt;

&lt;/div&gt;

&lt;/nav&gt;

  


&lt;div class="container"&gt;

 {% for store in stores %}

&lt;div class="store"&gt;

&lt;h2&gt;{{ store.name }}&lt;/h2&gt;

&lt;p&gt;{{ store.notes }}&lt;/p&gt;

&lt;/div&gt;

 {% endfor %}

&lt;/div&gt;

  


&lt;/body&gt;

&lt;/html&gt;

我們看到了一組新的 template tags for 與 endfor。這是 Django templates 中表示迴圈的方法，對 Python programmers 而言這應該不難理解。在 templates 裡沒辦法用縮排表示階層，所以我們是用一個 endfor 標籤來標示迴圈結束。取用物件內 attributes 的方法仍然是用一個點，不過我們必須在旁邊加上兩組大括弧，來標注這是 template 語法，而不是真的要在 HTML 中印出 store.name 這個字串。

  


把 server 跑起來，你應該可以在 http://localhost:8000/store/ 看到列表頁。不錯吧！接著是每個店家的獨立頁面。

  


第一件事情：我們要規劃如何把網址對應到合適的頁面。Django 的做法就是用 capturing groups：

  


\# lunch/u r ls.py

  


from django.conf.u r ls import include, u r l

from django.contrib import admin

from stores.views import home, store\_list, store\_detail  \# 記得 import

  


u r lpatterns = \[

 u r l\(r'^$', home, name='home'\),

 u r l\(r'^store/$', store\_list, name='store\_list'\),

 u r l\(r'^store/\(?P&lt;pk&gt;\d+\)/$', store\_detail, name='store\_detail'\), \# 新增這行

 u r l\(r'^admin/', include\(admin.site.u r ls\)\),

\]

\(?P&lt;name&gt;pattern\) 是 regular expression 的 named group，就是根據 pattern 把抓到的 group 取名為 name，這在 u r l tag 裡面會用到，稍後解釋。pk 在這裡指 primary key，是 Django 慣用的命名法。

  


接者是 view function。在 U r l 中被捕捉的值會直接被傳入，所以：

  


\# stores/views.py

  


from django.http import Http404

  


def store\_detail\(request, pk\):

 try:

 store = Store.objects.get\(pk=pk\)

 except Store.DoesNotExist:

 raise Http404

 return render\(request, 'store\_detail.html', {'store': store}\)

我們前面提過，model manager 與 query 物件會提供 query methods，用來建構新的 query object。但它們也有一些可以直接回傳單一物件的 query methods——例如這裡的 get。這個 method 的用途就是取得「一個」物件來回傳，或者如果找不到，就 raise 一個 Model.DoesNotExist exception。在這個例子裡，我們把這個 exception 用 try-except block 包起來，在找不到對應物件時回應 404 Not Found。\[註 1\]

  


不過為什麼我們可以直接使用 pk？我們的 model 裡沒有這個值吧？其實 pk 是 Django model 的特殊性質，永遠指向該 model 的 primary key（當然）。前面提過，Django model 一定要有 primary key，而且如果你沒有特別設定，預設會在每一個 model 加上一個 id 欄位來當作 primary key。所以在這個例子中使用 pk 就等同於 id，但是用 pk 在大多時候比較有彈性，因為事實上 primary key 可以隨意設定，不見得要是 id。

  


最後是 template：

  


{\# stores/templates/store\_detail.html \#}

  


&lt;!DOCTYPE html&gt;

&lt;html&gt;

&lt;head&gt;

&lt;title&gt;{{ store.name }} \| 午餐系統&lt;/title&gt;

&lt;link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css"&gt;

&lt;/head&gt;

  


&lt;body&gt;

&lt;nav class="navbar navbar-default navbar-static-top" role="navigation"&gt;

&lt;div class="container"&gt;

&lt;div class="navbar-header"&gt;

&lt;a class="navbar-brand" href="{% u r l 'home' %}"&gt;午餐系統&lt;/a&gt;

&lt;/div&gt;

&lt;/div&gt;

&lt;/nav&gt;

  


&lt;div class="container"&gt;

&lt;h1&gt;{{ store.name }}&lt;/h1&gt;

&lt;p&gt;{{ store.notes }}&lt;/p&gt;

&lt;table class="table"&gt;

&lt;thead&gt;

&lt;tr&gt;&lt;th&gt;品項&lt;/th&gt;&lt;th&gt;單價&lt;/th&gt;&lt;/tr&gt;

&lt;/thead&gt;

&lt;tbody&gt;

 {% for item in store.menu\_items.all %}

&lt;tr&gt;&lt;td&gt;{{ item.name }}&lt;/td&gt;&lt;td&gt;{{ item.price }}&lt;/td&gt;&lt;/tr&gt;

 {% endfor %}

&lt;/tbody&gt;

&lt;/table&gt;

&lt;/div&gt;

  


&lt;/body&gt;

&lt;/html&gt;

我們這裡又多玩了一個小花樣。還記得 Store 與 MenuItem 之間的多對一關聯嗎？我們這裡用了 menu\_items 這個 reverse relation key，來獲得所有與 Store 物件有關聯的 MenuItem 實例。由於 Store 到 MenuItem 是多對一，所以 menu\_items 同樣會回傳一個 model manager；我們這裡同樣用 all 把所有物件列出。注意在 template 中，呼叫方法時並不用加括弧！

  


現在我們有 detail 頁了，所以可以在列表頁加上連結。把 stores/templates/store\_list.html 的 &lt;h2&gt; 那行改成這樣：

  


&lt;h2&gt;&lt;a href="{% u r l 'store\_detail' pk=store.pk %}"&gt;{{ store.name }}&lt;/a&gt;&lt;/h2&gt;

我們再次使用了 u r l tag，不過這次除了 name 之外多加了一個參數。這裡傳入的參數會被用在 U r l 的 capturing group 上，所以這樣就可以得到某個 store 的 U r l！

  


來比對一下 template 中的 u r l tag 與 u r ls.py的內容幫助理解順便複習概念：

  


{% u r l 'store\_detail' pk=store.pk %}

u r l\(r'^store/\(?P&lt;pk&gt;\d+\)/$', store\_detail, name='store\_detail'\)

u r l tag 的第一個 arg 會先去 u r ls.py 中找到 name='store\_detail' 的 u r l，但由於我要拿到該 Store 的 u r l 還需要一個 pk 參數（不然誰知道你要拿哪家 store 的 u r l？），所以第二個 arg 我們就指定為該 store的pk、把store.pk傳入給\(?P&lt;pk&gt;\d+\)。

  


如果你熟悉 CRUD 的話，我們今天實作的其實就是那個 R。其實不難吧！不過上面的程式其實不是很優秀，所以我們明天會花一點來改寫，讓它符合 best practice。

  




註 1：那麼，如果有很多個物件符合，會回傳哪一個？答案是都不會！如果 get 找到超過一個結果，會 raise Model.MultipleObjectReturned 例外。不過在實務上，這個 method 通常是用在有 UNIQUE 屬性的欄位，所以這個 exception 比較少被使用。詳細的用法可參考官方文件。

