# GB_Spark_Streaming
Здесь представлен "черновой" код для проекта курса "Потоковая обработка данных".  
Используемые технологии: Spark Structured Streaming + Kafka + Cassandra.  
За основу взят датасет с информацией о статьях на Хабре - habr_data.csv 

Датасет содержит 40 столбцов. Общий вид из Цеппелина:
![](https://github.com/Filkin-S/GB_Spark_Streaming/blob/main/imgs/zeppelin.jpg)

Общая идея.
В исторических данных у каждой статьи есть мета информация о тексте (длина текста, кол-во слов, предложений, первые 5 предложений, 10 самых используемых слов и т.п.), а также оценки статьи - рейтинг, кол-во комментариев, кол-во позитивных\негативных голосов.

Задача: на основании рейтинга прошлых статей, разделить их на 4 класса (“A”, “B”, “C”, “D”) и попытаться предсказать рейтинг только, что опубликованной статьи по мета информации о тексте. Рейтинг использовать, например, для рекомендации \ продвижения новых «хороших» статей на главной странице.

Для симуляции идеи изначальный датасет разделил: 90% - история на обучение, 10% новые статьи в кафку (удалив оценки, оставив только данные о тексте).

Чтобы было, что джойнить, создал author_id и убрал из данных author_name и author_type в отдельную таблицу в Кассандре habr_users.

Для упрощения обучал только по полю «ten_most_common_words», но можно много полей сразу (и еще фичи генерировать). Анализ как лучше разбивать на классы (распределение статей по рейтингу), а также оценку модели метрикой провел заранее в Цеппелине.

Весь процесс несложен. Получаем из Кафки данные о «новой» статье, парсим. Т.к. обучение только по одному полю, преобразований данных внутри forEachBatch не требуется, сразу передаем данные в модель. Добавляем в данные имя и тип пользователя джойном с данными из таблицы Кассандры habr_users по полю  author_id. Оставляем только основные колонки (без мета-информации) и записываем данные в таблицу Кассандры habr_recs.

Итог выглядит так, Спарк:
![](https://github.com/Filkin-S/GB_Spark_Streaming/blob/main/imgs/pyspark.jpg)

Кассандра:
![](https://github.com/Filkin-S/GB_Spark_Streaming/blob/main/imgs/cassandra.jpg)
