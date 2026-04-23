```ino

//C114137102 蔡秉璁 職種技能實習(二) 期中考
#include <Wire.h>
#include <Adafruit_LiquidCrystal.h>
  
Adafruit_LiquidCrystal lcd(0);

void C_lcd(String text, int Setx, int Sety)
/*LCD輸出通用函數*/
{
  int len = text.length();     //讀取長度
  for(int i=0;i<len;i++)
  {
    lcd.setCursor(Setx+i,Sety);//設定顯示位置
    lcd.print(text[i]);        //顯示
    delay(100);
  }
}

void setup()
{
  lcd.begin(16,2);
  pinMode (A0,INPUT);  //亮度變化
  pinMode (10,INPUT);  //亮度-
  pinMode (9,INPUT);   //亮度+
  pinMode (8,INPUT);   //恢復自動
  for(int a=2;a<=6;a++)//5顆LED
    pinMode (a,OUTPUT);
  Serial.begin(9600);
  Serial.println("OK!");
  Serial.println("Please enter your password.");
  C_lcd("System Lock!",0,0);//系統鎖定！
}
//  t=是否由光敏控制 i=LED亮顆數 lock=是否已解鎖 q=密碼錯誤次數
int t = 1,i = 0,lock = 0,q=0;
String password = "123456"; //密碼
String pass = "";            //輸入密碼 完整
char r;                      //讀取密碼的第?個
void loop()
{
  if(lock== 1)
  {//密碼正確
    delay(100);
  	for(int a=2;a<=6;a++)
   	   digitalWrite(a, 0);    //關燈
  	for(int a=2;a<=i+1;a++)
  	   digitalWrite(a,1);     //依照 i 值開啟對應數量的 LED
  	if(digitalRead(8)== 1)    //開啟光敏控制
  	{
  	   Serial.println("Detection on!");
       t=1;
    }
    if(digitalRead(9)== 1)    //手動控制 LED+1
    {
       Serial.println("Detection off!");
       t=0;
       if(i<5)
        i++;
    }
    if(digitalRead(10)== 1)   //手動控制 LED-1
    {
       Serial.println("Detection off!");
       t=0;
       if(i>0)
         i--;
    }
    if (t == 1)
    {//根據不同亮度調整LED亮顆數
	   if(analogRead(A0) <= 344) i = 0;
	   else if(analogRead(A0) <= 400) i = 1;
	   else if(analogRead(A0) <= 600) i = 2;
	   else if(analogRead(A0) <= 800) i = 3;
	   else if(analogRead(A0) <= 1000) i = 4;
	   else i = 5;
    }
  }
  else
  {//密碼還沒正確
    for(int a=2;a<=6;a++)
    	digitalWrite(a, 0); //關燈
    while (1)
    {
      if(q==3)
      { //密碼錯三次
        digitalWrite(2,1);
        Serial.println("The system is locked！");
        lcd.clear();
        C_lcd("Fully locked",0,0);//完全鎖定
        while (1);//鎖定
      }
      if(Serial.available() >0)
      {//有收到數據
        char r = (char)Serial.read();//合併密碼
        if(r == '*')
        {//看到結尾
          if(pass == password)
          {//驗證密碼
            lock = 1;
            Serial.println("Password correct！");
            lcd.clear();
            C_lcd("Unlock",0,0);//開鎖
            break;
          }
          else
          {
            lock = 0;
            Serial.println("Incorrect password！");
            q++;//錯誤次數+1
            if(q==1)
               lcd.clear();
            C_lcd("Incorrect",0,0);//錯誤
            String T = String(q)+" time remaining";//合併字串and數次
            C_lcd(T,0,1);//錯誤?次
            for(int z=1;z<=3;z++)
            {//密碼錯誤閃逤(全部3次)
              for(int a=2;a<=6;a++)
                digitalWrite(a, 1);
              delay(500);
              for(int a=2;a<=6;a++)
                digitalWrite(a, 0);
              delay(500);
            }
          }
          pass = "";//讀取密碼清空
        }
        else
        {
          pass+= r;//+1讀取下一個
        }
      }
    }
  }
  
}
```