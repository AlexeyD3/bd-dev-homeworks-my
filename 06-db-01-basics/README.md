# Домашнее задание к занятию 1. «Типы и структура СУБД»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/virt-11/additional).

## Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД 
лучше выбрать для хранения определённых данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:

- электронные чеки в json-виде,
> Документоориентированные: MongoDB, CouchDB. Т.к. схема данных не может быть спрогнозированна.
- склады и автомобильные дороги для логистической компании,
> Графовые: Neo4j OrientDB т.к. нужна установка связей между данными, либо реляционные решения типа MySQL со связями между таблицами
- генеалогические деревья,
> думаю для этого случая так же подойдут графовые БД
- кэш идентификаторов клиентов с ограниченным временем жизни для движка аутенфикации,
> Redis, Memcached
- отношения клиент-покупка для интернет-магазина.
> БД ключ-значение: Redis, Memcached (+ БД для постоянного хранения данных) или реляционные СУБД типа MySQL

Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.

## Задача 2

Вы создали распределённое высоконагруженное приложение и хотите классифицировать его согласно 
CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если 
(каждый пункт — это отдельная реализация вашей системы и для каждого пункта нужно привести классификацию):

- данные записываются на все узлы с задержкой до часа (асинхронная запись);
> AP - доступность и устойчивость к распределению, данные в силу задержки неконсистентны
> По PACELC: PA/EL - при наличии сетевого соединения упор идёт на доступность, а при отсутствии на скорость с которой "клиент" получает ответ (пусть и неконсистентный в силу задержки до часа в актуальности данных)

- при сетевых сбоях система может разделиться на 2 раздельных кластера;
> AP (или даже просто P) - доступна и устойчива к распределению, но неконсистентна т.к. данные в отдельных кластерах могут различаться
> По PACELC: PA/EL - 2 раздельных кластера явно ухудшают консистентность в угоду доступности

- система может не прислать корректный ответ или сбросить соединение.
> CP (возможно даже CA) - система обеспечивает консистентность данных, но теряет доступность в случае когда теряет связь с частью узлов
> По PACELC: PC/EC максимальная консистентность данных, не присылает клиенту ответ в угоду Latency, стараясь максимально приблизиться к парадигме ACID.

Согласно PACELC-теореме как бы вы классифицировали эти реализации?

## Задача 3

Могут ли в одной системе сочетаться принципы BASE и ACID? Почему?

> Не могут, т.к. эти принципы преследуют разные цели и имеют противоположности в своих основных аспектах, ACID консистентна и атомарна в любой момент времени, обеспечивает жизнеспособность транзакций и их изолированность, 
  BASE же в свою очередь жертвует этими свойставми в угоду доступности, сохраняя при этом конечную консистентность.

## Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

- фиксация некоторых значений с временем жизни,
- реакция на истечение таймаута.

Вы слышали о key-value-хранилище, которое имеет механизм Pub/Sub. 
Что это за система? Какие минусы выбора этой системы?

> Система в которой данные храняться в формате ключ-значение.
> Механизм Pub/Sub - издатель-подписчик или издатель-(брокер/шина)-подписчик в этом механизме сообщения публикуются в каналы, подписчики могут подписываться на один или несколько каналов для получения сообщения, а так же в определенных случаях сами выступать издателями.
> Издатель события и получатель не контактируют напрямую и не обмениваются запросами. Так же этот механизм никак не связан с пространством ключей

> Минусы:
> На мой взгляд в лекции и доп материалах недостаточно информации, чтобы полноценно ответить на данный вопрос, но если рассматривать Redis то минусом можно назвать семантику доставки, заключающуюся в том, 
  что сообщение отправляется всего один раз и если подписчик в этот момент не в состоянии обработать сообщение или не доступен, то сообщение навсегда теряется (для более надежной доставки есть Redis stream).
> Возможны проблемы с масштабируемостью при увеличении нагрузки, в случае с Redis это связанно с однопоточностью.
> Необходимо дополнительное решение для персистенции данных.

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

