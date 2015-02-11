# Simplest Delegate Keynote

撰寫人：林立
日期：2014.5.13

---
# 簡述 Delegate
Delegate負責代為處理其他物件要做的工作。

---
# 入門教學 

## 生活上的例子

委託代辦的情況在生活上很常發生。例如我們要出國旅遊，護照和機票會由旅行社代辦；要找房子，會請房仲業者幫忙尋找；要修馬桶，會打電話請水電工人幫忙修理。在程式中，我們也常將A物件要做的事委託B物件來做，此時，B物件就是A物件的delegate，也就是受委託的物件。在以 Java 為主的 Design Pattern 的教科書上，delegate pattern，通常是這個意思。

不過，在 Objective-C/Cocoa Framework 中，delagate 通常是另外一個意思－我們希望旅行社幫我們辦機票與護照，但是旅行社往往不知道一些只有我們知道的必要資訊，例如，旅行社需要我們的照片與身分證字號，所以，在需要照片與身分證字號的時候，旅行社會回來問我們。在 Objective-C 的觀念中，我們會是旅行社的 delegate－旅行社必須問我們必要的問題，才能完成工作。

Objective-C 語言中的 delegate，更像是一個專門處理多種不同 callback 的統一物件。說到 callback，我們先從 C 語言說起。

## C語言的 Callback Function

在C當中，每個 function 都有對應的指標（pointer），和其他形態的指標並無特殊之處，我們可以稱它叫function pointer。有了這個pointer，就可以進行呼叫對應的 function。

聰明的人會想到，可以把它拿來當參數傳遞。這個被傳遞的，要拿來被執行的function，就叫做「callback function」，簡稱「callback」，而我們通常用callback來做到委託這件事。

C語言當中沒有物件導向的觀念，所以直接實作callback，而 Objective-C當中，各個物件分別處理不同的任務，我們便可以將物件A要做的事交給物件B去做。我們就不再用 function pointer 傳遞 callback，指定一個被委託做事的物件，受委託實作callback的物件，就是 delegate。

##iOS如何做到委託

在iOS當中，我們一定會看到，雖然是由 Controller 要求 View 更新，但 View 更新時，會回頭跟 Controller 要資料，這樣的動作算是一種 callback，於是，我們會把 Controller 設為 View 的 delegate。

以UITableView 為例，UITableViewController 就是 UITableView 的 delegate，如果是 UITableViewController 的 subclass，就可以自行實作 delegate 提供資料的方法。這些方法，就稱作 delegate method。

UITableView 要實作的 delegate method 包括：有幾行資料要顯示？每一行顯示什麼資料？可不可以進行新增刪除的動作？若編輯某一行的資料，原始資料會怎麼改變？因為要委託的工作有這麼多，我們不確定受委託的對象，可以做好所有的工作。

所以，就像 App 公司發包請設計公司設計App畫面一樣，委託需要先經過確定工作內容、簽訂合約的過程。View 委託 Controller，把 Controller 當 Delegate 提供資料時，同樣如此：View 與 Controller 之間的協定，有個術語叫做 Protocol，像 UITableView 與 UITableViewController 之間，就有 UITableViewDelegate 與 UITableViewController 兩個 Protocol。

##設定 Delegate 的步驟

要讓 View 委託 Controller 來顯示資料有底下幾個步驟：

1. View 要先擁有一個 delegate 
2. Controller 要把自己設成 View 的 delegate
3. 定義 delegate 可以為View做哪些事，也就是定義 Protocol
4. 對 Delegate 賦予 Protocol的身份
5. 實作 delegate method
6. 做好一切準備，開始使用

### View 要先擁有一個 delegate 

首先我們在行定義一個MyView，在裡面加一個成員變數 delegate。

```
@interface MyView : UIView
@property (weak, nonatomic) id delegate;
@end
```


這裡以一個繼承 UIVIew （iPhone 的view）的 自製 class - MyView 為例。我們產生了一個 UIView subclass，裡頭有一個成員變數叫做 delegate。其中 id 是一個指標，它指向未知類別的物件。這個 delegate 將會用來指向要做事的 Controller。

View擁有了一個 delegate 之後，接下來 Controller 就要把那個 delegate 設成 Controller 自己。

### Controller 要把自己設成 View 的 delegate

比方說我們要使用的 UIVewController 是 DisplayController，它負責前面講到的MyView。同時它有一個dataArray的成員變數，裡面是 MyView 所需繪畫的資料的陣列，可以一連串的點的座標陣列。裡面宣告可能是這樣：

```
@interface DisplayController : UIViewController
@property (strong, nonatomic) NSArray *dataArray;
@property (strong, nonatomic) IBOutlet MyView *myView;
@end
```

有了 Controller，接下來要怎麼將 delegate 設成自己呢？要在 DisplayController 裡面這樣做：

```
dataArray = [[NSArray alloc] init];
myView.delegate = self;
```

可以在 viewDIdLoad （iPhone）或 awakeFromNib 做這件事情。

### 定義 delegate 可以為View做哪些事，也就是定義 Protocol

設定了 Controller 為 View 做事情，接下來就要宣告要做哪些工作。這就像發包專案一些，我們要先定義好有哪些工作項目。這些定義好的工作項目，在這裡就叫做Protocol。Protocol是長這樣子的：

```
@protocol MyViewDataSource
- (NSArray *)viewRequestArrayForDrawing:(MyView *)myView;
@end
```

因為在這裏 MyView 向 DisplayController 要 array 這資料，所以我們給了一個方法叫 viewRequestArrayForDrawing 來提供資料。

### 對 Delegate 賦予 Protocol的身份

定義了這個 Protocol 之後，我們要將它賦予到 delegate 身上。分別有兩個地方要這樣做，一個是擁有 delegate 的 myView 身上，一個是實際上擔任 delegate 的 DisplayController 本身。

在 View 的部分是這樣：

```
@interface MyView : UIView
@property (weak) id <MyViewDataSource> delegate;
@end
```

這表示 delegate 擁有 MyViewDataSource 這個身份。

在 Controller 的部分是這樣：

```
@interface DisplayController : UIViewController <MyViewDataSource>
@end
```

這表示 DisplayController 擁有 MyViewDataSource 這個身份。

### 實作 delegate method 

差不多要完成了。因為 DisplayController 被賦予了 MyViewDataSource 這個身份。所以它必須將 MyViewDataSource 所要求的方法實作出來。

```
- (NSArray *)viewRequestArrayForDrawing:(MyView *)myView
{
 	return self.array;
}
```

實作了這個方法之後，就可以開始把 DisplayController 當 delegate 使用了。

### 做好一切準備，開始使用

在 View 當中是在 drawRect 這個方法畫圖的。畫圖時我們就可以向剛剛準備好的 Controller 來拿資料：

```
- (void)drawRect:(CGRect)rect
{
    NSArray *array = [self.delegate viewRequestArrayForDrawing:self];
    // 拿到 array 之後，來看要怎麼畫..
}
@end

```

在這裡， self.delegate 實際上就是 DisplayController。所以 viewRequestArrayForDrawing 是由 DisplayController 來做，而非 MyView 本身的方法來做。

好好的委託一項工作是不容易的。經過以上步驟，我們就成功委託 DisplayController 來做 MyView 的工作。




---

# 進階分析

## 細講 Protocol 與 Delegate

上面為了教學的方便，省略了一些實作的細節和應注意的事項。在這裡將 Protocol 和 Delegate 幾個實作細節一一說明。

### Protocol的 Require 與 Optional

Protocol其實有 Require 和 Optional 兩種選項。若不寫，預設是 Require，像上面的範例一樣：

```
@protocol MyViewDataSource

- (NSArray *)viewRequestArrayForDrawing:(MyView *)myView;

@require
- (MyPoint *)viewRequestPointForDrawing:(MyView *)myView;

@optional
- (void)cleanData:(MyView *)myView;

@end
```

而當遇到要使用 Optional 的方法時，物件必須確定 delegate 真的有實作這個方法，才去呼叫。要怎麼確定該方法是否存在呢？我們會使用 respondsToSelector 檢查。

```
if (self.delegate && [self.delegate respondsToSelector:@selector(refreshState)]) {
    [self.delegate refreshState];
}
```

### 命名 Delegate Method 的習慣

在設計 delegate method 的時候，我們往往會把是哪個物件傳來這個要求當做參數，來判斷現在是為哪個 View 做事，這是由於，同一個 Controller，可以同時為很多個 View 做事。比方說，在上面提到的 -viewRequestArrayForDrawing:，就傳遞了 myView，於是，我們可以透過 view 的指標判斷－要資料畫圖的 view 到底是哪一個，然後我們可以給不同的 view 不同的資料。

這個 delegate 是指標的性質，也代表了我們可以動態地改變被 delegate 指向的物件。可能在前面我們使用 ControllerA為 view 做事，五分鐘後我們會用 ControllerB 為 View 做事。這是 delegate 的優點，讓我們可以靈活地去運用它。

### delegate 是匿名類型，且是弱指標

delegate 是一個匿名類型(anonymous type)的指標，這是由於我們要避免限制 delegate的形態。因為 delegate 可以是不同類別的物件，因此我們採用 id 。如下面程式：

```
@interface MyView : UIView
@property (weak, nonatomic) id delegate;
@end
```

同時，因為避免 retain cycle 造成記憶體洩漏，我們使用弱指標的 delegate。

在 Objective-C 當中有兩種指標，一種是強指標 Strong（也就是MRC當中的 retain ），一種是弱指標 weak（也就是 MRC 當中的 assign）。假如物件之間循環地以強指標指向，就會導致 retain cycle。

因為 Controller 會 retain 住 view，而 view 會 retain 住 delegate，倘若 delegate 再 retain 住 Contoller，就會形成 retain cycle。相對於強指標的 retain，我們就要用弱指標的 weak，才可以解決這個問題。

但是 iOS 當中也有唯一仍使用 Strong 的 delegate 的例外，就是 NSURLConnection。


### 可多重賦予的 Protocol

Protocol 本身是可以多重賦予的。例如 UITableViewController 有兩種 delegate，UITableViewDataSource 和 UITableViewDelegate。

```
@interface SimpleTableViewController : UIViewController <UITableViewDelegate, UITableViewDataSource>
```

這邊的 SimpleTableViewController 被賦予了兩個 Protocol，UITableViewDataSource 和 UITableViewDelegate。

除此之外，為了要擁有 -respondsToSelector 的方法，也需要讓自製的 Protocol 被賦予 <NSObject>。



###Delegate的優點


使用委託者模式有幾個優點：  

* 分清楚class的職責，符合MVC的架構。
* 搭配訪問者模式（Accessors pattern），處理非同步的callback。 
* 方便更換delegate，由不同物件實作委託的方法。

Delegate 可以幫助分清楚 class 的職責，符合 MVC 的架構，可以分清楚每個class的職責，讓每個class只處理一項單一的事務。在 MVC 架構中，我們通常希望把 View 和 Model 兩個不變的類型，從 Controller 當中拆出來，好讓他們能被重複利用。因此 譬如上面所舉的 View - Controller 例子，就讓 Controller 處理提供資料的邏輯，而讓 View 單單處理畫面呈現的樣子。

在網路溝通當中，我們也常使用delegate。例如一個Controller要做網路請求，它就生成NSURLConnection的物件Connection來作網路溝通。因為網路溝通常是非同步的，傳出資料後，我們並不能立刻在NSURLConnection的回傳參數拿到，所以只能在NSURLConnection建立方法來處理回傳的資料。但是NSURLConnection的職責應該專注在網路溝通上，不應該接觸到Model及處理回傳內容，因此就必須設定Controller為Connection的delegate，接收回傳的資料做處理。

我們有時也會切換使用不同的delegate來實作要做的工作。例如我們若要用同一個TableView當中，切換顯示不同種類的動物列表，就可以將它的datasource更換掉，也就是由不同的資料來源提供，這樣就能快速地切換顯示不同套的資料。

### 比較 Delegate 與其他類似的方法

除了 Delegate 之外，還有其他方法也可以請其他物件代為做事。例如 KVO、Notification、Block。相比之下， delegate 只能同時間委派一個對象做事，但又比 block 更具型別安全的特性。
