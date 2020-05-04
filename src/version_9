// Термостат для поддержания температуры в камере ферментации
// v.9
// Рабочая стабильная прошивка
// Обновлено меню
// Схема интегрируется в схему кнопочной станции!
// Управление реле для вентилятора температуры в двухпозиционном режиме дублируя цепи управления кнопочной станции.
// Управление реле для вентилятора кислорода в двухпозиционном режиме через магнитный пускатель, только в автоматическом режиме, ручного режима нет.
// Сделано запоминание состояния автоматического режима и установок температуры и настроек при выключении питания
// Если включен автоматический режим, то ручной выключен. Схема ручного управления полностью шунтирована.
// Добавлен wathdog. Пришлось прошить бутлоадер от UNO !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
// Релейные модули arduino включаются включаются низким уровнем !!!
// Добавить: запрет работы при срабатывании теплового реле.
//
// Коды символов диспреля 1602 http://arduino.ru/forum/pesochnitsa-razdel-dlya-novichkov/pytayus-sodat-menyu#comment-531165

#define CLK 2 // энкодер
#define DT 3 // энкодер
#define SW 4 // энкодер
#define HL_VENT_PIN 9     // Индикация состояния режима работы вентилятора -> кислород
#define KM2NO_VENT_PIN 10 // пин реле для включения цепи магнитного пускателя вентилятора кислорода
#define KM1NO_AUTO_PIN 11 // пин реле для включения цепи пускателя вентилятора отопления
#define KM1NC_AUTO_PIN 8 // пин реле для выключения цепи пускателя вентилятора отопления
#define DS_PIN 12      // 1-wire
#define HL_AUTO_PIN 13 // Индикация работы в автоматическом режиме

#include <Arduino.h>
#include <LCD_1602_RUS.h>
LCD_1602_RUS lcd(0x3F, 16, 2); // 0x27 // 0x3F

#include <GyverEncoder.h>
Encoder enc1(CLK, DT, SW, TYPE2);

#include <microDS18B20.h>
MicroDS18B20 sensor1(DS_PIN);

#include "EEPROM.h"
#include "avr/wdt.h"

bool flagMenuRedraw = 0, flagSwitchMenu = 0, flagChildMenu = 0, flagAuto = 0, flagMode = 0,
     flagOxygen = 0, flagTimerVentWork = 0, flagWorkKM1 = 0, flagHL_AUTO = 0, flagHL_VENT = 0;
unsigned int amountParentMenu = 2, countChoseParentMenu = 0, countChoseLineMenu = 0, amountChildLineMenu = 2;
unsigned int menuIdParent, menuIdChild;
float tempNeed = 30, getTemp, hysteresis = 4;
String arrow = "\xB7E";
int timeWorkHours = 1, timePauseHours = 1;
unsigned long timerOxigen, timerRequestTemp, timerRedraw;
int addresstempNeed = 15, // разносим адреса через десятки, а то глючит.
    addressHysteresis = 25,
    addressTimeWorkHours = 35,
    addressTimePauseHours = 100,
    addressFlagAuto = 65,
    addressFlagOxygen = 5; ///  !!!! Ячейка памяти по умолчанию имеет значени 255

void setup()
{
  timerOxigen = timerRedraw = timerRequestTemp = millis();
  enc1.setType(TYPE1);
  lcd.init();
  lcd.backlight();
  pinMode(HL_AUTO_PIN, OUTPUT);
  pinMode(KM1NO_AUTO_PIN, OUTPUT);
  pinMode(KM1NC_AUTO_PIN, OUTPUT);
  pinMode(KM2NO_VENT_PIN, OUTPUT);
  pinMode(HL_VENT_PIN, OUTPUT);
  sensor1.setResolution(9);
  EEPROM.get(addressFlagAuto, flagAuto);
  delay(10);
  EEPROM.get(addressFlagOxygen, flagOxygen);
  delay(10);
  EEPROM.get(addresstempNeed, tempNeed);
  delay(10);
  EEPROM.get(addressHysteresis, hysteresis);
  delay(10);
  EEPROM.get(addressTimeWorkHours, timeWorkHours);
  delay(10);
  EEPROM.get(addressTimePauseHours, timePauseHours);
  delay(10);
  wdt_enable(WDTO_500MS);
  // === Первоначальная запись в ячейку памяти, т.к. в новых записано значение 255
  // EEPROM.put(addressFlagAuto, (bool)0);
  // delay(10);
  // EEPROM.put(addressFlagOxygen, (bool)0);
  // delay(10);
  // EEPROM.put(addresstempNeed, (bool)1);
  // delay(10);
  // EEPROM.put(addressHysteresis, (bool)1);
  // delay(10);
  // EEPROM.put(addressTimeWorkHours, (bool)1);
  // delay(10);
  // EEPROM.put(addressTimePauseHours, (bool)1);
  // delay(10);
  // == Отладка
  // Serial.begin(9600); // инициализируем порт, скорость 9600
}

void lcdInit()
{
  delay(10);
  lcd.clear();
  lcd.setCursor(0, 0);
}

void switchMode()
{
  if (flagMode == 1 && menuIdParent == 0)
  {
    flagAuto = !flagAuto;
    EEPROM.put(addressFlagAuto, flagAuto);
    delay(10);
    flagMode = 0;
  }
  if (flagMode == 1 && menuIdParent == 1 && flagAuto == 1)
  {
    flagOxygen = !flagOxygen;
    EEPROM.put(addressFlagOxygen, flagOxygen);
    delay(10);
    flagMode = 0;
  }
}

void menu1_1()
{
  lcdInit();
  lcd.print(L"t=");
  lcd.setCursor(2, 0);
  lcd.print(getTemp);
  lcd.setCursor(6, 0);
  lcd.print(L"\xBDF");
  lcd.setCursor(7, 0);
  lcd.print(L"C");
  lcd.setCursor(10, 0);
  lcd.print(tempNeed);
  lcd.setCursor(14, 0);
  lcd.print(L"\xBDF");
  lcd.setCursor(15, 0);
  lcd.print(L"C");
  lcd.setCursor(0, 1);
  if (flagAuto == 0)
  {
    lcd.print(L"Ручной");
  }
  else
  {
    lcd.print(L"Автомат.");
  }
  lcd.setCursor(10, 1);
  lcd.print(hysteresis);
  lcd.setCursor(14, 1);
  lcd.print(L"\xBDF");
  lcd.setCursor(15, 1);
  lcd.print(L"C");
}

void menu1_2()
{
  lcdInit();
  lcd.print(L"Oxygen");
  lcd.setCursor(9, 0);
  lcd.print("Тр=");
  lcd.setCursor(12, 0);
  lcd.print((float)timeWorkHours * 0.1);
  lcd.setCursor(15, 0);
  lcd.print(L"ч");
  lcd.setCursor(0, 1);
  if (flagOxygen == 0)
  {
    lcd.print(L"Выкл.");
  }
  else
  {
    lcd.print(L"Вкл.");
  }
  lcd.setCursor(9, 1);
  lcd.print("Тп=");
  lcd.setCursor(12, 1);
  lcd.print((float)timePauseHours * 0.1);
  lcd.setCursor(15, 1);
  lcd.print(L"ч");
}

void menu1_1_1()
{
  lcdInit();
  if (countChoseLineMenu == 0)
  {
    lcd.print(arrow);
  }
  lcd.setCursor(1, 0);
  lcd.print(L"t=");
  lcd.setCursor(3, 0);
  lcd.print(tempNeed);
  lcd.setCursor(7, 0);
  lcd.print(L"\xBDF");
  lcd.setCursor(8, 0);
  lcd.print(L"C");
  lcd.setCursor(0, 1);
  if (countChoseLineMenu == 1)
  {
    lcd.print(arrow);
  }
  lcd.setCursor(1, 1);
  lcd.print(L"\xBE0");
  lcd.setCursor(2, 1);
  lcd.print(L"=");
  lcd.setCursor(3, 1);
  lcd.print(hysteresis);
  lcd.setCursor(7, 1);
  lcd.print(L"\xBDF");
  lcd.setCursor(8, 1);
  lcd.print(L"C");
}

void menu1_2_1()
{
  lcdInit();
  if (countChoseLineMenu == 0)
  {
    lcd.print(arrow);
  }
  lcd.setCursor(1, 0);
  lcd.print(L"работа=");
  lcd.setCursor(8, 0);
  lcd.print((float)timeWorkHours * 0.1);
  lcd.setCursor(12, 0);
  lcd.print(L"ч");
  lcd.setCursor(0, 1);

  if (countChoseLineMenu == 1)
  {
    lcd.print(arrow);
  }
  lcd.setCursor(1, 1);
  lcd.print(L"пауза=");
  lcd.setCursor(7, 1);
  lcd.print((float)timePauseHours * 0.1);
  lcd.setCursor(11, 1);
  lcd.print(L"ч");
}

void menuError()
{
  lcdInit();
  lcd.print(L"Ошибка выбора");
  lcd.setCursor(0, 1);
  lcd.print(L"Ошибка выбора");
}

// =================== в loop =============================================================================

void requestTemp()
{
  if (millis() - timerRequestTemp >= 2000)
  {                              // ищем разницу в мс
    timerRequestTemp = millis(); // сброс таймера
    sensor1.requestTemp();       // выполнить действие
  }
}

void menuRedraw()
{
  if (millis() - timerRedraw >= 2000)
  {                         // ищем разницу в мс
    timerRedraw = millis(); // сброс таймера
    flagMenuRedraw = 1;
  }
}

void writeKM1_AUTO_PIN() // Для релейного регулирования с обратной связью
{
  getTemp = sensor1.getTemp(); // сообщаем регулятору текущую температуру
  if (flagAuto == 1)
  {

    if ((getTemp < (tempNeed - hysteresis / 2)) && flagWorkKM1 == 0)
    {
      flagWorkKM1 = 1;
      digitalWrite(KM1NC_AUTO_PIN, HIGH); // Управление по низкому уровню. Выключить при выполнении условия.
      digitalWrite(KM1NO_AUTO_PIN, LOW); // Управление по низкому уровню. Включить при выполнении условия.
    }

    if ((getTemp > (tempNeed + hysteresis / 2)) && flagWorkKM1 == 1)
    {
      flagWorkKM1 = 0;
      digitalWrite(KM1NO_AUTO_PIN, HIGH);
      digitalWrite(KM1NC_AUTO_PIN, LOW);
    }
    if (flagHL_AUTO == 0)
    {
      digitalWrite(HL_AUTO_PIN, HIGH);
      flagHL_AUTO = 1;
    }
  }
  if (flagAuto == 0)
  {
    if (flagHL_AUTO == 1)
    {
      flagHL_AUTO = 0; flagWorkKM1 = 0;
      digitalWrite(HL_AUTO_PIN, LOW);
      digitalWrite(KM1NO_AUTO_PIN, HIGH);
      digitalWrite(KM1NC_AUTO_PIN, HIGH);
    }
  }
}

void writeKM2NO_VENT_PIN() // Включаем либо выключаем подачу кислорода
{
  if (flagAuto == 1 && flagOxygen == 1)
  {
    if ((millis() - timerOxigen >= timePauseHours * 60 * 60 *1000) && flagTimerVentWork == 0)
    // 60 * 60 * 1000 !!!!
    {
      flagTimerVentWork = 1;
      timerOxigen = millis();             // сброс таймера
      digitalWrite(KM2NO_VENT_PIN, LOW); // На замыкание котактов. Если низкая то греем.
    }
    if ((millis() - timerOxigen >= timeWorkHours * 60 * 60 *1000) && flagTimerVentWork == 1)
    // 60*60*1000 !!!!
    {
      flagTimerVentWork = 0;
      timerOxigen = millis(); // сброс таймера
      digitalWrite(KM2NO_VENT_PIN, HIGH);
    }
    if (flagHL_VENT == 0)
    {
      flagHL_VENT = 1;
      digitalWrite(HL_VENT_PIN, HIGH);
    }
  }
  if (flagAuto == 1 && flagOxygen == 0)
  {
    if (flagHL_VENT == 1)
    {
      flagHL_VENT = 0;
      digitalWrite(HL_VENT_PIN, LOW);
      digitalWrite(KM2NO_VENT_PIN, HIGH);
    }
  }

  if (flagAuto == 0)
  {
    if (flagHL_VENT == 1)
    {
      flagHL_VENT = 0;
      digitalWrite(KM2NO_VENT_PIN, HIGH);
      digitalWrite(HL_VENT_PIN, LOW);
    }
  }
}

void switchMenu()
{
  if (flagChildMenu == 0 && flagSwitchMenu == 1)
  {
    flagSwitchMenu = 0;
    flagChildMenu = 1;
  }
  else if (flagChildMenu == 1 && flagSwitchMenu == 1)
  {
    if (menuIdChild == 0)
    {
      EEPROM.put(addresstempNeed, tempNeed);
      delay(10);
      EEPROM.put(addressHysteresis, hysteresis);
      delay(10);
    }
    else if (menuIdChild == 1)
    {
      EEPROM.put(addressTimeWorkHours, timeWorkHours);
      delay(10);

      EEPROM.put(addressTimePauseHours, timePauseHours);
      delay(10);
    }
    flagSwitchMenu = 0;
    flagChildMenu = 0;
  }
}

void parentMenuEnc()
{
  if (enc1.isRelease() && flagChildMenu == 0) // Однокраное нажание. Переключение режима работы с ручного на автоматический.
  {
    flagMode = 1;
  }

  if (enc1.isHolded()) // Удержание. Переключение между меню.
  {
    flagSwitchMenu = 1;
  }
  if (enc1.isRight()) // Вращение ручки енкодера. Циклическое перемещение стрелки по пунктам.
  {
    if (flagChildMenu == 0)
    {
      flagMenuRedraw = 1;
      countChoseParentMenu++;
      if (countChoseParentMenu >= amountParentMenu)
      {
        countChoseParentMenu = 0;
      }
    }
    if (flagChildMenu == 1)
    {
      flagMenuRedraw = 1;
      countChoseLineMenu++;
      if (countChoseLineMenu >= amountChildLineMenu)
      {
        countChoseLineMenu = 0;
      }
    }
  }
  if (enc1.isLeft()) // Вращение ручки енкодера. Циклическое перемещение стрелки по пунктам.
  {
    if (flagChildMenu == 0)
    {
      flagMenuRedraw = 1;
      countChoseParentMenu++;
      if (countChoseParentMenu >= amountParentMenu)
      {
        countChoseParentMenu = 0;
      }
    }
    if (flagChildMenu == 1)
    {
      flagMenuRedraw = 1;
      countChoseLineMenu++;
      if (countChoseLineMenu >= amountChildLineMenu)
      {
        countChoseLineMenu = 0;
      }
    }
  }
}

void childMenuEnc()
{
  if (enc1.isRightH() && flagChildMenu == 1) // Удержание кнопки с вращением ручки енкодера. Изменение переменной.
  {
    flagMenuRedraw = 1;

    if (menuIdChild == 0 && countChoseLineMenu == 0)
    {
      tempNeed++;
      if (tempNeed > 100)
      {
        tempNeed = 100;
      }
    }
    else if (menuIdChild == 0 && countChoseLineMenu == 1)
    {
      hysteresis++;
      if (hysteresis > 100)
      {
        hysteresis = 100;
      }
    }
    else if (menuIdChild == 1 && countChoseLineMenu == 0)
    {
      timeWorkHours++;
      if (timeWorkHours > 240)
      {
        countChoseParentMenu = 240;
      }
    }
    else if (menuIdChild == 1 && countChoseLineMenu == 1)
    {
      timePauseHours++;
      if (timePauseHours > 240)
      {
        timePauseHours = 240;
      }
    }
  }

  if (enc1.isLeftH() && flagChildMenu == 1) // Удержание кнопки с вращением ручки енкодера. Изменение переменной.
  {
    flagMenuRedraw = 1;

    if (menuIdChild == 0 && countChoseLineMenu == 0)
    {
      tempNeed--;
      if (tempNeed < 0)
      {
        tempNeed = 0;
      }
    }
    else if (menuIdChild == 0 && countChoseLineMenu == 1)
    {
      hysteresis--;
      if (hysteresis < 0)
      {
        hysteresis = 0;
      }
    }
    else if (menuIdChild == 1 && countChoseLineMenu == 0)
    {
      timeWorkHours--;
      if (timeWorkHours < 1)
      {
        timeWorkHours = 0;
      }
    }
    else if (menuIdChild == 1 && countChoseLineMenu == 1)
    {
      timePauseHours--;
      if (timePauseHours < 1)
      {
        timePauseHours = 0;
      }
    }
  }
}

void parentMenu()
{
  switchMode();
  switch (countChoseParentMenu)
  {
  case 0:
    menuIdParent = 0;
    menu1_1();
    break;
  case 1:
    menuIdParent = 1;
    menu1_2();
    break;
  default:
    menuError();
    break;
  }
}

void childMenu()
{
  switch (menuIdParent)
  {
  case 0:
    menuIdChild = 0;
    menu1_1_1();
    break;
  case 1:
    menuIdChild = 1;
    menu1_2_1();
    break;
  default:
    menuError();
    break;
  }
}

void loop()
{
  enc1.tick();
  requestTemp();
  parentMenuEnc();
  childMenuEnc();
  switchMenu();
  menuRedraw();
  if (flagChildMenu == 0 && flagMenuRedraw == 1)
  {
    parentMenu();
    flagMenuRedraw = 0;
  }
  writeKM1_AUTO_PIN();
  writeKM2NO_VENT_PIN();
  if (flagChildMenu == 1 && flagMenuRedraw == 1)
  {
    childMenu();
    flagMenuRedraw = 0;
  }
  wdt_reset();

  // == Отладка
  //  Serial.println("\nflagAuto=");
  // Serial.println(flagAuto);
  // Serial.println("\ntemp=");
  // Serial.println(temp);
  //   Serial.println("\nflagWorkKM1=");
  // Serial.println(flagWorkKM1);
}