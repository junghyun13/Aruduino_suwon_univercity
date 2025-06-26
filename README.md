# Aruduino_suwon_univercity
정보통신학과 로봇경진대회 아두이노 자율주행RC카 코드
자세한 설명: https://velog.io/@junghyun13/%EC%88%98%EC%9B%90%EB%8C%80%EB%A1%9C%EB%B4%87%EA%B2%BD%EC%A7%84%EB%8C%80%ED%9A%8C
#include <IRremote.h>


int RightMotor_E_pin = 5;	// 오른쪽 모터의 Enable & PWM
int LeftMotor_E_pin = 6;   // 왼쪽 모터의 Enable & PWM
int RightMotor_1_pin = 8;  // 오른쪽 모터 제어
int RightMotor_2_pin = 9;  // 오른쪽 모터 제어
int LeftMotor_3_pin = 10;  // 왼쪽 모터 제어
int LeftMotor_4_pin = 11;  // 왼쪽 모터 제어
int Remote_pin = 3;// 적외선 수신센서 핀
int triggerPin=13;
int echoPin=12;

int L_Linesen = A5;       // 왼쪽 라인트레이서 센서
int C_Linesen = A4;       // 가운데 라인트레이서 센서
int R_Linesen = A3;       // 오른쪽 라인트레이서 센서

int L_MotorSpeed = 230;  // 왼쪽 모터 속도
int R_MotorSpeed = 230;  //153 오른쪽 모터 속도

int SL = 1;
int SC = 1;
int SR = 1;
int m=1;
int state=1;
unsigned long before=millis();
              
IRrecv irrecv(Remote_pin);        // 적외선 송수신 통신을 위한 객체
decode_results IR_Signal;

void motor_role(int R_motor, int L_motor, int R_MotorSpeed, int L_MotorSpeed) {
  digitalWrite(RightMotor_1_pin, R_motor);
  digitalWrite(RightMotor_2_pin, !R_motor);
  digitalWrite(LeftMotor_3_pin, L_motor);
  digitalWrite(LeftMotor_4_pin, !L_motor);

  analogWrite(RightMotor_E_pin, R_MotorSpeed);  // 우측 모터 속도값
  analogWrite(LeftMotor_E_pin, L_MotorSpeed);   // 좌측 모터 속도값
  irrecv.enableIRIn();
}



long readTravelTime(int triggerPin, int echoPin){
  long distance, duration;
  pinMode(triggerPin, OUTPUT);
  digitalWrite(triggerPin, LOW);
  delayMicroseconds(2);
  digitalWrite(triggerPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerPin, LOW);
  pinMode(echoPin, INPUT);
  duration=pulseIn(echoPin, HIGH);
  return duration;
}

void linetracer(int SL, int SC, int SR){
  int L = digitalRead(L_Linesen);
  int C = digitalRead(C_Linesen);
  int R = digitalRead(R_Linesen);
  if ( L == LOW && C == LOW && R == LOW ) {           // 0 0 0
    L = SL; C = SC; R = SR;
  }

  if ( L == LOW && C == HIGH && R == LOW ) // 1 0 1
  {
   motor_role(HIGH, HIGH, R_MotorSpeed, L_MotorSpeed);
   Serial.println("직진");
  }
  
  else if (L == LOW && R == HIGH )	// 0 0 1, 0 1 1
  {
   motor_role(LOW, HIGH, R_MotorSpeed, L_MotorSpeed);
   Serial.println("우회전");
  }
 
  else if (L == HIGH && R == LOW ) // 1 1 0, 1 0 0
  {
   motor_role(HIGH, LOW, R_MotorSpeed, L_MotorSpeed);
   Serial.println("좌회전");
  }
  SL = L; SC = C; SR = R;
}

void setup() {
  pinMode(RightMotor_E_pin, OUTPUT);
  pinMode(LeftMotor_E_pin, OUTPUT);
  pinMode(RightMotor_1_pin, OUTPUT);
  pinMode(RightMotor_2_pin, OUTPUT);
  pinMode(LeftMotor_3_pin, OUTPUT);
  pinMode(LeftMotor_4_pin, OUTPUT);

  Serial.begin(9600);  // PC와의 시리얼 통신 9600bps로 설정
  Serial.println("start Serial");
}
void loop() {
  //motor_role(HIGH, HIGH, R_MotorSpeed, L_MotorSpeed);
  float cm;
  unsigned long now=millis();
  while(now-before<=100){
    linetracer(SL, SC, SR);
    now=millis();
  }
  do{
    cm=0.01723*readTravelTime( triggerPin,  echoPin);
    Serial.println(cm);
    before=millis();
    if(cm<=10)
      motor_role(HIGH, HIGH, 0, 0);
  }while(cm<=10);
   /*else if ( L == HIGH && C == LOW && R == HIGH ) {                // 1 1 1, 1 0 1
    analogWrite(RightMotor_E_pin, 0);
    analogWrite(LeftMotor_E_pin, 0);
    Serial.println("정지");
  }*/	//진행한 센서신호 저장*/
}

/* IR remote 이름과 주소 
 *  이름,  16진수 주소 , 10진수 주소
  { "0",    0xFF6897 , 16738455 } 
  { "1",    0xFF30CF , 16724175 }
  { "2",    0xFF18E7 , 16718055 }
  { "3",    0xFF7A85 , 16743045 }
  { "4",    0xFF10EF , 16716015 }
  { "5",    0xFF38C7 , 16726215 }
  { "6",    0xFF5AA5 , 16734885 }
  { "7",    0xFF42BD , 16728765 }
  { "8",    0xFF4AB5 , 16730805 }
  { "9",    0xFF52AD , 16732845 }
  { "100+", 0xFF9867 , 16750695 }
  { "200+", 0xFFB04F , 16756815 }
  { "-",    0xFFE01F , 16769055 }
  { "+",    0xFFA857 , 16754775 }
  { "EQ",   0xFF906F , 16748655 }
  { "<<",   0xFF22DD , 16720605 }
  { ">>",   0xFF02FD , 16712445 }
  { ">|",   0xFFC23D , 16761405 }
  { "CH-",  0xFFA25D , 16753245 }
  { "CH",   0xFF629D , 16736925 }
  { "CH+",  0xFFE21D , 16769565 }
*/


