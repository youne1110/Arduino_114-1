// 功能：
// 1) 預設藍色
// 2) 按鈕按 10 次 -> 黃色；按 20 次 -> 紅色
// 3) 超過 3 秒沒有按鍵，顏色會平滑漸變回藍色

const int R_PIN = 9;   // 使用 PWM 腳位
const int G_PIN = 10;  // 使用 PWM 腳位
const int B_PIN = 11;  // 使用 PWM 腳位
const int BTN_PIN = 12; // 按鈕接到 12，使用 INPUT_PULLUP

// 按鈕去抖與計數
int lastButtonState = HIGH;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

// 記錄按下次數與時間
int pressCount = 0;
unsigned long lastPressTime = 0; // 最近一次按下的時間 (millis)

// 超過這段時間沒按則開始漸變回藍色
const unsigned long idleToFadeMs = 3000; // 3 秒

// 當前顏色（0-255）
int curR = 0, curG = 0, curB = 255; // 初始為藍色

void setup() {
  pinMode(R_PIN, OUTPUT);
  pinMode(G_PIN, OUTPUT);
  pinMode(B_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP); // 使用內建上拉

  analogWrite(R_PIN, curR);
  analogWrite(G_PIN, curG);
  analogWrite(B_PIN, curB);

  Serial.begin(9600);
  Serial.println("Start - Default Blue");
}

void loop() {
  int reading = digitalRead(BTN_PIN);

  // 去抖處理（簡單版）
  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    // 穩定的狀態改變，檢查是否為按下瞬間 (HIGH -> LOW)
    static int stableState = HIGH;
    if (reading != stableState) {
      stableState = reading;
      if (stableState == LOW) {
        // 按下一次
        pressCount++;
        if (pressCount > 255) pressCount = 255; // 防止溢位（只是保險）
        lastPressTime = millis();
        Serial.print("Pressed: ");
        Serial.println(pressCount);
        // 立即根據 pressCount 設定目標色（會在下面應用）
      }
    }
  }

  lastButtonState = reading;

  // 根據 pressCount 決定目標顏色
  int targetR, targetG, targetB;
  if (pressCount >= 20) {
    // 紅色
    targetR = 255; targetG = 0;   targetB = 0;
  } else if (pressCount >= 10) {
    // 黃色 (R + G)
    targetR = 255; targetG = 255; targetB = 0;
  } else {
    // 預設藍色
    targetR = 0;   targetG = 0;   targetB = 255;
  }

  // 若 idle 超過閾值且目前不是藍色，開始漸變回藍色
  if (millis() - lastPressTime > idleToFadeMs && (curR != 0 || curG != 0 || curB != 255)) {
    // 平滑漸變到藍色 (0,0,255)
    fadeToColor(0, 0, 255);
    // 漸變完成後重置按鈕計數（回到預設藍色）
    pressCount = 0;
    Serial.println("Idle -> fade back to Blue. pressCount reset.");
  } else {
    // 非 idle 或尚未達到 3 秒，直接朝目標顏色移動 (小幅平滑過渡)
    // 這裡讓變化平滑（每 loop 移動一小步）
    smoothMoveTowards(targetR, targetG, targetB);
  }

  // 輸出實際 PWM
  analogWrite(R_PIN, curR);
  analogWrite(G_PIN, curG);
  analogWrite(B_PIN, curB);

  delay(20); // 小延遲，使平滑過渡更穩定
}

// 小幅向目標顏色移動（避免瞬間跳色）
void smoothMoveTowards(int tR, int tG, int tB) {
  const int step = 8; // 每次最大變化量，數字越小變化越平滑
  if (curR < tR) curR = min(curR + step, tR);
  else if (curR > tR) curR = max(curR - step, tR);

  if (curG < tG) curG = min(curG + step, tG);
  else if (curG > tG) curG = max(curG - step, tG);

  if (curB < tB) curB = min(curB + step, tB);
  else if (curB > tB) curB = max(curB - step, tB);
}

// 平滑完整漸變到指定顏色（阻塞式，確保視覺完整漸變）
void fadeToColor(int tR, int tG, int tB) {
  const int steps = 40;
  int startR = curR;
  int startG = curG;
  int startB = curB;

  for (int i = 1; i <= steps; i++) {
    float f = (float)i / steps;
    curR = startR + (int)((tR - startR) * f);
    curG = startG + (int)((tG - startG) * f);
    curB = startB + (int)((tB - startB) * f);

    analogWrite(R_PIN, curR);
    analogWrite(G_PIN, curG);
    analogWrite(B_PIN, curB);
    delay(20);
  }

  // 確保精準到目標值
  curR = tR; curG = tG; curB = tB;
  analogWrite(R_PIN, curR);
  analogWrite(G_PIN, curG);
  analogWrite(B_PIN, curB);
}
