# API для системы классификации людей по фотографии

Все запросы и ответы работают в формате JSON.

- [Загрузка фотографии](#Загрузка-фотографии)
- [Проверка фотографии](#Проверка-фотографии)
- [Проверка готовности архива](#Проверка-готовности-архива)
- [Скачивание архива сжатых фотографий](#Скачивание-архива-сжатых-фотографий)
- [Скачивание архива](#Скачивание-архива)

## Общее краткое описание работы:

1) Загружаются фотографии на сервер через `/upload`.
2) Загружается фотография для проверки через `/check`, возвращается номер запроса проверки.
3) Терминал проверяет готовность архива (к примеру, каждую секунду) через `/checknumber`.
4) Как только возвращается `"status" : "done"`, получает json через `/downloadarchive` или `/downloadcompressed`, декодирует base64 представления и сохраняет.



## Загрузка фотографии
Функция: **Загрузка фотографии**
Адрес: `/upload`
Тип запроса: **POST**


Описание: Отправляется изображение формата только .jpg (.jpeg не подойдёт) любого разрешения в формате base64 в JSON. 
JSON содержит два поля «terminal» (номера терминала в системе) и «file» (base64 изображение). 

Пример JSON: 
```json
{
	"terminal" : "1",
	"file": "(здесь находится base64 string представление изображения)"
}
```

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
Ответ: Высылается два вида json.
При успешной загрузке фотографии: 
```json
{
	"status" : "done"
}
```
При провале:
```json
{
	"status" : "fail"
}
```

## Проверка фотографии
**Функция проверки фотографии**
Адрес: **/check**
Тип запроса: **POST**

Описание: Отправляется изображение формата только .jpg (.jpeg не подойдёт) любого разрешения в формате base64 в JSON. 
JSON содержит два поля «terminal» (номера терминала в системе) и «file»(base64 изображение). 
Пример JSON: 
```json
{
	"terminal" : "1",
	"file": "(здесь находится base64 string представление изображения)"
}
```

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

Ответ: При успешной загрузке выдаст номер запроса (id). Он нужен для того, чтобы в дальнейшем проверять готовность архива с фотографиями человека, который встал на проверку лица.

 При успешной:
 ```json
{
	"status" : "done",
	"id" : (здесь номер запроса)
}
```

При провале:
```json
{
	"status": "fail"
}
```
## Проверка готовности архива
3) Проверка готовности архива.
Адрес: /checknumber
Тип запроса: POST

Описание: Проверяет готовность архива на сервере по присылаемому номеру запроса, выданному на предыдущем этапе — проверка фотографии.

Формат JSON:
```json
{
	"terminal" : "1"
	"queryNumber" : (полученный номер запроса)
}
```

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

## Скачивание архива фотографий по номеру запроса
Скачивание архива по номеру запроса.
Адрес: /downloadarchive
Тип запроса: POST
Описание: По запросу отправляется архив в формате base64 в JSON.
Пример запроса:
```json
{
	"terminal": "1",
	"queryNumber" : (номер запроса)
}
```

Пример ответа:
```json
{
	"terminal": "1",
	"queryNumber": "0",
	"archive": (здесь .zip-файл представленный base64 строкой)
}
```
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

## Скачивание архива сжатых фотографий
Скачивание архива уменьшенных фотографий по номеру запроса.
Адрес: /downloadcompressed
Тип запроса: POST
Описание: По запросу отправляется архив в формате base64 в JSON.
Пример запроса:
```json
{
	"terminal": "1",
	"queryNumber" : (номер запроса)
}
```

Пример ответа:
```json
{
	"terminal": "1",
	"queryNumber": "0",
	"archive": (здесь .zip-файл представленный base64 строкой)
}
```
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


