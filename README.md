# Реализация рейтинговой системы пользователей и ее интеграция в пользовательский интерфейс
Отчет по лабораторной работе #3 выполнил(а):
- Паханов Александр Александрович
- РИ-300018

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Создание интерактивного приложения с рейтинговой системой пользователя и интеграция игровых сервисов в готовое приложение.

## Задание 1
### Используя видео-материалы практических работ 1-5 повторить реализацию игровых механик.
### Ход работы:

- Практическая работа «Механизм ловли объектов»

Напишем код, с помощью которого реализуем передвижение энергетического щита с помощью курсора мыши и исчезновение яиц при пересечении:
```C#
using UnityEngine;

public class EnergyShield : MonoBehaviour
{
    void Update()
    {
        Vector3 mousePos2D = Input.mousePosition;
        mousePos2D.z = -Camera.main.transform.position.z;
        Vector3 mousePos3D = Camera.main.ScreenToWorldPoint(mousePos2D);
        Vector3 pos = this.transform.position;
        pos.x = mousePos3D.x;
        this.transform.position = pos;
    }

    private void OnCollisionEnter(Collision coll)
    {
        GameObject Collided = coll.gameObject;
        if (Collided.tag == "Dragon Egg")
            Destroy(Collided);
    }
}
```
Сам код привязываем к префабу энергетического щита.
Также добавим объект Canvas с текстовым полем, в котором будет отображаться количество очков. Настройки Canvas и TMP:

![](/Pics/z1_1.jpg)
![](/Pics/z1_2.jpg)

- Практическая работа «Добавляем счетчик»

Добавим в скрипт EnergyShield следующий код, который будет обнулять счетчик на старте и обновлять его при поимке яйца:
```C#
public TextMeshProUGUI scoreGT;
void Start()
{
    GameObject scoreGO = GameObject.Find("Score");
    scoreGT = scoreGO.GetComponent<TextMeshProUGUI>();
    scoreGT.text = "0";
}
private void OnCollisionEnter(Collision coll)
{
    GameObject Collided = coll.gameObject;
    if (Collided.tag == "Dragon Egg")
        Destroy(Collided);
    int score = int.Parse(scoreGT.text);
    score += 1;
    scoreGT.text = score.ToString();
}
```

В скрипт DragonPicker добавим метод, который будет уничтожать все яйца на сцене:
```C#
public void DragonEggDestroyed()
{
    GameObject[] tDragonEggArray = GameObject.FindGameObjectsWithTag("Dragon Egg");
    foreach (GameObject tGO in tDragonEggArray)
        Destroy(tGO);
}
```

Данный метода будем вызывать в скрипте DragonEgg, когда игроку не удасться поймать яйцо, тем самым мы будем уничтожать следующее.
```C#
void Update()
{
    if(transform.position.y < bottomY)
    {
        Destroy(this.gameObject);
        DragonPicker apScript = Camera.main.GetComponent<DragonPicker>();
        apScript.DragonEggDestroyed();
    }
}
```

- Практическая работа «Уменьшение жизни. Добавление текстуры»

Для реализации механики жизней игрока добавим в скрипт DragonPicker следующие строчки кода:
В Start():
```C#
for(int i = 1; i <= numEnergyShield; i++)
    shieldList.Add(tShieldGo);
```

В DragonEggDestroyed():
```C#
int shieldIndex = shieldList.Count - 1;
GameObject tShieldGo = shieldList[shieldIndex];
shieldList.RemoveAt(shieldIndex);
Destroy(tShieldGo);

if (shieldList.Count == 0)
    SceneManager.LoadScene("_0Scene");
```
Таким образом, при уничтожении всех щитов сцена будет перезапускаться.

Добавим немного визуальных эффектов из UnityAssetStore:

![](/Pics/z1_3.jpg)

- Практическая работа «Прибираемся в папке»

После структурирования файлов проекта, мы имеем его следующий вид:

![](/Pics/z1_4.jpg)

Проверим, что мы перенесли все, что нам нужно:

![](/Pics/z1_5.jpg)

- Практическая работа «Интеграция игровых сервисов в готовое приложение»

Встроим плагин Yandex Games в нашу игру, выставим необходимые настройки, сделаем и выложим билд в черновик Яндекс Игр. Результат:
https://yandex.ru/games/app/199729?draft=true&lang=ru

## Задание 2
### Добавить в приложение интерфейс для вывода статуса наличия игрока в сети (онлайн или офлайн)
### Ход Работы:

Чтобы вывести статус игрока(подключен ли к Яндекс SDK) напишем следующий скрипт:
```C#
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using YG;

public class OnlineChecker : MonoBehaviour
{
    public TextMeshProUGUI statusText;
    public Image statusImage;
    void Start()
    {
        statusText = GameObject.Find("StatusText").GetComponent<TextMeshProUGUI>();
        statusImage = GameObject.Find("StatusImage").GetComponent<Image>();
    }

    void Update()
    {
        statusText.text = YandexGame.SDKEnabled == true ? "Online" : "Offline";
        statusImage.color = YandexGame.SDKEnabled == true ? Color.green : Color.red;
    }
}
```

Результат:

![](/Pics/z2_1.jpg)

## Задание 3
### 1. Произвести сравнительный анализ игровых сервисов Яндекс Игры и VK Game;
### 2. Дать сравнительную характеристику сервисов, описать функционал;
### 3. Описать их методы интеграции с Unity;
### 4. Произвести сравнение, сделать выводы;
### 5. Подготовить реферат по результатам выполнения пунктов 1-4.
### Реферат:

Яндекс игры и VK Game - это две игровые платформы, на которых можно играть в HTML-5 игры.
Что общего у этих сервисов:
- главная страница - это каталог с различными играми, отсортированными по популярности и на различные жанры;
- профиль игрока;
- категории игр по жанрам;
- имеется поиск по ключевым словам;
- отдельное поле с играми, в которые уже заходил(категория "Мои игры");
- в большинство игр можно поиграть как на ПК, так и на телефоне;
- в обоих сервисах можно сделать аккаунт разработчика и выкладывать свои игры, при прохождении проверки.

Различия:
- VK Game интегрирован в профиль человека в социальной сети ВКонтакте, из-за чего игры могут быть также привязаны к различным сообществам или ивентам. В свою очередь, аккаунт Яндекс игр привязан к аккаунту Яндекс;
- в играх VK Game есть много социальных фишек из-за привязки аккаунта к ВК;
- в Яндекс Играх есть рекламные баннеры на главной странице;
- у VK Game имеются игры, которые можно скачать(сервис привязан к Игры Mail.ru).

У VK Game взаимодействие с ВКонтакте происходит через библиотеку VK Bridge.
На платформе Яндекс Игры используется SDK Яндекс.

Вывод:
VK Games больше сосредоточена больше на социальные игры, т.к. она привязана к соц сети. В Яндекс больше одиночных игр, хотя в ней также присутствую проекты с онлайн составляющей. По своей сути эти два сервиса имеют довольно схожий функционал, который, в некоторых случаях, различается только в деталях.

## Выводы

Мы сделали первые шаги по созданию игры Dragon Picker. Узнали, что в Unity есть удобный магазин с ассетами, в котором можно найти как платные, так и бесплатные продукты. Также мы познакомились с двумя игровыми площадками, на которых можно играть в HTML-5 игры. Каждая из платформ имеет похожий широкий функционал, однако эти платформы отличаются по игровым критериям.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
