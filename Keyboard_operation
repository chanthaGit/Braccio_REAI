#include <Braccio.h>
#include <Servo.h>
#include <string.h>

Servo base, shoulder, elbow, wrist_rot, wrist_ver, gripper;

const byte numChars = 64;

int stepdelay = 20;     // 10..30 ms
int stepInc   = 5;      // 1..10
int m1 = 0, m2 = 165, m3 = 0, m4 = 0, m5 = 0, m6 = 73;

char groupBuf[numChars];
char tempChars[numChars];
bool inGroup = false;
size_t groupIdx = 0;

//----------------- Utilities -----------------
int clampInt(int v, int lo, int hi) {
  if (v < lo) return lo;
  if (v > hi) return hi;
  return v;
}

void clampAll() {
  m1 = clampInt(m1, 0, 180);
  m2 = clampInt(m2, 15, 165);
  m3 = clampInt(m3, 0, 180);
  m4 = clampInt(m4, 0, 180);
  m5 = clampInt(m5, 0, 180);
  m6 = clampInt(m6, 10, 73);
  stepdelay = clampInt(stepdelay, 10, 30);
  stepInc   = clampInt(stepInc, 1, 10);
}

void printHelp() {
  Serial.println("===== Braccio Teleop (Immediate) =====");
  Serial.println("พิมพ์ตัวอักษรเพื่อสั่งทันที (ต้องใช้เทอร์มินัลที่ส่งอักขระทันที เช่น PuTTY/TeraTerm/CoolTerm)");
  Serial.println(" Base: q(+), a(-)");
  Serial.println(" Shoulder: w(+), s(-)");
  Serial.println(" Elbow: e(+), d(-)");
  Serial.println(" Wrist Vertical: r(+), f(-)");
  Serial.println(" Wrist Rotation: t(+), g(-)");
  Serial.println(" Gripper: y ปิด(+), h เปิด(-)");
  Serial.println(" ตั้งค่า: [1..9]=step, 0=step 10, '>' เพิ่ม stepdelay, '<' ลด stepdelay");
  Serial.println(" ชอร์ตคัท: o เปิดกริปเปอร์สุด(10), c ปิดกริปเปอร์สุด(73)");
  Serial.println(" พิมพ์ p เพื่อดูคู่มือนี้");
  Serial.println(" ป้อนค่าทั้งหมด: (m1,m2,m3,m4,m5,m6) เช่น (90,100,110,120,130,20)");
  Serial.println(" ข้อกำหนด: M2 15..165, M6 10..73, stepdelay 10..30");
  Serial.println("======================================");
}

void printStatus() {
  Serial.print("Step delay: "); Serial.print(stepdelay);
  Serial.print(" | Step inc: "); Serial.println(stepInc);
  //Serial.print("Base: "); Serial.print(m1);
  //Serial.print(" | Shoulder: "); Serial.print(m2);
  //Serial.print(" | Elbow: "); Serial.print(m3);
  //Serial.print(" | Wrist V: "); Serial.print(m4);
  //Serial.print(" | Wrist R: "); Serial.print(m5);
  //Serial.print(" | Gripper: "); Serial.println(m6);
}

void moveAndReport() {
  clampAll();
  Braccio.ServoMovement(stepdelay, m1, m2, m3, m4, m5, m6);
  //Serial.println("Robot moved");
  //printStatus();
}

//----------------- Command Handlers -----------------
void parseCSVBufferAndApply() {
  // groupBuf contains inside of parentheses e.g. "90,100,110,120,130,20"
  strncpy(tempChars, groupBuf, numChars);
  tempChars[numChars - 1] = '\0';

  char *token;
  token = strtok(tempChars, ","); if (token) m1 = atoi(token);
  token = strtok(NULL, ","); if (token) m2 = atoi(token);
  token = strtok(NULL, ","); if (token) m3 = atoi(token);
  token = strtok(NULL, ","); if (token) m4 = atoi(token);
  token = strtok(NULL, ","); if (token) m5 = atoi(token);
  token = strtok(NULL, ","); if (token) m6 = atoi(token);

  moveAndReport();
}

void applyKey(char c) {
  if (c >= 'A' && c <= 'Z') c = c - 'A' + 'a';

  switch (c) {
    case 'q': m1 += stepInc; moveAndReport(); break;
    case 'a': m1 -= stepInc; moveAndReport(); break;

    case 'w': m2 += stepInc; moveAndReport(); break;
    case 's': m2 -= stepInc; moveAndReport(); break;

    case 'e': m3 += stepInc; moveAndReport(); break;
    case 'd': m3 -= stepInc; moveAndReport(); break;

    case 'r': m4 += stepInc; moveAndReport(); break;
    case 'f': m4 -= stepInc; moveAndReport(); break;

    case 't': m5 += stepInc; moveAndReport(); break;
    case 'g': m5 -= stepInc; moveAndReport(); break;

    case 'y': m6 += stepInc; moveAndReport(); break;  // close
    case 'h': m6 -= stepInc; moveAndReport(); break;  // open

    case 'u': m6 = 10; moveAndReport(); break;        // open fully
    case 'j': m6 = 73; moveAndReport(); break;        // close fully

    case '>': stepdelay++; clampAll(); printStatus(); break;
    case '<': stepdelay--; clampAll(); printStatus(); break;

    case 'p': printHelp(); printStatus(); break;

    default:
      if (c >= '1' && c <= '9') {
        stepInc = c - '0';
        clampAll();
        printStatus();
      } else if (c == '0') {
        stepInc = 10;
        clampAll();
        printStatus();
      }
      break;
  }
}

//----------------- Immediate Serial Handler -----------------
void handleSerialImmediate() {
  while (Serial.available() > 0) {
    char c = Serial.read();

    // Ignore CR/LF entirely for immediate mode
    if (c == '\r' || c == '\n') continue;

    if (!inGroup) {
      if (c == '(') {
        inGroup = true;
        groupIdx = 0;
      } else {
        applyKey(c);  // executes immediately
      }
    } else {
      if (c == ')') {
        // end of group
        groupBuf[groupIdx] = '\0';
        inGroup = false;
        parseCSVBufferAndApply();
      } else {
        if (groupIdx < numChars - 1) {
          groupBuf[groupIdx++] = c;
        } // else: silently drop extra chars
      }
    }
  }
}

//----------------- Arduino -----------------
void setup() {
  Serial.begin(9600);
  while (!Serial) { ; }

  Serial.println("Teleop Braccio (ส่งทันทีต่ออักขระ)");
  Serial.println("- ใช้เทอร์มินัลที่ส่งอักขระทันที (PuTTY/TeraTerm/CoolTerm/screen/minicom)");
  Serial.println("- 9600 baud, 8N1, ไม่มี flow control");
  Serial.println("- คีย์ลัด: q/a, w/s, e/d, r/f, t/g, y/h, 1..9, 0, </>, u/j, p");
  Serial.println();

  Braccio.begin();
  Braccio.ServoMovement(10, 0, 165, 0, 0, 0, 73);

  printHelp();
  printStatus();
}

void loop() {
  handleSerialImmediate();
}
