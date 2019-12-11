# Car-china-radio-remote-control
https://www.youtube.com/watch?v=470r13HLdKc

Система состоит из двух устройств: передатчик и приемник. Схемы обоих устройств представлены в файле Schematic_TX_RX-buttons.png

Для сборки используются готовые платы, аналоги Arduino Nano, на микроконтроллерах Atmega328/Atmega168 или их китайском клоне GLT8F328P (кстати, очень не плохой чип). Так же используются готовые модули NRF24L01, в любом исполнении. 

Передатчик обеспечивает передачу 10 дискретных команд, в том числе регулировку звука с помощью рототативного энкодера от, пришедшей в негодность, компьютерной мышки.



Передатчик:

Исполюзуются три кнопки, кнопка размещенная под энкодером от мыши и сам энкодер для регулировки звука.

Кратковременое нажатие на любую из кнопок формирует кодовую посылку передающуюся на приемник. 

Удержание кнопки более 300мс формирует модифицированную кодовую посылку. Таким образом, кнопки могут формировать 8 независимых команд. Дополнительные две команды (предусмотренны как Vol+ и Vol-) формирует вращение энкодера.

Одновременное вращение и нажатие кнопки энкодера трактуется передатчиком как нажатие кнопки энкодера, и не формирует дополнительные команды. Так как кнопку энкодера логично привязывать к команде Mute, это сделано во избежание ошибок во время вождения.

Передатчик не отправляет код отмены команды (отжатия кнопок), поэтому логика обработки команд реализована в приемнике.

Для переключения режимов буззера - включен\выключен, нужно на обесточенном передатчике зажать кнопку "Центр" (указана на схеме как SW3) и подать на передатчик питание. Если буззер был включен, он выключится, и наоборот. Состояние сохраняется в ЕЕПРОМ.

Для включения режима "Обучение магнитолы" нужно обесточить передатчик, и зажав кнопки "Вверх" (SW2) и "Вниз" (SW4), подать на него питание. Кодовая посылка будет модифицировна и приемник будет понимать, что нужно перейти в режим "Обучение магнитолы". В этом режиме приемник будет передавать каждую команду магнитоле на протяжении около 3-х секунд. Этого времени достаточно, чтобы магнитола определила команду и прописала его в память. Выход из этого режима - кратковременно обесточить передатчик, или нажать на нем кнопку RESET на плате ардуино.

Так же, может быть так, что у вас в руле уже установлены некие кнопки управления (как у меня в KIA Magentis II), и на них, при включении фар, приходит сигнал подсветки +12 вольт. В таком случае есть смысл реализовать ту часть схемы, которая отвечает за подсветку. Это обеспечит вам подсветку кнопок своей конструкции, а так же передать этот сигнал приемнику, который в свою очередь передаст его на вход ILLUMINATION магнитолы .



Приемник:

Получив кодовую посылку от передатчика, приемник идентифицирует команду и подключает резисторы разного номинала, расположенные на пинах микроконтроллера, к массе, второй вывод резистора, соответственно, подключен к выводу KEY1 или KEY2. И вслучае если это команды от кнопок, задерживается в таком состоянии на 500мс, что обеспечивает гарантированное считывание команды магнитолой. В случае если используется вращение энкодера, длительность подключения соответствующих резисторов равна 100мс.
Эти два параметра описаны и могут бить изменены в скетче приемника (читайте описание констант hold и keyHold).
В случае если передатчик прислал состояние "Обучение магнитолы), эти временные задержки возрастают на 2500мс, для энкодера равнаются 2600мс, для кнопок 3000мс.

После получения команды, приемник отсылает передатчику подтверждение приема. Если по каким либо причинам передатчик не получил подтверждения приема команды, он повторит попытку 15 раз, и если снова нет, то вы услышите три которких попискивания, в случае если буззер активирован.



Специфика сборки и настройки:

Как показала практика, энкодеры мышки бывают некоторых разновидностей, и распиновка их неочевидна. Для правильного использования, вам нужно внимательно проанализировать плату, и определится с "земляным" выводом (на схеме обозначен как C). Чаще всего, им окажется средний вывод, но, как в моем случаи, им может оказаться любой крайний. Идентифицировать выводы не сложно -  два сигнальных вывода (A/B) всегда уходят к выводам специализированной микросхемы мыши, а "земляной" приходит на массу\общий питания.

Если ваш энкодер ведет себя странно то: а) вы неправильно определили распиновку; б) вам нужно поменять параметр c1.setType(TYPE1) на TYPE2, в скетче передатчика; возможно вам придется поставить по коденсатору 10-100нФ между выводами A\B и массой.

Буззер для передатчика может быть любым, даже маленьким динамиком. Но все же, рекомендую использовать пасивный пьезоэлемент или пасивный буззер. Стандартный ардуиновский активный тоже будет работать нормально, но звук будет хриплым.

Не пробуйте запитать ардуино от батареи автомобиля на пин VIN, это чревато выходом ее из строя. Теоретически это делать можно, а практически - качество установленных на ней китайских AMS1117-5.0 очень далеко от заявленных параметров. Сэкономив на копеечном LM7805, вы можете потерять целую плату.

Не пробуйте запитать NRF24L01 от выхода +3.3V ардуино. Ток, который потребляет радиомодуль, в несколько раз превышает ток, который способно обеспечить на этом выходе ардуино. Вы либо не получите стабильную работу радимодуля, либо сожжете выход +3.3V ардуино. Питание NRF24L01 через отдельный линейный стабилизатор AMS1117-3.3 - обязательно.

Питание приемника лучше всего получить прямо с магнитолы, с выхода управления внешним усилителем. На этом выходе напряжение около +9V, что уменьшит нагревание стабилизатора питания LM7805, и отделит питание приемника радиоудлиннителя от помех по питанию, от системы зажигания автомобиля. Напряжение на выходе управления внешним усилителем магнитолы появляется после полной загрузки андроид.

Купить мне пиво можно тут https://www.paypal.me/kostyamat
