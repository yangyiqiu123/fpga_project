module divfreq(input clk, output reg clk_div);
	reg [24:0] Count;
	reg [24:0] num;
	always @(posedge clk)
		begin
			if(Count > 3000)		
				begin
					
					Count <= 25'b0;
					clk_div <= ~clk_div;
				end
			else Count <= Count + 1'b1;
		end
endmodule
//除頻器



module fianl_block(

	input [0:5] D,//0又1左2重製3是發球4復活5暫停
	input speed, CLK,

	output reg [0:27] LED,//0到7是紅色8到15是綠色16到23是藍色
	output reg [0:3] life,//寫條
	output reg beep,//撞到就教遺下
	output reg [0:6] lcd//得分
);

 reg [3:0] Count;
 reg [3:0] plat_1= 3'b000, plat_2=3'b001, plat_3=3'b010, ball_x=3'b001;//PLAT
 
 

 reg [3:0] kplat_1 = 3'b000, kplat_2 = 3'b001, kplat_3 = 3'b010;
 reg [3:0] plat_pos = 3'b010;//平台最左那科的位置
 reg last_status_d1 = 1'b0, last_status_d0 = 1'b0,last_status_d4 = 1'b0;//存上一次的輸入
 reg start = 1'b0;//d3控制
 reg [10:0] time_pass =11'b0;//過幾秒做一個動作
 
 
 
 reg [7:0] ball_y = 8'b00000010;
 reg [15:0] block = 16'b0000000000000100;//遊戲起始磚塊
 reg up = 1;//看球上網上還是往下
 reg [2:0] ball_status;//往左往右或垂直
 //reg [1:0] remain_life = 3;
 
 reg [1:0] remain_life = 2;
 
 reg [3:0] score = 4'b0;
 reg stage_1 = 1;//=1第一關還沒完
 reg a=0, b=0, c=0, d=0;
 
 
 divfreq (CLK,CLK_div);
 

	
always @(posedge CLK_div)
 begin

	//除頻部分
	
	if(speed)  //球的速度
	begin
		time_pass <= time_pass + 1'b1;
	end
	else
	begin
		time_pass <= time_pass + 2'b11;
	end
	


	if(Count<8)
		Count <= Count + 1'b1;
			
	else
		Count <= 3'b000;

//顯示一開始的關卡
//show
	 LED[24:26] = Count;

	 if(stage_1 == 0 && score == 0)//stage_1==0第一關已經過了 並且把錢一局的分數打完(通關)
	 begin//通關畫面
		if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[16:23] <= 8'b11111111;
		if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[0:7] <= 8'b11111111;
		if(Count == 0) LED[8:15]<=8'b11110001;
		else if(Count == 1) LED[8:15]<=8'b11110101;
		else if(Count == 2) LED[8:15]<=8'b00000100;
		else if(Count == 3) LED[8:15]<=8'b00011110;
		else if(Count == 4|| Count == 5|| Count == 6) LED[8:15]<=8'b11011110;
		else if(Count == 7) LED[8:15]<=8'b11000000;
		beep <= 0;
	 end
	 
	 else if(remain_life!=0 || ball_y!=8'b00000000)
	 begin
		if(Count==plat_3 || Count==plat_2 ||Count==plat_1) LED[0:7]<=8'b11111110;	//板子發亮的原理
		else LED[0:7] = 8'b11111111;
	 
		if (Count==ball_x) LED[8:15]<=~ball_y;//求發亮的原理
		else LED[8:15] = 8'b11111111;
	 
	 
		if(block[Count]==1 && block[Count+8]==1) LED[16:23] <= 8'b00111111;//磚塊發亮的原理
		else if(block[Count]==1 && block[Count+8]==0) LED[16:23] <= 8'b01111111;
		else if(block[Count]==0 && block[Count+8]==0) LED[16:23] <= 8'b11111111;
		else if(block[Count]==0 && block[Count+8]==1) LED[16:23] <= 8'b10111111;
	 end
	 
	 else
	 begin//死亡畫面
	 if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[16:23] <= 8'b11111111;
		if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[0:7] <= 8'b11111111;
		if(Count == 0) LED[8:15]<=8'b11110001;
		else if(Count == 1) LED[8:15]<=8'b11110101;
		else if(Count == 2) LED[8:15]<=8'b00000100;
		else if(Count == 3) LED[8:15]<=8'b00011110;
		else if(Count == 4|| Count == 5|| Count == 6) LED[8:15]<=8'b11011110;
		else if(Count == 7) LED[8:15]<=8'b11000000;
	 /*
		if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[16:23] <= 8'b11111111;
		if(Count == 0||Count == 1||Count == 2||Count == 3||Count == 4|| Count == 5|| Count == 6 ||Count == 7)LED[0:7] <= 8'b11111111;
		if(Count==0 || Count==7) LED[0:7] = 8'b11111111;
		else if(Count==1) LED[0:7] = 8'b00000000;
		else if(Count==2 || Count==3) LED[0:7] = 8'b01111110;
		else if(Count==4) LED[0:7] = 8'b01110010;
		else if(Count==5) LED[0:7] = 8'b01110110;
		else if(Count==6) LED[0:7] = 8'b00110000;
	 */
	 end
//7段顯示器得分的部分
	d <= score[0];
	c <= score[1];
	b <= score[2];
	a <= score[3];
 
 //update status
	 //設定板子
	 plat_1 <= kplat_1 + plat_pos;
	 plat_2 <= kplat_2 + plat_pos;
	 plat_3 <= kplat_3 + plat_pos;
	
	 if(start==0)//start是s4/d3發射球
	 begin
		ball_x <= plat_2;//把球放在中間
		beep <= 0;
	 end
	 
	 //restart重新開始
	 if(D[2]==1'b1)//3滴血遊戲重來
	 begin
	 
		plat_pos <= 3'b010;//板子回到四的位置
		ball_y <= 8'b00000010;//ball由下屬上來兩隔
		start <= 0;
		block <= 16'b0000000000000100;//重製方塊的位置 看遊戲起始面把16拆成8 8 比較好看
		up <= 1;
		remain_life <= 2;
		beep <= 0;
		score <= 4'b0;
		stage_1 = 1;
		
	 end	
	 
	 //Life血量
	 if(remain_life==2'b11) life = 4'b1110;
	 else if(remain_life==2'b10) life = 4'b1100;
	 else if(remain_life==2'b01) life = 4'b1000;
	 else life = 4'b0000;
	 
	 
	 if(D[5]==0)//D(5)放在迴圈最外面 因為是暫停鍵 非常不確定
	 begin
//復活
		if(D[4]==1 && last_status_d4==0 && remain_life!=0)//上一下1這一下0 才會往重生 (10)
		begin
			start <= 0;
			ball_x <= plat_2;//求應概要在平台2的位置上面推測plat應該是給求定位用的
			ball_y <= 8'b00000010;
			remain_life <= remain_life - 1;
		end

		//平台左右移動
		//plat right
		else if(D[1]==0 && last_status_d1==1 && plat_pos<5) plat_pos <= plat_pos + 1'b1;//上一下1這一下0 才會往又 (10)
		//plat left
		else if(D[0]==0 && last_status_d0==1 && plat_pos>0) plat_pos <= plat_pos - 1'b1;//上一下1這一下0 才會往左 (10)
		//start game
		else if(D[3]==1) start<=1;
		
		
	 
		last_status_d1 <= D[1];
		last_status_d0 <= D[0];
		last_status_d4 <= D[4];
	  

	 //ball status
		if(start==1 && time_pass==11'b11111111111)
		
		begin
			beep <= 0;
		//ball raising
			if(up==1)//球往上 
			begin
				if(ball_x == plat_1 && ball_y == 8'b00000010) ball_status=2'b0;//打到板子左邊
				else if(ball_x == plat_2 && ball_y == 8'b00000010) ball_status=2'b01;//中間
				else if(ball_x == plat_3 && ball_y == 8'b00000010) ball_status=2'b10;//右邊
				
				if(ball_status==0)//打到板子左邊 
				begin
					if(ball_x == 3'b001 || ball_y == 8'b10000000) ball_status = 2'b10;//如果求在最左二狀到板子要往又飛
					ball_x <= ball_x-1;
					ball_y <= ball_y*2;
				end
			
				else if(ball_status==1)//求垂直往上
				begin
					ball_y <= ball_y*2;
				end
			
				else if(ball_status==2)//如果求在最又二邊狀到板子要往左飛
				begin
					if(ball_x==3'b110 || ball_y==8'b10000000) ball_status = 2'b0;
					ball_x <= ball_x+1;
					ball_y <= ball_y*2;
				end
				else if(ball_x==3'b000 && ball_x==plat_1 && ball_y==8'b00000010)
				begin
					ball_x <= ball_x+1;
					ball_y <= ball_y*2;
				end
				else if(ball_x==3'b111 && ball_x==plat_3 && ball_y==8'b00000010)
				begin
					ball_x <= ball_x-1;
					ball_y <= ball_y*2;
				end
			end
		//ball falling
			else 
			begin
		
				if(ball_status==0)//球往左調 
				begin
					if(ball_x==3'b001) ball_status = 2'b10;
					ball_x <= ball_x-1;
					ball_y <= ball_y/2;
				end
			
				else if(ball_status==1)//球垂直調 
				ball_y <= ball_y/2;
				
				else if(ball_status==2)//球往又調 
				begin
					if(ball_x==3'b110) ball_status = 2'b0;
					ball_x <= ball_x+1;
					ball_y <= ball_y/2;
				end
			
			end		
			time_pass <= 11'b0;
		end
	end
	
	 //hit detect
	if(ball_y==8'b01000000 && block[ball_x+8]==1)
	begin//球撞到第二排的方塊
		block[ball_x+8]<=0;
		up <= 0;
		beep <= 1;
		if(stage_1 == 1) score <= score + 1'b1;
		else score <= score - 1'b1;
	end
	else if(ball_y==8'b10000000 && block[ball_x]==1)
	begin//球撞到最上面的方塊
		block[ball_x]<=0;
		up <= 0;
		beep <= 1;
		if(stage_1 == 1) score <= score + 1'b1;
		else score <= score - 1'b1;
	end
	else if(ball_y==8'b00000010 && (plat_1==ball_x || plat_2==ball_x || plat_3 == ball_x))
	begin//球有撞到板子
		up <= 1;
		if(start==1) beep <= 1;
	end
	else if(ball_y==8'b10000000)
	begin//球碰到最上面
		up<=0;
		beep <= 1;
	end;
	 
	if(block == 16'b0 && stage_1 == 1)
	begin//第二關的開始
		ball_y <= 8'b00000010;
		start <= 0;
		block <= 16'b0101010010101000;
		up <= 1;
		beep <= 0;
		stage_1 = 0;
	end
	 
	lcd[0] = ~((~b & ~c & ~d)|(a & ~b & ~c)|(~a & b & d)|(~a & c));
   lcd[1] = ~((~a & ~b)|(~b & ~c)|(~a & ~c & ~d)|(~a & c & d));
	lcd[2] = ~((~a & b)|(~b & ~c)|(~a & d));
	lcd[3] = ~((a & ~b & ~c)|(~a & ~b & c)|(~a & c & ~d)|(~a & b & ~c & d)|(~b & ~c & ~d));
	lcd[4] = ~((~b & ~c & ~d)|(~a & c & ~d));
	lcd[5] = ~((~a & b & ~c)|(~a & b & ~d)|(a & ~b & ~c)|(~b & ~c & ~d));
   lcd[6] = ~((a & ~b & ~c )|(~a & ~b & c)|(~a & b & ~c)|(~a & c & ~d));
	 

 end
 endmodule
