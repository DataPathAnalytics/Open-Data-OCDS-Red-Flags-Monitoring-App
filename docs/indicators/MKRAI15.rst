######################################################################################################################################################
MKRAI15. Закупка товаров методом прямого заключения договора превышает 5% стоимости договора, заключенного на основании проведенного конкурса при сохранении цены и технических спецификаций. Дозакупка товаров, которые ранее не закупались. Дозакупка товаров с отсылкой не на конкурс.
######################################################################################################################################################

***************
Суть индикатора
***************

Индикатор отслеживает случаи, когда закупающая организация закупает товары методом прямого заключения договора, используя обоснование "осуществления дополнительного приобретения товаров, не превышающих 5 процентов стоимости договора, заключенного на основании проведенного конкурса при сохранении цены и технических спецификаций".

****
Риск
****

Избегание применения конкурсных процедур с целью заключения договора с "удобным" поставщиком. Представители закупающей организации нарушают норму Закона, чтобы избежать проведения конкурса на общих основаниях. Данный факт, кроме непосредственно нарушения нормы Закона, может служить индикатором сговора представителей закупающей организации и поставщика (подрядчика).

*******************************
Нарушение норм/принципов закона
*******************************

Часть 4 статьи 21 Закона "О государственных закупках": "1) осуществления дополнительного приобретения товаров, не превышающих 5 процентов стоимости договора, заключенного на основании проведенного конкурса при сохранении цены и технических спецификаций".


***********************************
Основание для разработки индикатора
***********************************

Индикатор вводится, так как в системе не реализован контроль соответствия цены, технических требований и соотношений сумм вышеописанных причин.


******************************
Методология расчета индикатора
******************************

Уровень расчета
===============
Индикатор рассчитывается на уровне *конкурса*.

Источники данных для расчета
============================

Для расчета индикатора используются следующие источники данных:

- API системы государственных закупок в OCDS формате
- Транзакционная переменная :ref:`tv_tenderCPVList`

Методы закупок
==============

Индикатор рассчитывается для следующих методов закупок:

- метод прямого заключения договора.


Статусы закупок
---------------

Индикатор расчитывается для закупок, которые:

- находятся в статусе ``published``
- находятся в статусе ``changed``


Качество данных
---------------

Индикатор расчитывается только для конкурсов, у которых значение переменной ``tv_badQuality=0``.



Частота расчета
===============

Если выполнены все условия для активации расчета индикатора, он рассчитывается каждый раз после изменений в JSON-документе конкурса. Также индикатор расчитывается один раз в сутки, если выполнены все условия для его расчета.


Поля для расчета
================

Для расчета индикатора используются следующие поля API модуля системы гос. закупок:

- ``data.tender.procurementMethodRationale``
- ``data.relatedProcesses.relationship``
- ``data.tender.procurementMethodDetails``
- ``data.relatedProcesses.identifier``
- ``data.tender.items.classification.scheme``
- ``data.tender.items.classification.id``
- ``data.item.relatedLot``
- ``data.awards.relatedLot``
- ``data.awards.relatedBid``
- ``data.bids.details.priceProposal.unit.value.amount``
- ``data.bids.relatedLots.value.amount``

Для расчета используются следующие транзакционные переменные:

- :ref:`tv_tenderCPVList`

Формула расчета
===============

1. Выбираем только процедуры, у которых ``data.tender.procurementMethodRationale = 'additionalProcurement10'`` (в новом экспортере ``additionalProcurement5``).

2. Если в процедуре отсутствует контейнер ``data.relatedProcesses``, индикатор принимает значение ``-1``. Расчет заканчивается.

3. Выбираем предыдущую процедуру открытых торгов: такой номер ``data.relatedProcesses.identifier``, которму соответствует ``data.relatedProcesses.relationship = 'prior'``.

4. Если у найденной процедуры ``data.tender.procurementMethodDetails`` не равно ``oneStage``, ``downgrade`` или ``simplicated``, индикатор принимает значение ``1``. Расчет заканчивается.

5. Если статус найденной процедуры ``data.tender.statusDetails`` не равен ``contractSigned`` или ``evaluationComplete``, индикатор принимает значение ``1``. Расчет заканчивается.

6. Все элементы списка переменной :ref:`tv_tenderCPVList` должны находиться в соответствующей переменной найденной процедуры. Иначе, индикатор принимает значение ``1``. Расчет заканчивается.

7. Для каждого предмета закупки проводим следующие действия.
    - В исследуемой процедуре находим элемент ``data.item``, в котором ``data.tender.items.classification.id`` равен нашему.
    - Определяем идентификатор лота ``data.item.relatedLot``, к которому относится найденный ``data.item``.
    - Находим блок определения победителя, где ``data.awards.relatedLot = data.item.relatedLot`` и ``data.awards.status = 'active'``.
    - В найденном блоке определения победителя находим идентификатор победившего предложения ``data.awards.relatedBid``.
    - По найденному идентификатору находим выигравшее предложение ``data.awards.relatedBid = data.bids.details.id``.
    - В выигравшем предложении в блоке ``data.bids.priceProposal`` находим цену единицы измерения предмета закупки ``data.bids.details.priceProposal.unit.value.amount``.
    - По такой же схеме находим стоимость исследуемого предмета закупки в предыдущей процедуре открытых торгов.
    - Если найденные цены единиц измерения в нашей процедуре не находятся в предыдущей, индикатор принимает значение ``-1``. Расчет заканчивается.

8. В найденных выигравших предложения из предыдущего шага сравниваем суммы всех сумм выигравших предложений ``data.awards.value.amount``. Если сумма исследуемой процедуры составляет больше 5% от суммы предшествующей конкурентной процедуры, индикатор принимает значение ``1``. Расчет заканчивается.

9. Если мы дошли до этого пункта, индикатор принимает значение ``0``.

Факторы, которые влияют на корректное срабатывание индикатора
=============================================================

Индикатор может срабатывать неправильно, если код предмета закупки, указанный закупающей организацией не детализирован достаточно для точной идентификации предмета закупки.
