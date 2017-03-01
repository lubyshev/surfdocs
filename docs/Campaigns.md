# Campaigns

**Campaigns** это система, позволяющая без особых затрат ввести в эксплуатацию любую кампанию проекта.

## Ключевые особенности системы

* **Campaigns** максимально абстрагирована от исходных данных и при желании вообще не будет их изменять.
Система использует собственные таблицы *campaigns* и *campaign_partners* для
отслеживания происходящих событий кампании.

* Кампанию можно создать для любой таблицы проекта.

* Вся логика проводящейся кампании полностью отделена от **Campaigns**, и
реализуется через методы интефейса **ICampaign** классом *партнера* кампании.

## Соглашения

* *Кампания* &mdash; некоторые действия предпринимаемые системой при определенных
условиях для каждого из *партнеров* кампании.

* *Партнер* или *Участник* кампании &mdash; каждая запись из *связанной таблицы*
привязанная к кампании.

* *Связанная таблица* &mdash; таблица, некоторые (или все) записи которой привязаны к кампании.
Для каждой *кампании* возможна только одна *связанная таблица*.
Но для каждой таблицы можно создать сколько угодно *кампаний*

## Описание

### Класс Campaigns

Основной инcтрумент, позволяющий привязывать, запускать и удалять *кампании*.

```php

class Campaign {

  /**
   * Добавляет партнера к кампании
   *
   * @param ICampaign $model Партнер (запись в таблице-источнике)
   *
   * @return bool
   *
   * @throws CampaignsException
   */
  public static function addPartner( $model );

  /**
   * Проверяет существует ли кампания
   *
   * @param ICampaign $model
   *
   * @return bool
   */
  public static function campaignExists( $model );

  /**
   * Закрывает связанную кампанию
   *
   * @param ICampaign $model Родительская модель кампании
   *
   * @return bool
   *
   * @throws CampaignsException
   */
  public static function closeCampaign( $model );

  /**
   * Создает новую кампанию
   *
   * @param CActiveRecord $model Класс-источник кампании
   * @param int $dtStart Время старта кампании
   * @param int $dtStop Время окончания кампании. Если NULL - действует всегда.
   *
   * @return bool
   *
   * @throws CampaignsException
   */
  public static function createCampaign( $model, $dtStart, $dtStop = null );

  /**
   * Удаляет связанную кампанию со всеми партнерами
   *
   * @param ICampaign $model Партнер кампании или имя класса партнера
   *
   * @return bool
   *
   * @throws CampaignsException
   */
  public static function deleteCampaign( $model );

  /**
   * Устанавливает статус для партнера (участника) кампании
   *
   * @param ICampaign $model Участник кампании
   * @param int $state Статус
   * @param int $nextDate Следующий вызов смены статуса
   *
   * @return bool
   *
   * @throws CampaignsException
   */
  public static function setState( $model, $state, $nextDate );

}

```

### Интерфейс ICampaign

Основной инcтрумент, позволяющий реализовать логику *кампании* для *партнеров*.

```php

interface ICampaign {

  /**
   * Подтверждение удаления кампании в состоянии 'deleted'
   *
   * @param CampaignsModel $campaign Кампания
   *
   * @return bool Если TRUE - кампания будет удалена, FALSE - кампания останется.
   */
  public static function confirmDeleteCampaign( $campaign );

  /**
   * Подтверждение установки кампании в неактивное состояние
   * после окончания срока действия
   *
   * @param CampaignsModel $campaign Кампания
   *
   * @return bool Если TRUE - перевод состояния будет осуществлен, FALSE - останется в том же состоянии.
   */
  public static function confirmExpireCampaign( $campaign );

  /**
   * Возвращает сохраненный ранее Id связанной кампании
   *
   * @return int
   *
   * @see ICampaign::setCampaignId()
   */
  public static function getCampignId();

  /**
   * Новое событие партнера кампании.
   *
   * @param CampaignPartnersModel $campaignPartner
   *
   * @return void
   */
  public function partnerEvent( $campaignPartner );

  /**
   * Инициализация партнера кампании.
   * Вызывается при campaign_partners.state = null;
   *
   * @param CampaignPartnersModel $campaignPartner
   *
   * @return void
   */
  public function partnerInit( $campaignPartner );

  /**
   * Сохраняет Id связанной кампании
   *
   * @param int $id
   *
   * @return bool
   */
  public static function setCampaignId( $id );

}

```

### Класс CampaignPartnersModel

Экземпляр передается в класс партнера через ICampaign-методы **partnerInit** и
**partnerEvent**  позволяет управлять последующими событиями *кампании* для этого партнера.

```php

class CampaignPartnersModel extends CActiveRecord {

  /**
   * Устанавливает состояние для партнера кампании.
   *
   * @param int $state Код состояния
   * @param int $time Время нового события.
   * Если параметр не передан - время не меняется.
   * Если параметр NULL - событие больше не будет вызвано.
   *
   * @return bool
   *
   * @throws CampaignException
   */
  public function setState( $state, $time = 'undefined');

}

```

То есть, класс *партнера* имеет доступ к текущему своему состоянию, а так же имеет
способ его изменить.

**Внимание!** Не изменяйте напрямую поля этого класса во избежание коллизий.
Используйте функцию *setState( $state, $time )*.

Параметр *$state* в функции *setState( $state, $time )* отдается в произвольное
использование классу *партнера* для определения логики поведения *кампании*.

**Внимание!** Параметр *$time* в функции *setState( $state, $time )* имеет следующую
особенность: если установить его в *NULL*, то *партнер* не будет больше участвовать
в *кампании* до дальнейших изменений его состояния.

## Реализация

Для того чтобы запустить новую *кампанию*, нужно

1. Запустить или убедитися в присутствии в кроне задачи CampaignsCommand (см. **CampaignsCommand.md**)

2. От модели *связанной таблицы* наследовать класс *партнера*, в котором реализовать ICampaign.
```php

class КлассПартнера
  extends МодельСвязаннойТаблицы
  implements ICampaign
{ }

```

3. Создать *кампанию* посредством
```php

Campaign::createCampaign( ИмяКлассаПартнера, $timeStart[, $timeStop]);

```

4. Чтобы запустить другую *кампанию* на той же таблице, нужно реализовать новый
производный класс от модели *связанной таблицы*,
```php

class КлассПартнераДва
  extends МодельСвязаннойТаблицы
  implements ICampaign
{ }

```
и зарегистрировать новую *кампанию*:

```php

Campaign::createCampaign( ИмяКлассаПартнераДва, $timeStart[, $timeStop]);

```
