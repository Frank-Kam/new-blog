---
title: "THJCC CTF 2026 Writeup"
description: "THJCC CTF 2026 Official Writeup (zh-TW ver)"
published: 2026-2-22
draft: false
tags: ['CTF', 'THJCC', 'Cyber-Sec']
---

好 那今天呢 風光明媚風和日麗 因爲我以前在唸書的時候 我常常覺得很奇怪就是我到學校然後看他們吃什麼早餐都是吃一份蛋餅配一杯奶茶 或是 一個漢堡配一杯奶茶 或是一份蘿蔔糕配一杯奶茶 每次吃完都說啊我吃飽了我心裡就想說這真的可以吃飽？因爲你知道我通常早餐我都點3份可能 3個 我想說這樣就能吃飽？一定是唬人的嘛 我才想到說大家都是要面子的 沒有沒有 我都帶好幾份去吃 那是你嘛 你不要臉嘛 我說正常的 我就想說 不行 我們一定要做個企劃 就是有一天要讓自己的胃滿足 大滿足 所以今天的企劃就是 當總召！  
  
以上改自張家兄弟神片早餐吃到飽，誠如各位所見我當上總召了，這故事留到都 2026 年 2 月了還沒寫的 2025 年度總結來說。
  
題目原始碼：[https://github.com/Frank-Kam/THJCC-CTF-2026](https://github.com/Frank-Kam/THJCC-CTF-2026)  
## Welcome to THJCC CTF (Welcome)  
### 題敘
```
In this CTF, unless otherwise specified, the flag format is THJCC{.\*}  
```
我們直接按 F12 然後重新整理，找到題目的封包然後搜尋 `THJCC{` 就可以得到 Flag  
`THJCC{We1c0m3-tO-tHjcC-c7F_2O26}`  

## Las Vegas (Web)
### 題敘
```
Lucky 7 7 7  
```
我們進到網站會有一個 `pull` 按鈕可以按，開 F12 或 Burp 可以看到會 POST 一個類似 `/?n=789` 之類的數字，所以我們使用 cURL 就可以解了  
```bash
curl -X POST http://chal.thjcc.org:14514/?n=777
```
得到  
`^THJCC\{LUcKy_sEVen_[0-9a-f]{16}\}$` 

## Ear👂(Web)
### 題敘
```
[CWE-698](https://cwe.mitre.org/data/definitions/698.html)
> Author: Frank
```
[CWE-698](https://cwe.mitre.org/data/definitions/698.html) 其實是一個水洞，但通常會造成資訊洩露或 CSRF 這種嚴重的結果，具體的成因就像  

```php
<?php
require_once 'config.php';

if (empty($_SESSION['username'])) {
    header('Location: index.php');
 // exit;
 // 如果沒有 Exit 的話就會繼續往下執行底下的程式，像是資料庫或靜態網頁之類的
}
Flag (?
```
所以我們直接盲猜 `admin.php`
```bash
curl http://chal.thjcc.org:1234/admin.php
```
可以看到
```html
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Admin Panel</title></head>
<body>
<p>Admin Panel</p>
<p><a href="status.php">Status page</a></p>
<p><a href="image.php">Image</a></p>
<p><a href="system.php">Setting</a></p>
</body>
```
繼續往 `system.php` 送的話就有 Flag 了
```html
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Admin Panel</title></head>
<body>
<p>System settings</p>
<p>^THJCC\{[0-9a-f]{16}_U_kNoW-HOw-t0_uSe-EaR\}$</p>
</body>
</html
```

## Happy Cat Jail
### 題敘
```
meow ?
```
題目有給一點點原始碼
```go
package main

import (
	"fmt"
	"unsafe"
)
……
```
裡面提到 `unsafe` 這個看起來很危險的東西，從 [Go 官網](https://pkg.go.dev/unsafe) 可以看到他的介紹，~~應該蠻明顯這東西有空指針了吧~~，如果要直接硬炸的話可以用  
```go
type catInterface struct {
	t uintptr
	v unsafe.Pointer
}

p := unsafe.Pointer(&target)

iface := (*catInterface)(p)

catStr := (*string)(iface.v)

fmt.Println(*catStr)
```
如果你通靈到 `secret` 這東西還有一個更短的 Payload - Found By 燒餅
```go
fmt.Println(target.(*secret).flag)
_ = unsafe.Sizeof(0)
```
以上兩個都可以得到 Flag  
`^THJCC\{iT'[1-9][a-z][1-9][a-zA-Z][1-9]Sm[1-9]Y_gO[a-zA-Z]!!!!!L[a-z]AnG\}$`  
  
btw 我從去年 HITCON ExCTF 結束的時候就有 Go jail 這個想法了，到 2 月初才把程式寫出來，原本打算放白箱的但~~拿去給很多 AI 試都可以 One shot~~ 所以就只留 unsafe 這個提示。~~下一次 NHNC 可能會有 Perl jail 還是 Java jail 之類奇怪的 jail 出現~~

## IMAGE? (Misc)
### 題敘
```
Check the hex of this image
```
![](/posts/images/THJCC-CTF-2026/IMAGE_Q.png)
如果用 [Hexed](https://hexed.it/) 這種 Hex 編輯器來看的話，可以發現 [PNG IEND](https://www.libpng.org/pub/png/spec/1.2/PNG-Chunks.html) 後面還有其他內容，後面出現了壓縮檔裡面的檔名，至於爲什麼知道是壓縮檔，~~自己去學吧反正這題的考點只有 PNG Chunk 而已~~，直接 unzip
```bash
[frank@Laptop ~]$ unzip IMAGE_Q.zip
Archive:  IMAGE_Q.zip
warning [IMAGE_Q.zip]:  3297649 extra bytes at beginning or within zipfile
  (attempting to process anyway)
   creating: cute/
  inflating: cute/F.png
  inflating: cute/F3.png
```
可以看到裡面有兩個檔案，點開 `F3.png` 就有 Flag 了
`THJCC{fRierEN-SO_cUTe:)}`

## TV (Forensics)
### 題敘
```
Amazing Sound
```

[檔案](https://github.com/Frank-Kam/THJCC-CTF-2026/tree/main/Forensics/TV)
其實這蠻明顯是 SSTV 的，GPT 照題目名稱和檔案類型也直接猜中，編碼是 Martin M1，使用 [Robot36](https://play.google.com/store/apps/details?id=xdsopl.robot36&hl=zh_TW) 可以成功解碼。
![](/posts/images/THJCC-CTF-2026/SSTV.png)

## Metro (Misc)

### 題敘
```
I took this photo at a MRT station in a certain city/county in Taiwan. Please identify which station it is and which floor it was taken on.

Flag format: THJCC{Station Code-Floor} (Case insensitive)

Example: If the station code for Taipei Metro's Shuanglian Station is R12, and the floor is the 1st floor (using American English numbering), the flag would be THJCC{R12-1F}
```

![](/posts/images/THJCC-CTF-2026/Metro.jpg)  
把圖片拉到過曝可以發現後面的第二月臺的紫色貼紙，臺灣使用紫色系的捷運系統只有桃園捷運了，在 OSINT 照片的時候如果有 Ubike 在附近會非常好用，剛好這張照片有拍到，我們可以去 [Ubike 站點地圖](https://www.youbike.com.tw/region/main/stations/) 尋找 `」` 這種路口和捷運站的相對位置一樣的站，~~桃園捷運才幾站一下找就有了~~
![](/posts/images/THJCC-CTF-2026/Metro-2.png)  
樓層的部分[桃園捷運官網](https://www.tymetro.com.tw/tymetro-new/upload/station/20250507084749_1.png)有。  
`THJCC{A10-3F}`

## YRSK (Misc)
### 題敘
```
Notice RIFF ChunkSize and size limits
Flag format: Case Insensitive, no spaces
```
[檔案](https://drive.proton.me/urls/6HMDMBCTY0#SsqH9Fv6WhMa) (稍大)
這題其實和 IMAGE 一樣都在考 Chunk，但 WAV 用的是比較通用的 RIFF 模式，[RIFF (WAV) Chunk](https://www.recordingblogs.com/wiki/list-chunk-of-a-wave-file)，一樣用 Hex 編輯器可以看到 data 之後多了許多奇怪的 Chunk，用 binwalk 之類的分析一下應該可以發現有 MP3 的蹤跡。  
至於 MP3 怎麽藏的和解法就看 PoC
```python
import struct

with open("out.wav", "rb") as f:
    data = f.read()

offset = data.find(b"FLAG")
if offset == -1:
    raise Exception("Err")

size = struct.unpack("<I", data[offset+4:offset+8])[0]
payload = data[offset+8:offset+8+size]

with open("extract.mp3", "wb") as f:
    f.write(payload)

print("Done")
```
播放輸出出來的 MP3 就可以聽到 Flag 了  
`THJCC{YRSKITSWONDERFULL0L}`  
  
btw 外面那層的音樂其實是電子專輯，音質非常好歐。

## ExBaby Shark Master (Forensics)
### 題敘
```
Just Search  
```
[檔案](https://github.com/Frank-Kam/THJCC-CTF-2026/tree/main/Forensics/ExBaby-Shark-Master)
這題蠻水的
```
strings THJCC_ExBaby-Shark-Master.pcapng | grep THJCC{
```
`THJCC{1t'S-3Asy*-r1gh7?????}`  
  
~~有點後悔出這題~~

## Chat
### 題敘
```
這是新來的 THJCC 客服機器人，但他好像有一些「祕密」……  
This is the new THJCC customer service bot... but it seems like he’s hiding a little "secret"  
```
[提示詞](https://github.com/Frank-Kam/THJCC-CTF-2026/tree/main/AI/Chat) (使用 GPT 5.2 Auto 模型)
這題其實蠻吃運氣的，只能看我們工人解的[參考](https://discord.com/channels/1214908408239620156/1214945609639923792/1475126588809941175)
  
## 總結
我覺得這次當工人的氛圍蠻愉快的，沒有遇到什麼會爭執的事情，~~靶機也沒有像 NHNC 的慘況~~  
感謝所有參賽者和工作人員。  
  
如果你有題目或比賽方面的問題可以透過 Footer 的連接聯絡我。
