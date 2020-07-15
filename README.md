# ajax_mynews課題

１，下記のコードを全部記述

２，`npm run watch`を実行

３，サーバーを起動して/apiviewにアクセス

４，表示されたchange colorボタンやadd helloボタンが効くか試す

５，/apiviewに書いてある課題に取り組む

----

↓【既存のファイルに追記】↓

routes/web.php
```
Route::get('/apiview', 'NewsController@apiview');
```

routes/api.php
```
//URLが/apiから始まる
//csrf保護無効=外部からアクセスできる
Route::group(['middleware' => ['api']], function(){
  Route::get('news', 'Api\NewsController@index');
});
```

app/Http/Controllers/NewsController.php
```
public function apiview()
{
    return view('news.apiview');
}    
```

app.js
```
require('./bootstrap');
require('./ajax_practice.js'); //この行を追記
```

----

↓【新たにファイルを作成】↓

resources/views/news/apiview.blade.php
```
<!doctype html>
 <html lang="{{ app()->getLocale() }}">
     <head>
         <meta name="csrf-token" content="{{ csrf_token() }}">
         <script src="{{ secure_asset('js/app.js') }}" defer></script>
        <link href="{{ secure_asset('css/app.css') }}" rel="stylesheet">
        <title>apiview</title>
     </head>
     <body>

         <div class='test'>hello view</div>
         <button id='change_color'>change color</button>
         <button id='add_hello'>add hello</button>
         <button id='get_news'>get news</button>

         <br>
         <div class='target_hello'></div>

         <br>
         <table class='news' border='2'>
             <tr>
                 <th>id</th>
                 <th>title</th>
                 <th>body</th>
             </tr>
         </table>

         <br>

         <h5>【課題】newsテーブルにデータを保存できるadd_newsボタンのAjaxを実装しましょう（値は決め打ちでokです）</h5>
         <button id='add_news'>add news</button>
         <br>
         ヒント：
         <ul>
             <li>Ajaxのdataに連想配列(json)を渡すことで、サーバーにデータの送信が可能です</li>
             <li>コントローラでデータを取得する方法→<a href='https://readouble.com/laravel/5.5/ja/requests.html'>JSON入力値の取得の項目</a></li>
         </ul>
         <h5>【応用】画面で入力した値をnewsテーブルに保存できるよう改修しましょう</h5>
     </body>
 </html>
 ```

resources/js/ajax_practice.js
```
$(function() {
     //ボタンを押すと色が変わる
     $('#change_color').click('on',function(){
         var colors = ['red', 'blue', 'yellow','white'];
         $('body').css("background-color", colors[getRandomIntInclusive(0,colors.length-1)]);
     });

     //HTMLを追加することもできる
     $('#add_hello').click('on',function(){
         var msg = '<p>hello</p>';
         $('.target_hello').append(msg);
     });

     //通信して取得したデータを元に、HTMLを追加することもできる
     $('#get_news').click('on', function(){
         $.ajax({
             url: '/api/news',
             type: 'GET',
             data: {}
         })

         .done(function(response) {
             console.log(response);
             var row;
             for(var i=0; i<Object.keys(response).length; i++){
                 row = row + "<tr>"
                 row = row + "<td>"+ response[i].id +"</td>";
                 row = row + "<td>"+ response[i].title +"</td>";
                 row = row + "<td>"+ response[i].body +"</td>";
                 row = row + "</tr>";
             }
             $('.news').append(row);
         })

         .fail(function() {
             alert('エラー');
         });
     })
 });

 /*
   min-max間のランダムな整数を返す
 */
 function getRandomIntInclusive(min, max) {
     min = Math.ceil(min);
     max = Math.floor(max);
     return Math.floor(Math.random() * (max - min + 1)) + min; //The maximum is inclusive and the minimum is inclusive
 }
 ```

app/Http/Controllers/Api/NewsController.php
```
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\HTML;

use App\News;

class NewsController extends Controller
{
    public function index(Request $request)
    {
        $news = News::all()->sortByDesc('updated_at');
        return $news;
        /*
           この場合$newsは返すだけで何もしていないので、
           return News::all()->sortByDesc('updated_at');　
           の方がスマート
        */
    }    
}
```
