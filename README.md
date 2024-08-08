## 制作sim打电话工具
***

> ## *硬件材料*

| 组件       | 详细说明            |
|------------|---------------------|
| 单片机     | Arduino Uno         |
| GSM 模块   | SIM800L 模块        |
| 按键模块   | 4x4 矩阵键盘        |
| 屏幕       | SSD1306 OLED 屏幕   |
| 电源模块   | 5v 稳定供电电源     |
| 天线       | 5dBi GSM Antenna    |
| 麦克风模块 | MAX9814             |
| 扬声器模块 | 1W Mini Speaker     |


> ## *硬件连接*

### Arduino Uno 与 SIM800L 模块连接

| SIM800L 引脚 | Arduino Uno 引脚      | 说明              |
|--------------|------------------------|-------------------|
| TXD          | RX (Pin 2)             | 数据发送          |
| RXD          | TX (Pin 3)             | 数据接收          |
| GND          | GND                    | 地线              |
| VCC          | 5V 稳定供电电源        | 电源              |

### Arduino Uno 与 4x4 矩阵键盘连接

| 4x4 矩阵键盘 引脚 | Arduino Uno 引脚   | 说明       |
|-------------------|---------------------|------------|
| 行引脚（R1-R4）   | 数字引脚（8-11）    | 行信号     |
| 列引脚（C1-C4）   | 数字引脚（4-7）     | 列信号     |

### Arduino Uno 与 SSD1306 OLED 屏幕连接

| OLED 引脚 | Arduino Uno 引脚 | 说明           |
|-----------|--------------------|----------------|
| VCC       | 5V                 | 电源           |
| GND       | GND                | 地线           |
| SCL       | A5 (SCL)           | 时钟信号       |
| SDA       | A4 (SDA)           | 数据信号       |

### Arduino Uno 与 MAX9814 麦克风模块连接

| MAX9814 引脚 | Arduino Uno 引脚 | 说明           |
|--------------|--------------------|----------------|
| VCC          | 5V                 | 电源           |
| GND          | GND                | 地线           |
| OUT          | 模拟输入引脚 A0    | 声音信号       |

### Arduino Uno 与 1W Mini Speaker 扬声器连接

| 扬声器引脚 | Arduino Uno 引脚 | 说明           |
|------------|--------------------|----------------|
| 正极       | D9 (PWM 输出)      | 音频输出       |
| 负极       | GND                | 地线           |

### SIM800L 模块与 5dBi GSM Antenna 天线连接

| SIM800L 模块 引脚 | 天线              | 说明             |
|-------------------|-------------------|------------------|
| 天线接口          | 配套 GSM 模块的标准天线 | 天线连接         |


> ## *示例代码（Arduino 与 SIM800L）*

```cpp
#include <SoftwareSerial.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Keypad.h>

// OLED 屏幕设置
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// GSM 模块串口
SoftwareSerial gsmSerial(2, 3); // RX, TX

// 4x4 矩阵键盘配置
const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};
byte rowPins[ROWS] = {8, 9, 10, 11}; // 行引脚
byte colPins[COLS] = {4, 5, 6, 7};   // 列引脚
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

void setup() {
  // 启动串口通信
  Serial.begin(9600);
  gsmSerial.begin(9600);

  // 初始化 OLED 屏幕
  if (!display.begin(SSD1306_I2C_ADDRESS, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for (;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.println(F("Welcome!"));
  display.display();

  // 初始化 GSM 模块
  delay(1000);
  gsmSerial.println("AT"); // 检查模块是否响应
  delay(1000);
  gsmSerial.println("AT+CMGF=1"); // 设置 GSM 模块为文本模式
  delay(1000);
}

void loop() {
  // 读取键盘输入
  char key = keypad.getKey();
  
  if (key) {
    display.clearDisplay();
    display.setCursor(0,0);
    display.print(F("Dialing: "));
    display.print(key);
    display.display();

    if (key == '#') {
      String phoneNumber = "+1234567890"; // 替换为实际的电话号码
      gsmSerial.print("ATD");
      gsmSerial.print(phoneNumber);
      gsmSerial.println(";"); // 添加分号以指示拨号结束
      display.clearDisplay();
      display.setCursor(0,0);
      display.println(F("Dialing..."));
      display.display();
    }
  }

  // 读取 GSM 模块的响应并将其输出到串口监视器
  if (gsmSerial.available()) {
    Serial.write(gsmSerial.read());
  }
}

```
> ## *操作步骤*
### 代码解释：
- **初始化**：初始化串口通信、GSM 模块和 OLED 屏幕
- **键盘输入处理**：读取键盘的输入并显示在 OLED 屏幕上。当按下 # 键时，使用 GSM 模块拨打电话号码
- **拨号和显示**：通过 GSM 模块发送 AT 指令拨打电话，并在 OLED 屏幕上显示拨号状态
