# Комплексный проект анализа пользователей

### Стек:
Python, Pandas, NumPy, Plotly, Seaborn, Matplotlib, bootstrap, SQL, pandahouse, A\B-тестирование.

## A/B–тестирование:
В ходе тестирования одной гипотезы целевой группе была предложена новая механика оплаты услуг на сайте, у контрольной группы оставалась базовая механика. Необходимо проанализировать итоги эксперимента и сделать вывод, стоит ли запускать новую механику оплаты на всех пользователей.

**Этапы**:
1. *Распределяем и выделяем* 5 различных выборок по признаку действий пользователей:
- Активные участники эксперимента;
- Оплатившие участники эксперимента;
- Не активные участники эксперимента, но оплатившие;
- Активные участники эксперимента, но не оплатившие;
- Активные и оплатившие участники эксперимента.

2. *Проводим* анализ каждой выборки (размерность, распределение оплаты по группам, описательная статистика групп), затем для дальнейшего анализа использовала только 1 полученную выборку (активные и оплатившие участники эксперимента).
  
**Получаем следующие выводы, исходя из анализа выборок**:
- Количество участников двух групп (объём выборки) сильно разнится: 20% участников от общего количества в группе А и 80% участников в группе В. Для точной статистической оценки необходимо распределение групп 50% на 50%.
- Стандартное отклонение в группе В в отличие от группы уменьшилось, что говорит о том, что разброс значений уменьшился.
- Количество пользователей, которые произвели оплату в целевой группе увеличилось на 402% в сравнении с тестовой группой.

3. *Определяем* далее ключевые метрики для анализа группы и статистической значимости показателей:
Поскольку нам необходимо оценить, правда ли, что новый алгоритм улучшил качество сервиса. Будем использовать:

- `CR` — коэффициент конверсии (= число посетителей, совершивших целевое действие / Общее кол-во посетителей),
- `ARPU` — доход от пользователя (= доход за определённый период / количество пользователей, которые купили товар за это время),
- `ARPPU` — доход от платной подписки (= доход за период / число платных подписок за это время).

4. *Производим* расчёты каждой метрики и *проводим* гипотезы с помощью статистического теста.

*Первая метрика* - `CR` (проверка с помощью метода Хи-квадрат для категориальных переменных (в нашем случае 0 - нет оплаты, rev>0 - оплата есть))

**Результат**: Поскольку p-value меньше 0,5, то мы не отклоняем нулевую гипотезу. Следовательно, отличий между конверсиями в покупку между тестовой и контрольной группами нет, новая механика оплаты услуг на сайте не влияет на снижение конверсии.

*Вторая метрика* - `ARPU` (проверка с помощью метода бутстрап, который более точен и работает с большими выбросами в данных).

**Результат**: Поскольку p-value меньше 0,5, а 0 попадает в доверительный интервал, то мы не можем отклонить нулевую гипотезу. Следовательно, отличия между доходность от одного пользователя между тестовой и контрольной группами значимо не отличаются, новая механика оплаты услуг на сайте может не влиять на доходность одного пользователя.

*Третья метрика* - `ARPPU` (проверка с помощью метода бутстрап).

**Результат**: Поскольку p-value меньше 0,5, а 0 не входит в доверительный интервал, то мы отклоняем нулевую гипотезу. Следовательно, отличия между доходностью от платящего пользователя между тестовой и контрольной группами существует, новая механика оплаты услуг на сайте влияет на доходность одного платящего пользователя.

#### Вывод:
Таким образом, исходя из проведённых тестов, новая механика оплаты на сайте оказала положительное влияние на метрики ARPU (однако данная метрика не показала статистически значимых различий между группами), ARPPU, конверсия в покупку (CR) имела отрицательный эффект, однако, является статистически незначимой метрикой, то есть не оказывает влияния на наши выборки.

Следует распределить данные в подвыборках равномерно для более точной оценки поставленной гипотезы.


## SQL: 
Необходимо написать оптимальный запрос, который даст информацию о количестве очень усердных студентов.

**Этапы**:
- Подключаемся к таблице из Clickhouse с помощью pandahouse.
- Первая часть задания: Пишем запрос в 2 этапа для получения информации о количестве очень усердных студентов:
  1. Найдём интервал дат, возьмём текущий месяц.
  2. Найдём количество усердных учеников, которые решили хотя бы 20 и более заданий.

**Результат** : Количество усердных учеников равно 136 как за текущий месяц (октябрь), так и за всё время.

#### Оптимизация воронки
Необходимо в одном запросе выгрузить следующую информацию о группах пользователей:

- ARPU 
- ARPAU 
- CR в покупку 
- СR активного пользователя в покупку 
- CR пользователя из активности по математике (subject = ’math’) в покупку курса по математике
- ARPU считается относительно всех пользователей, попавших в группы.

**Этапы**:
1. Формируем из таблицы датафрейм с необходимыми колонками (подготовка данных).
2. Находим количество всех баллов по каждому студенту и количество баллов у студента по математике.
3. Пишем запрос по подсчёту каждой метрики.

## Python
1. Реализовать функцию, которая будет автоматически подгружать информацию из дополнительного файла groups_add.csv и на основании дополнительных параметров пересчитывать метрики.
2. Реализовать функцию, которая будет строить графики по получаемым метрикам.

**Этапы**:
1. Подклучаем модули `requests` и `urllib.parse`
2. Пишем функцию с использованием api и получаем загрузочную ссылку, присваиваем каждому датафрейму верные ссылки для чтения.
3. Пишем функцию с автоматическим пересчётом метрик.
4. Пишем функцию визуализацию метрик.
