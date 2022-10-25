# Интеграция сервиса для получения данных профиля пользователя.
Отчет по лабораторной работе #2 выполнил(а):
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
Создание интерактивного приложения и изучение принципов интеграции в него игровых сервисов.

## Задание 1
### По теме видео практических работ 1-5 повторить реализацию игры на Unity.
### Ход работы:
- Создание основных игровых объектов

![](/Pics/z1_1.jpg)

- Реализация передвижения дракона и сбрасывания яиц

```C#
using UnityEngine;

public class EnemyDragon : MonoBehaviour
{
    public GameObject dragonEggPrefab;
    public float speed = 1;
    public float timeBetweenEggDrop = 1f;
    public float leftRightDistance = 10f;
    public float chanceDirection = 0.1f;
    void Start()
    {
        Invoke("DropEgg", 2f);
    }
    
    void DropEgg()
    {
        Vector3 MyVector = new Vector3(0.0f, 5.0f, 0.0f);
        GameObject egg = Instantiate<GameObject>(dragonEggPrefab);
        egg.transform.position = transform.position + MyVector;
        Invoke("DropEgg", timeBetweenEggDrop);
    }

    void Update()
    {
        Vector3 pos = transform.position;
        pos.x += speed * Time.deltaTime;
        transform.position = pos;

        if(pos.x < -leftRightDistance)
        {
            speed = Mathf.Abs(speed);
        }
        else if (pos.x > leftRightDistance)
        {
            speed = -Mathf.Abs(speed);
        }
    }

    private void FixedUpdate()
    {
        if(Random.value < chanceDirection)
        {
            speed *= -1;
        }
    }
}
```

- Добавление платформы и новых материалов

![](/Pics/z1_2.jpg)

- Создание визуальных эффектов

![](/Pics/z1_3.jpg)

Скрипт поведения яйца:
```C#
using UnityEngine;

public class DragonEgg : MonoBehaviour
{
    public static float bottomY = -30f;

    private void OnTriggerEnter(Collider other)
    {
        ParticleSystem ps = GetComponent<ParticleSystem>();
        var em = ps.emission;
        em.enabled = true;

        Renderer rend;
        rend = GetComponent<Renderer>();
        rend.enabled = false;
    }

    void Update()
    {
        if(transform.position.y < bottomY)
        {
            Destroy(this.gameObject);
        }
    }
}
```

Скрипт DragonPicker для генерации щитов:

```C#
using UnityEngine;

public class DragonPicker : MonoBehaviour
{
    public GameObject energyShieldPrefab;
    public int numEnergyShield = 3;
    public float energyShieldBottomY = -6f;
    public float energyShieldRadius = 1.5f;
    void Start()
    {
        for(int i = 1; i <= numEnergyShield; i++)
        {
            GameObject tShieldGo = Instantiate<GameObject>(energyShieldPrefab);
            tShieldGo.transform.position = new Vector3(0, energyShieldBottomY, 0);
            tShieldGo.transform.localScale = new Vector3(1 * i, 1 * i, 1 * i);
        }
    }
}

```

## Задание 2
### В проект, выполненный в предыдущем задании, добавить систему проверки того, что SDK подключен (доступен в режиме онлайн и отвечает на запросы)
### Ход Работы:

Чтобы подключить SDK Яндекс к проекту, нужно добавить в заголовок head HTML-страницы строку следующего вида:
```javascript
<!-- Yandex Games SDK -->
<script src="https://yandex.ru/games/sdk/v2"></script>
```

Далее требуется инициализировать SDK, используя метод init объекта YaGames:

```C#
YaGames
    .init()
    .then(ysdk => {
        console.log('Yandex SDK initialized');
        window.ysdk = ysdk;
    });
```

При этом в консоли выведется сообщение, если все прошло успешно. Тем самым мы проверим, подключен ли SDK.

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
