# Требования к входным данным: 

<code> 1. Требования к фото
* Фото делается на камеру, находящуюся на расстоянии 45 см от пола (допустима погрешность в 5 см), камера смотрит перпендикулярно естественной поверхности 
* Разрешение фото: 4032 x 3024
* Фон на фотографии должен быть целиком заполнен белым листом бумаги
* Освещение должно быть равномерным
2. Требования к многоугольнику
* Многоугольник задается массивом координат вершин на естественной плоскости
* Координаты должны быть записаны в csv-файл в виде четырех строчек. Каждая строчка соответствует одной координате. Внутри строчки записаны сначала координата х, потом координата y через запятую 
* Допускаются только координаты следующего вида: [[x,y],[x,y+h],[x+h,y+h],[x+h,y]] 
3. Требования к расположению объектов внутри многоугольника
* Предметы не могут перекрывать друг друга
* Один предмет не может находиться на фотографии больше одного раза
</code>  

# Требования к выходным данным:
Программа выводит одну строку:
* "Да" --- если предметы со входных изображений можно разместить в том же положении, что и на шаблонах, на поверхности с учетом всех требований из пункта 3 так, чтобы они находились целиком внутри многоугольника, либо задевали его границу.
* "Нет" --- если предметы со входных изображений нельзя разместить в том же положении, что и на шаблонах, на поверхности с учетом всех требований из пункта 3 так, чтобы они находились целиком внутри многоугольника, либо задевали его границу.

# Фотографии предметов:
![image](https://user-images.githubusercontent.com/55063415/132926718-31eb092c-6b09-42ff-ad45-4f06e41f09b6.png)
![image](https://user-images.githubusercontent.com/55063415/132926729-9505fac4-bf1e-42aa-8900-2fd45cc2d498.png)
![image](https://user-images.githubusercontent.com/55063415/132926755-37429755-686d-4b22-bc77-027dcd249aab.png)
![image](https://user-images.githubusercontent.com/55063415/132926764-ebb54b5c-55bd-4e86-884b-ccdc6ef2414f.png)
![image](https://user-images.githubusercontent.com/55063415/132926778-4a92046c-94f4-49e0-85e1-98d8ee3d0062.png)
![image](https://user-images.githubusercontent.com/55063415/132926788-11f4ff92-584f-447b-a82f-d2bfad6aca99.png)
![image](https://user-images.githubusercontent.com/55063415/132926793-04bc56d9-483c-41a6-bd39-6e02cfa70535.png)
![image](https://user-images.githubusercontent.com/55063415/132926797-f9e93c62-02de-4b46-b06c-aafb038ec837.png)
![image](https://user-images.githubusercontent.com/55063415/132926888-f6147e6f-dcaa-403c-ba19-743636ab91a9.png)
![image](https://user-images.githubusercontent.com/55063415/132953629-7733353f-fb95-4110-87f5-f907574a2b83.png)


# Фотография поверхности:
![image](https://user-images.githubusercontent.com/55063415/132927007-155c7dcf-3b5d-4b81-b49a-88938cdf7f6d.png)

# Алгоритм решения:


## Подготовительная работа
<code>
1. Зафиксируем некоторое пороговое значение для алгорита Харриса. <br>


2. Производим следущие действия с каждым из шаблонов: <br>


* Используем алгоритм Canny для нахождения границ объекта шаблона <br>


* Для найденных границ используем морфологическое преобразование binary_closing, чтобы получить четкие замкнутые границы. <br>


* С помощью преобразования Харриса с пороговым значением, зафиксированным на шаге 1, находим особые точки на полученной карте границ (будем использовать меру Ферстнера). <br>


* Опишем объект шаблона приближающим его прямоугольником. Метод cv.minAreaRect(). <br>


</code>


## Распознавание объектов на входном фото

<code>

1. Используем алгоритм Canny для нахождения границ объектов на входном изображении. <br>


2. Для найденных границ используем морфологическое преобразование binary_closing чтобы получить четкие замкнутые границы. <br>  


3. Применяем к полученной в пункте 2 карте границ алгоритм Харриса с использованием адаптивного радиуса и пороговым значением, выбранным во время подготовительной работы. Будем использовать меру Ферстнера. Мы получили карту особых точек на входном ихображении. <br>


5. Получим дескрипторы особых точек с помощью SIFT.<br>


6. Заводим пустое множество. Будем складывать туда шаблоны, соответствующие предметам на изображении.<br>


7. В цикле сравниваем входное изображение с каждым шаблоном и с помощью метода skimage.feature.match_descriptors() матчим предметы и шаблоны. Если предмет совпал с каким-то шаблоном, сохраняем шаблон в множество. <br>


</code>

## Проверка того, попадают ли обекты в многоугольник
Мы получили множество шаблонов, соответствующих объектам со входной фотографии, причем для каждого шаблона получен прямоугольник, описывающий находящийся на нем объект.
<code>
1. Остортируем это множество в порядке убывания диаметров прямоугольников. <br>
2. Вычислим диаметр входного многоугольника.<br>
3. В цикле по элементам множества:
* Если диаметр прямоугольника, описанного вокруг предмета шаблона больше диаметра входрного многоугольника, возвращаем "Нет".
* Строим разность по Минковскому прямоугольника и многоугольника с помощь метода MinkowskiDiff из pyclipper. Если она пустая, возвращаем "Нет". Если непустая, вырезаем прямоугольник из многоугольника с помощью  Execute из pyclipper и переходим к следующей итерации цикла.
4. Возвращаем "Да".
</code>

## Оценка качества
Вычисляется как доля размеченных примеров, на которых алгоритм работает верно.

# Улучшения

Нужно научиться решать задачу в случае, когда предметы разрешено вращать внутри многоугольника. <br>
Стоит придумать какой-то алгоритм заплнения многоугольника, кроме жадного.<br>
Возможно, описывать предметы не прямоугольниками, а треугольниками и решать задачу с помощью триангуляции входного многоугольника.<br>
Можно добавить дополнительную карту особых точек(с мерой Харриса или другоим пороговым значением), чтобы сматчилось больше предметов.<br>
В случае плохих результатов стоит отказаться от поиска особых точек по катре границ, а вместо этого искать текстурные и цветовые признаки. Это может помочь при матчинге.