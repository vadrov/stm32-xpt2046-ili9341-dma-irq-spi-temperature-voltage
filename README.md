# XPT2046 controller driver (stm32f4, ili9341, xpt2046, dma, irq, temperature, voltage)
 XPT2046 controller driver (HR2046 and other compatible). Supports all the functionality of the controller: work with the touchscreen, temperature measurement by an internal sensor, voltage measurement at the VBAT and AUX inputs.

Драйвер контроллера XPT2046 (HR2046 и других совместимых). Поддерживает весь функционал контроллера: работа с тачскрином, измерение температуры внутренним датчиком, измерение напряжений на входах VBAT и AUX.

**Функции и возможности:**
 *  - Опрос тачскрина (в прерывании и вне прерывания) с получением информации о координатах и длительности касания;
 *  - Определение статуса текущего касания тачскрина: нет касания, клик, удержание;
 * 	- Измерение температуры встроенным датчиком;
 * 	- Измерение напряжения батареи на входе VBAT контроллера (7 вывод м/с в корпусе TSSOP-16);
 * 	- Измерение напряжения на входе AUX контроллера (8 вывод м/с в корпусе TSSOP-16).
 * 	- Допускает работу на одном spi нескольких устройств.

Схема подключения дисплея с тачскрином:
![stm32 + ili9341+xpt2046 схема подключения](https://user-images.githubusercontent.com/111627147/211880060-12eb392f-d982-4026-aa97-a971dd6c6dfe.jpg)

Автор: **VadRov**

Контакты: [Youtube](https://www.youtube.com/@VadRov) [Дзен](https://dzen.ru/vadrov) [VK](https://vk.com/vadrov) [Telegram](https://t.me/vadrov_channel)

Донат: [Поддержать автора](https://yoomoney.ru/to/4100117522443917)
