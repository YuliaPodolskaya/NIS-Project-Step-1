# API для системы классификации фотографии людей

Все запросы и ответы работают в формате JSON.

1. [Загрузка фотографии](#Загрузка-фотографии)
2. [Проверка фотографии](#Проверка-фотографии)
3. [Проверка готовности архива](#Проверка-готовности-архива)
4. [Скачивание архива сжатых фотографий](#Скачивание-архива-сжатых-фотографий)
5. [Скачивание архива](#Скачивание-архива)
</br> 

## Общее краткое описание работы:

1) Фотография загружается на сервер через команду `/upload`.
2) Фотография проверяется через команду `/check`.
3) Возвращается номер запроса проверки.
4) Терминал проверяет готовность архива (к примеру, каждую секунду) через `/checknumber`. Примр частоты проверки: раз в секунду. <!--- если это реккомендация, а обновить текст --->
5) Если в параметре `"status"` возвращается значеенеие `"done"`, терминал получает JSON-файл через команду `/downloadarchive` или `/downloadcompressed`, декодирует base64 представления и сохраняет <!--- куда --->.
</br>

## Загрузка фотографии <!--- все эти действия независимы друг от друга, поэтому последовательность действий и нумерованный список здесь излишние --->
Функция: **Загрузка фотографии**  
Адрес: `/upload`  
Тип запроса: **POST**  


Описание: Отправляется изображение в формате .jpg (.jpeg недоступен) любого разрешения в JSON-файл. Формат записи данных: [base64](https://ru.wikipedia.org/wiki/Base64).
JSON-файл содержит следующие параметры:
* «terminal» (номера терминала в системе)
* «file» (изображение, записанное в формате base64). 

Пример JSON: 
```json
{
	"terminal" : "1",
	"file": "(здесь находится base64 string представление изображения)"
}
```
</br>

Пример запроса на языке Python 3.6:

```python
import requests
import json
import base64

url = "http://192.168.0.10/upload"
path_to_image = "home/files/image.jpg"

with open(path_to_image, "rb") as image_file:
	encoded_string = base64.b64encode(image_file.read())

data = {
	"terminal": "1",
	"file": str(encoded_string)[2:len(encoded_string) - 1]
}

json_data = json.dumps(data)

answer = requests.post(url, json=data)

print(answer)

response = answer.json()
print(response)

```
</br>

Ответ: Отпраавляется два вида JSON-файла. <!--- прописать какие и оформить в виде списка --->
При успешной загрузке фотографии: <!--- статусы ответов лучше описывать отдельно, вне самого кода --->
```json
{
	"status" : "done"
}
```
При возможной ошибке: <!--- дописать, когда это может возникать --->
```json
{
	"status" : "fail" 
}
```
</br>

## Проверка фотографии
Функция: **Проверка фотографии**  
Адрес: `/check`  
Тип запроса: **POST**  


Описание: Отправляется изображение в формате .jpg (.jpeg недоступен) любого разрешения в json-файл. Формат записи данных: [base64](https://ru.wikipedia.org/wiki/Base64).
JSON-файл содержит следующие параметры:
* «terminal» (номера терминала в системе)
* «file» (изображение, записанное в формате base64). 

Пример JSON: 
```json
{
	"terminal" : "1",
	"file": "(здесь находится base64 string представление изображения)"
}
```

</br>
Пример запроса на языке Python 3.6:

```python
import requests
import json
import base64

url = "http://192.168.0.10/check"
path_to_image = "home/files/image.jpg"

with open(path_to_image, "rb") as image_file:
	encoded_string = base64.b64encode(image_file.read())

data = {
	"terminal": "1",
	"file": str(encoded_string)[2:len(encoded_string) - 1]
}

json_data = json.dumps(data)

answer = requests.post(url, json=data)

print(answer)

response = answer.json()
print(response)
```
</br>

Ответ: При успешной загрузке отправляется номер запроса (id). Он нужен для того, чтобы в дальнейшем проверять готовность архива с фотографиями. <!--- про проверку лица я бы опустила, поскольку из названия самой API документации это понятно --->

 При успешной загрузке:
 ```json
{
	"status" : "done",
	"id" : "(номер запроса)" <!--- описание параметров лучше выносить из кода (применимо ко всем запросам) --->
}
```

При возможной ошибке: <!--- дописать, когда это может возникать --->
```json
{
	"status": "fail"
}
```
</br>

## Проверка готовности архива
Функция: **Проверка готовности архива**  
Адрес: `/checknumber`  
Тип запроса: **POST**  

Описание: Проверяет готовность архива на сервере по id, полученному в методе [Проверка фотографии](#Проверка-фотографии).

Пример JSON:
```json
{
	"terminal" : "1",
	"queryNumber" : "(полученный номер запроса)"  <!--- почему здесь параметр уже на id называется, но означает все также номер запроса? если это другой параметр, нужно точнее описать его --->

}
```
</br>

Пример запроса на языке Python 3.6:

```python
import requests
import json
import base64

url = "http://192.168.0.10/checknumber"

data = {
	"terminal": "1",
	"queryNumber" : "0" 

}

json_data = json.dumps(data)

answer = requests.post(url, json=data)

print(answer)

response = answer.json()
print(response)
```
</br>

## 4. Скачивание архива фотографий по номеру запроса
Функция: **Скачивание архива по номеру запроса**  
Адрес: `/downloadarchive`  
Тип запроса: **POST**  

Описание: Отправляется архив в JSON-файл. Формат записи данных: [base64](https://ru.wikipedia.org/wiki/Base64). 
Пример запроса:
```json
{
	"terminal": "1",
	"queryNumber" : "(номер запроса)"
}
```

Пример ответа:
```json
{
	"terminal": "1",
	"queryNumber": "0",
	"archive": "(.zip-файл, представленный строкой в формате base64)"
}
```
</br>
Пример отправки и декодирования на Python 3.6:  

```python
import requests
import json
import base64

url = "http://192.168.0.10/downloadarchive"

data = {
	"terminal": "1",
	"queryNumber": "0" 
}

json_data = json.dumps(data)

answer = requests.post(url, json=data)

print(answer)

response = answer.json()
print(response)

decoded = base64.b64decode(response["archive"])

with open("your_path_and_name_of_zip.zip", "wb") as f:
	f.write(decoded)

print("done")
```

</br>

## Скачивание архива сжатых фотографий
Функция: **Скачивание архива сжатых фотографий по номеру запроса**  
Адрес: `/downloadcompressed`  
Тип запроса: **POST**  

Описание: Отправляется архив <!--- в json-файл? --->. Формат записи данных: [base64](https://ru.wikipedia.org/wiki/Base64).   
Пример запроса:
```json
{
	"terminal": "1",
	"queryNumber" : "(номер запроса)"
}
```

Пример ответа:
```json
{
	"terminal": "1",
	"queryNumber": "0",
	"archive": "(.zip-файл в формате base64)"
}
```

</br>

Пример отправки и декодирования на Python 3.6:

```python
import requests
import json
import base64

url = "http://192.168.0.10/downloadcompressed"

data = {
	"terminal": "1",
	"queryNumber": "0" 
}

json_data = json.dumps(data)

answer = requests.post(url, json=data)

print(answer)

response = answer.json()
print(response)

decoded = base64.b64decode(response["archive"])

with open("your_path_and_name_of_zip.zip", "wb") as f:
	f.write(decoded)

print("done")
```


