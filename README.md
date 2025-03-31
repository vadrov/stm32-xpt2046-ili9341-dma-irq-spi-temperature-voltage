Copyright (C) 2019, VadRov, all right reserved / www.youtube.com/@VadRov / www.dzen.ru/vadrov
# XPT2046 controller driver (stm32f4, ili9341, xpt2046, dma, irq, temperature, voltage)
 XPT2046 controller driver (HR2046 and other compatible). Supports all the functionality of the controller: work with the touchscreen, temperature measurement by an internal sensor, voltage measurement at the VBAT and AUX inputs.
 
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
