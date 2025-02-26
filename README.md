# fpga_project
## Authors: 110321002 何雨叡 110321005 邱暘壹 110321029 黃柏源
![image](https://github.com/user-attachments/assets/060beaa3-294c-4752-9493-f644d03a9307)
## DEMO 影片
https://drive.google.com/file/d/1Mm9-EfjWEZ9URJ82ZRNmmbz4O2RvGrgl/view?usp=drive_link
### Input/Output unit:

* 8*8 Led 矩陣顯示遊戲畫面
* 7段顯示器顯示分數
* LED 陣列顯示生命

### 功能說明
打磚塊，下面的板子左右移動反彈球去打市面的方塊，第一關打到一個加一分，第二關打到一個扣一分，直到分數變成0，顯示結束畫面，生命規0顯示失敗畫面。


### 程式模組說明: 

divfreq() 除頻
reg [3:0] plat_pos = 3'b010;//平台最左那科的位置
reg last_status_d1 = 1'b0, last_status_d0 = 1'b0,last_status_d4 = 1'b0;//存上一次的輸入
reg start = 1'b0;//開始
reg [10:0] time_pass =11'b0;//過幾秒做一個動作
reg [7:0] ball_y = 8'b00000010; // 球y軸
reg [15:0] block = 16'b0000000000000100;//遊戲起始磚塊
reg up = 1;//看球上網上還是往下
reg [2:0] ball_status;//往左往右或垂直
reg [1:0] remain_life = 3;// 生命
reg [3:0] score = 4'b0; //分數
reg stage_1 = 1;  //第一關

