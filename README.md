## В текущем репозитарии указана открытая информация о проекте.

Решение задчаи 
Оценка взаимосвязи между пресс-релизами кредитных рейтинговых 
агентств и присвоенными кредитными рейтингами по 
национальной рейтинговой шкале для Российской Федерации с учетом методологии.

### Структура проекта
1. site_for_model - папка с сайтом, где fronted - разметка страницы, static - статичный файл, в том числе стили, site_for_model - папка с настройками сайта, analysys_pressrelease - папка с бэкэндом сайта (файл views.py)
2. src - папка с моделью, файлами для обучения, выделения ключевых конструкций, препроцессингом данных, датасетом. 

### Базовое использование обученной модели

#### Загрузите модель:
Для загрузки используется стандартная функция load_model из пакета tf.keras. \
Правильная загрузка требует указания customs_objects
```
import tensorflow as tf
import tensorlow_addons as tf

PATH = '/src/models/saved_models/'
MODEL_NAME = 'transformer_exp_5'

custom_objects = {'target_1': tfa.metrics.F1Score(num_classes=7, average='weighted'),
                  'target_2': tfa.metrics.F1Score(num_classes=17, average='weighted')}

model = tf.keras.load_model(PATH + MODEL_NAME, custom_objects = custom_objects)
```

#### Предсказания:
После загрузки модели можно исользовать предсказания. Текст предается в модель в формате tf.Tensor с размерами (1, 1). Максимальная длина токенов, при whitespace токенизации, 512. Текст должен быть написан на кириллице без каких либо обработок и токенизации. \
Модель возвращает список с двумя тензорами, размерность которых (1, 7) и (1, 17) соответственно.
```
text = tf.convert_to_tensor(' text ', dtype=tf.string)

predict = model(text)

type(predict) -> list

predict[0].shape -> (1, 7)
predict[1].shape -> (1, 17)

```
#### Получение ключевых слов:
Для получния ключевых слов необходимо импортировать функицю get_attention_words(). Она принимает в качестве арументов tf.keras.Model и tf.Tenor с размерами (1, 1), возвращает словарь, содержащий массив слов и массив весов внимания для этих же слов.

```
from src.preprocess.attention_processing import get_attention_words

text = tf.convert_to_tensor(' text ', dtype=tf.string)
returns = get_attention_words(model, text)

returns.keys() -> dict_keys(['words_1', 'words_2', 'attention_wieghts'])

returns['words_1'] -> np.array()
returns['words_2'] -> np.array()

returns['attention_wieghts'] -> np.array()
```

#### Сайт - обертка для классификации текстовых пресс релизов
### ВНИМАНИЕ! Для запуска сайта перейти по ссылке: https://a880-95-165-191-10.ngrok-free.app/
Для того, чтобы классифицировать пресс релиз нужно:
 1. Написать его в поле для текста или загрузить файл word или txt. Нажать отправить
 2. Модель выдает 3 вида информации. Предсказания уровня рейтинга по 2 градациям: укрупненный (7 классов), детализированный (17 классов), а также ключевые конструкции.
 3. В поле "Вывод с пометками" показывается пресс релиз с выделением ключевых конструкций. По кнопке "Скачать", загружается архив текст со всеми ключевыми конструкциями и сами конструкции.
### Будущее проекта
1. Расширение датасета.
2. Создание модели seq-to-seq для выделения ключевых слов.
3. Использовать алгоритм обучения с подкреплением https://github.com/KitStandart/rl_lib для указания ошибок модели и дообучения на исправленных данных.
