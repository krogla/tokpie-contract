# Аудит смарт контрактов TKP.

Статус - в работе

## Общие замечания

Код контрактов проверен на предмет программных закладок и критических ошибок спобоных привести к потере денег инвесторами.

* Присутствует возможность пересылки токенов до периода окончания ICO. Наличие бонусных периодов, позволяет держателям продавать купленные ранее токены во время ICO по цене ниже, чем в контракте.
* Присутствует возможность обойти ограничение по максимальной доле инветосров при покупке токенов, если один инвестор осуществит покупку с нескольких пренадлежащих ему аккаунтов.

Токен TKP наследует функционал контрактов ERC20Basic, ERC20, BasicToken, StandardToken, MintableToken, а так же Ownable - реализующего механизм владения контрактом и позволяющего устновить агента, которому разрешено выпускать токены, и Pauseble - позволяющиго поставить распродажу на паузу.
Распродажа токенов TKP осуществляется через контракт preICO, наследующего функционал FinalizableCrowdsale, и ICO. Оба так же наследует функционал контрактов Ownable - реализующего механизм владения контрактом и Pauseble - позволяющиго поставить распродажу на паузу.
Финализация ICO осуществляется с помощью контракта postICO, в котором производится дополнительная эмиссия токенов, контракт наследует Ownable - реализующего механизм владения контрактом.
Некоторые контракты используют методы библиотеки MathLib для безопасных вычислений.

Не зафиксирована версия компилятора, устновка pragma solidity допускает более новые версии компилятора, однако уже сейчас компиляция в них вызывает множественные warning сообщения.
Рекомендация: устновить pragma solidity 0.4.20;  (без ^)

## Обзор контрактов

MathLib

1) (Не критично) Использование оператора require вместо assert позволит сэкономить газ в случае ошибочных вычислений в методах контракта.

ERC20Basic

1) (Примечание) Упрощенная веерсия интерфейса ERC20

ERC20

1) (Примечание) ERC20 интерфейс

ShortAddressProtection

1) (Не критично) Использование оператора require вместо assert позволит сэкономить газ в случае ошибочных вычислений в методах контракта.

BasicToken

1) (Важно) Контракты ICO и preICO, использующие данный токен, наследуют модификаторы контраткта Pausable, которые запрещают покупку токенов во время паузы, однако функция transfer это не учитывает, что может позволить участникам перепродавать уже имеющиеся токены во время остановленного этапа preICO или ICO.   

StandardToken

1) (Важно) функция transferFrom не учитывает паузу в контрактах ICO и preICO (см. замечание к BasicToken)   

Ownable

1) (Примечание) Владелец контракта может назначить другого владельца.

MintableToken

1) (Важно) Выпуск токенов может производиться только с адреса агента
2) (Важно) Владелец может установить любой адрес агента, потенциально есть возможно выпустить токены со стороннего контракта до вызова завершающей функии в postICO.    

Token

1) (примечание) Задаются публичные параметры токена

Pausable

1) Наследует Ownable
2) (Примечание) Контракт содержит функционал, позволяющий его владельцу переключать триггер paused во включеное и выключеное положение, и модификаторы позволяющие вызов некоторых функций только в режиме паузы или только в режиме вне паузы.

FinalizableCrowdsale

1) Наследует Pausable
2) (Примечание) Только владелец может может завершить краудсейл

RefundVault

1) (Примечание) контракт хранит все средства на время краудсейла, владельцем vault является контракт preICO

preICO

1) Наследует FinalizableCrowdsale, владелец может сменить владельца и ставить на паузу
2) (Примечание) жестко прописано ограничение на минимальное количество покупаемых токенов - 100
3) (Примечание) в момент деплоя задается ограничение на максимальное количество полученных средств от каждого инвестора
4) (Важно) токены можно перепродавать вне контракта preICO во время паузы (см. замечание к BasicToken), что может позволить владельцу злоупотребить возможностью поставить ICO на паузу
5) (Важно) ограничения на покупку токенов можно обойти если переводить токены вне контракта preICO, например можно купить по 100000 токенов с нескольких адресов и перевести все на один 
6) (Примечание) После деплоя контракта, для корректной работы, надо установить его адрес в качестве saleAgent в токене
7) (Средне) В случае если preICO несостоится (не закроется softCap), предусмотрен возврат средств инвесторам, но не предусмотрен возврат/сжигание токенов, в случае если, например, планируются повторные попытки ICO.

ICO

1) Наследует Pausable, владелец может сменить владельца и ставить на паузу
2) (Примечание) жестко прописано ограничение на минимальное количество покупаемых токенов - 100
3) (Примечание) в момент деплоя задается ограничение на максимальное количество полученных средств от каждого инвестора
4) (Важно) токены можно перепродавать вне контракта preICO во время паузы (см. замечание к BasicToken), что может позволить владельцу злоупотребить возможностью поставить ICO на паузу
5) (Важно) ограничения на покупку токенов можно обойти если переводить токены вне контракта preICO (см. коментарий к preICO)
6) (Важно) Нет ограгичений по отправке (покупке/продаже) токенов в периоды проведения ICO (с учетом того, что в данном ICO предусмотрены периоды со скидками, покупатель может купить токены со скидкой и с профитом перепродать их на следующих периодах по более низкой, чем установленная в контракте, цене).
7) (Не критично) рассчитанные периоды повышения цены токена смещены на 2 минуты от старта ICO, т.е. фактически дествуют от 0:01:00 в день старта и далее в кажом периоде до 23:59 6-го дня, соответственно начало периода не в 0:00:00 первого дня очередного периода, а в 23:59:00 последнего дня предыдущего периода 
8) (Среднее) После деплоя контракта, для корректной работы, надо установить его адрес в качестве saleAgent в токене
9) (Примечание) При деплое контракта надо точно задать даты начала и конца ICO, чтобы не пересеклись в preICO

postICO

1) Наследует Ownable, владелец может сменить владельца
2) (Примечание) При деплое контракта надо точно задать дату конца ICO, чтобы не пересеклась c ICO
3) (Среднее) После деплоя контракта, для корректной работы, надо установить его адрес в качестве saleAgent в токене
4) (Не критично) функции claim имеют неоптимальный код и в некоторых случаях незначительный перерасход газа 

Контракт Migrations не связан с остальными контрактами.

## Предупреждение об ответственности

Этот аудит касается только исходных кодов смарт контрактов и не должен рассматриваться как одобрение платформы, команды или компании.

## Авторы

Аудит проведен Михаилом Семёнкиным, команда [EthereumWorks](https://github.com/EthereumWorks)
По вопросам проведения аудитов и разработки смарт контрактов обращайтесь: Telegram - @SlavaPoe, Skype - v.poskonin (MousePo).