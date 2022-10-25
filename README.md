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

Добавим немного текстур из UnityAssetStore:

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
### Добавить в приложение интерфейс для вывода статуса наличия игрока в сети (онлайн или офлайн).
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
        statusText.text = YandexGame.SDKEnabled ? "Online" : "Offline";
        statusImage.color = YandexGame.SDKEnabled ? Color.green : Color.red;
    }
}
```

Результат при включенном игровом объекте YandexGame:

![](/Pics/z2_1.jpg)

Результат при выключенном YandexGame(ситуация схожа с тем, если бы игрок не был онлайн):

![](/Pics/z2_2.jpg)

## Задание 3
### Реализовать вывод в консоль количества времени отсутствия игрока в сети если пользователь офлайн.
### Ход работы:

Немного перепишем скрипт из прошлого задания:
```C#
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using YG;
using System;

public class OnlineChecker : MonoBehaviour
{
    public TextMeshProUGUI statusText;
    public Image statusImage;
    public DateTime LastLoginTime;
    void Start()
    {
        statusText = GameObject.Find("StatusText").GetComponent<TextMeshProUGUI>();
        statusImage = GameObject.Find("StatusImage").GetComponent<Image>();
    }

    void Update()
    {
        if(YandexGame.SDKEnabled)
        {
            statusText.text = "Online";
            statusImage.color = Color.green;
            LastLoginTime = DateTime.Now;
        }
        else
        {

            statusText.text = "Offline";
            statusImage.color = Color.red;
            var offlineDurationTime = DateTime.Now - LastLoginTime;
            Debug.Log("Player offline " + offlineDurationTime.Days + " days, " 
                + offlineDurationTime.Hours + " hours, " + offlineDurationTime.Minutes + " minutes.");
        }
    }
}
```

В итоге мы можем получить вывод в консоль по типу:

![](/Pics/z3_1.jpg)

## Выводы

Мы сделали играбельный прототип игры Dragon Picker и выложили его на платформу Яндекс Игры. Познакомились с плагином Яндекс, сделали проверку статуса игрока(онлайн или оффлай) и вывод в коносль продолжительности отсутствия пользователя. Яндекс SDK имеет большое количество различных фич и с их помощью можно настроить проект самым разнообразным способом.

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
