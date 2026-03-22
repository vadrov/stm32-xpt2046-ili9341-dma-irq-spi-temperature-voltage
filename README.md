Copyright (C) 2019, VadRov, all right reserved / www.youtube.com/@VadRov / www.dzen.ru/vadrov
# XPT2046 controller driver (stm32f4, ili9341, xpt2046, dma, irq, temperature, voltage)
 XPT2046 controller driver (HR2046 and other compatible). Supports all the functionality of the controller: work with the touchscreen, temperature measurement by an internal sensor, voltage measurement at the VBAT and AUX inputs.

**ВАЖНОЕ ЗАМЕЧАНИЕ**
- В этом драйвере может быть использован, как внутренний, так и внешний источники опорного напряжения для измерений АЦП.
- Если на вашей плате к 9 выводу микросхемы VREF подведено питание (как правило, накоротко соединено с VCC - 1 вывод м/с, IOVDD - 10 вывод м/с и VREF - 9 вывод м/с), то при измерениях АЦП всегда в качестве опорного напряжения будет использовать то напряжение, которое подведено в внешнему источнику VREF. Поэтому в даташите указано, что, если планируется использование внутреннего источника опорного напряжения, то вывод VREF должен оставаться неподключенным. То есть вывод VREF - это двунаправленный вывод, работающий как вход, если подключен источник внешнего опорного напряжения и как выход внутреннего источника опорного напряжения, если он включен соответствующим битом в управляющем байте. Эти тонкости не важны при использовании тачскрина, но критичны, если XPT2046 используется для измерения температуры (датчиком внутри самого чипа, по сути, измеряющим температуру самого кристалла микросхемы, а не окружающей вас среды) и напряжений (на выводах VBAT и AUX). Неверная настройка управляющего байта и неправильное указание значения опорного напряжения в таких случаях существенно исказят измерения.
- В файле xpt2046.h есть соответствующее макроопределение, влияющее на формат управляющих байтов:
```c
#define XPT2046_INTERNAL_VREF //Используем внутренний источник опорного напряжения.
							    //Если используется внешний источник опорного напряжения,
							    //то строку закомментируйте. Это для случая, если к выводу 9
								//VREF не подключено питание, т.е. вывод свободен (не распаян, не соединен с другими выводами и т.п.).
```
- Величину опорного напряжения для внутреннего и внешнего источников можно задать в том же файле соответствующими макроопределениями:
```c
#ifdef XPT2046_INTERNAL_VREF
#define XPT2046_VREF  2500 //Напряжение внутреннего источника опорного напряжения.
						   //Оно определено в спецификации на контроллер XPT2046 (2500 мВ +/- 50 мВ).
                           //и измеряется на 9 выводе м/с в корпусе TSSOP-16, если он не подключен к питанию 

#define XPT2046_VREF  3490 //Напряжение внешнего источника опорного напряжения.
						   //Измеряется на 9 выводе микросхемы XPT2046 в корпусе TSSOP-16.
						   //Здесь напряжение для моей платы. У вас может быть иным.
						   //Это для случая, если к выводу 9 подключено питание (соединен с выводами VCC и IOVDD).
#endif
```
- Для последующего относительно точного определения температуры кристалла XPT2046 (заявлена точность ±2°C) и напряжений на входах VBAT и AUX вы должны измерить фактическое значение напряжения на выводе VREF даже для случая использования внутреннего источника опорного напряжения, несмотря на то, что в спецификации указано его значение 2500 мВ. Но не забывайте, что, во-первых, оно указано с допуском в ±50 мВ, а, во-вторых, мои измерения для микросхем разных изготовителей и разных серий показывают, что это значение в спецификации иногда не соблюдается и имеет существенный разброс: 2400...2650 мВ. Измерение производите прибором с большим входным сопротивлением (мегаомы).
- В идеале, в момент включения вашего устройства, находившегося продолжительное время в помещении, температура кристалла м/с XPT2046 должна примерно соответствовать температуре в этом помещении. Для неответственных задач, можете просто "откалибровать" значение VREF по известной температуре. Прошего говоря, не измерить его, а "подогнать" под значение известной температуры, уменьшая-увеличивая значение VREF от известной базы (2500 мВ, если используется внутренний источник и, например, 3300 мВ, если используется внешний источник, в качестве которого выступает, как правило, питание вашего дисплейного модуля).

**IMPORTANT NOTE**
- This driver can use either internal or external reference voltage sources for ADC measurements.
- If your board has power supplied to pin 9 of the VREF chip (usually shorted to VCC - pin 1 of the I/O module, IOVDD - pin 10 of the I/O module, and VREF - pin 9 of the I/O module), the ADC will always use the voltage supplied to the external VREF source as the reference voltage for measurements. Therefore, the datasheet states that if the internal reference voltage is to be used, the VREF pin should be left unconnected. This means that the VREF pin is bidirectional, functioning as an input if an external reference voltage source is connected, and as an output of the internal reference voltage source if enabled by the corresponding bit in the control byte. These subtleties aren't important when using a touchscreen, but they are critical if the XPT2046 is used to measure temperature (using a sensor inside the chip itself, essentially measuring the temperature of the chip itself, not the surrounding environment) and voltage (on the VBAT and AUX pins). Incorrectly setting the control byte and specifying the incorrect reference voltage value in such cases will significantly distort the measurements.
- The xpt2046.h file contains a corresponding macro definition that affects the format of the control bytes:
```c
#define XPT2046_INTERNAL_VREF //Use the internal reference voltage.
//If an external reference voltage is used,
//then comment out this line. This applies if pin 9
//VREF is not powered, i.e., the pin is free (not soldered, not connected to other pins, etc.).
```
- The reference voltage value for the internal and external sources can be set in the same file using the corresponding macro definitions:
```c
#ifdef XPT2046_INTERNAL_VREF
#define XPT2046_VREF 2500 //Voltage of the internal reference voltage.
//It is defined in the XPT2046 controller datasheet (2500 mV +/- 50 mV).
//and is measured at pin 9 of the m/s in the TSSOP-16 package if it is not connected to power.

#define XPT2046_VREF 3490 //Voltage of the external reference voltage.
//Measured at pin 9 of the XPT2046 chip in the TSSOP-16 package.
//This voltage is for my board. Yours may be different.
//This is for the case when power is connected to pin 9 (connected to the VCC and IOVDD pins).
#endif
```
- To determine the XPT2046's crystal temperature (claimed accuracy of ±2°C) and the voltages at the VBAT and AUX inputs, you should measure the actual voltage at the VREF pin, even when using the internal reference voltage source, despite the specification stating its value of 2500 mV. However, keep in mind that, firstly, it is specified with a tolerance of ±50 mV, and secondly, my measurements for chips from different manufacturers and different series show that this value in the specification is sometimes not adhered to and has a significant spread: 2400...2650 mV. Perform the measurement with a device with a high input impedance (megohms).
- Ideally, when your device is turned on after being left indoors for an extended period, the crystal temperature of the XPT2046 microcontroller should roughly correspond to the room temperature. For less demanding applications, you can simply "calibrate" the VREF value using a known temperature. In other words, don't measure it, but rather "adjust" it to the known temperature by increasing or decreasing the VREF value from a known baseline (2500 mV if using an internal source, and, for example, 3300 mV if using an external source, typically the power supply for your display module).

**Functions and features:**
- Touch screen polling (in and out of the interrupt) with information about the coordinates and duration of the touch;
- Determination of the status of the current touch of the touchscreen: no touch, click, hold;
- Temperature measurement by built-in sensor;
- Measurement of battery voltage at the input of the VBAT controller (7 pin m/s in the TSSOP-16 housing);
- Measurement of the voltage at the AUX input of the controller (8 pin m/s in the TSSOP-16 housing).
- Allows operation on one spi of several devices.

Драйвер контроллера XPT2046 (HR2046 и других совместимых). Поддерживает весь функционал контроллера: работа с тачскрином, измерение температуры внутренним датчиком, измерение напряжений на входах VBAT и AUX.

**Проект собран для МК stm32f401ccu6.**

**Функции и возможности:**
- Опрос тачскрина (в прерывании и вне прерывания) с получением информации о координатах и длительности касания;
- Определение статуса текущего касания тачскрина: нет касания, клик, удержание;
- Измерение температуры встроенным датчиком;
- Измерение напряжения батареи на входе VBAT контроллера (7 вывод м/с в корпусе TSSOP-16);
- Измерение напряжения на входе AUX контроллера (8 вывод м/с в корпусе TSSOP-16).
- Допускает работу на одном spi нескольких устройств.

**Схема подключения дисплея с тачскрином (к плате stm32f401ccu6, stm32f411ceu6):**
![stm32 + ili9341+xpt2046 схема подключения](https://user-images.githubusercontent.com/111627147/211880060-12eb392f-d982-4026-aa97-a971dd6c6dfe.jpg)

## Использование:
- В среде STM32CubeIDE создайте новый проект для своего МК stm32f4xx. Выполните общую настройку проекта (тактирование, отладка и т.п.). Определите spi для работы с контроллером. Если к этому spi будет подключено еще одно устройство, например, дисплей, то настройте spi (задайте параметры spi) под используемый дисплей. При этом помните, что XPT2046 требует 4-проводного spi (полнодуплексный режим). Определите выводы МК T_CS (выход, GPIO mode -> Output Push Pull) и T_IRQ (вход с внешним прерыванием по спадающему фронту, GPIO mode -> External Interrupt Mode with Falling edge trigger detection). Подтяните вывод T_CS к питанию внутренней подтяжкой (GPIO Pull-up/Pull-down -> Pull-up). Если к одному spi будет подключено несколько устройств, то CS выводы других устройств также притяните к питанию. Кроме того, задайте высокий уровень на всех выходах CS (GPIO output level -> High). Подтяните вывод T_IRQ к питанию внутренней подтяжкой (GPIO Pull-up/Pull-down -> Pull-up), если на вашей плате не распаян резистор, подтягивающий к питанию вывод PENIRQ контроллера XPT2046. Скорость выводов определите в High, скорость линий spi в Very High.
- В файле stm32f4xx_it.c
1. Подключить заголовочный файл драйвера
```c
#include "xpt2046.h"
```
2. В обработчик SysTick_Handler добавить вызов:
```c
 XPT2046_TIMCallback(touch);
```
3. В обработчик внешнего прерывания EXTIxx_IRQHandler (xx определяет линию прерывания) добавить вызов:
```c
 XPT2046_EXTICallback(touch);
 ```
- Определить параметры подключения МК к контроллеру XPT2046:
```c
 XPT2046_ConnectionData cnt_touch = {	.spi 	  = SPI1, //Используемый spi
					.speed 	  = 4, //Скорость spi 0...7 (0 - clk/2, 1 - clk/4, ..., 7 - clk/256)
					.cs_port  = T_CS_GPIO_Port, //Порт для управления T_CS
					.cs_pin	  = T_CS_Pin, //Вывод порта для управления T_CS
					.irq_port = T_IRQ_GPIO_Port, //Порт для управления T_IRQ
					.irq_pin  = T_IRQ_Pin, //Вывод порта для управления T_IRQ
					.exti_irq = T_IRQ_EXTI_IRQn //Канал внешнего прерывания
};
```
- Объявить переменную обработчика XPT2046:
```c
 XPT2046_Handler touch1;
```
- Инициализировать обработчик XPT2046:
```c
 XPT2046_InitTouch(&touch1, 20, &cnt_touch);
```
Параметр 20 определяет период счета внутреннего таймера обработчика. Таймер обработчика привязан к системному таймеру, который должен быть настроен на генерацию прерываний 1000 раз в секунду. Таким образом, опрос тачскрина в прерывании будет происходить 1000/20 = 50 раз в секунду.
- Произвести калибровку тачскрина:
```c
  XPT2046_CalibrateTouch(&touch1, lcd); //Запускаем процедуру калибровки
```
Опрос тачскрина может осуществляться в прерывании драйвера (touch1.fl_interrupt = 1) либо в программе пользователя (touch1.fl_interrupt = 0). Если опрос осуществляется вне прерывания драйвера, то необходимо вызывать в программе пользователя перед получением статуса и координат касания:
```c
uint8_t res = XPT2046_GetTouch(&touch1); //res = 0 - опрос успешный, res = 1 - нет касания, res = 2 - spi занято
```
- Для получения координат касания:
```c
tPoint point_d; //координаты касания на дисплее: x соответствует point_d.x, а у - point_d.y
XPT2046_ConvertPoint(&point_d, &touch1.point, &touch1.coef); //Преобразуем координаты тачскрина в дисплейные
```
Текущий статус касания определяет параметр touch1.status. Если touch1.status = XPT2046_STATUS_NOCLICK, то нет клика; touch1.status = XPT2046_STATUS_CLICK - есть клик,  touch1.status = XPT2046_STATUS_HOLD - есть удержание. Время перехода касания из статуса "клик" в статус "удержание" определяет параметр XPT2046_TOUCH_HOLD_TIME (файл xpt2046.h).


Автор: **VadRov**

Контакты: [Youtube](https://www.youtube.com/@VadRov) [Дзен](https://dzen.ru/vadrov) [VK](https://vk.com/vadrov) [Telegram](https://t.me/vadrov_channel)

Поддержать автора: [donate.yoomoney](https://yoomoney.ru/to/4100117522443917)
