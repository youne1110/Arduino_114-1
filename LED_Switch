const int r = 7;
const int g = 8;
const int b = 9;
const int bt = 12;

int buttonState = HIGH;       // 當前按鈕狀態 (預設HIGH，因為用了PULLUP)
int lastButtonState = HIGH;   // 上一次的按鈕狀態
int c = 0;                    // 顏色模式
String color ;

void setup() {
  for (int i = 7; i < 10; i++) {
    pinMode(i, OUTPUT);
  }
  pinMode(bt, INPUT);   // 使用內建上拉電阻
  Serial.begin(9600);
}

void loop() {
  buttonState = digitalRead(bt);

  // 偵測「按下去的一瞬間」(HIGH → LOW)
  if (buttonState == LOW && lastButtonState == HIGH) {
    c += 1;
    if (c > 7) {
      c = 0;  // 循環回到 0
    }
    delay(50); // 去抖動
  }
  lastButtonState = buttonState;

  switch (c) {
    case 0: 
      digitalWrite(r, LOW);
      digitalWrite(g, HIGH);
      digitalWrite(b, HIGH);
      color = "RED";
      break;
    case 1: 
      digitalWrite(r, HIGH);
      digitalWrite(g, LOW);
      digitalWrite(b, HIGH);
      color = "GREEN";
      break;
    case 2:
      digitalWrite(r, HIGH);
      digitalWrite(g, HIGH);
      digitalWrite(b, LOW);
      color = "BLUE";
      break;
    case 3: 
      digitalWrite(r, LOW);
      digitalWrite(g, LOW);
      digitalWrite(b, HIGH);
      color = "YELLOW";
      break;
    case 4: 
      digitalWrite(r, HIGH);
      digitalWrite(g, LOW);
      digitalWrite(b, LOW);
      color = "CYAN";
      break;
    case 5:
      digitalWrite(r, LOW);
      digitalWrite(g, HIGH);
      digitalWrite(b, LOW);
      color = "PURPLE";
      break;
    case 6:
      digitalWrite(r, HIGH);
      digitalWrite(g, HIGH);
      digitalWrite(b, HIGH);
      color = "OFF";
      break;
    case 7:
      digitalWrite(r, LOW);
      digitalWrite(g, LOW);
      digitalWrite(b, LOW);
      color = "WHITE";
      break;
  }

  Serial.print("Current Color: ");
  Serial.println(color);
}
