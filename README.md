Решение задачи классификации людей по фотографии лица
====================================================

(на фотографиях автор проекта)

Предобработка
-------------
Перед началом обучения нужно подготовить обучающую выборку. 
Я взял SoF Dataset , который представляет набор из 1000 фотографий лиц 112 людей. 

Чтобы воспользоваться этими фотографиями, их нужно обработать, т.е. вырезать область лица. Для этой задачи существует детектор лица на базе библиотеки dlib. Он работает так :  
1.	строим плотности распределения цветов точек для фона и для объекта;
2.	изображение примеряется на шаблон, для каждой точки вне шаблона вычисляется вероятность принадлежности к фону и к объекту;
3.	используем эти вероятности для вычисления "расстояния" между точками и запускаем алгоритм поиска кратчайших расстояний на графе.
В итоге точки, которые ближе к объекту (лицу), относим к нему и, соответственно, те, что ближе к фону, относим к фону.
Определив нужные контуры лица алгоритм выделяет ключевые точки на изображении. 

 
 Ключевые точки лица
 
 ![face_dots](/images/res.jpg)

Среди них необходимы только 3 точки, которые главным образом определяют плоскость лица.

 ![face_dots_main](/images/res2.jpg)

 

Эти три точки нужны для того, чтобы получить матрицу отображения исходного лица на заранее известную область, 
которая будет для всех изображений одинакова.
Получив координаты 3 опорных точек, необходимо получить уравнения для Аффинного преобразования.
Пусть (x1,y1) (x2,y3) (x3,y3) — исходные точки, которые мы хотим перевести в (x’1,y’1) (x2',y'2) (x’3,y’3). 
Тогда аффинное преобразование, выраженное матрицей T можно найти из соотношения:

![afin](/images/afin.JPG)
 

Результат всех действий – обрезанные фотографии с определенным размером и с одинаковым расположением главных точек лица:

![face](/images/res3.jpg)
 
Создание модели
---------------

После создания обучающей выборки нужно создать модель сети. Так как VGG16  заточен на классификацию 1000 классов, то для решения задачи распознавания, например, 28 человек (по 11-20 фоток на человека), нужно заменить в последнем сплошном слое 1000 нейронов на 28. 
По итогу, первому сверточному слою модели будет подаваться массив из тензоров размером 128х128х3 , где 128 – размер изображения по двум осям, а 3 – количество каналов изображения (в данном случае 3 так как ргб)
Параметры сети установлены такие:
1) 	количество прогонов 18
2)	размер обучающей за раз выборки  12
3)	метрика ответа categotical_accuracy
4)	метрика ошибки categorical_crossentropy
5)	функция минимизации ошибки RMSprop


Результат
-----------

 Обучение 18 эпох:
 
 
 ![result](/images/result.JPG)
 
 Проверки на тестовой выборке:
 
 ![test](/images/test.JPG)
 
Итог
----

Готовая модель VGG16 хорошо подходит для решения задачи классификации, однако она 
довольно тяжелая. Из-за этого процесс обучения становится более требовательным по времени.
Полученные результаты говорят о том, что на модель хорошо предсказывает результат, однако под конец обучения val_categorical_accruracy равен 1,
 что говорит о наличии переобучения. Также об этом свидетельстует test_accuracy , равный 1. Как итогЖ нужно брать естовую выборку больше и 
 лучше перемешивать исходный датасет. 
 
  
 
 
 
