__Soulplatform__ – это mBaaS (mobile backend as a service) облачный сервис, позволяющий создавать и эксплуатировать приложения без необходимости написания server-side кода, поднятия собственных серверов и их поддержки.

# Руководство для Android

__SoulSDK__ предоставляет собой набор удобных инструментов для работы с Soulplatform API.

## Инициализация

 Добавте следующие строки в классе Application в метод onCreate():
```java
 @Override
    public void onCreate() {
        super.onCreate();
        SoulSDK.initialize("API_KEY", this);
    }
```
Вместо “API_KEY” вставьте ключ, соответствующий вашему аккаунту в Soul.

## Клиентская кастомизация

Soul предоставляет вам возможность создавать свое уникальное приложение на базе существующих решений, для того, чтобы кастомизировать ваше приложение вы можете  устанавливать те или иные значения параметров на сайте в админ-панели Soul, а также настроить SoulSDK при помощи  специального класса SoulConfigs. 

После того, как вы инициализировали SoulSDK, вы можете вызвать методы установки параметров. Это действие не является обязательным, если его проигнорировать все параметры получат значения по умолчанию.

```java
        SoulConfigs.setGCMSenderId(ApplicationConfig.SENDER_ID);
        SoulConfigs.setSearchLifeTimeValue(SoulConfigs.ONE_HOUR);
        SoulConfigs.setLikeReactionLifeTime(SoulConfigs.ONE_MONTH);
        SoulConfigs.setDislikeReactionLifeTime(SoulConfigs.ONE_YEAR);
        SoulConfigs.setUserAgentAppVersion(BuildConfig.VERSION_NAME);
       
        // other
        
```

## Список возможностей

* `SoulSDK` связывает ваше приложение с Soul Backend и предоставляет возможность использовать в вашем приложении готовые функции, которые почти всегда используются практически в каждом приложении для знакомств.

Вот основные возможности реализованные на данный момент:

* `SoulAuth` - набор методов необходимых для авторизации пользователя в системе. На данный момент доступна возможность авторизации через подтверждение номера мобильного телефона через sms.

* `SoulCurrentUser` - набор методов для работы с сущностью авторизованного пользователя. После авторизации из любого места приложения можно легко получить объект авторизованного пользователя, а также отдельно его свойства. При использовании методов SoulCurrentUser SDK само синхронизирует информацию с сервером, поэтому производя любые действия с авторизованным пользователем через класс SoulCurrentUser, вы всегда можете быть уверены, что вы работаете с актуальными данными и любые изменения свойств пользователя будут сохранены.

* `SoulMedia` - набор методов для работы с изображением, аудио и видео. Получение объектов альбомов пользователя, создание, удаление и редактирование альбомов с изображением, а также самих изображений.

* `SoulUsers` - набор методов для поиска других пользователей по определенным критериям. SoulSDK избавляет вас от необходимости описывать логику получения новых пользователей, а также случаи с обрывом соединения. 

* `SoulReactions` - набор методов для отправки ваших “реакций” на других пользователей. Сами реакции могут быть абсолютно произвольными и указанными вами на сайте Soul в админ панели. Однако самые очевидные и популярные вынесены в SoulReactions в простые удобные методы, не требующие излишней информации.

* `SoulCommunication` - набор методов для коммуникации между пользователями. На данный момент SoulSDK поддерживает коммуникацию через текстовые чаты. SoulSDK упрощает разработку чатов, достаточно указать с каким пользователем нужно создать чат и подписаться на прием/отправку сообщений вызвав один простой метод.

* `SoulEvents` - набор методов для быстрой и удобной актуализации информации по всем событиям и объектам, которые так или иначе касаются текущего авторизованного пользователя.

* `SoulPurchases` - набор методов для упрощения и безопастности (валидация на сервере) произведенных покупок в Google Play.

* `SoulSystem` - набор методов для получения информации о настройках, параметрах связанных с сервером Soul. Например: getServerTime()


### Возвращаемые значения

Любой метод из перечисленных выше классов вызывается в качестве статического метода этого класса и не требует никаких дополнительных зависимостей. 
Например:  SoulReactions.likeUser(userId) 

Все методы `SoulSDK` предоставляют возможность получения результатов в двух вариантах: 

* через универсальный интерфейс `SoulCallback`

```java
public interface SoulCallback<T> {
    void onSuccess(T responseEntity);
    void onError(SoulError error);
}
```

```java
public class SoulResponse<T> {
      private SoulError error;
      private boolean hasError = false;
      private T response;
    
    // ...
}
```

```java
public class SoulError {
      private String description = "";
      private int code = 0;

  // ...
}
```

* для поклонников реактивности через универсальный `Observable<SoulResponse<T>>`

```java
 SoulUsers.getNextSearchResult()
        .subscribe(
                res -> sowUsers(res.getResponse().getUsers()),
                err -> Log.e(TAG, "error " + err.getMessage()));
}
```



## Пример авторизации

Ниже рассмотрен пример реализации авторизации через подтверждение мобильного телефона на примере `SoulResponse`:

```java
public class AuthActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // при старте приложения автоматически пробуем авторизоваться
        SoulAuth.login(authCallback);
    }


    private SoulCallback authCallback = new SoulCallback<AuthorizationResponse>() {
        @Override
        public void onSuccess(AuthorizationResponse responseEntity) {
            // после каждого успешного SoulSDK автоматически сохраняет все необходимые данные 
            // для успешной авторизации в следующий раз, поэтому  никаких параметров в данный метод 
            // передавать не требуется
            startActivity(new Intent(this, NextBeautifulActivity.class));
        }

        @Override
        public void onError(SoulError error) {
            switch (error.getCode()) {
                case SoulError.NO_LOGIN_CREDENTIALS:
                    // в случае отсутствия необходимых данных для логина в фоне, показываем экран 
                    // с просьбой ввести номер телефона для авторизации
                    showPhoneInput();
                    break;
                case SoulError.PHONE_WRONG_CODE:
                    showWrongCodeError(error.getDescription());
                    break;
                default:
                    showError(error.getDescription());
            }
        }
    };

    // после того, как пользователь ввел номер телефона отправляем запрос SMS на этот 
    // номер мобильного телефона ...
    public void requestPhoneNumber(String phoneNumber) {
        SoulAuth.requestPhone(phoneNumber);
        //... и отображаем форму для ввода проверочного кода
        showVerificationCodeInput();
    }

    // после того, как пользователь получил проверочный код на указанный ранее номер телефона, 
    // отправляем код на валидацию
    public void verifyPhone(String verificationCode) {
        SoulAuth.verifyPhone(verificationCode, authCallback);
    }
}
```


## Пример работы с авторизованным пользователем

### CurrentUser

Авторизованного пользователя всегда можно получать данным методом здесь находится актуальная информация, т.к. объект всегда синхронизируется с сервером после каждой успешной авторизации.
        
```java
    User authorizedUser = SoulCurrentUser.getCurrentUser();
    Log.e(TAG, "Usersid is " + authorizedUser.getId());
```



```java
    SoulCurrentUser.updateSelfGender(myGenderET.getText().toString())
            .subscribe(
                    res -> Log.e(TAG, "success " + res.getResponse().getCurrentUser().getId()),
                    err -> Log.e(TAG, "error " + err.getMessage())));
```

### UsersParameters

Объект `CurrentUser` содержит в себе `UsersParameters`,

public class User {

```java
    private String id;
    private int recordId;
    private NotificationTokens notificationTokens;
    private SubscriptionServices subscriptionServices;
    private UsersParameters parameters; // <- параметры
    private List<Album> albums;
    private Reactions reactions;
    private Chat chat;
    
    // ...
}
```

которые в свою очередь представлены четырьмя разными группами параметров:

* __filterable__ - параметры, которые влияют или могут влиять на фильтрацию результатов при поиске пользователей  
* __publicVisible__ - информация о пользователях, доступная другим пользователям
* __publicWritable__ - данные пользователя, которые могут быть изменены другими пользователями
* __privateVisible__ - данные, доступные для изменения и просмотра только самим пользователем

Данные пармаметры являются свободным JSON и могут иметь любую структуру.

```java
    String filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
    JSONObject jObject = new JSONObject(filterable);
    String gender = jObject.getString("gender");
    Log.d(TAG, "gender: " + gender);
}
```

Создать свое произвольное свойство пользователя можно очень просто при помощи класса `SoulParameterObject`:

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    soulParameterObject.setProperty("myOwnProperty", "value Of Property");
    parameters.setFilterable(soulParameterObject);
    
    SoulCurrentUser.updateUserParameters(parameters).subscribe();
    // Обратите внимание, что ничего не сохраниться без вызова .subscribe()
    // т.к. метод можвращает Observable<SoulResponse<CurrentUserRESP>>
    // иначе с параметрами нужно передать дополнительно SoulCallback
```

Объект созданный при помощи `SoulParameterObject` можно наполнять свойствами при помощи метода `setProperty(...)`,
который принимает на вход String, людые Numbers, Boolean, а также любой валидный `SoulParameterObject`, в который также может быть вложены вышеперечисленные свойства или опять же `SoulParameterObject`.

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    soulParameterObject.setProperty("myOwnProperty", "value Of Property");

    SoulParameterObject nestedObject = new SoulParameterObject();
    nestedObject.setProperty("prop_1", 38);
    nestedObject.setProperty("prop_2", true);

    soulParameterObject.setProperty("myOwnProperty", "value Of Property");
    soulParameterObject.setProperty("nested", nestedObject);

    parameters.setFilterable(soulParameterObject);
        
    SoulCurrentUser.updateUserParameters(parameters).subscribe();
```

Получать результаты из SoulSDK (полученные от сервера по API) также гораздо удобней не в виде Json-строки, а в качестве SoulParameterObject.
В этом случае вам не требуется заниматься парсингом json.

```java
    SoulParameterObject filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
    String gender = filterable.getString("myOwnProperty");
    Log.d(TAG, "myOwnProperty: " + gender);

    SoulParameterObject nestedObj = filterable.getSoulParameterObject("nested");
    int prop1 =  nestedObj.getInt("prop_1");
    boolean prop2 =  nestedObj.getBoolean("prop_2");
    Log.d(TAG, "prop1: " + prop1 + " prop2: " + prop2);
```

Таким образом можно сохранить местоположение пользователя:

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    SoulParameterObject nestedObject = new SoulParameterObject();
    nestedObject.setProperty("lat", 38.74123);
    nestedObject.setProperty("lng", -9.1400233);
    soulParameterObject.setProperty("location", nestedObject);
    parameters.setFilterable(soulParameterObject);
    SoulCurrentUser.updateUserParameters(parameters).subscribe();

    SoulParameterObject filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
```

А затем прочитать его:

```java
    SoulParameterObject locationObject = filterable.getSoulParameterObject("location");
    Location location = new Location("");
    location.setLatitude(locationObject.getDouble("lat"));
    location.setLongitude(locationObject.getDouble("lng"));
```

### Популярные свойства пользователя

Однако, существуют такие свойства пользователей, которые используются почти любыми приложениями для знакомств.
Например, пол пользователя или рассмотренное выше местоположение. Работа с такими наиболее популярными свойствами в SoulSDK представлена специальными методами.  

На данный момент реализованные следующие "быстрые" методы для работы с такими свойствами:

В Soulplatform существует такое понятие как __`availability`__ - это способность находить других пользователей и быть видимым для поиска другими пользователями. Соответствует свойству {"filterable":{"availableTill": [unix time here]}},
где `availableTill` - это временная метка, время до которого пользователь обладает данной способностью.

* __`SoulCurrentUser.getTimeLeft()`__ - возвращает время оставшееся до конца __availability__

* __`SoulCurrentUser.turnSearchOn()`__ - устанавливает значение __availableTill__ равным сумме текущего времени сервера и длительности __availability__. Значение длительности задано по умолчанию в SoulSDK, однако ее всегда можно сменить вызовом метода  `SoulConfigs.setSearchLifeTimeValue(long val)`

* __`SoulCurrentUser.turnSearchOff()`__ - сбрасывает значение __availableTill__ и пользователь теряет способность __availability__

* __`SoulCurrentUser.updateGCMToken(String token)`__ - самый простой и быстрый способ сообщить серверу о токене для получения push уведомлений. Альтернативой может быть создание объекта `UsersParameters` самостоятельно и передачей собранного json на сервер.

* __`SoulCurrentUser.updateSelfGender(String gender)`__ - самый простой и быстрый способ передать на сервер пол авторизованного пользователя

* __`SoulCurrentUser.updateTargetGender(String gender)`__ - самый простой и быстрый способ передать на сервер пол пользователей, которые будут найдены в процессе поиска

* __`SoulCurrentUser.updateLocation(Location location)`__ - нет проще способа сообщить серверу о своем местоположении

Актуальное текущее время сервера всегда можно получить при помощи метода __`SoulSystem.getServerTime()`__


## Работа с изображениями

SoulPlatform дает возможность загружать, читать и удалять изображения. Любые изображения на сервере хранятся в __альбомах__, следовательно существует возможность добавлять, удалять и изменять альбомы. 
Для каждого пользователя существует как минимум один предустановленный альбом с именем __"default"__

### Пример получения фотографий из альбома

```java
    SoulMedia.getPhotosFromMyAlbum("default", 0, 10, new SoulCallback<AlbumRESP>() {
        @Override
        public void onSuccess(AlbumRESP responseEntity) {
            List<Photo> photoList =  responseEntity.getAlbum().getPhotos();
            String avatarPhoto =  responseEntity.getAlbum().getMainPhoto().getOriginal().getUrl();
        }

        @Override
        public void onError(SoulError error) {
            Log.e(TAG, "Error: " + error.getDescription());
        }
    });
```

Следующим способом можно удалить свою фотографию (необходимо знать альбом в котором она находится):
```java
    List<Photo> photoList =  responseEntity.getAlbum().getPhotos();
    String photoId = photoList.get(1).getId();
    SoulMedia.deleteMyPhoto("default", photoId, new SoulCallback<Boolean>() {...});
```

Чтобы добавить фотографию в альбом:
```java
    SoulMedia.addPhotoToMyAlbum("default", photoFile, new SoulCallback<PhotoRESP>() {...});
```


## Другие пользователи

Если пользователь авторизовался и обладает свойством __`availability`__, то ему доступен поиск других пользователей, а также отправка различных "реакций" найденным пользователям. 


### Поиск пользователей
В системе Soulplatform (в админ-панели в своем аккаунте можно задать набор фильтров и правил, по которым будут происходить поиск пользователей.
Однако, в SoulSDK существует предустановленный фильтр (по-умолчанию), использовать который элементарно:

```java
    SoulUsers.getNextSearchResult(new SoulCallback<UsersSearchRESP>() {
        @Override
        public void onSuccess(UsersSearchRESP responseEntity) {
            List<User> users = responseEntity.getUsers();
        }
    
        @Override
        public void onError(SoulError error) {
            //proceed error
        }
    });
```

Обратите внимание на то, что метод вызывается без передачи каких-либо параметров. Вызывая метод вы всегда будете получать __"новых"__ пользователей.

В тот момент, когда вы впервые вызвали данный метод, на сервере создается "сессия поиска" закрепленная за данным пользователем, от имени которого выполняется поиск.
Далее, каждый раз вызывая данный метод, в колбек будут приходить пользователи, которые еще ни разу не были получены через данный метод в рамках текущей "сессии поиска".

Однако, вы можете сбросить "сессию поиска" и тогда, снова вызвав данный метод, вы начнете получать пользователей, которых вы, возможно, получали в рамках предыдущей сессии.
Тем не менее, вы не будете получать пользователей,которым вы отправили когда-либо "реакцию" в рамках любой текущей или предыдущей сессии.

Для того, чтобы сбросить "сессию поиска" достаточно вызвать метод:
```java
    SoulUsers.refreshSearchResult();
```

### Взаимодействие с пользователями (Реакции)

Получив в результате поиска других пользователей вы можете отправить кому-либо из них произвольную "реакцию".
В Soulplatform существуют "группы реакций" и сами "реакции". 

Пример структуры: 
группы: `"liking"`, `"blocking"`
реакции:
 __"liking":__ "liked", "disliked"
 __"blocking":__ "blocked", ...
 
Сами отправляемые на сервер реакции представлены следующей структурой:
```java
public class ReactionREQ extends GeneralRequest {
    private String value;
    private long expiresTimeInSec;
    
    // ...
}
```
Обратите внимание на то, что у каждой реакции есть время жизни, по истечении которой, реакцию можно считать невалидной.

Пример отправки реакции "liked" в группу "liking" со временем жизни заявки в один год:
```java
    long reactionLifetime = 365 * 24 * 60 * 60; // 1 год
    ReactionREQ reaction = new ReactionREQ();
    reaction.setValue("liked");
    reaction.setExpiresTimeInSec(SoulSystem.getServerTime() / 1000 + reactionLifetime);
    SoulReactions.sendReactionToUser(likedUserId, "liking", reaction).subscribe();
```

Стоит отметить то, что отправленные разные реакции в одну и ту же группу, затирают друг друга, т.е. останется только последняя отправленная.

Таким образом отправив "like" какому-либо пользователю в группу "liking", вы можете снова отправить в эту группу "dislike", и именно "dislike" будет считаться актуально реакцией на данного пользователя в данной группе реакций. 

Однако, если вы хотите просто удалить какую-либо реакцию, а не перезаписать на другую, то для этого можно использовать метод:
```java
    SoulReactions.deleteReactionFromUser(likedUserId, "liking").subscribe();
```

### Популярные реакции
Как и в других разделах SoulSDK в `SoulReactions` также есть предустановленные реакции -
группы и значения, которые используются в большинстве дейтинг приложений.

* __`SoulReactions.likeUser(String userId)`__ - отправляет реакцию `"liked"` 
в группу `"liking"` со временем жизни установленным по умолчанию в SoulSDk, и который можно легко 
сменить при помощи метода `setLikeReactionLifeTime(long val)`

* __`SoulReactions.dislikeUser(String userId)`__ - отправляет реакцию `"disliked"` 
в группу `"liking"` со временем жизни установленным по умолчанию в SoulSDk, и который можно легко 
сменить при помощи метода `setDislikeReactionLifeTime(long val)`

* __`SoulReactions.blockUser(String userId)`__ - отправляет реакцию `"blocked"` 
в группу `"blocking"` со временем жизни установленным по умолчанию в SoulSDk, и который можно легко 
сменить при помощи метода `getBlockReactionLifeTime(long val)`


## Чаты

SoulSDK позволяет пользователям создавать чаты друг с другом. В дефолтном варианте 
чаты между пользователями возможны при условии, что оба пользователя имеют "метч" оба имеют реакции
"liked" друг на друга. Однако, это можно регулировать в админ-панели Soulplatform.

Для того, чтобы чаты работали в SoulSDK должны быть переданы ключи: `PUBLISH_KEY` и `SUBSCRIBE_KEY`

```java
    SoulConfigs.setChatsCredentials(PUBLISH_KEY, SUBSCRIBE_KEY);
```

Получить все имеющиеся чаты можно следующим способом:
```java
    SoulCommunication.getAll(0, 100, false, new SoulCallback<ChatsRESP>() {
        @Override
        public void onSuccess(ChatsRESP responseEntity) {
            List<Chat> chatList = responseEntity.getChats();
        }

        @Override
        public void onError(SoulError error) {

        }
    });
```

Также можно получить детали одного чата, если известно его `chatId`:
```java
    SoulCommunication.getOne(chatId, new SoulCallback<ChatRESP>() {
        @Override
        public void onSuccess(ChatRESP responseEntity) {
            Chat chat = responseEntity.getChat();
            String channelName = chat.getChannelName(); // получение уникального названия канала
        }

        @Override
        public void onError(SoulError error) {

        }
    });
```
Или удалить чат:
```java
    SoulCommunication.delete(chatId).subscribe();
```

Для того, чтобы начать отправлять и получать сообщения необходимо "присоединится" к чату:
```java
    SoulCommunication.connectToChat(chat, new SoulCallback<ChatMessage>() {
        @Override
        public void onSuccess(ChatMessage responseEntity) {
            showMessageInUIChatRoom(responseEntity); //сообщения один за одним приходят сюда
        }

        @Override
        public void onError(SoulError error) {

        }
    })
```

Как только вы подсоединились к чату, SoulSDK пришлет вам все сообщения, которые были вам присланы,
но вы их по какой-либо причине не получили. Таким образом произойдет синхронизация с историей сообщений.
Следующие поступающие входящие сообщения будут приходить в этот же колбек.

Для того, чтобы отправить сообщение, в зависимости от того, что вы хотите отправить,
необходимо просто вызвать один из следующих методов класса `SoulCommunication`:

* __`sendChatMessage(String channelName, String message)`__ - отправка обычного текстового сообщения

* __`sendChatPhoto(String channelName, File file)`__ - отправка изображения

* __`sendChatLocation(String channelName, Location location)`__ - отправка указанного местоположения

* __`sendChatCurrentLocation(String channelName)`__ - само определит теукщее местоположение и отправит

Для того, чтобы сообщения приходили также в виде push уведомлений (на случай, когда соединение с
чатом закрыто - например, закрыто приложение) следует передать SENDER_ID в

```java
    SoulConfigs.setGCMSenderId(ApplicationConfig.SENDER_ID);
```

## История событий

Пользователи в Soulplatform постоянно появляются, удаляются, взаимодействуют друг с другом, меняют свои статусы или другие свойства,
добавляются в чаты и удаляются. О большинстве из этих событий ваш авторизованный пользователь может получать уведомления в виде push нотификаций.

Однако, такие нотификации иногда могут быть не настроены, push-нотификации могут опаздывать или вовсе не приходить.
Пользователь мог отключить свой девайс на время и в течение него не получать никаких уведомлений.

За это время с интересующими его пользователями произошло множество изменений,
а также могут быть открыты чаты с вами или полученные от других пользователей важные реакции на вас.

Все исторические события, что вы пропустили за время отсутствия можно получить при помощи "синхронизации событий":
```java
    SoulEvents.get(Long sinceDate, Integer after, Integer limit, new SoulCallback<EventsRESP>() {...});
```
При вызове метода следует указать или время, начиная с которого вы хотели бы получить "события" либо последний присланный вам recordId "события".
Соответственно вам нужно хранить дату последней синхронизации или recordId последнего известного вам сообщения.

Однако, вам ничего из этого не нужно делать, т.к. SoulSDK делает все это за вас. 
Просто вызовите метод:

```java
    SoulEvents.get(new SoulCallback<EventsRESP>() {...});
```
В колбек вам прийдет результат соответствующий запросу:
after - сохраненный в SoulSDK recordId последнего известного события.
limit - значение задано в SoulSDK по-умолчанию, которое можно сменить используя метод: 
```java
    SoulConfigs.setEventsLimit(...);
```





