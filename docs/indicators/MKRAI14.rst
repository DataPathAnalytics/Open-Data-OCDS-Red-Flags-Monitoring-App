######################################################################################################################################################
MKRAI14. Поставщик (подрядчик) никогда не побеждал в конкурсных процедурах других закупающих организаций.
######################################################################################################################################################

***************
Суть индикатора
***************

Индикатор отслеживает случаи, когда поставщики в конкурсе никогда не побеждали в конкурсах других закупающих организаций.

****
Риск
****

Риск сговора между представителями закупающей организации и участником конкурса. Если поставщик (подрядчик) никогда не побеждает в конкурсах других закупающих организаций, значит у него не самое лучшее тендерное предложение: высокая цена, отсутствие необходимой квалификации, непрофессионально подготовленная конкурсная заявка и т. д. И если такой поставщик (подрядчик) побеждает у конкретной закупающей организации (риск усиливается, если он побеждает неоднократно), это может служить сигналом о наличии сговора. Если поставщик не может победить у других, как он побеждает у данной закупающей организации? При исследовании важно учитывать региональный фактор, а именно: возможно данный поставщик (подрядчик) единственный, кто поставляет определенный предмет закупки в регионе. Если же он не единственный, то риск существенно усиливается.


*******************************
Нарушение норм/принципов закона
*******************************

Часть 1 Статьи 1 Закона "О государственных закупках": "Целью настоящего Закона является обеспечение экономичности и эффективности использования государственных средств при осуществлении государственных закупок".


***********************************
Основание для разработки индикатора
***********************************

Индикатор вводится так как в системе не отслеживается история побед поставщиков.

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
- Аналитическая таблица :ref:`tbl_PESupplier`

Методы закупок
==============

Индикатор рассчитывается для следующих методов закупок:

- прямого заключения договора.


Статусы закупок
---------------

Индикатор рассчитывается для конкурсов, которые начаты в текущем году (``data.tender.datePublished``) и находятся в статусах:

- находятся в статусе ``evaluationComplete``
- находятся в статусе ``contractSignPending``


Качество данных
---------------

Индикатор расчитывается только для конкурсов, у которых значение переменной ``tv_badQuality=0``.



Частота расчета
===============

Если выполнены все условия для активации расчета индикатора, он рассчитывается каждый раз после изменений в JSON-документе конкурса. Также индикатор расчитывается один раз в сутки, если выполнены все условия для его расчета.


Поля для расчета
================

Для расчета индикатора используются следующие поля API модуля системы гос. закупок:

- ``data.parties.id``
- ``data.parties.identifier.scheme``
- ``data.parties.identifier.id``
- ``data.parties.roles``
- ``data.tender.datePublished``

Формула расчета
===============

1. В исследуемой процедуре выбираем всех поставщиков-победителей, таких ``data.parties``, у которых ``data.parties.roles = 'supplier'``.
2. Выбираем их идентификаторы - конкатенация ``data.parties.identifier.scheme`` и ``data.parties.identifier.id``.
3. Выбираем идентификатор закупающей организации (конкатенация ``data.parties.identifier.scheme`` и ``data.parties.identifier.id``), такой, для которого ``data.parties.roles = 'buyer, procuringEntity'``.
4. Выбираем дату оглашения процедуры ``data.tender.datePublished``.
5. Для каждого идентификатора поставщика проводим поиск в аналитической таблице :ref:`tbl_PESupplier` строк, в которых дата меньше даты оглашения нашей процедуры.
6. Если в найденных строках присутствует только один уникальный идентификатор закупающей организации и он совпадает с идентификатором закупающей организации из исследуемой процедуры, индикатор принимает значение ``1``. Расчет заканчивается.
7. Если мы дошли до этого пункта, индикатор принимает значение ``0``. Расчет заканчивается.

Факторы, которые влияют на корректное срабатывание индикатора
=============================================================

Индикатор может срабатывать неправильно, если закупающая организация не отображает на портале все фактически пройденные этапы процедуры закупки.
