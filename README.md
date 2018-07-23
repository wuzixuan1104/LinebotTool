# Linebot
## Step1. 創建line app
https://developers.line.me/en/docs/messaging-api/overview/
## Step2. 開發者建立機器人
https://developers.line.me/en/docs/messaging-api/building-sample-bot-with-heroku/
## Step3. 設定webhook
+ 機器人會call這隻api回傳訊息, 所以這隻api如同index.php進入點

![](https://i.imgur.com/iXFYCff.png)
+ 所有圖片、檔案或連結必須使用https

## Step4. 匯入linebot SDK
+ https://github.com/line/line-bot-sdk-php

## [補充] 在本機端測試 (非正規)
+ 將vendor中的LINEBot/Event/Parser/EventRequestParser.php
    + 將某行註解
```php=
public static function parseEventRequest($body, $channelSecret, $signature)
{
    if (!isset($signature)) {
        throw new InvalidSignatureException('Request does not contain signature');
    }

    // if (!SignatureValidator::validateSignature($body, $channelSecret, $signature)) {
    //     throw new InvalidSignatureException('Invalid signature has given');
    // }
    ....
}
```
+ Header

![](https://i.imgur.com/BLp6t9o.png)

+ Body

![](https://i.imgur.com/HqVSMwq.png)

## Step4. 匯入linebot SDK
+ 連結：https://github.com/line/line-bot-sdk-php
+ 安裝line message api sdk
```bash=
$ composer require linecorp/line-bot-sdk
```
+ bot client instance
```php=
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient('<channel access token>');
$bot = new \LINE\LINEBot($httpClient, ['channelSecret' => '<channel secret>']);
```
+ 發送請求
```php=
$body = file_get_contents ("php://input"); //如同撈取php後端的所有資料
$events = $bot->parseEventRequest ($body, $_SERVER["HTTP_" . LINE\LINEBot\Constant\HTTPHeader::LINE_SIGNATURE]);
```
+ 取得回應token
```php=
foreach( $events as $event ) {
 //可使用以下兩種方式取得回應
 //$response = $bot->replyText($event->getReplyToken(), 'hello cherry!');
  $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder('hello');
  $response = $bot->replyMessage($event->getReplyToken(), $textMessageBuilder);
}
```
+ 更多回應類型參考：https://developers.line.me/en/docs/messaging-api/reference/#webhook-event-objects

## Step5. 發送訊息樣式
+ 文字：Text message
```php=
MyLineBotMsg::create()
    ->text($event->getText())
    ->reply($event->getReplyToken());
```
+ 圖片：Image message
```php=
MyLineBotMsg::create ()
    ->image($url, $url)
    ->reply ($event->getReplyToken());
```
+ 影音：Video message
```php=
MyLineBotMsg::create()
    ->video($url, $url)
    ->reply ($event->getReplyToken())
```
+ 語音：Audio message
```php=
MyLineBotMsg::create()
    ->audio($url, $duration)
    ->reply ($event->getReplyToken());
```
+ 位置：Location message
```php=
MyLineBotMsg::create()
    ->location('my location', '〒150-0002 東京都渋谷区渋谷２丁目２１−１', '35.65910807942215', '139.70372892916203')
    ->reply ($event->getReplyToken());
```

+ 地圖：Imagemap message
```php=
MyLineBotMsg::create()
    ->imagemap('https://angel.ioa.tw/res/image/t5', 'test', 1040, 1040,
       [
          MyLineBotActionMsg::create()->imagemapMsg('文字', 0, 0, 100, 100),
       ])
    ->reply ($event->getReplyToken());
```
+ 按鈕樣板：Button Template message
```php=
MyLineBotMsg::create()->template('抬頭',
    MyLineBotMsg::create()->templateButton('按鈕', '説明', 'https://example.com/bot/images/image.jpg', [
      MyLineBotActionMsg::create()->message('是', 'true'),
      MyLineBotActionMsg::create()->postback('否', 'bbb=123'),
    ])
)->reply($event->getReplyToken());
```
+ 確認：Confirm Template message
```php=
Log::info($event->getText());
  $builder = MyLineBotMsg::create()->template('這訊息要用手機的賴才看的到哦',
      MyLineBotMsg::create()->templateConfirm( '你是女生？', [
        MyLineBotActionMsg::create()->message('是', 'true'),
            MyLineBotActionMsg::create()->message('否', 'false'),
      ])
  )->reply ($event->getReplyToken());
```
+ 滾動多個樣板：Carousel Template message
```php=
$builder = MyLineBotMsg::create()->template('這訊息要用手機的賴才看的到哦',
    MyLineBotMsg::create()->templateCarousel( [
      MyLineBotMsg::create()->templateCarouselColumn('標題', '哈哈哈哈哈', 'https://cdn.adpost.com.tw/adpost/production/uploads/adv_details/pic/00/00/00/00/00/00/06/5e/_29753e27ceb64b0f35b77aca7acf9a3e.jpg', [
        MyLineBotActionMsg::create()->datetimePicker('date', date('Y-m-d'), 'date', '', '', ''),
        // MyLineBotActionMsg::create()->message('label', 'test'),
        MyLineBotActionMsg::create()->uri("Google","http://www.google.com"),
        MyLineBotActionMsg::create()->postback('label', 'postback', 'postback'),
      ]),
      MyLineBotMsg::create()->templateCarouselColumn('標題', '哈哈哈哈哈', 'https://cdn.adpost.com.tw/adpost/production/uploads/adv_details/pic/00/00/00/00/00/00/06/5e/_29753e27ceb64b0f35b77aca7acf9a3e.jpg', [
        MyLineBotActionMsg::create()->datetimePicker('date', date('Y-m-d'), 'date', '', '', ''),
        MyLineBotActionMsg::create()->message('label', 'test'),
        // MyLineBotActionMsg::create()->uri("Google","http://www.google.com"),
        MyLineBotActionMsg::create()->postback('label', 'postback', 'postback'),
      ]),
    ])
)->reply ($event->getReplyToken());
```
+ 滾動圖片：Image Carousel messge
```php=
MyLineBotMsg::create()->template( '這訊息要用手機的賴才看得到喔！',
  MyLineBotMsg::create()->templateImageCarousel([
    MyLineBotMsg::create()->templateImageCarouselColumn(
      'https://cdn.adpost.com.tw/adpost/production/uploads/adv_details/pic/00/00/00/00/00/00/06/5e/_29753e27ceb64b0f35b77aca7acf9a3e.jpg',
      MyLineBotActionMsg::create()->uri("Google","http://www.google.com")
    ),
    MyLineBotMsg::create()->templateImageCarouselColumn(
      'https://cdn.adpost.com.tw/adpost/production/uploads/adv_details/pic/00/00/00/00/00/00/06/5e/_29753e27ceb64b0f35b77aca7acf9a3e.jpg',
      MyLineBotActionMsg::create()->uri("Google","http://www.google.com")
    ),
    MyLineBotMsg::create()->templateImageCarouselColumn(
      'https://cdn.adpost.com.tw/adpost/production/uploads/adv_details/pic/00/00/00/00/00/00/06/5e/_29753e27ceb64b0f35b77aca7acf9a3e.jpg',
      MyLineBotActionMsg::create()->uri("Google","http://www.google.com")
    )
  ]))
->reply ($event->getReplyToken());

```
+ 傳送多個（可一次傳送多筆樣板）：Multi messages
```php=
MyLineBotMsg::create ()
 ->multi ([
   MyLineBotMsg::create ()->text ($event->getText()),
   MyLineBotMsg::create ()->text ('hello')
 ])
 ->reply ($event->getReplyToken());
```
---
## 彈性樣板訊息
+ 官網：https://developers.line.me/en/reference/messaging-api/#action-objects
+ 線上樣板模擬器：https://developers.line.me/console/fx/
+ 其他範例：https://medium.com/linedevth/using-flex-message-to-create-world-cup-line-bot-60e0591f9d02

`[由於sdk尚未更新，以下自己撰寫]`

+ 類似組成html的概念，以區塊化分，需吐出json格式
```json=
{
    "type": "bubble",
    "header": {
        "type": "box",
        "layout": "vertical",
        "contents": [
            {
                "type": "text",
                "text": "Header text"
            }
        ]
    },
    "body": {
        "type": "box",
        "layout": "vertical",
        "contents": [
            {
                "type": "button",
                "action": {
                    "type": "postback",
                    "data": "Buy",
                    "text": "action=123"
                },
                "style": "primary",
                "color": "#0000ff"
            }
        ]
    },
    "footer": {
        "type": "box",
        "layout": "vertical",
        "contents": [
            {
                "type": "text",
                "text": "Footer text"
            }
        ]
    },
    "hero": {
        "type": "image",
        "url": "https://example.com/flex/images/image.jpg"
    },
    "styles": {
        "header": {
            "backgroundColor": "#00ffff"
        },
        "hero": {
            "separator": true,
            "separatorColor": "#000000"
        },
        "footer": {
            "backgroundColor": "#00ffff",
            "separator": true,
            "separatorColor": "#000000"
        }
    }
}
```
+ 單個訊息：Flex bubble messages
```php=
MyLineBotMsg::create()->flex('test',
  MyLineBotMsg::create()->flexBubbleBuilder()
    ->setHeader( FlexComponent::create()->setType('box')->setLayout('vertical')
        ->setContents([
           FlexComponent::create()->setType('text')->setText('Header text')])
    )->setBody( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
      FlexComponent::create()->setType('text')->setText('Body text')])
    )->setFooter( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
      FlexComponent::create()->setType('text')->setText('Footer text')])
    )->setHero( FlexComponent::create()->setType('image')->setUrl('https://example.com/flex/images/image.jpg')
    )->setStyles( FlexComponent::create()
       ->setHeader( FlexComponent::create()->setBackgroundColor('#00ffff') )
       ->setHero( FlexComponent::create()->setSeparator(true)->setSeparatorColor('#000000') )
       ->setFooter( FlexComponent::create()->setBackgroundColor('#00ffff')->setSeparator(true)->setSeparatorColor('#000000') )
))->reply($event->getReplyToken())
```
+ 輪播訊息：Flex Carousel messages
```php=
MyLineBotMsg::create()->flex( 'test', MyLineBotMsg::create()->flexCarouselBuilder([
  MyLineBotMsg::create()->flexBubbleBuilder()
     ->setHeader( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
     FlexComponent::create()->setType('text')->setText('Header text')])
     )->setBody( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
     FlexComponent::create()->setType('text')->setText('Body text')])
     )->setFooter( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
     FlexComponent::create()->setType('text')->setText('Footer text')])
     )->setHero( FlexComponent::create()->setType('image')->setUrl('https://example.com/flex/images/image.jpg')
     )->setStyles( FlexComponent::create()
       ->setHeader( FlexComponent::create()->setBackgroundColor('#00ffff') )
       ->setHero( FlexComponent::create()->setSeparator(true)->setSeparatorColor('#000000') )
       ->setFooter( FlexComponent::create()->setBackgroundColor('#00ffff')->setSeparator(true)->setSeparatorColor('#000000') )
),

MyLineBotMsg::create()->flexBubbleBuilder()
  ->setHeader( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
    FlexComponent::create()->setType('text')->setText('Header text')])
 )->setBody( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
    FlexComponent::create()->setType('text')->setText('Body text')])
 )->setFooter( FlexComponent::create()->setType('box')->setLayout('vertical')->setContents([
    FlexComponent::create()->setType('text')->setText('Footer text')])
 )->setHero( FlexComponent::create()->setType('image')->setUrl('https://example.com/flex/images/image.jpg')
 )->setStyles( FlexComponent::create()
    ->setHeader( FlexComponent::create()->setBackgroundColor('#00ffff') )
    ->setHero( FlexComponent::create()->setSeparator(true)->setSeparatorColor('#000000') )
    ->setFooter( FlexComponent::create()->setBackgroundColor('#00ffff')->setSeparator(true)->setSeparatorColor('#000000') )
)
]))->reply($event->getReplyToken());
```
