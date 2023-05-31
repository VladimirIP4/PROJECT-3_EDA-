# PROJECT-3_EDA

## [SF-DST] Booking reviews
Прогнозирование рейтинга отеля на Booking

# Описание:

Представьте, что вы работаете датасаентистом в компании Booking. Одна из проблем компании — это нечестные отели, которые накручивают себе рейтинг. Одним из способов нахождения таких отелей является построение модели, которая предсказывает рейтинг отеля. Если предсказания модели сильно отличаются от фактического результата, то, возможно, отель играет нечестно, и его стоит проверить. Вам поставлена задача создать такую модель.

# Результат: MAPE: 0.12534952556169107

# Этапы проекта:

## 1. Загрузка данных и исследование
Загружаем данные в отдельные датасеты. Временно объединяем датасеты для обработки данных на следующих этапах. Помечаем датасеты признаком sample, для дальнейшего разделения перед этапом моделирования. Знакомимся со структурой данных:
- в датасете есть как числовые, так и строковые признаки
- один из признаков является датой - далее переведем его в формат datetime для удобства использования
- есть два признака с одинковым количеством пропусков, скорее всего это одни и те же строки, исходя из количества пропусков будем их заполнять
- среди числовых признаков только average_score напоминает нормальное распределение
- признаки lat и lng визуально и логически непригодны для наших целей, но их можно задействовать при проектировании новых признаков

## 2. Очистка данных
### 2.1. Очистка от дубликатов
Анализ списка дубликатов показал, что за дубликаты метод все же принимает строки с отличающимися признаками. При этом метрика улучшается на 0,024% при их удалении. Не исключаю, что метод восприниматет неполные дубликаты, но при этом и сам алгоритм обучения их также воспринимает некорректно. С учетом отсутствия более глубокого понимания принципов работы ML на данном этапе и относительно небольшого количества дубликатов, принимаю решение их удалить.

### 2.2. Очистка от пропусков
Проверяем данные на пропуски. Признаки lat и lng имеют по 3268 пропусков. Выясняем, что всего 17 отелей имеют пропуски координат. Заполним их вручную в отдельную таблицу и внесем информацию в датасет.

## 3. Разведывательный анализ данных
### 3.1. План действий по каждому из нечисловых признаков
- **hotel_address:** извлечь 2 признака - страну (для обучения модели) и город (для дальнейшего вычисления расстояния от центра)
- **review_date:** создать новый признак сезонности
- **hotel_name:** сам по себе признак бесполезен, удаляем (можно выделить сетевые отели, но это очень объемная работа) 
- **reviewer_nationality:** скорее всего полезный признак, приведем к необходимому для кодирования виду
- **negative_review:** измеряем эмоциональную оценку
- **positive_review:** измеряем эмоциональную оценку
- **tags:** анализируем теги, создаем несколько новых признаков
- **days_since_review:** переводим в число (дней)
- **lat и lng:** создаем признак "расстояние до центра города"

### 3.2. Проектирование новых признаков
### hotel_address
Из данного признака можно достать несколько параметров, выясняется что все отели представлены 6 странами по одному городу в каждой, т.к. в данном случае признаки страны и города идентичны, то для обучения остановимся на стране, а город пока оставим для определения координат его центра и создания в дальнейшем признака расстояния отеля до центра. Остальные данные признака не задействуем.

### review_date
Т.к. у нас нет данных о датах проживания автора отзыва, то идея создать признак "уикенд - да/нет" в данном случае не будет корректным, ведь мы не можем быть уверены когда именно (через сколько дней) был оставлен отзыв. Но мы можем предположить, что отзыв оставлен через несколько дней, максимум через 1-2 недели, поэтому попробуем создать признак сезонности - довольно важный параметр в отельном бизнесе. В среднем по Европе сезонность по месяцам года следующая (в целом как и у нас в РФ) - high (июнь-август), low (декабрь-февраль) и shoulder (март-май, сентябрь-ноябрь).

### hotel_name
В идеале выделить сети и одиночные, но это очень объемная работа (не всегда имя сети присутствует в названии, есть много "подбрендов" и т.п.), для данного проекта сам по себе признак бесполезен, поэтому удаляем

### reviewer_nationality
Приведем к необходимому для кодирования виду - удалим лишне пробелы и оставим ТОП-100, а остальные пометим как "другие"

### negative_review
Измеряем эмоциональную оценку при помощи SentimentIntensityAnalyzer. Дальнейшая оценка значимости признаков для модели а также ознакомление с наиболее успешными работами на Kaggle позвоят сделать вывод о том, что признаки negative_review и positive_review, а также тщательная их очистка и подготовка (в т.ч. с помощью других инструментов) позволяют добится наилучших результатов в показателе MAPE.
На данном этапе я принял решение сделать эмоцинальную оценку "сырых" данных и создать на ее основе один признак по параметру 'compound'. 
Далее при желании достигнуть лучших показателей в соревновании следует большую часть усилий прилагать именно к обработке этих признаков.

### positive_review
Аналогично поступаем с положительными отзывами

### tags
Анализ тэгов привел к следующему заключению - берем наиболее часто встречающиеся и выделяем по типу тэгов 3 дополнительных признака:
- trip_typе - может быть leisure или business, если нет этих тэгов ставим NaN
- mobile_submitt - тэг либо есть, либо его нет (1 или 0)
- guests - разделим на solo, couple, group и with children

Дальнейшая оценка покажет, что данные признаки не сильно эффективны в обучении.

### days_since_review
С логической точки зрения данный признак не должен быть полезен, но на практике он довольно ценен (по графику в конце моделирования) и улучшает метрику на 0,125%. Есть предположение, что он, например, косвенно отображает возраст отеля (т.к. значения довольно большие). А в отельном бизнесе возраст отеля (износ номерного фонда, или наоборот рост популярности и узнаваемости) может играть важную роль. Оставляем признак, вынув из изначального столбца число дней.

### lat и lng
Используем для вычисления расстояния отеля до центра города, получим координаты центра каждого города и вычислим для каждого отеля расстояние от него. Оценка покажет, что признак довольно полезный.

### 3.3. Преобразование признаков

### 3.4. Отбор признаков (мультиколлениарность)
По итогам оценки тепловой карты относительно сильная корреляция прослеживается только между признаками additional_number_of_scoring и total_number_of_reviews. Исключение одного из них при обучении на стационарном ПК дает улучшение MAPE на 0,048%. Однако при обучении на Kaggle показатель MAPE ухудшился. Не трогаем эти признаки.

## 4. Моделирование

### 4.1. Подготовка данных

### 4.2. Обучение модели и предсказание, оценка MAPE

### 4.3. Прогноз тестовых данных, запись submission для Kaggle
