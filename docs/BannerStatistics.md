# BannerStatistics

## Введение

**BannerStatistics** это хелпер для быстрого добавления данных статистики рекламных блоков.

## Структура

Хелпер имеет следующие функции с одинаковыми параметрами вызова

```php
class BannerStatistics {

  /**
   * Добавляет показ футера к статистике
   *
   * @param int $id ID рекламного обявления
   * @param int $userId ID пользователя
   * @param int $ip IP-адрес пользователя
   * @param bool $delayed Отложить вставку до применения функции ::apply()
   *
   * @return void
   */
  public function addFooterView( $id, $userId, $ip, $delayed = true );

  /**
   * Добавляет показ текста к статистике
   */
  public function addTextView( $id, $userId, $ip, $delayed = true );

  /**
   * Добавляет показ топика к статистике
   */
  public function addTopicView( $id, $userId, $ip, $delayed = true );

  /**
   * Добавляет клик футера к статистике
   */
  public function addFooterClick( $id, $userId, $ip, $delayed = true );

  /**
   * Добавляет клик текста к статистике
   */
  public function addTextClick( $id, $userId, $ip, $delayed = true );

  /**
   * Добавляет клик топика к статистике
   */
  public function addTopicClick( $id, $userId, $ip, $delayed = true );

}
```

где

*$delayed* &mdash; параметр, отвечающий за способ вставки данных в таблицу.
Если TRUE &mdash; то данные будут вставлены все вместе, одним запросом, по завршению работы,
если FALSE &mdash; данные будут вставлены немедленно.

## Использование

Для более непринужденного использования создан компонент **Statistic**.

Для его подключения нужно прописать в *app/config/main.php*

```php
return [
    ...
    'components' => [
        ...
        'stats' => [ 'class'=>'Statistic', ],
        ...
    ],
    ...
];
```

И тогда, в любом месте кода используем

```php
...

Yii::app()->stats->banners->addFooterView( $id, $userId, $ip, $delayed);
Yii::app()->stats->banners->addTextView( $id, $userId, $ip, $delayed);
Yii::app()->stats->banners->addTopicView( $id, $userId, $ip, $delayed);
Yii::app()->stats->banners->addFooterClick( $id, $userId, $ip, $delayed);
Yii::app()->stats->banners->addTextClick( $id, $userId, $ip, $delayed);
Yii::app()->stats->banners->addTopicClick( $id, $userId, $ip, $delayed);

...
```
