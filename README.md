# Создание приложения для автоматизации измерения характеристик сердца по видео УЗИ
## Общая информация о проекте
### Потребности, приведшие к созданию данного проекта
Ультразвуковое исследование сердца (УЗИ), называемое также эхокардиографией, представляет собой значимый метод для диагностики и оценки состояния сердечно-сосудистой системы. Во время УЗИ, помимо визуального обследования структур сердца, выполняются расчеты различных его характеристик, о которых подробно будет рассказано позже. Они используются для оценки состояния сердечно-сосудистой системы, включая такие состояния, как сердечная недостаточность и другие формы патологии. По завершении исследования специалист по УЗИ выбирает несколько изображений, соответствующих максимальному сокращению и расширению сердечной мышцы и производит их полуавтоматическую разметку, на основе которой вычисляются необходимые характеристики. Данная процедура занимает около 15-20 минут рабочего времени специалиста.

### Цель данного проекта 
Cоздать продукт, который позволит полностью автоматизировать процесс разметки и получения характеристик сердца, что освободит ресурсы врача.

### User story
Я как специалист по УЗИ хочу автоматизировать процесс измерения характеристик сердца по видео УЗИ, чтобы сократить свои временные затраты на эту задачу.

### Use case

|                                   |                                                                                                                                                                                                                            |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Действующее лицо                  | Специалист по УЗИ                                                                                                                                                                                                          |
| Предусловие                       | Получено видео УЗИ длинной не менее 3 цикла сердца.<br>Видео загружено на компьютер с установленным приложением.                                                                                                           |
| Триггер                           | В любой момент, когда запущено приложение                                                                                                                                                                                  |
| Описание основного сценария       | 1. Специалист загружает видео в приложение<br>2. Приложение производит обработку видео и выгружает результаты в виде таблицы с требуемыми характеристиками сердца и размеченным видео для проверки корректности обработки. |
| Описание альтернативного сценария | 2.1. Приложение сообщает об ошибке, если не удалось обработать видео.                                                                                                                                                      |

### Требования к продукту
* Ожидаемая погрешность работы продукта должна составлять менее **15%**, от предсказанных специалистом по УЗИ значений, что, судя по промежуточным результатам, является вполне достижимым требованием.
* Обработка должна выполняться за разумный промежуток времени и исчисляться минутами.
* Характеристики сердца, которые должны определяться по УЗИ:
1. Левый желудочек (далее ЛЖ):
	1.1. КДР – конечно-диастолический размер.
	1.2. КСР - конечно-систолический размер.
	1.3. КДО - конечно-диастолический объем.
	1.4. КСО - конечно-систолический объем.
	1.5. ФВ – фракцию выброса. (КДО - КСО) / КДО * 100%
2. Правый желудочек (далее ПЖ):
	2.1. КДР ПЖ.
	2.2. ФИП – фракция изменения площади. (S_Диастолы – S_Систолы) / S_Диастолы. Норма ФИП 36%
3. КДР ЛЖ/КДР ПЖ 
### Методы определения параметров

#### 1. КДР и КСР

КДР (конечно-диастолический размер) — это максимальный размер, который достигает сердце в диастолу, когда сердечные камеры наполнены кровью. КСР (конечно-систолический размер) — это размер сердца в систолу, когда сердечная мышца сокращается и выталкивает кровь из камер. Более подробное описание приведено на изображении ниже.

<img src="assets/measurements.jpg" width="450" />

В ходе согласования со специалистами был принят следующий алгоритм расчета КДР и КСР (здесь алгоритм описан в общих чертах, более подробное описание находится в `notebooks/0_research/1_visualization_image.ipynb`):
1. От середины митрального (для ЛЖ) или трёстворчатого (для ПЖ) клапана до вершины желудочка строится длинная ось.
2. На расстоянии 1/4 от начала длинной оси откладывается точка, от которой рисуется два перпендикуляра вправо и влево, до границ желудочка.
Соответственно КДР определяется при максимальном размере желудочка, КСР при минимальном.

#### 2. КДО и КСО

КДО (конечно-диастолический объем) — это объём крови, который присутствует в желудочках сердца в конце диастолы, перед следующим систолическим сокращением. Это максимальный объём, который может находиться в желудочках после их наполнения кровью. КСО (конечно-систолический объем) — это объём крови, который остаётся в желудочках после систолы, то есть после сокращения сердца. Это минимальный объём крови в желудочках, который сохраняется в конце систолы. Эти два параметра рассчитываются *методом (каким?)*.
1. Желудочек по длинной оси разделяется на 16 секторов.
2. По ширине и высоте каждого сектора строится цилиндр.
3. Рассчитывается объём каждого цилиндра, объёмы суммируются.
Соответственно КДО определяется при максимальном размере желудочка, КСО при минимальном. На основе этих параметров рассчитывается ФВ

#### 3. ФИП
ФИП (фракция изменения площади) — это соотношение изменения площади поперечного сечения сердца (обычно желудочка) между диастолой (расслаблением) и систолой (сокращением). Это значение позволяет оценить, насколько хорошо сердце сокращается и, соответственно, насколько оно эффективно перекачивает кровь. Площадь желудочка в диастоле и систоле определяется стандартными инструментами из библиотек `python`. По ним рассчитывается ФИП.

## Данные

Для решения поставленной задачи собраны и размечены около 200 картинок для обучения модели. Тестирование модели происходит на 25 видео. Все данные получены с помощью аппарата **Philips Affiniti 70**.

## Описание решения

Задача разделена на три подзадачи:
1. Задача сегментации, целью которой является определение на видео УЗИ границ желудочков и предсердий. Для решения этой задачи выбрана модель `YOLO8`. Модель дообучена на размеченных врачами изображениях УЗИ. Более подробно можно посмотреть в *указать_ноутбук*. На заре проекта была опробована модель `MaskRCNN`, эксперимент не принёс результатов (скорее всего из-за малого объёма тренировочной выборки). Альтернативное решение - предобученая `U-Net`.
2. Задача вычисления параметров по полученным сегментам. Для решения этой задачи разработаны классы `Visualization` и `Video_processing`. Они собраны в библиотеку `UZI.py`, а их разработка описана в `notebooks/0_research/1_visualization_image.ipynb` и `notebooks/0_research/2_visualization_video.ipynb`.
3. Оценка результатов.
* Для первой задачи выбрана метрика `accuracy`, которая высчитывается для каждого сегмента как отношение количества появлений сегментов на кадрах без ошибок ко всему количеству кадров в видео. Затем для видео высчитывается среднее значением метрики по всем сегментам. И так как количество кадров в видео различается не сильно (100-120), то метрика по всем видео усредняется и получается итоговая `accuracy`. Чтобы понять, правильно ли детектирован сегмент, для каждого видео нужно создать подобие "разметки" в виде квадратов, которые задаются центром и шириной стороны. Подробнее на видео ниже. Так же стоит отметить, что при оценке результатов первая задача рассматривается как задача классификации, а не сегментации. При визуальной оценке было видно, что модель предсказывается границы сегментов довольно хорошо, но путает или вовсе не определяет сегменты. Поэтому нам важно оценить, насколько хорошо наша модель отличает сегменты друг от друга и обнаруживает ли их. Так же значительная часть валидации происходила на видео, не имеющих разметки. 
* Для второй задачи выбрана метрика `MAE` за её лёгкую интерпретацию. Она рассчитывается как среднее абсолютное отклонение значений параметров по каждому видео предоставленных специалистами и определённых моделью.
* Для решения этой задачи разработан и помещён в `UZI.py` класс `Evaluation`.

## Результаты

На данный момент по 25 видео достигнуты следующие результаты:
* Скорость обработки - больше 10 кадров в секунду. То есть видео длительностью 4 секунды обрабатывается за 10-12 секунд.
* `Accuracy` - **0.91
* `MAE` (КДД и КСД - длинная ось):

Левый желудочек

| ЛЖ_КДР | ЛЖ_КСР | ЛЖ_КДД | ЛЖ_КСД | ЛЖ_КДП | ЛЖ_КСП | ЛЖ_КДО | ЛЖ_КСО | ЛЖ_ФВ |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ----- |
| см     | см     | см     | см     | cм^2   | cм^2   | cм^3   | cм^3   | %     |
| 0.41   | 0.46   | 0.45   | 0.94   | 3.44   | 3.98   | 14.00  | 13.01  | 10.84 |

Правый желудочек

| ПЖ_КДР | ПЖ_КДП | ПЖ_КСП | ПЖ_ФИП |
| ------ | ------ | ------ | ------ |
| см     | см     | cм^2   | %      |
| 0.36   | 2.23   | 1.55   | 7.39   |

* Среднее отклонение значений определённых моделью параметров от оценок специалистов - **менее 15%**.

По итогам обработки видео так же строятся графики изменения параметров:

<p float="left">
  <img src="assets/II%2012%20ЛЖ%20КДР%20и%20КСР.png" width="400" />
  <img src="assets/II%2012%20ЛЖ%20КДО%20и%20КСО.png" width="400" /> 	
  <img src="assets/II%2016%20ЛЖ%20КДР%20и%20КСР.png" width="400" />
  <img src="assets/II%2016%20ЛЖ%20КДО%20и%20КСО.png" width="400" /> 
  <img src="assets/II%2024%20ЛЖ%20КДР%20и%20КСР.png" width="400" />
  <img src="assets/II%2024%20ЛЖ%20КДО%20и%20КСО.png" width="400" />
</p>
Примеры получаемых видео (для воспроизведения откройте в новой вкладке):

<div style="display: flex;">
    <a href="https://github.com/user-attachments/assets/fda442f7-77bf-4684-b112-579ef0672533">
        <img src="assets/II_12_img.PNG" alt="II_12" width="300">
    </a>
    <a href="https://github.com/user-attachments/assets/b677f43a-0534-4cb7-9786-f1f9d5af137d">
        <img src="assets/II_16_img.PNG" alt="II_16" width="300">
    </a>
    <a href="https://github.com/user-attachments/assets/8a9a9c0b-75da-4749-beef-f648a3b2c17f">
        <img src="assets/II_23_img.PNG" alt="II_23" width="300">
    </a>
    <a href="https://github.com/user-attachments/assets/0c15bcb2-480d-4daa-aa94-9165494c3cf2">
        <img src="assets/II_24_img.PNG" alt="II_24" width="300">
    </a> 
</div>

На видео отмечаются следующие элементы:
## Архитектура папок

```
UZI/
│
├── assets/                         # Папка для вложений readme.md
│
├── data/                           # Директория для хранения данных
│   ├─ processed/                   # Обработанные и подготовленные данные для     │   │   │                             моделирования
│   │   ├── datasets/               # Подкаталог для организации наборов данных
│   │   │   ├── train/              # Набор данных для обучения модели
│   │   │   ├── val/                # Набор данных для валидации (проверки) модели
│   │   │   └── test/               # Набор данных для тестирования модели
│   │   └── Elena_marking/          # Разметка, созданная Еленой
│   │
│   └── raw/                        # Исходные неочищенные данные
│       ├── images/                 # Подкаталог для изображений
│       └── videos/                 # Подкаталог для видеофайлов
│           └── videos_II/          # Дополнительный подкаталог для видео          │                                     (возможно, для второй версии)
│
├── docs/                           # Документация проекта
│   ├── ИИ_ИЮ_26.txt                # Оценка параметров сердца по ИИ видео,
│   │                               # выполненная врачами
│   └── ТЗ для УЗИ 1.1.txt          # Техническое задание проекта, описывающее его
│                                   # цели и требования
│
├── models/                         # Директория с исходным кодом проекта
│   └── best.pt                     # Дообученная модель
│
├── notebooks/                      # Директория для Jupyter ноутбуков,
│   │                               # используемых в проекте
│   ├── 0_research                  # Подкаталог с ноутбуками для подробного
│   │                               # описания исследования
│   ├── 1_processing                # Подкаталог с ноутбуками для обработки файлов
│   ├── 2_work                      # Подкаталог для рабочих ноутбуков
│   └── 3_temp                      # Подкаталог для временных файлов
│
├── project_docs/                   # Документация проекта
│       ├── ИИ_ИЮ_25.txt            # Оценка параметров сердца по видео ИИ врачами
│       └── ТЗ для УЗИ 1.1.txt      # Техническое задание проекта в текстовом      │                                     формате
│
├── results/                        # Директория для хранения результатов
│   │                               # экспериментов
│   ├── 0_research                  # Подкаталог для результатов работы ноутбуков
│   │                               # из 0_research
│   ├── 1_processing                # Подкаталог для результатов работы ноутбуков
│   │   │                           # из 1_processing
│   │   └── videos_II               # Подкаталог для обработанных ИИ видео
│   │       └── graphs              # Подкаталог для графиков изменения параметров
│   │                               # сердца на ИИ видео
│   └── 2_work                      # Подкаталог для результатов работы ноутбуков
│                                   # из 2_work
│
├── src/                            # Основная директория с исходным кодом проекта
│   ├── models/                     # Директория для хранения файлов моделей
│   │   ├── base.pt                 # Файл с базовой моделью
│   │   └── best.pt                 # Файл с лучшей моделью на данный момент
│   └── UZI_paths.py                # Скрипт, содержащий класс с путями к ресурсам
│                                   # и данным проекта (не используется)
│
├── requirements.txt                # Файл, содержащий список зависимостей
│                                   # (библиотек) проекта для установки (будет
│                                   # добавлен)
├── README.md                       # Документация проекта, описание его целей,
│                                   # структуры и инструкции по использованию
└── .gitignore                      # Файл, указывающий на файлы и папки, которые
                                      # не должны быть включены в систему контроля
                                       # версий git

```
