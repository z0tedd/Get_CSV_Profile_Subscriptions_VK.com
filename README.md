# Парсинг подписок профиля ВК

## Первое, что необходимо - запросить архив со своими данными

https://vk.com/faq18145

![image](https://gist.github.com/user-attachments/assets/5ca56bce-539e-49f8-8f65-6dbc119cf7ee)
![image](https://gist.github.com/user-attachments/assets/7622e43e-e23b-4310-8a37-b54c3af87220)

Спустя несколько дней вам придет сообщение о том, что архив подготовлен и его можно скачать
![image](https://gist.github.com/user-attachments/assets/c4224b41-ff0a-4f2b-aa6b-a4af14720fb8)

Открываем архив, заходим в папку profile, видим html файлы с названием subscription0. Их мы и будем парсить.

![image](https://gist.github.com/user-attachments/assets/9b5e8e8a-3ccd-4710-8f04-635b91f689be)
![image](https://gist.github.com/user-attachments/assets/9d2a5cd4-4fbe-497d-ab02-d00d87d43743)

Код для обработки(кидаем его в папку profile, меняем название файла и запускаем):

```python
import os
from chardet import detect
from bs4 import BeautifulSoup
import csv
import re


# get file encoding type
def get_encoding_type(file):
    with open(file, "rb") as f:
        rawdata = f.read()
    return detect(rawdata)["encoding"]


# Получение названия паблика в формате <имя> <Ссылка>
def getName(s: str) -> str:
    match = re.search(r'<a href="(.*?)">(.*?)</a>', s)
    if match:
        return [f"{match.group(2)}", f"{match.group(1)}"]
    return []


# Создание таблицы
def GetCSV(rowList):
    with open("profile.csv", "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerows(rowList)


FileName = "subscriptions1.html"
# Гении из вк используют не utf-8, а windows-1251. Спасибо им
# за 20 веселых минут непонимания, почему у меня не работает File.read()
from_codec = get_encoding_type(FileName)
File1 = open(FileName, encoding=from_codec)

a = File1.read()
soup = BeautifulSoup(a, "html.parser")
SoupResult = soup.find_all("a")

# Замена амперсанта реплейсом, чтобы таблица нормально обрабатывала значение
ResultForCSV = [
    getName(str(i).replace("&amp;", "&")) for i in SoupResult if getName(str(i))
]
GetCSV(ResultForCSV)
File1.close()

```

Получаем profile.csv, который читается екселем.
![image](https://gist.github.com/user-attachments/assets/8d0cdda8-0f0c-4ca0-a002-276027a6b26d)
