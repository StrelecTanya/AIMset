# AIMset
Summer work in BTC - отчет

Моей основной задачей на летней практике было повторение статьи 2020 года «How to choose sets of ancestry informative markers: A supervised feature selection approach»1. В этой статье приводился новый способ нахождение информативных маркеров происхождения (AIM), которые могут быть полезны в криминалистике и популяционной генетике (эффективно различать популяции по генетическим данным). Основные нововведения ученых для поиска стали использование наивного байесого классификатора и logloss вместо ошибки неправильной классификации при отборе AIM в AIMset. В качестве датасета они использовали открытые данные из проекта 1000 геномов. Их код для отбора AIMsets написан на R с использованием пакета vcfR для работы с vcf файлами (https://github.com/fbaumdicker/AIMsetfinder/tree/master). Моя работа заключалась в том, чтобы воспроизвести их код на тех же данных, но с использованием софта hail (вместо vcfR).
Основные шаги по воспроизведение эксперимента:
1.	Шаг Prepare (не обязательный) - предобработка файлов - удаляем группу AMR (смешанные американцы).
2.	Шаг 0 - фильтрация vcf файла и разделение образцов на тестовую и обучающую выборки.
В ходе фильтрации мы оставляли только биаллельные варианты и однонуклеотидные полиморфизмы (SNP). При делении образцов на тестовую и обучающую отбиралось по 100 образцов из каждой группы в тестовую выборку, остальные образцы собирались в обучающую. Например, в проекте 1000 геномов 2504 образца и 5 популяций ('AFR' – африканцы, 'AMR' – американцы, 'EAS' – восточные азиаты, 'EUR'- европейцы, 'SAS' – южные азиаты) и при нашем делении у нас получится тестовая выборка, состоящая из 500 образцов и обучающая из 2004.

3.	Шаг 1 - подсчет частоты встречаемости SNP в популяции
Считаем частоты каждого SNP в популяциях и получаем подобную таблицу.
 
4.	Шаг 2 - фильтруем SNP по информативности и получаем vcf файл только с информативными вариантами
Информативность считается по этой формуле
 
, где p – это частота встречаемости SNP (посчитана в шаге 1),  К – это количество групп, а h(x) ≔ x log(x) + (1 − x) log(1 − x).
После того, как мы посчитали информативность – отбираем 10% самых информативных в новый vcf файл.
5.	Шаг 3 – для более быстрой обработки данных дробим полученный vcf файл на более мелкие с шагом 10.000.
6.	Шаг 4 - считаем количество SNP в vcf файле
7.	Шаг 5 - переводим полученные vcf файлы на шаге 3 в более быстрый для чтения формат - hail matrix table – и пересчитываем генотипы – вместо 0|0 пишем 0, 0|1 = 1 и 1|1 = 2
8.	Шаг 6 – на обучающей выборке с помощью байесого наивного классификатора находим AIMset с заданным размером (мы сами задаем в начале программы сколько информативных маркеров происхождения мы хотим найти в результате работы программы).
