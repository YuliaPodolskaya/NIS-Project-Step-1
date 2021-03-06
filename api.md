# API для системы классификации фотографии людей

Все запросы и ответы работают в формате JSON.

1. [Загрузка фотографии](#Загрузка-фотографии)
2. [Проверка фотографии](#Проверка-фотографии)
3. [Проверка готовности архива](#Проверка-готовности-архива)
4. [Скачивание архива сжатых фотографий](#Скачивание-архива-сжатых-фотографий)
5. [Скачивание архива](#Скачивание-архива)
</br> 

## Общее краткое описание работы:

1) Загружаются фотографии на сервер через `/upload`.
2) Загружается фотография для проверки через `/check`, возвращается номер запроса проверки.
3) Терминал проверяет готовность архива (к примеру, каждую секунду) через `/checknumber`.
4) Как только возвращается `"status" : "done"`, получает json через `/downloadarchive` или `/downloadcompressed`, декодирует base64 представления и сохраняет.
</br>

## 1. Загрузка фотографии
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
</br>

## 2. Проверка фотографии
Функция: **Проверка фотографии**  
Адрес: `/check`  
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

Ответ: При успешной загрузке выдаст номер запроса (id). Он нужен для того, чтобы в дальнейшем проверять готовность архива с фотографиями человека, который встал на проверку лица.

 При успешной:
 ```json
{
	"status" : "done",
	"id" : "(здесь номер запроса)"
}
```

При провале:
```json
{
	"status": "fail"
}
```
</br>

## 3. Проверка готовности архива
Функция: **Проверка готовности архива**  
Адрес: `/checknumber`  
Тип запроса: **POST**  

Описание: Проверяет готовность архива на сервере по присылаемому номеру запроса, выданному на предыдущем этапе — проверка фотографии.

Формат JSON:
```json
{
	"terminal" : "1",
	"queryNumber" : "(полученный номер запроса)"

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

Описание: По запросу отправляется архив в формате base64 в JSON.  
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
	"archive": "(здесь .zip-файл представленный base64 строкой)"
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

## 5. Скачивание архива сжатых фотографий
Функция: **Скачивание архива сжатых фотографий по номеру запроса**  
Адрес: `/downloadcompressed`  
Тип запроса: **POST**  

Описание: По запросу отправляется архив в формате base64 в JSON.  
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
	"archive": "(здесь .zip-файл представленный base64 строкой)"
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


