# Music.Reviews.Corpus

Корпус состоит из англоязычных рецензий на музыкальные альбомы в период с января 2012 по октябрь 2018, собранных с сайтов pitchfork.com, exclaim.ca и slantmagazine.com. 
Корпус хранится в виде базы данных с метаданными рецензий и директории с текстовыми файлами

## Структура базы данных
База данных sqilte хранится в файле reviews.db, состоит из одной таблицы reviews вида:

| Поле | Описание |
|------|----------|
| _id | guid статьи |
| album | название альбома |
| artist | имя исполнителя/название группы |
| rating | оценка автором рецении альбома по 100-балльной шкале |
| genre | жанр альбома из рецензии |
| author | автор рецензии |
| album_year | год выхода альбома |
| review_year | год публикации рецензии |
| source | Pitchfork, Exclaim или Slant Magazine |
| url | ссылка на рецензию |

## Способ хранения рецензий
Каждая статья расположена в отдельном файле в виде сплошного текста, название файла совпадает со значением _id рецензии в базе reviews.db.

## Сбор данных
Сбор данных осуществляется с помощью [scrapy](https://scrapy.org/) тремя spider'ами: Pitchfork_spider, Exclaim_spider, Slantmagazine_spider. 

## Статистика корпуса
| Показатель | Значение |
|------|----------|
| количество текстов | 20 943 |
| количество источников | 3 (Pitchfork, Slant Magazine, Exclaim) |
| количество текстов Pitchfork | 8 172 |
| количество текстов Slant Magazine | 1 059 |
| количество текстов Exclaim | 11 712 |
| количество токенов | 9 706 356 |
| количество предложений | 350 785 |

## Токенизация и сегментация
Токенизация и сегментация осуществляются с помощью библиотеки NLTK: для разбиения на предложения использовался sent_tokenize, на токены - SpaceTokenizer. Файлу с оригнальным текстом с определённым guid'ом соответствует файл <guid>_sentences с разбиением этого текста на предложения: одна строка - одно предложение. В файле <guid>_tokens хранятся кортежи с парами значений (индекс начала токена, индекс конца токена)  

## Поиск по корпусу
Консольное приложение. Поиск реализован с помощью библиотеки Whoosh. Поиск проводится по самим текстам рецензий, по названиям альбомов, по исполнителям/группам и по изданиям рецензий. Форма поискового запроса: "field":"term/phrase" "OPERATOR" "field":"term/phrase" "OPERATOR" ... , где field - название поля из множества {"text", "album", "artist", "source"}, по которому происходит поиск терма, operator - логический оператор большими буквами, term/phrase - искомое слово или высказывание, ? и * можно заменять один любой символ или любое количество любых символов. Если поля не указаны ни для одного терма, то поиск по умолчанию происходит по текстам рецензий.
  
 Например:
 artist:Slowdive OR text:shoegaze AND album:Soulvaki - найти все рецензии, у которых исполнитель - Slowdive или альбом Soulvaki и при этом в тексте встречается shoegaze 

Результат выводится в формате "Terms {terms set} were found in <id рецензии>" для всех совпадений

## Классификация

Выделено три класса: flop - отрицательные рецензии с оценками от 0 до 40, meh - смешанные рецензии с оценками от 41 до 60, top - положительные рецензии с оценками от 61 до 100. В качестве признака были взяты 3-грамы слов. Классификатор - логистическая регрессия. Оценка качества классификации - средние значения F1_micro и F1_weighted на кросс-валидации с пятью фолдами.

F1_micro = 82%, F1_weighted = 71%
