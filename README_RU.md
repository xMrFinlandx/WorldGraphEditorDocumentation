# Содержание  
  
- [Введение](#введение)  
  - [Материалы](#материалы)
  - [Основные возможности](#основные-возможности)
  - [Дополнительные возможности](#дополнительные-возможности)
  - [Технические детали](#технические-детали)
- [Использование](#использование)  
  - [С чего начать](#с-чего-начать)  
  - [Сохранение графа и настройка компонентов](#сохранение-и-настройка)  
  - [Проверка](#проверка)  
- [Кастомизация](#кастомизация)  
  - [Собственная реализация портов](#собственная-реализация-портов)  
  - [Класс Dropdown](#класс-dropdown)  
  - [Выталкивание из проходов с помощью PushData](#выталкивание-из-проходов)  
  - [Реализация запуска TransitionManager](#собственная-реализация-запуска-transitionmanager)  
  - [События и вызов асинхронного кода](#события-и-вызов-асинхронного-кода)  
- [Дорожная карта](#дорожная-карта)  

# Введение  

World Graph Editor — это инструмент, предоставляющий удобный редактор на основе узлов (нод), который позволяет легко определять связи между сценами, упрощая создание проектов с большим количеством сцен. Помимо этого, инструмент содержит встроенные механизмы осуществления переходов с возможностью ожидания выполнения асинхронных задач.  
## Материалы

- Страница в [AssetStore](https://assetstore.unity.com/packages/slug/306228)  
- Туториал на [YouTube](https://www.youtube.com/watch?v=-ATVm00ctiA)

## Основные возможности

- Интуитивно понятное и простое в использовании окно редактора 
- Возможность задать один из трех типов связей между сценами
- Визуальное выделение и подписи для GameObject-ов, представляющих порты на графе
- Автоматическое добавление сцен в Build Settings
- Проверка корректности данных при входе в Play Mode или сборке проекта
- Совместимость с 2D и 3D проектами
- Поддержка отмены/повтора действий (Undo/Redo)
- Поддержка Unity 6

## Дополнительные возможности

- Префаб TransitionManager для управления переходами между сценами во время игры
- Возможность автоматически создавать объект игрока при загрузке новой сцены
- Поддержка выполнения асинхронного кода во время переходов между сценами
- Демо проект с 10 сценами
- Поддержка написания собственных компонентов

## Технические детали

**Поддерживаемые операционные системы:**  
Windows, macOS, Linux

Инструмент работает внутри редактора Unity и был протестирован на Windows и macOS. Совместимость с Linux ожидается, но не тестировалась должным образом.

**Поддерживаемые платформы:**  
Windows, macOS, WebGL

Никаких ограничений, связанных с другими платформами, обнаружено не было, однако на них не проводилось отдельного тестирования.

**Демо:**  
Инструмент не зависит от используемого рендер-пайплайна, однако демонстрационные сцены используют URP и старую систему ввода (Input System).

# Использование
## С чего начать
### Создание World Graph Container  
  
1. В папке проекта создайте `WorldGraphContainer`:  
    - **ПКМ → Create → World Graph Editor → World Graph Container**  
2. Дважды нажмите ЛКМ по созданному объекту, чтобы открыть окно редактора  
  
### Добавление сцен  
  
Перетащите нужные сцены в открывшееся окно для создания нод. Каждая нода представляет собой сцену, а каждый порт на ноде обозначает переход на другую сцену.  
  
> [!WARNING]  
> Избегайте дубликатов сцен, так как это приведёт к ошибкам. Дубликаты будут подсвечены красным.  
  
### Виды портов  
  
При создании портов с помощью кнопки "+" на ноде открывается контекстное меню, позволяющее создать один из следующих портов:  
- **Left Passage**, **Right Passage**, **Top Passage**, **Bottom Passage** — связующие порты (проходы), для создания связей, которые используются для соединения нод  
- **Additional Port** — порт, не предназначенный для связи, используется в случаях, требующих, например, реализации системы быстрого перемещения   
  
### Создание портов и связей  
  
1. Нажмите на "+" на выбранной ноде и создайте связующий порт  
2. Перетащите связь из порта первой ноды к середине второй ноды  
3. Если все сделано правильно, ноды окажутся связаны между собой  
  
Таким образом, вы указали какими проходами сцены будут связаны между собой. Дополнительная настройка портов доступна в окне **Inspector**. Выберите нужную ноду, чтобы изменить её параметры.    
  
> [!WARNING]  
> Каждый порт на ноде должен иметь уникальное название, а также оно не может быть пустым или состоять только из пробелов. В противном случае это приведет к ошибкам.  
  
### Типы возможных связей между нодами  
  
- **Non Directed** — Связь по умолчанию. Перемещение между проходами возможно в любом направлении  
- **Shortcut** — Указывает, что проход изначально доступен только с одной стороны (например, запертая дверь, открывающаяся с другой стороны). Может учитываться в [собственной реализации портов](#собственная-реализация-портов)  и блокировать переход в зависимости от условий
- **Single Directed** — Связь, позволяющая проходить исключительно в заданном направлении  
  
Тип связи настраивается в контекстном меню, которое открывается с помощью клика ПКМ по уже существующей связи.  
## Сохранение и настройка  
### Сохранение графа  
  
Когда закончите работу с графом, нажмите кнопку "Save" чтобы сохранить его.  
  
> [!NOTE]  
> При сохранении, все несвязанные порты, кроме дополнительных (Additional Port), будут удалены.  
  
> [!NOTE]  
> При сохранении будет предложено добавить все использованные сцены в **Build Settings**, так как это необходимо для дальнейшей работы.  
  
### Настройка TransitionManager  
  
1. Убедитесь, что менеджер существует по пути: `Assets/WorldGraphEditor/Resources/TransitionManager`. Если нет,   
создайте его с помощью пункта меню:   
   - **Tools → World Graph Editor → Create Transition Manager Prefab**  
1. Перетащите сохранённый `WorldGraphContainer` в соответствующее поле в `TransitionManager`. Таким образом, вы указываете какой контейнер будет использоваться  
2. Установите `AutoLoad` в значение `true`. Это позволит менеджеру загружаться автоматически при старте игры  
3. Установите чекбокс в правой части поля `PlayerPrefab` в значение `true` и перетащите в него префаб игрока  
  
Благодаря этому на каждой сцене автоматически будет создаваться объект игрока. Если вам не подходит данный способ загрузки менеджера, вы можете [использовать свой](#собственная-реализация-запуска-transitionmanager).  
  
### Настройка сцен  
  
Для представления портов на сценах добавьте соответствующие компоненты.    
По умолчанию доступны:  
- **Passage2D** - Перемещает игрока к противоположному порту, опираясь на заданную связь между ними в графе  
- **Teleport2D** - Перемещает игрока к любому другому существующему порту на графе  
  
Вы также можете задать позицию по умолчанию, добавив на сцену объект с компонентом `DefaultSpawnPosition`.  Благодаря этому, если на сцене не будет найден выходной порт `OutputTransitionComponent`, игрок появится на этой позиции.  
  
### Настройка компонентов  
  
- **Passage2D**  
    - В поле `AssignedPort` выберите имя прохода, который будет соответствовать порту в графе  
  
- **Teleport2D**  
    - В поле `AssignedPort` выберите имя прохода, который будет соответствовать порту в графе  
  - В поле `GoTo` выберите порт, к которому должен быть осуществлён переход  
  
> [!TIP]  
> Каждый порт на сцене в поле `AssignedPort` должен ссылаться к одному порту на ноде.
> В поле `AssignedPort` для каждого порта на сцене должно быть указано соответствие определённому порту на ноде.  
  
При необходимости, вы можете написать [собственные реализации](#собственная-реализация-портов).  
  
## Проверка  
### Ожидаемое поведение  
  
- При запуске в **DontDestroyOnLoad** автоматически создаётся объект `TransitionManager`  
- При контакте с объектом порта, персонаж перемещается на другую сцену, в позицию соответствующую целевому порту  
  
### Ошибки  
  
Если целевой порт отсутствует на сцене:  
- Объект игрока появится в позиции, заданной в `DefaultSpawnPoint`, или, если она не задана — в позиции (0, 0, 0)  
- В консоль будет выведено сообщение об ошибке  
  
> [!WARNING] 
> Игра автоматически остановится, если:  
> - **TransitionManager** отсутствует по пути:`Assets/WorldGraphEditor/Resources/TransitionManager`  
> - Поле `Container` в `TransitionManager` пустое  
> - Контейнер содержит в себе ошибки  
> - Сцены в **BuildSettings** были удалены, отключены или изменён их порядок  
>  
> В консоль будет выведена ошибка с инструкцией по её исправлению.  
  
  
# Кастомизация  
  
## Собственная реализация портов  
  
### Общие рекомендации  
  
Для создания собственной реализации:  
- Наследуйтесь от `PassageBase` или `TeleportBase`, либо реализуйте `ITransitionComponent`
- Используйте класс [Dropdown](#класс-dropdown) и [соответствующие методы](#заполнение-dropdown-данными-из-графа) в `WorldGraphContainer` для создания выпадающих списков для инициализации текущего порта или выбора любого порта на графе  
- Реализуйте метод `GetGuid()`, возвращающий `Guid` текущего порта  
- Используйте метод `Refresh()` для получения данных из `TransitionManager` и `WorldGraphContainer` для обновления полей класса  
  
> [!TIP]  
> Метод `Refresh()` рекомендуется обернуть в `#if UNITY_EDITOR`, так как он нужен только в редакторе. Вы можете вызвать у всех компонентов на сцене метод `Refresh()` используя `TransitionManager.RefreshPorts()`.  
  
> [!NOTE]  
> Метод `Refresh()` может передать `null`, если `TransitionManager` отсутствует по указанному пути.  
  
### Методы переходов в TransitionManager  
  
```csharp  
Task LoadScene(int buildIndex)  
```  
Загружает сцену по переданному индексу.  
  
Параметры:  
  
| Имя          | Тип   | Описание          |
| ------------ | ----- | ----------------- |
| `buildIndex` | `int` | Build Index сцены |

  
---  
  
  
```csharp  
void GoFrom(string currentPortGuid, bool ignoreShortcuts)  
```  
Перемещает игрока к противоположному порту, **опираясь** на связи в графе.  
  
Параметры:  
  
| Имя               | Тип      | Описание                 |  
|-------------------|----------|--------------------------|  
| `currentPortGuid` | `string` | `Guid` текущего порта    |  
| `ignoreShortcuts` | `bool`   | Игнорировать ли шорткаты |  
  
---  
  
  
```csharp  
void GoTo(string currentPortGuid, string targetPortGuid)  
```  
Перемещает игрока на любой выбранный порт. **Не опирается** на связи в графе.  
  
Параметры:  
  
| Имя               | Тип       | Описание                                    |  
|-------------------|-----------|---------------------------------------------|  
| `currentPortGuid` | `string`  | `Guid` текущего порта                       |  
| `targetPortGuid`  | `string`  | `Guid` целевого порта                       |  
  
---  
  
  
#### Пример  
  
Пример реализации телепорта с возможностью выбора конечного места для перемещения   
  
```csharp  
using System.Linq;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace WorldGraphEditor.Examples
{
    public class MyExampleTeleport : MonoBehaviour, ITransitionComponent, IInteractable
    {
        [SerializeField] private Dropdown<TransitionData> _assignedPort = new();

        [SerializeField, ReadOnlyField] private string _currentGuid;
        
#if UNITY_EDITOR
        private WorldGraphContainer _container;
        
        public void Refresh(TransitionManager manager)
        {
            if (manager != null)
                _container = manager.Container;

            if (_container == null)
                return;

            // Получения индекса сцены
            var sceneBuildIndex = SceneManager.GetActiveScene().buildIndex;
            
            // Получение данных обо всех портах на конкретной сцене
            var data = _container.GetTransitionsDropdownData(sceneBuildIndex);
            
            // Установка значений в выпадающий список
            _assignedPort.SetData(data);

            // Получение выбранного значения из выпадающего списка
            _currentGuid = _assignedPort.GetSelectedValue().CurrentPassageGuid;
        }
#endif

        private void OnValidate()
        {
            TransitionManager.RefreshPorts();
        }

        public Vector3 GetSpawnPosition() => transform.position;

        public string GetGuid() => _currentGuid;

        public void Interact()
        {
            // Примерная логика получения информации о посещенных телепортах
            var data = MyRuntimeData.VisitedTeleports
                .Where(item => item.guid != GetGuid())
                .ToArray();
            
            // Вывод окна для выбора телепорта, к которому нужно осуществить перемещение
            TeleportSelectionWindow.Instance.ShowDialog(data, OnSelect);
        }

        private void OnSelect(string selectedGuid)
        {
            // Закрытие окна для выбора телепорта
            TeleportSelectionWindow.Instance.CloseDialog();
            
            if (selectedGuid == string.Empty)
                return;
            
            // Вызов метода для перемещения
            TransitionManager.Instance.GoTo(GetGuid(), selectedGuid);
        }
    }
}
```  
---  
  
### Класс Dropdown  
  
Dropdown — это класс, предназначенный для создания выпадающих списков, связанных с портами в графе. Он обеспечивает сохранность выбранного значения даже при переименовании портов или изменении их количества на ноде.  
  
Класс содержит два метода:  
  
```csharp  
T GetSelectedValue()  
```  
  
Возвращает выбранное значение.  
  
Возвращаемые значения:  
  
| Тип | Описание                                                                             |     |
| --- | ------------------------------------------------------------------------------------ | --- |
| `T` | Выбранное значение, если такое найдено, иначе — значение по умолчанию (`default(T)`) |     |
  
---  
  
```csharp  
void SetData(IEnumerable<(string Guid, string DisplayName, T Value)> data)  
```  
Обновляет данные для выпадающего списка, состоящие из `Guid`, отображаемого имени и значения.  
  
Параметры:  
  
| Имя    | Тип                                                       | Описание                                                                      |  
|--------|-----------------------------------------------------------|-------------------------------------------------------------------------------|  
| `data` | `IEnumerable<(string Guid, string DisplayName, T Value)>` | Данные, где каждый элемент содержит`Guid`, имя для отображения и значение `T` |  
  
---  
  
### Заполнение Dropdown данными из графа  
  
В `WorldGraphContainer` предусмотрены **EditorOnly** методы для упрощенного заполнения `Dropdown` данными из графа.  
  
  
```csharp  
IEnumerable<(string Guid, string Name, TransitionData Data)> GetTransitionsDropdownData(int buildIndex)  
```  

Параметры:  
  
| Имя          | Тип    | Описание                                        |     |
| ------------ | ------ | ----------------------------------------------- | --- |
| `buildIndex` | `int`  | Индекс сцены, для которой нужно получить данные |     |

Возвращаемые значения:  
  
| Тип                                                            | Описание                        |  
|----------------------------------------------------------------|---------------------------------|  
| `IEnumerable<(string Guid, string Name, TransitionData Data)>` | Данные обо всех портах на сцене |  
  
  
#### Пример  
  
```csharp  
using UnityEngine;
using UnityEngine.SceneManagement;

namespace WorldGraphEditor.Examples
{
    public class MyComponentWithDropdown : MonoBehaviour, ITransitionComponent
    {
        // Выпадающий список с типом TransitionData
        [SerializeField] private Dropdown<TransitionData> _assignedPortDropdown = new();

        private string _currentPortGuid;
        
#if UNITY_EDITOR
        private WorldGraphContainer _container;
        
        public void Refresh(TransitionManager manager)
        {
            if (manager != null)
                _container = manager.Container;

            if (_container == null)
                return;

            // Получение индекса текущей сцены
            var currentSceneBuildIndex = SceneManager.GetActiveScene().buildIndex;
        
            // Получение данных обо всех портах на сцене
            var scenePortsData = _container.GetTransitionsDropdownData(currentSceneBuildIndex);
            
            // Установка данных в выпадающий список
            _assignedPortDropdown.SetData(scenePortsData);
        
            // Получение выбранного значения из списка
            _currentPortGuid = _assignedPortDropdown.GetSelectedValue().CurrentPassageGuid;
        }
#endif
    }
}
```  
  
---  
  
```csharp  
IEnumerable<(string Guid, string Path, PortData Data)> GetAllPortsDropdownData()  
```  
  
Возвращает данные обо всех портах на графе, группируя их по сценам.  
  
Возвращаемые значения:  
  
| Тип                                                      | Описание                         |  
|----------------------------------------------------------|----------------------------------|  
| `IEnumerable<(string Guid, string Path, PortData Data)>` | Данные обо всех портах на графе  |  
  
#### Пример  
  
```csharp  
using UnityEngine;

namespace WorldGraphEditor.Examples
{
    public class MyComponentWithDropdown : MonoBehaviour, ITransitionComponent
    {
        // Выпадающий список с типом PortData
        [SerializeField] private Dropdown<PortData> _targetPortDropdown = new();

        private string _targetPortGuid;
        
#if UNITY_EDITOR
        private WorldGraphContainer _container;
        
        public void Refresh(TransitionManager manager)
        {
            if (manager != null)
                _container = manager.Container;

            if (_container == null)
                return;
        
            // Получение данных обо всех портах в графе
            var allPortsData = _container.GetAllPortsDropdownData();
        
            // Установка данных в выпадающий список
            _targetPortDropdown.SetData(allPortsData);
        
            // Получение выбранного значения из списка
            _targetPortGuid = _targetPortDropdown.GetSelectedValue().Guid;
        }
#endif
    }
}
```  
  
---  
  
### Выталкивание из проходов  
  
Вы можете добавить "силу выталкивания" для порта, реализовав интерфейс `IPusher` и метод `GetPushData()`,  возвращающий `PushData`.  
  
PushData это структура, содержащая в себе два поля:  
- `Force` — Сила выталкивания  
- `Exists` — Указывает, существует сила или нет  
  
```csharp  
public readonly struct PushData
{
    public readonly Vector3 Force;
    public readonly bool Exists;

    public PushData(Vector3 force)
    {
        Force = force;
        Exists = true;
    }
} 
```  
  
#### Пример  
  
```csharp  
using UnityEngine;

namespace WorldGraphEditor.Examples
{
    public class MyPushingPassage : PassageBase, IPusher
    {
        [SerializeField] private Vector2 _pushForce;
        
        public PushData GetPushData()
        {
            // Если сила (0, 0), то можно вернуть PushData со значениями по умолчанию
            if (_pushForce == Vector2.zero)
                // Возвращает значение по умолчанию, где Exists будет false
                return default; 
            
            // Если сила есть, то ее можно передать в конструктор. Exists в таком случае станет true
            return new PushData(_pushForce);
        }
        
        // Прочие методы
        public override Vector3 GetSpawnPosition() //...
        private void OnValidate() //...
    }
}
```  
  
Чтобы применить эту силу к объекту игрока, используйте `TransitionManager.Instance.PushData` для получения актуального значения.  
  
#### Пример  
  
```csharp  
using UnityEngine;

namespace WorldGraphEditor.Examples
{
    public class PlayerController : MonoBehaviour
    {
        [SerializeField] private Rigidbody2D _rigidbody;

        private void OnTransitionEnded()
        {
            // Получение силы выталкивания
            var pushData = TransitionManager.Instance.PushData;
            
            // Применение силы к Rigidbody только если она существует
            if (pushData.Exists) 
                _rigidbody.AddForce(pushData.Force, ForceMode2D.Impulse);
        }

        private void Awake()
        {
            TransitionManager.OnTransitionEnded += OnTransitionEnded;
        }
        
        private void OnDestroy()
        {
            TransitionManager.OnTransitionEnded -= OnTransitionEnded;
        }
    }
}
```  
  
## Собственная реализация запуска TransitionManager  
  
По умолчанию `TransitionManager` запускается автоматически при старте игры, однако вы можете создать его в удобный для вас момент.  
  
### Отключение автозагрузки  
  
1. Установите `AutoLoad` в `false`  
2. Чтобы создать менеджер, вызовите `TransitionManager.CreateInstance()`  
  
#### Пример  
```csharp  
using UnityEngine;

namespace WorldGraphEditor.Examples
{
    public class MyCustomTransitionManagerLoader : MonoBehaviour
    {
        [SerializeField] private float _loadDelay = 2f;
        
        private void Start()
        {
            Invoke(nameof(MyLoadMethod), _loadDelay);
        }

        private void MyLoadMethod()
        {
            // Создание менеджера
            TransitionManager.CreateInstance();
        }
    }
}
```  
> [!NOTE]  
> `TransitionManger` так же содержит в себе метод `LoadFromResources()`, однако он лишь возвращает экземпляр класса из папки **Resources** и не создает объект в **DontDestroyOnLoad.** В первую очередь этот метод используется в режиме редактора для получения ссылки. Рекомендуется использовать именно `CreateInstance()`, поскольку в нем предусмотрена защита от создания нескольких экземпляров класса.  

---  
## События и вызов асинхронного кода  
  
`TransitionManager` обладает событиями, которые оповещают о текущих фазах перехода:  
  
- **OnInitialized**: Вызывается после успешной инициализации `TransitionManager`  
- **OnPortEntered**: Вызывается при вызове `GoFrom()` или `GoTo()`  
- **OnTransitionStarted**: Вызывается перед началом загрузки следующей сцены  
- **OnSceneLoaded**: Вызывается после загрузки сцены и после того, как был найден "выходной порт"  `OutputTransitionComponent` а так же, определена позиция появления игрока `PlayerSpawnPosition`  
- **OnTransitionEnded**: Вызывается после создания префаба игрока. Если префаб не был указан, событие все равно будет вызвано  
- **OnPortLeaved**: Вызывается после `OnTransitionEnded`  
  
### Задержка между фазами перехода  
  
Вы можете задать задержку между фазами перехода перед вызовом асинхронных методов с помощью `EventCallConfig`
  
1. Создание  
    - **ПКМ → Create → World Graph Editor → Event Call Config**  
2. Укажите один из доступных видов задержки  
   - **This Frame** — В текущем кадре  
   - **Next Frame** — В следующем кадре  
   - **Physics Update** — В `FixedUpdate()`  
   - **Custom Delay** — Через заданное время  
3. Перетащите `EventCallConfig` в соответствующее поле `TransitionManager`  
  
### Ожидание завершения асинхронных методов  
  
Если нужно дождаться выполнения операций, таких, как генерация уровня, вы можете зарегистрировать асинхронные методы с учётом их приоритета с помощью `TransitionManager.RegisterAsyncHandler()`. Чем выше значение приоритета, тем раньше метод будет вызван.   
  
> [!TIP]  
> Таким образом, порядок вызова при каждой фазе перехода будет таким:  
> 1. Вызов события перехода (**OnPortEntered / OnTransitionStarted / OnSceneLoaded / OnTransitionEnded**)  
> 2. Ожидание задержки `EventCallConfig` 
> 3. Ожидание выполнения асинхронных методов  
  
Тип события, после которого начинается выполнение зарегистрированных асинхронных методов, указывается с помощью перечисления `EventType`. 

События, для которых можно зарегистрировать асинхронные методы:  
  
```csharp  
namespace WorldGraphEditor
{
    public enum EventType
    {
        OnPortEntered,
        OnTransitionStarted,
        OnSceneLoaded,
        OnTransitionEnded,
    }
}
```  
---  
  
### Работа с асинхронными методами  
  
```csharp  
static void RegisterAsyncHandler(EventType eventType, Func<Task> func, int priority)  
```  
  
Параметры:  

| Имя         | Тип          | Описание                                                              |     |
| ----------- | ------------ | --------------------------------------------------------------------- | --- |
| `eventType` | `EventType`  | Тип события, после которого должен начаться вызов асинхронных методов |     |
| `func`      | `Func<Task>` | Метод, возвращающий `Task`, который нужно зарегистрировать            |     |
| `priority`  | `int`        | Приоритет вызова методов                                              |     |
  
---
  
```csharp  
static void UnregisterAsyncHandler(EventType eventType, Func<Task> func)  
```  
  
Параметры:  
  
| Имя          | Тип          | Описание                                          |  
|--------------|--------------|---------------------------------------------------|  
| `eventType`  | `EventType`  | Тип события, для которого нужно удалить метод     |  
| `func`       | `Func<Task>` | Метод, возвращающий `Task`, который нужно удалить |  
  
---
  
> [!NOTE]  
> `TransitionManager` в методе `OnDestroy` автоматически удаляет все зарегистрированные в нем методы.  
  
#### Пример  
  
```csharp  
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.UI;

namespace WorldGraphEditor.Examples
{
    public class MyWorldGenerator : MonoBehaviour
    {
        [SerializeField] private Text _worldTextMeshStatus;
        [SerializeField] private Text _lootTextMeshStatus;
        [SerializeField] private Text _enemiesTextMeshStatus;

        [SerializeField] private float _worldGenerationTime;
        [SerializeField] private float _spawnLootTime;
        [SerializeField] private float _spawnEnemiesTime;
        
        // Событие, после которого будут вызываться асинхронные операции. В данном случае - после загрузки сцены
        private EventType _eventType = EventType.OnSceneLoaded;

        private void Awake()
        {
            // Выполнится первым из-за высокого приоритета
            TransitionManager.RegisterAsyncHandler(_eventType, GenerateWorld, 2);
            
			// Будут выполнены после, в порядке их регистрации, поскольку приоритет одинаковый
            TransitionManager.RegisterAsyncHandler(_eventType, SpawnLoot, 1);
            TransitionManager.RegisterAsyncHandler(_eventType, SpawnEnemies, 1);
            
            _lootTextMeshStatus.text = "Waiting...";
            _enemiesTextMeshStatus.text = "Waiting...";
        }
        
		// Асинхронные методы. 
		// Каждый метод сначала обновляет текст, затем ждёт заданное время и снова обновляет текст
        private async Task GenerateWorld()
        {
            _worldTextMeshStatus.text = "Generating...";
            await Awaitable.WaitForSecondsAsync(_worldGenerationTime);
            _worldTextMeshStatus.text = "Complete!";
        }
        
        private async Task SpawnLoot()
        {
            _lootTextMeshStatus.text = "Spawning...";
            await Awaitable.WaitForSecondsAsync(_spawnLootTime);
            _lootTextMeshStatus.text = "Complete!";
        }

        private async Task SpawnEnemies()
        {
            _enemiesTextMeshStatus.text = "Spawning...";
            await Awaitable.WaitForSecondsAsync(_spawnEnemiesTime);
            _enemiesTextMeshStatus.text = "Complete!";
        }
        
        private void OnDisable()
        {
            // Удаление методов 
            TransitionManager.UnregisterAsyncHandler(_eventType, GenerateWorld);
            TransitionManager.UnregisterAsyncHandler(_eventType, SpawnLoot);
            TransitionManager.UnregisterAsyncHandler(_eventType, SpawnEnemies);
        }
    }
}
```  
  ---
## Дорожная карта  
  
- Поддержка нескольких сцен (вариаций) для одной ноды  
- Валидация всех сцен, указанных в графе (обнаружение не назначенных портов, дубликатов портов и тд.)  
- Поддержка написания собственных реализаций `TransitionManager`  
- Кастомизация нод  
- Алгоритм для поиска пути через выбранные сцены