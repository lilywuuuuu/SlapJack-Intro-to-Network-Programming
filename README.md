# 網路程式設計概論期末專題 — SlapJack


## 簡介
### 專題題目簡介

在 _SlapJack_ 遊戲中，一間房間能容納四位玩家，最多可同時支援四間房間。在遊戲開始時，每位玩家都會輪流進行翻牌的操作，當一位玩家翻開一張牌後，所有玩家就會看到目前的牌面和 Counter 數字。如果目前牌面的數字與 Counter 相符，最快擊牌的玩家將獲得一分。但如果有玩家在不對的牌面擊牌，則會被扣一分。首先累積三分的玩家將成為本輪的勝利者。這款遊戲老少閒宜，能訓練玩家的反應速度、專心能力、以及對數字的靈敏度。_SlapJack_ 不只提供玩家們一個有趣的互動平台，更能讓男女老少能夠一同參與並享受遊戲帶來的挑戰和樂趣。

### 成員分工

__吳承瑀__ 110550091: 負責撰寫 Client 端程式碼以及控制專題的進度和遊戲的發想。

__黃仁駿__ 111550125: 負責撰寫 Server 端程式碼以及遊戲整體架構的構想。

__游惠晴__ 111550101: 負責撰寫遊戲的 Graphic Design & User Interface 以及遊戲整體的美編。

### 開發與執行環境

兩位 Windows 使用者是在 Virtual Box 內的 Ubuntu 22.04 開發與執行，另一位 Mac 使用者則是在電腦本身的 Terminal 上開發與執行。

### 特殊需求

此遊戲需要在電腦上安裝 `ncurses library` 才可遊玩。


## 研究方法與設計：
### Server 與 Client 程式互動規則與資料傳輸格式

Server 與 Client 端溝通皆使用 "String\n" 格式，使用 TCP。

### Server

main 與 room 函式利用全域變數溝通，使用共用鎖避免衝突。

#### main 函式

* 負責變數與 listening socket 的初始化
* 利用 pthread 呼叫 room 函式建立遊戲房間
* 負責處理新使用者加入相關功能（包含記錄使用者註冊的名稱、分配使用者 ID、人數超過伺服器上限），可加入已開始遊戲中人數未滿的房間。
* 利用共用鎖避免與 room 函式間 socket file descriptor 傳遞相關的潛在衝突，僅在房間有空位的才等待房間該回合結束（釋出共用鎖）以加速客戶端進入房間的時間

#### room 函式

負責遊戲邏輯
* 持續檢查房間內是否已滿四人，開始遊戲
* 維護當回合的答案、翻出的牌和記分板
* 玩家翻牌、擊牌的時限都為三秒
* 每回合開始時向所有玩家傳輸當前的記分板（所有玩家名、ID、分數）
* 檢查使用者是否斷線與斷線後的資料處理（重置ID、名字、分數等）若人未滿利用全域變數通知 main 函式 
* 勝利時通知並移除所有使用者後重置房間，勝利條件包含達到目標分數或 $3/4$ 以上玩家斷線




### Client
#### 各函式功能說明
* `handle_alarm(int sig)`：用於計時中斷某些動作。
* `scoreboard(int score[5], int id[5], char name[5][15])`：顯示計分板。
* `card()、draw()、counter()、show_card()、flip_card()、before_flip()`：處理遊戲中的圖形界面，如牌的繪製和翻轉效果。
* `title()、welcomeframe()、gameover()、endframe()、endscoreboard()`：顯示遊戲的不同階段和界面，比如遊戲開始、結束、等待界面等。

#### 流程
1. 使用者輸入名稱和 IP 後與 Server 建立連線
2. 等待 Server 分配房間
3. 收到伺服器已滿則結束，否則進入等待畫面，等待未開始的房間滿四人或已開始的房間進入下一回合
4. 遊戲開始後，使用者按下 q 可隨時離開，每回合的開始更新計分板
5. 若輪到自己翻牌，則使用者須在三秒內按下任意鍵否則 Server 會強制翻牌
6. 有玩家翻牌後，進入翻牌動畫，畫面顯示當前的答案與撲克牌，玩家有三秒可擊牌，以收到 Server 傳送的牌相關訊息的時間為基準
7. 若遊戲結束則進入對應的結束畫面



## 成果：最後成果的主要功能與特色。

玩家在終端機輸入 `./finalcli <server_ip> <player_name>` 之後即可連線到伺服器。
1. 進入遊戲畫面後會收到歡迎的訊息以及自己的名字和 ID，每人都會有獨特的 ID 是為了預防有多數玩家取的 player name 相同，以至於分不清自己在記分板上是哪一位。畫面下方會看到伺服器是否已經將玩家分配到一個房間內，且目前房間內有幾位玩家。
![Screenshot 2024-01-08 at 10.47.46 AM](https://hackmd.io/_uploads/SkvIQJFda.png)

2. 若房間內集滿四位玩家，則遊戲即將開始。
![Screenshot 2024-01-08 at 10.48.08 AM](https://hackmd.io/_uploads/ryMuEyFOa.png)

3. 每位玩家都會輪流翻牌，輪到你翻牌時，畫面的左方會提醒你按任何一個鍵來翻牌。
![Screenshot 2024-01-08 at 10.48.19 AM](https://hackmd.io/_uploads/rkR9V1tOp.png)

4. 翻牌後每位玩家都會同時看到一張牌以及目前 Counter 的數字。Counter 會從 1~13 循環，牌則是由伺服器隨機分發。
![Screenshot 2024-01-08 at 10.48.32 AM](https://hackmd.io/_uploads/HkURVyKu6.png)

5. 若牌上的數字和 Counter 相等，則玩家須在三秒內擊牌，花費的時間會顯示在畫面下方，這個數字會被回傳給伺服器，再由伺服器判斷哪位玩家最快擊牌。最快者則加一分。若在錯誤的牌擊牌，則此玩家扣一分。
![Screenshot 2024-01-08 at 11.00.23 AM](https://hackmd.io/_uploads/BkFuHJYua.png)

6. 若想要在遊戲中離開，則隨時可以按 'q' 離開，當然按 'Ctrl + C'也有同樣的效果。若有玩家中離遊戲，則其他玩家的 Score Board 上會更新此資訊。中離玩家的 Score & ID 將被清零，Name 則會變成 -。房間若有空位，則下一位連至伺服器的玩家會優先彌補空缺，若沒有房間有空位，才會開啟一間新房間。
![Screenshot 2024-01-08 at 11.09.05 AM](https://hackmd.io/_uploads/HyWiDktu6.png)

7. 如果在遊戲進行中，有三位玩家退出，最後留在房間內的玩家則成為最終贏家。每位玩家及可以按任何一鍵退出遊戲。若想要遊玩下一局，再次輸入程式碼即可。
![Screenshot 2024-01-08 at 11.26.11 AM](https://hackmd.io/_uploads/H1dwjyFda.png)


8. 某位玩家獲得三分後即遊戲結束，贏家會用紅字顯示。每位玩家及可以按任何一鍵退出遊戲。若想要遊玩下一局，再次輸入程式碼即可。
![Screenshot 2024-01-08 at 10.49.09 AM](https://hackmd.io/_uploads/HkwIIJYuT.png)

9. 遊戲進行中的示範。
    
![ScreenRecording2024-01-08at11.30.57AM-ezgif.com-video-to-gif-converter (2)](https://hackmd.io/_uploads/HJYh1et_a.gif)


## 結論：
### 專題製作心得

在本次的報告中，透過組員之間的協調以及合作，不斷討論跟溝通以產出最終的成果，儘管我們的主題是已經存在的遊戲「心臟病」，但是將人與人面對面的即時感轉換成透過電腦的連線也能有所互動的過程，激發了我們的創意。

把遊戲的進行轉換成 Server 與 Client 之間的訊息交換也讓我們認知到流程規劃的重要性，實現的過程則是將老師一學期所教授的知識學以致用；使用者介面的部分， ncurses 對我們而言是一個新的東西，上網找資料以及最後在我們的報告中排版的整個歷程使我們對這個函式庫有更多的了解，才發現 C 語言也能做到如此精緻的使用者介面。

透過本次專題，結合所學知識以及創意，還有透過專題獲得的新知，讓我們得以做出最終的 _SlapJack_。

### 遭遇困難及解決經過

由於 Server 與 Client 的程式碼是由不同的人負責，完成專題的過程中遭遇的最大困境是彼此的訊息傳遞順序以及內容都要在正確的時間以正確的格式傳送，因此除了撰寫自己的程式碼之外，我們花了很多時間檢查以及溝通訊息傳送的內容。

延續上述的問題，因為兩方的訊息交流量很大，有出現 Client 傳遞的資訊並非 Server 所預期的格式導致錯誤發生，為尋找錯誤發生的地方，我們在 Server 端把所有收到的資訊輸出，藉此找到格式錯誤的部分。

另外是 UI 介面的部分，翻牌的動畫需使用 usleep() 讓肉眼能分辨撲克牌移動的過程，但同時也需要 wclear() 來清除視窗，否則撲克牌的移動軌跡會停留在視窗上面，但是 UI 介面除了翻牌動畫之外有一些其他的資訊是我們不需要在翻牌時清除的，像是計分板的內容，然而 wclear() 會清除整個視窗，導致計分板消失。因此透過開新的子視窗專門負責翻牌的動畫，並且只對子視窗使用 wclear()，如此便解決了更新畫面之後，部分介面消失的問題。

### 成果未來改進或延伸方向

這次的專題僅能同時開 4 間房間，每間 4 位玩家，也就是最多 16 人遊玩，未來如果有機會的話可以試著提高房間數量以及每個房間的人數上限，讓更多人可以同時玩這個遊戲。也期望不論是繼續利用 ncurses 或是改用另一種螢幕控制的方式 ANSI Escape Sequence 來美化現有的 UI 介面，提升玩家的遊玩意願。最後是聊天室的功能，有鑑於每輪翻牌之間的間隔時間大約 3~5 秒，可以增加發送表情符號的功能增加玩家跟玩家之間的互動以及趣味性。


## 參考文獻與附錄：
### 參考文獻列表

1. ncurses 安裝教學：https://linux.cn/article-9693-1.html
2. Ascii 動畫：https://github.com/sahilgajjar/ascii_art
