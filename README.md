# patterns
## Порождающие(creational)


**Фабричный метод** — это порождающий паттерн проектирования, который делегирует создание объектов подклассам. То есть он предоставляет интерфейс для создания объектов, но позволяет подклассам решать, какой класс инстанцировать.
Зачем он нужен?
- Разделение ответственности — логика создания объекта отделяется от его использования.
- Расширяемость — легко добавлять новые типы продуктов, не меняя существующий код.
- Инкапсуляция — клиентский код не знает, какие именно классы используются, и работает через абстракции.


```
class Transport {
public:
    virtual void deliver() = 0;
    virtual ~Transport () = default;
};
class Truck : public Transport {
public:
    void deliver() override {
        std::cout << "Deliver by land in a truck" << std::endl;
    }
};
class Ship : public Transport{
public:
    void deliver() override {
        std::cout << "Deliver by sea in a ship" << std::endl;
    }
};
class Logistics {
public:
    virtual std::unique_ptr<Transport> createtransport() = 0;
    void plane_delivery() {
        auto transport = createtransport();
        transport->deliver();
    }
    virtual ~Logistics() = default;
};
class RoadLogistics : public Logistics {
public:
    std::unique_ptr<Transport> createtransport() override {
        return std::make_unique<Truck>();
    }
};
class SeaLogistics : public Logistics {
public:
    std::unique_ptr<Transport> createtransport() override {
        return std::make_unique<Ship>();
    }
};
int main()
{
    RoadLogistics R;
    SeaLogistics S;
    R.plane_delivery();
    S.plane_delivery();

    return 0;
}
```


**Абстрактная фабрика** — это порождающий паттерн, который предоставляет интерфейс для создания семейств связанных объектов, не уточняя их конкретные классы.
Зачем он нужен?
Когда у тебя:
- есть группы объектов, которые должны сотрудничать друг с другом (например, кнопка и окно из одного GUI),
- и ты хочешь переключаться между ними динамически (например, между Windows и macOS),
- и при этом ты не хочешь писать кучу if и switch, и жёстко привязываться к конкретным классам.


```
class Button {
public:
    virtual void paint() = 0;
    virtual ~Button() = default;
};

class Checkbox {
public:
    virtual void render() = 0;
    virtual ~Checkbox() = default;
};
class WinButton : public Button {
public:
    void paint() override {
        std::cout << "Render Windows Button\n";
    }
};

class WinCheckbox : public Checkbox {
public:
    void render() override {
        std::cout << "Render Windows Checkbox\n";
    }
};
class MacButton : public Button {
public:
    void paint() override {
        std::cout << "Render Mac Button\n";
    }
};

class MacCheckbox : public Checkbox {
public:
    void render() override {
        std::cout << "Render Mac Checkbox\n";
    }
};
class GUIFactory {
public:
    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<Checkbox> createCheckbox() = 0;
    virtual ~GUIFactory() = default;
};
class WinFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<WinButton>();
    }
    std::unique_ptr<Checkbox> createCheckbox() override {
        return std::make_unique<WinCheckbox>();
    }
};

class MacFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<MacButton>();
    }
    std::unique_ptr<Checkbox> createCheckbox() override {
        return std::make_unique<MacCheckbox>();
    }
};
void renderUI(GUIFactory& factory) {
    auto button = factory.createButton();
    auto checkbox = factory.createCheckbox();

    button->paint();
    checkbox->render();
}

int main() {
    WinFactory winFactory;
    MacFactory macFactory;

    renderUI(winFactory);  // Render Windows Button + Checkbox
    renderUI(macFactory);  // Render Mac Button + Checkbox
}
```


**Строитель** — это порождающий паттерн, который позволяет поэтапно создавать сложные объекты. Он отделяет процесс создания объекта от его представления.
Зачем нужен?
- Когда конструктор с 10 параметрами становится неудобным;
- Когда нужен гибкий способ настройки объекта перед его созданием;
- Когда у объекта есть разные "варианты сборки", но он должен быть одним и тем же классом в итоге.

  
```
class Computer {
public:
    std::string cpu;
    std::string gpu;
    std::string ram;
    std::string storage;

    void specs() const {
        std::cout << "CPU: " << cpu << "\n"
            << "GPU: " << gpu << "\n"
            << "RAM: " << ram << "\n"
            << "Storage: " << storage << "\n";
    }
};
class ComputerBuilder {
public:
    virtual void setCPU() = 0;
    virtual void setGPU() = 0;
    virtual void setRAM() = 0;
    virtual void setStorage() = 0;
    virtual Computer getResult() = 0;
    virtual ~ComputerBuilder() = default;
};
class GamingComputerBuilder : public ComputerBuilder {
private:
    Computer pc;
public:
    void setCPU() override { pc.cpu = "Intel i9"; }
    void setGPU() override { pc.gpu = "NVIDIA RTX 4090"; }
    void setRAM() override { pc.ram = "32GB DDR5"; }
    void setStorage() override { pc.storage = "2TB NVMe SSD"; }
    Computer getResult() override { return pc; }
};
class OfficeComputerBuilder : public ComputerBuilder {
private:
    Computer pc;
public:
    void setCPU() override { pc.cpu = "Intel i5"; }
    void setGPU() override { pc.gpu = "Integrated Graphics"; }
    void setRAM() override { pc.ram = "16GB DDR4"; }
    void setStorage() override { pc.storage = "512GB SSD"; }
    Computer getResult() override { return pc; }
};
class Director {
public:
    void construct(ComputerBuilder& builder) {
        builder.setCPU();
        builder.setGPU();
        builder.setRAM();
        builder.setStorage();
    }
};
int main() {
    Director director;

    GamingComputerBuilder gamingBuilder;
    OfficeComputerBuilder officeBuilder;

    std::cout << "== Gaming PC ==\n";
    director.construct(gamingBuilder);
    Computer gamingPC = gamingBuilder.getResult();
    gamingPC.specs();

    std::cout << "\n== Office PC ==\n";
    director.construct(officeBuilder);
    Computer officePC = officeBuilder.getResult();
    officePC.specs();
}
```


**Одиночка** — это порождающий паттерн, который гарантирует, что у класса есть только один экземпляр, и предоставляет глобальную точку доступа к нему.
Зачем он нужен?
- Когда тебе нужно, чтобы:
- был только один объект какого-то класса (например, конфигурация, логгер, подключение к БД);
- к нему можно было получить доступ из любой части программы.


```
/*
	Mutex (mutual exclusion) - класс, для примитивной синхронизации потоковой
	обработки данных или объектов.
*/

class Singleton {
private:
	static Singleton* instance;
	static std::mutex mutex;
	Singleton() { std::cout << "Singleton instance created." << std::endl; }
	~Singleton() { std::cout << "Singleton instance destroyed." << std::endl; }
public:
	// Удаляем конструктор копирования
	Singleton(const Singleton&) = delete;
	// Удаляем оператор присваивания
	Singleton& operator=(const Singleton&) = delete;

	// Метод для получения единственного экземпляра
	static Singleton* getInstance() {
		std::lock_guard<std::mutex> lock(mutex);
		if (instance == nullptr) {
			instance = new Singleton();
		}
		return instance;
	}

	void printSingleton() {
		std::cout << "Singleton ptr: " << instance << std::endl;
	}

};

// Инициализация статического члена
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex;

int main() {
	Singleton* singleton = Singleton::getInstance();
	singleton->printSingleton();

	Singleton* anotherSingleton = Singleton::getInstance();
	anotherSingleton->printSingleton();
}
```


**Прототип (Prototype)** — это порождающий паттерн, который используется тогда, когда нужно создавать копии объектов, а не создавать их с нуля.
Что такое паттерн Прототип?
Прототип — это паттерн, который позволяет:
- создавать новые объекты через клонирование уже существующих (прототипов),
- избегать повторного создания дорогих объектов «с нуля»,
- скрыть детали создания объекта от клиента.
Когда он нужен?
- Когда создание объекта — дорогостоящая операция (много полей, вложенные объекты);
- Когда нужно копировать объекты, не зная их точный тип во время компиляции;
- Когда используется абстрактная фабрика и нужно клонировать объекты из семейства.


```
class Shape {
public:
		virtual std::unique_ptr<Shape> clone() const = 0;
		virtual void draw() const = 0;
		virtual ~Shape() {}
	};
	
	class Circle : public Shape {
	public:
		Circle() { std::cout << "Circle created." << std::endl; }
		std::unique_ptr<Shape> clone() const override {
			return std::make_unique<Circle>(*this);
		}
		void draw() const override { std::cout << "Drawing a Circle." << std::endl; }
	};
	
	class Square : public Shape {
	public:
		Square() { std::cout << "Square created." << std::endl; }
		std::unique_ptr<Shape> clone() const override {
			return std::make_unique<Square>(*this);
		}
		void draw() const override { std::cout << "Drawing a Sqaure." << std::endl; }
	};
	
	int main(){
		std::unique_ptr<Shape> circle = std::make_unique<Circle>();
		std::unique_ptr<Shape> square = std::make_unique<Square>();
	
		circle->draw();
		square->draw();
	
		std::unique_ptr<Shape> circleClone = circle->clone();
		std::unique_ptr<Shape> squareClone = square->clone();
	
		circleClone->draw();
		squareClone->draw();
	}
```


## Структурные(structural)


**Адаптер (Adapter)** — это структурный паттерн, который позволяет объектам с несовместимыми интерфейсами работать вместе.
Он оборачивает один класс в другой, чтобы преобразовать интерфейс существующего класса в тот, который ожидает клиент.
Зачем нужен?
Когда:
- ты хочешь использовать существующий класс, но его интерфейс не совпадает с тем, который ожидает твой код;
- переопределить чужой код ты не можешь (например, сторонняя библиотека);
- нужно интегрировать старый и новый код, не переписывая всё.


```
class EuropeanSocket {
public:
    virtual void power220V() const {
        std::cout << "Powering with 220V (European socket)\n";
    }
    virtual ~EuropeanSocket() = default;
};
class USPlug {
public:
    virtual void power110V() const {
        std::cout << "Powering with 110V (US plug)\n";
    }
   
};
class USPlugAdapter : public EuropeanSocket {
private:
    USPlug& usPlug;

public:
    USPlugAdapter(USPlug& plug) : usPlug(plug) {}
    void power220V() const override {
        std::cout << "Converting 220V to 110V for US plug...\n";
        usPlug.power110V();
    }
};
int main() {
    USPlug americanDevice;
    USPlugAdapter adapter(americanDevice);

    // Клиент ожидает EuropeanSocket, но получает адаптированный USPlug
    adapter.power220V();
}
```


Паттерн **Мост (Bridge)** — это структурный паттерн проектирования, который разделяет абстракцию и реализацию, чтобы их можно было изменять независимо друг от друга.
Что делает паттерн Мост?
Он позволяет:
- Избежать жёсткой связи между абстрактным интерфейсом и его реализацией;
- Изменять абстракцию и реализацию независимо (например, без перекомпиляции);
- Упростить код при росте количества комбинаций подклассов (вместо множественного наследования).
Когда использовать?
- Когда есть разные виды объектов и множество реализаций поведения, и их комбинации могут взрывать количество подклассов;
- Когда ты хочешь переключать реализацию во время выполнения;
- Когда хочешь разделить интерфейс и реализацию по разным иерархиям.


```
class Device {
public:
    virtual void turnOn() = 0;
    virtual void turnOff() = 0;
    virtual void setVolume(int percent) = 0;
    virtual ~Device() = default;
};
class TV : public Device {
public:
    void turnOn() override { std::cout << "TV turned ON\n"; }
    void turnOff() override { std::cout << "TV turned OFF\n"; }
    void setVolume(int percent) override {
        std::cout << "TV volume set to " << percent << "%\n";
    }
};

class Radio : public Device {
public:
    void turnOn() override { std::cout << "Radio turned ON\n"; }
    void turnOff() override { std::cout << "Radio turned OFF\n"; }
    void setVolume(int percent) override {
        std::cout << "Radio volume set to " << percent << "%\n";
    }
};
class Remote {
protected:
    Device& device;

public:
    Remote(Device& d) : device(d) {}

    virtual void togglePower() {
        static bool isOn = false;
        if (isOn) {
            device.turnOff();
        }
        else {
            device.turnOn();
        }
        isOn = !isOn;
    }

    virtual void volumeUp() {
        device.setVolume(80);
    }

    virtual ~Remote() = default;
};
class AdvancedRemote : public Remote {
public:
    using Remote::Remote;  // использовать конструктор базового класса

    void mute() {
        std::cout << "Muting device...\n";
        device.setVolume(0);
    }
};
int main() {
    TV tv;
    Radio radio;

    AdvancedRemote tvRemote(tv);
    tvRemote.togglePower();
    tvRemote.volumeUp();
    tvRemote.mute();

    std::cout << "\n";

    Remote radioRemote(radio);
    radioRemote.togglePower();
    radioRemote.volumeUp();
}
```

Паттерн **Компоновщик (Composite)**  — это структурный паттерн проектирования, который позволяет одинаково обращаться к отдельным объектам и их группам (деревьям).
Что делает паттерн Компоновщик?
- Он позволяет строить иерархии объектов в виде дерева, где:
- Листья — это простые объекты (например, кнопка),
- Узлы (композитные объекты) — содержат другие объекты (например, окно, содержащее кнопки и текст).
Зачем нужен?
- Чтобы упростить работу с иерархией: клиент не должен отличать, работает он с отдельным объектом или с их группой.
- Чтобы унифицировать интерфейс для всех компонентов.
- Особенно полезен в GUI, файловых системах, иерархиях сцен и т.д.


```
class GUIComponent {
public:
    virtual void render() const = 0;
    virtual void add(std::shared_ptr<GUIComponent> component) {
        // по умолчанию: ничего не делает (лист)
    }
    virtual ~GUIComponent() = default;
};
class Button : public GUIComponent {
public:
    void render() const override {
        std::cout << "Rendering Button\n";
    }
};

class Text : public GUIComponent {
public:
    void render() const override {
        std::cout << "Rendering Text\n";
    }
};
class Panel : public GUIComponent {
private:
    std::vector<std::shared_ptr<GUIComponent>> children;
public:
    void add(std::shared_ptr<GUIComponent> component) override {
        children.push_back(component);
    }

    void render() const override {
        std::cout << "Rendering Panel:\n";
        for (const auto& child : children) {
            child->render();
        }
    }
};
int main() {
    auto root = std::make_shared<Panel>();
    auto btn = std::make_shared<Button>();
    auto txt = std::make_shared<Text>();

    root->add(btn);
    root->add(txt);

    auto subPanel = std::make_shared<Panel>();
    subPanel->add(std::make_shared<Button>());

    root->add(subPanel);

    root->render();
}
```

Паттерн **Декоратор (Decorator)** — это структурный паттерн, который позволяет динамически добавлять поведение объекту, не изменяя его класс.
Что делает паттерн Декоратор?
Он:
- Оборачивает объект в другой объект с таким же интерфейсом,
- Добавляет новое поведение до/после вызова оригинального метода,
- Позволяет комбинировать разные "дополнения" гибко и независимо.
Когда использовать?
- Когда нельзя или неудобно использовать наследование;
- Когда нужно гибко добавлять поведение к объектам во время выполнения;
- Когда хочется разделить обязанности без создания множества подклассов.


```
class Text {
public:
	virtual std::string render() const = 0;
	virtual ~Text() = default;
};
class PlainText : public Text {
    std::string content;
public:
    PlainText(const std::string& text) : content(text) {}

    std::string render() const override {
        return content;
    }
};
class TextDecorator : public Text {
protected:
    std::shared_ptr<Text> wrappee;
public:
    TextDecorator(std::shared_ptr<Text> component) : wrappee(component) {}
};
class BoldDecorator : public TextDecorator {
public:
    BoldDecorator(std::shared_ptr<Text> text) : TextDecorator(text) {}

    std::string render() const override {
        return "<b>" + wrappee->render() + "</b>";
    }
};

class ItalicDecorator : public TextDecorator {
public:
    ItalicDecorator(std::shared_ptr<Text> text) : TextDecorator(text) {}

    std::string render() const override {
        return "<i>" + wrappee->render() + "</i>";
    }
};
int main() {
    std::shared_ptr<Text> simple = std::make_shared<PlainText>("Hello, World");

    std::shared_ptr<Text> bold = std::make_shared<BoldDecorator>(simple);
    std::shared_ptr<Text> italic = std::make_shared<ItalicDecorator>(bold);

    std::cout << italic->render() << std::endl;  // <i><b>Hello, World</b></i>
}
```

Паттерн **Фасад (Facade)** — это структурный паттерн, который предоставляет упрощённый интерфейс к сложной системе классов, библиотек или подсистем.


Что делает фасад?
Он:
- Скрывает сложность системы за простым интерфейсом;
- Делает код удобнее и понятнее для пользователя;
- Помогает изолировать зависимости между клиентом и внутренними компонентами.
Когда использовать?
- Когда система слишком сложная или запутанная, и ты хочешь скрыть детали реализации;
- Когда есть много взаимосвязанных классов, но клиенту нужно только простое взаимодействие;
- Когда нужно облегчить миграцию, тестирование или разделение ответственности.


```
class Projector {
public:
    void on() { std::cout << "Projector ON\n"; }
    void setInput(const std::string& source) {
        std::cout << "Projector input set to " << source << "\n";
    }
};

class Screen {
public:
    void down() { std::cout << "Screen going down\n"; }
};

class SoundSystem {
public:
    void on() { std::cout << "Sound system ON\n"; }
    void setVolume(int level) {
        std::cout << "Volume set to " << level << "\n";
    }
};

class MoviePlayer {
public:
    void play(const std::string& movie) {
        std::cout << "Playing movie: " << movie << "\n";
    }
};
class HomeTheaterFacade {
private:
    Projector& projector;
    Screen& screen;
    SoundSystem& sound;
    MoviePlayer& player;

public:
    HomeTheaterFacade(Projector& p, Screen& s, SoundSystem& snd, MoviePlayer& mp)
        : projector(p), screen(s), sound(snd), player(mp) {}

    void watchMovie(const std::string& movie) {
        std::cout << "\n--- Preparing to watch a movie ---\n";
        screen.down();
        projector.on();
        projector.setInput("MoviePlayer");
        sound.on();
        sound.setVolume(50);
        player.play(movie);
    }
};
int main() {
    Projector projector;
    Screen screen;
    SoundSystem sound;
    MoviePlayer player;

    HomeTheaterFacade theater(projector, screen, sound, player);
    theater.watchMovie("Inception");
}
```

Паттерн **Приспособленец (Flyweight)** — это структурный паттерн, который позволяет экономить память, разделяя общее (повторяющееся) состояние между множеством объектов.

Зачем нужен паттерн Приспособленец?
Он нужен, когда:
- В программе много объектов, и они занимают слишком много памяти;
- Многие из этих объектов повторяются по содержанию (например, символы, части дерева, плитки, враги и т.п.);
- Повторяющееся состояние можно вынести в общее хранилище (внешнее состояние);
- Остаётся только уникальное состояние (внутреннее), которое не занимает много памяти.


```
class Glyph {
public:
	virtual void draw(int x, int y) const = 0;
	virtual ~Glyph() = default;
};
class Character : public Glyph {
private:
    char symbol; // общий для всех "A"
public:
    Character(char c) : symbol(c) {}

    void draw(int x, int y) const override {
        std::cout << "Drawing '" << symbol << "' at (" << x << ", " << y << ")\n";
    }
};
class GlyphFactory {
private:
    std::unordered_map<char, std::shared_ptr<Glyph>> cache;

public:
    std::shared_ptr<Glyph> get(char c) {
        if (cache.find(c) == cache.end()) {
            cache[c] = std::make_shared<Character>(c);
        }
        return cache[c];
    }
};
int main() {
    GlyphFactory factory;

    std::string text = "HELLO";
    int x = 0;

    for (char c : text) {
        auto glyph = factory.get(c); // reuse if already exists
        glyph->draw(x, 0);           // x — внешний (уникальный) параметр
        x += 10;
    }
}
```


Паттерн **Прокси (Proxy)** — это структурный паттерн, который предоставляет заместителя (или представителя) для другого объекта, чтобы контролировать доступ к этому объекту. Прокси может использоваться для отложенной загрузки, защищенного доступа, кэширования, или мониторинга.
Зачем нужен паттерн Прокси?
Паттерн прокси нужен, когда:
- Нужно контролировать доступ к объекту (например, ограничения доступа или логирование).
- Требуется ленивая загрузка (например, объект слишком тяжёлый и создаётся только по мере необходимости).
- Требуется кэширование результатов работы объекта.
- Нужно обеспечить удалённый доступ (например, к удалённым ресурсам).
- Безопасность: прокси может проверять права доступа к объекту.
Когда использовать?
- Когда объект слишком тяжёлый или дорогой в создании (например, загрузка большого изображения или доступ к базе данных).
- Когда нужно ограничить доступ к какому-либо ресурсу (например, только авторизованные пользователи могут выполнять определённые операции).
- Когда объект должен быть заменён или защищён другими объектами с дополнительным поведением (например, кэширование, логирование).


```
class Image {
public:
    virtual void display() const = 0;
    virtual ~Image() = default;
};

class RealImage : public Image {
private:
    std::string filename;

public:
    RealImage(const std::string& file) : filename(file) {
        loadFromDisk();  // Загружаем изображение
    }

    void loadFromDisk() const {
        std::cout << "Loading image: " << filename << std::endl;
    }

    void display() const override {
        std::cout << "Displaying image: " << filename << std::endl;
    }
};
class ProxyImage : public Image {
private:
    mutable std::shared_ptr<RealImage> realImage;  // mutable для возможности изменения в const методах
    std::string filename;

public:
    ProxyImage(const std::string& file) : filename(file), realImage(nullptr) {}

    void display() const override {
        // Если изображение ещё не загружено, загружаем его
        if (!realImage) {
            realImage = std::make_shared<RealImage>(filename);  // Создание реального изображения
        }
        realImage->display();  // Отображаем реальное изображение
    }
};
int main() {
    ProxyImage image("photo.jpg");  // Создаём объект прокси

    // Изображение загружается и отображается только при первом вызове display()
    image.display();

    // Второй раз изображение просто отображается, без загрузки
    image.display();

    return 0;
}
```

## Поведенческие (behavioral)
Паттерн **Цепочка обязанностей (Chain of Responsibility)** — это поведенческий шаблон проектирования, который позволяет избежать жёсткой привязки отправителя запроса к его получателю, позволяя обработать запрос нескольким объектам-обработчикам по цепочке.
Зачем он нужен?
- Чтобы разделить обязанности между разными объектами.
- Чтобы дать возможность нескольким объектам обработать запрос, не зная, кто именно это сделает.
- Для динамической настройки цепочек обработки.


```
class Handler {
protected:
    std::shared_ptr<Handler> next;

public:
    void setNext(std::shared_ptr<Handler> nextHandler) {
        next = nextHandler;
    }

    virtual void handle(const std::string& request) {
        if (next) {
            next->handle(request);
        } else {
            std::cout << "No handler could process the request: " << request << "\n";
        }
    }

    virtual ~Handler() = default;
};

class AuthHandler : public Handler {
public:
    void handle(const std::string& request) override {
        if (request == "auth") {
            std::cout << "AuthHandler: обработал запрос.\n";
        } else {
            std::cout << "AuthHandler: передал запрос дальше...\n";
            Handler::handle(request);
        }
    }
};

class LogHandler : public Handler {
public:
    void handle(const std::string& request) override {
        if (request == "log") {
            std::cout << "LogHandler: обработал запрос.\n";
        } else {
            std::cout << "LogHandler: передал запрос дальше...\n";
            Handler::handle(request);
        }
    }
};

class ErrorHandler : public Handler {
public:
    void handle(const std::string& request) override {
        std::cout << "ErrorHandler: никто не обработал '" << request << "', логирую как ошибку.\n";
    }
};

int main() {
    auto auth = std::make_shared<AuthHandler>();
    auto log = std::make_shared<LogHandler>();
    auto error = std::make_shared<ErrorHandler>();

    // Настраиваем цепочку
    auth->setNext(log);
    log->setNext(error);

    // Клиент отправляет запрос
    auth->handle("auth");
    auth->handle("log");
    auth->handle("unknown");

    return 0;
}
```

**Команда(command pattern)** — это объект, который инкапсулирует действие (запрос), параметры и получателя. Он позволяет:
отделить объект, инициирующий действие, от объекта, который выполняет это действие;
ставить команды в очередь, отменять, повторять и логировать операции.
Зачем нужен?
- Чтобы реализовать отмену/повтор операций.
- Чтобы логировать или отложить выполнение действий.
- Чтобы избежать прямого вызова методов у объектов (инверсия управления).
- Чтобы реализовать меню/кнопки в GUI, где каждая кнопка — это команда.


```
// Получатель (Receiver)
class Light {
public:
    void on() {
        std::cout << "Свет включен\n";
    }

    void off() {
        std::cout << "Свет выключен\n";
    }
};

// Интерфейс команды
class Command {
public:
    virtual void execute() = 0;
    virtual ~Command() = default;
};

// Конкретные команды
class LightOnCommand : public Command {
    std::shared_ptr<Light> light;

public:
    LightOnCommand(std::shared_ptr<Light> l) : light(l) {}

    void execute() override {
        light->on();
    }
};

class LightOffCommand : public Command {
    std::shared_ptr<Light> light;

public:
    LightOffCommand(std::shared_ptr<Light> l) : light(l) {}

    void execute() override {
        light->off();
    }
};

// Инициатор (Invoker)
class RemoteControl {
    std::vector<std::shared_ptr<Command>> commands;

public:
    void addCommand(std::shared_ptr<Command> command) {
        commands.push_back(command);
    }

    void run() {
        for (const auto& cmd : commands) {
            cmd->execute();
        }
    }
};

int main() {
    auto light = std::make_shared<Light>();

    auto lightOn = std::make_shared<LightOnCommand>(light);
    auto lightOff = std::make_shared<LightOffCommand>(light);

    RemoteControl remote;
    remote.addCommand(lightOn);
    remote.addCommand(lightOff);

    remote.run();  // Выполнение всех команд

    return 0;
}
```

**Интерпретатор (Interpreter)** — это поведенческий шаблон проектирования, который определяет представление грамматики языка и интерпретатор для предложений на этом языке.
Он используется, когда у вас есть язык с простой грамматикой, и вы хотите написать интерпретатор для его выполнения.
Зачем нужен?
- Для разбора и исполнения "мини-языков" (например, выражений, фильтров, запросов).
- Чтобы интерпретировать скрипты, математические выражения, формулы, фильтры и т. д.
- Упрощённый аналог парсера/интерпретатора языков.

  
```
// Контекст (хранит значения переменных)
class Context {
public:
    std::map<std::string, int> variables;

    int getValue(const std::string& name) const {
        auto it = variables.find(name);
        if (it != variables.end()) return it->second;
        throw std::runtime_error("Unknown variable: " + name);
    }
};

// Интерфейс выражения
class Expression {
public:
    virtual int interpret(const Context& context) const = 0;
    virtual ~Expression() = default;
};

// Терминальное выражение — число
class Number : public Expression {
    int value;

public:
    Number(int val) : value(val) {}
    int interpret(const Context&) const override {
        return value;
    }
};

// Терминальное выражение — переменная
class Variable : public Expression {
    std::string name;

public:
    Variable(const std::string& varName) : name(varName) {}
    int interpret(const Context& context) const override {
        return context.getValue(name);
    }
};

// Нетерминальное выражение — сумма
class Add : public Expression {
    std::shared_ptr<Expression> left;
    std::shared_ptr<Expression> right;

public:
    Add(std::shared_ptr<Expression> l, std::shared_ptr<Expression> r)
        : left(l), right(r) {}

    int interpret(const Context& context) const override {
        return left->interpret(context) + right->interpret(context);
    }
};

// Нетерминальное выражение — разность
class Subtract : public Expression {
    std::shared_ptr<Expression> left;
    std::shared_ptr<Expression> right;

public:
    Subtract(std::shared_ptr<Expression> l, std::shared_ptr<Expression> r)
        : left(l), right(r) {}

    int interpret(const Context& context) const override {
        return left->interpret(context) - right->interpret(context);
    }
};

// --- Пример использования ---
int main() {
    Context context;
    context.variables["x"] = 5;
    context.variables["y"] = 10;

    // выражение: x + y
    std::shared_ptr<Expression> expr = std::make_shared<Add>(
        std::make_shared<Variable>("x"),
        std::make_shared<Variable>("y")
    );

    std::cout << "Результат x + y = " << expr->interpret(context) << "\n"; // 15

    // выражение: x + y - 3
    std::shared_ptr<Expression> complexExpr = std::make_shared<Subtract>(
        expr,
        std::make_shared<Number>(3)
    );

    std::cout << "Результат x + y - 3 = " << complexExpr->interpret(context) << "\n"; // 12

    return 0;
}
```

**Iterator** — это поведенческий паттерн, который предоставляет способ последовательно обходить элементы коллекции, не раскрывая внутреннего представления этой коллекции (структуру хранения, указатели, и т.п.).
Зачем нужен?
- Чтобы одинаково обходить разные типы коллекций.
- Чтобы инкапсулировать логику обхода внутри объекта итератора.
- Чтобы предоставить несколько параллельных итераций по одной коллекции.
- Чтобы не зависеть от реализации самой коллекции (например, список, дерево, граф и т. д.).


```
// Интерфейс итератора
class Iterator {
public:
    virtual bool hasNext() const = 0;
    virtual std::string next() = 0;
    virtual ~Iterator() = default;
};

// Коллекция — список строк
class NameCollection {
    std::vector<std::string> names;

public:
    void add(const std::string& name) {
        names.push_back(name);
    }

    // Вложенный итератор
    class NameIterator : public Iterator {
        const std::vector<std::string>& names;
        size_t index = 0;

    public:
        NameIterator(const std::vector<std::string>& names) : names(names) {}

        bool hasNext() const override {
            return index < names.size();
        }

        std::string next() override {
            return names[index++];
        }
    };

    std::shared_ptr<Iterator> createIterator() const {
        return std::make_shared<NameIterator>(names);
    }
};

int main() {
    NameCollection collection;
    collection.add("Alice");
    collection.add("Bob");
    collection.add("Charlie");

    auto it = collection.createIterator();

    while (it->hasNext()) {
        std::cout << it->next() << "\n";
    }

    return 0;
}
```

**Посредник** — это поведенческий шаблон проектирования, который инкапсулирует взаимодействие между объектами, чтобы они не ссылались напрямую друг на друга, а взаимодействовали через центральный объект — посредника.
Зачем нужен?
- Чтобы уменьшить связность между компонентами системы.
- Чтобы упростить логику взаимодействия (например, между UI элементами).
- Чтобы централизовать контроль за сообщениями.


```
// Вперёд объявление
class ChatMediator;

// Интерфейс участника (Colleague)
class User : public std::enable_shared_from_this<User> {
protected:
    std::string name;
    std::shared_ptr<ChatMediator> mediator;

public:
    User(const std::string& name, std::shared_ptr<ChatMediator> mediator)
        : name(name), mediator(mediator) {}

    virtual void send(const std::string& message) = 0;
    virtual void receive(const std::string& message) = 0;

    std::string getName() const { return name; }
    virtual ~User() = default;

    // Позволяет другим классам использовать shared_from_this()
    std::shared_ptr<User> getSharedPtr() {
        return shared_from_this();
    }
};

// Интерфейс посредника
class ChatMediator {
public:
    virtual void sendMessage(const std::string& message, std::shared_ptr<User> sender) = 0;
    virtual void addUser(std::shared_ptr<User> user) = 0;
    virtual ~ChatMediator() = default;
};

// Конкретный посредник (чат)
class ConcreteChatMediator : public ChatMediator {
    std::vector<std::shared_ptr<User>> users;

public:
    void addUser(std::shared_ptr<User> user) override {
        users.push_back(user);
    }

    void sendMessage(const std::string& message, std::shared_ptr<User> sender) override {
        for (const auto& user : users) {
            if (user != sender) {
                user->receive(sender->getName() + ": " + message);
            }
        }
    }
};

// Конкретный участник
class ChatUser : public User {
public:
    ChatUser(const std::string& name, std::shared_ptr<ChatMediator> mediator)
        : User(name, mediator) {}

    void send(const std::string& message) override {
        std::cout << name << " отправляет сообщение: " << message << "\n";
        mediator->sendMessage(message, getSharedPtr());
    }

    void receive(const std::string& message) override {
        std::cout << name << " получил сообщение: " << message << "\n";
    }
};

int main() {
    auto chat = std::make_shared<ConcreteChatMediator>();

    auto alice = std::make_shared<ChatUser>("Alice", chat);
    auto bob = std::make_shared<ChatUser>("Bob", chat);
    auto charlie = std::make_shared<ChatUser>("Charlie", chat);

    chat->addUser(alice);
    chat->addUser(bob);
    chat->addUser(charlie);

    alice->send("Привет всем!");
    bob->send("Привет, Alice!");
    return 0;
}
```

**Хранитель (Memento)** — это поведенческий паттерн, который позволяет сохранить и восстановить состояние объекта без нарушения инкапсуляции, то есть без того, чтобы внешние объекты могли изменять это состояние.
Основная идея заключается в том, что:
- Хранитель (Memento) сохраняет состояние объекта.
- Originator — объект, состояние которого нужно сохранить, и который может восстанавливать своё состояние из Хранителя.
- Caretaker — объект, который управляет сохранёнными состояниями и может их запрашивать.
Зачем нужен?
- Для сохранения состояния объекта, чтобы в будущем можно было вернуться к этому состоянию.
- Для реализации отмены операций (например, в приложениях с undo/redo).
- Для реализации сохранений игры, отслеживания изменений и других сценариев.


```
// Мemento — Хранитель состояния
class Memento {
    std::string state;

public:
    Memento(const std::string& state) : state(state) {}

    std::string getState() const {
        return state;
    }
};

// Originator — объект, состояние которого сохраняем
class Editor {
    std::string text;

public:
    void setText(const std::string& newText) {
        text = newText;
    }

    std::string getText() const {
        return text;
    }

    // Создаёт Memento для сохранения состояния
    std::shared_ptr<Memento> save() const {
        return std::make_shared<Memento>(text);
    }

    // Восстанавливает состояние из Memento
    void restore(const std::shared_ptr<Memento>& memento) {
        text = memento->getState();
    }
};

// Caretaker — хранит Memento
class History {
    std::vector<std::shared_ptr<Memento>> savedStates;

public:
    void saveState(const std::shared_ptr<Memento>& memento) {
        savedStates.push_back(memento);
    }

    std::shared_ptr<Memento> getLastState() {
        if (savedStates.empty()) return nullptr;
        auto last = savedStates.back();
        savedStates.pop_back();
        return last;
    }
};

int main() {
    Editor editor;
    History history;

    editor.setText("Hello, world!");
    std::cout << "Текущий текст: " << editor.getText() << "\n";

    // Сохраняем состояние
    history.saveState(editor.save());

    editor.setText("Hello, C++!");
    std::cout << "Текущий текст: " << editor.getText() << "\n";

    // Восстанавливаем состояние
    editor.restore(history.getLastState());
    std::cout << "После восстановления: " << editor.getText() << "\n";

    return 0;
}
```
**Наблюдатель (Observer)** — это поведенческий паттерн, который позволяет объекту (известному как Subject) уведомлять другие объекты (известные как Observers) о каких-либо изменениях в его состоянии, не прибегая к жёсткому связыванию между объектами.
Зачем нужен?
- Чтобы отслеживать изменения состояния объекта и автоматически оповещать другие объекты.
- Для реализации реактивных интерфейсов (например, пользовательский интерфейс, который автоматически обновляется при изменении данных).
- Это очень полезно в публикациях/подписках, например, в системах событий.


```
// Интерфейс наблюдателя
class Observer {
public:
    virtual void update(const std::string& message) = 0;
    virtual ~Observer() = default;
};

// Интерфейс субъекта (Subject)
class Subject {
public:
    virtual void addObserver(std::shared_ptr<Observer> observer) = 0;
    virtual void removeObserver(std::shared_ptr<Observer> observer) = 0;
    virtual void notifyObservers() = 0;
    virtual ~Subject() = default;
};

// Конкретный субъект
class NewsAgency : public Subject {
    std::vector<std::shared_ptr<Observer>> observers;
    std::string news;

public:
    void setNews(const std::string& newNews) {
        news = newNews;
        notifyObservers();
    }

    void addObserver(std::shared_ptr<Observer> observer) override {
        observers.push_back(observer);
    }

    void removeObserver(std::shared_ptr<Observer> observer) override {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    void notifyObservers() override {
        for (const auto& observer : observers) {
            observer->update(news);
        }
    }
};

// Конкретный наблюдатель
class Subscriber : public Observer {
    std::string name;

public:
    Subscriber(const std::string& name) : name(name) {}

    void update(const std::string& message) override {
        std::cout << name << " получил новость: " << message << "\n";
    }
};

int main() {
    auto newsAgency = std::make_shared<NewsAgency>();

    auto subscriber1 = std::make_shared<Subscriber>("Subscriber 1");
    auto subscriber2 = std::make_shared<Subscriber>("Subscriber 2");

    newsAgency->addObserver(subscriber1);
    newsAgency->addObserver(subscriber2);

    newsAgency->setNews("Вышел новый паттерн Observer!");

    newsAgency->removeObserver(subscriber1);

    newsAgency->setNews("Новый паттерн — стратегия!");

    return 0;
}
```
**Состояние (State)** — это поведенческий паттерн, который позволяет объекту изменять своё поведение в зависимости от его состояния. Объект будет выглядеть как если бы он изменил свой класс.
Идея паттерна:
Вместо того, чтобы писать длинные условные конструкции (например, if или switch), объект делегирует работу по обработке состояния отдельным классам.
Каждый класс представляет одно состояние, а поведение объекта изменяется в зависимости от того, какое состояние активно.
Зачем нужен?
- Чтобы удобно управлять состоянием объекта, особенно если у объекта есть много состояний, каждое из которых должно вести себя по-разному.
- Уменьшает количество условных конструкций в коде и делает его более читаемым и поддерживаемым.
- Легче добавлять новые состояния без изменения существующего кода.


```
// Абстрактный класс состояния
class State {
public:
    virtual void insertCoin() = 0;
    virtual void ejectCoin() = 0;
    virtual void pressButton() = 0;
    virtual void dispense() = 0;
    virtual ~State() = default;
};

// Конкретные состояния автомата
class WaitingForCoinState;
class WaitingForSelectionState;
class DispensingState;

// Контекст (автомат)
class VendingMachine {
    std::shared_ptr<State> currentState;

public:
    VendingMachine() : currentState(nullptr) {}

    void setState(std::shared_ptr<State> state) {
        currentState = state;
    }

    void insertCoin() {
        if (currentState)
            currentState->insertCoin();
    }

    void ejectCoin() {
        if (currentState)
            currentState->ejectCoin();
    }

    void pressButton() {
        if (currentState)
            currentState->pressButton();
    }

    void dispense() {
        if (currentState)
            currentState->dispense();
    }
};

// Конкретное состояние: Ожидание монеты
class WaitingForCoinState : public State {
    std::shared_ptr<VendingMachine> machine;

public:
    WaitingForCoinState(std::shared_ptr<VendingMachine> machine) : machine(machine) {}

    void insertCoin() override {
        std::cout << "Монета вставлена.\n";
        machine->setState(std::static_pointer_cast<State>(std::make_shared<WaitingForSelectionState>(machine)));  // Меняем состояние на "Ожидание выбора напитка"
    }

    void ejectCoin() override {
        std::cout << "Монета не вставлена.\n";
    }

    void pressButton() override {
        std::cout << "Сначала вставьте монету.\n";
    }

    void dispense() override {
        std::cout << "Нужно сначала вставить монету.\n";
    }
};

// Конкретное состояние: Ожидание выбора напитка
class WaitingForSelectionState : public State {
    std::shared_ptr<VendingMachine> machine;

public:
    WaitingForSelectionState(std::shared_ptr<VendingMachine> machine) : machine(machine) {}

    void insertCoin() override {
        std::cout << "Монета уже вставлена. Выберите напиток.\n";
    }

    void ejectCoin() override {
        std::cout << "Монета возвращена.\n";
        machine->setState(std::static_pointer_cast<State>(std::make_shared<WaitingForCoinState>(machine)));  // Переходим обратно в состояние "Ожидание монеты"
    }

    void pressButton() override {
        std::cout << "Вы выбрали напиток.\n";
        machine->setState(std::static_pointer_cast<State>(std::make_shared<DispensingState>(machine)));  // Меняем состояние на "Выплата напитка"
    }

    void dispense() override {
        std::cout << "Нужно сначала выбрать напиток.\n";
    }
};

// Конкретное состояние: Выплата напитка
class DispensingState : public State {
    std::shared_ptr<VendingMachine> machine;

public:
    DispensingState(std::shared_ptr<VendingMachine> machine) : machine(machine) {}

    void insertCoin() override {
        std::cout << "Автомат уже выплачивает напиток.\n";
    }

    void ejectCoin() override {
        std::cout << "Невозможно вернуть монету, напиток уже выплачивается.\n";
    }

    void pressButton() override {
        std::cout << "Невозможно нажать кнопку, напиток уже выплачивается.\n";
    }

    void dispense() override {
        std::cout << "Напиток выплачивается...\n";
        machine->setState(std::static_pointer_cast<State>(std::make_shared<WaitingForCoinState>(machine)));  // Переходим обратно в состояние "Ожидание монеты"
    }
};

int main() {
    // Создаём автомат с пустым состоянием
    auto machine = std::make_shared<VendingMachine>();

    // Устанавливаем начальное состояние
    machine->setState(std::static_pointer_cast<State>(std::make_shared<WaitingForCoinState>(machine)));

    machine->insertCoin();  // Вставляем монету
    machine->pressButton(); // Выбираем напиток
    machine->dispense();    // Выплачиваем напиток

    return 0;
}
```

Паттерн **Стратегия (Strategy)** позволяет изменить поведение объекта во время выполнения, выбирая алгоритм из набора доступных. Это позволяет клиенту выбирать стратегию, которая будет использоваться в зависимости от контекста, не изменяя код самого объекта.
Основная идея заключается в том, чтобы инкапсулировать различные алгоритмы в отдельные классы, а затем использовать один из них в зависимости от условий. Таким образом, паттерн помогает избежать условных операторов для выбора алгоритма и улучшает расширяемость кода.

```
// Интерфейс стратегии
class ShippingStrategy {
public:
    virtual double calculateCost(double weight) = 0;
    virtual ~ShippingStrategy() = default;
};

// Конкретная стратегия для авиа доставки
class AirShipping : public ShippingStrategy {
public:
    double calculateCost(double weight) override {
        std::cout << "Calculating air shipping cost...\n";
        return weight * 5.0;  // Примерная стоимость для авиа
    }
};

// Конкретная стратегия для железнодорожной доставки
class TrainShipping : public ShippingStrategy {
public:
    double calculateCost(double weight) override {
        std::cout << "Calculating train shipping cost...\n";
        return weight * 2.0;  // Примерная стоимость для поезда
    }
};

// Конкретная стратегия для морской доставки
class SeaShipping : public ShippingStrategy {
public:
    double calculateCost(double weight) override {
        std::cout << "Calculating sea shipping cost...\n";
        return weight * 1.0;  // Примерная стоимость для моря
    }
};

// Контекст
class ShippingContext {
private:
    std::shared_ptr<ShippingStrategy> strategy;  // Стратегия, используемая в контексте

public:
    void setStrategy(std::shared_ptr<ShippingStrategy> newStrategy) {
        strategy = newStrategy;  // Устанавливаем новую стратегию
    }

    double calculateShippingCost(double weight) {
        if (strategy) {
            return strategy->calculateCost(weight);  // Используем выбранную стратегию
        }
        return 0.0;
    }
};

int main() {
    // Контекст доставки
    ShippingContext context;

    // Устанавливаем стратегию авиа доставки
    context.setStrategy(std::make_shared<AirShipping>());
    double airCost = context.calculateShippingCost(10.0);  // Вес 10 кг
    std::cout << "Air shipping cost: " << airCost << " USD\n";

    // Устанавливаем стратегию железнодорожной доставки
    context.setStrategy(std::make_shared<TrainShipping>());
    double trainCost = context.calculateShippingCost(10.0);
    std::cout << "Train shipping cost: " << trainCost << " USD\n";

    // Устанавливаем стратегию морской доставки
    context.setStrategy(std::make_shared<SeaShipping>());
    double seaCost = context.calculateShippingCost(10.0);
    std::cout << "Sea shipping cost: " << seaCost << " USD\n";

    return 0;
}
```

Паттерн **Шаблонный метод (Template Method)** — это поведенческий паттерн, который определяет основу алгоритма, оставляя детали некоторых шагов на усмотрение подклассов. Он позволяет переопределить только отдельные шаги алгоритма, не изменяя его структуру.
Основная идея паттерна заключается в том, что вы определяете общий алгоритм в базовом классе (шаблонный метод), а конкретные шаги этого алгоритма могут быть реализованы в подклассах. Это позволяет избежать дублирования кода, сохраняя гибкость.


```
// Абстрактный класс
class CoffeeMaker {
public:
    // Шаблонный метод
    void makeCoffee() {
        boilWater();          // Общий шаг для всех видов кофе
        brewCoffee();         // Общий шаг для всех видов кофе
        pourInCup();          // Общий шаг для всех видов кофе
        addCondiments();      // Шаг, который может изменяться в подклассах
    }

    virtual ~CoffeeMaker() = default;

protected:
    virtual void brewCoffee() = 0;     // Абстрактный метод, который будут реализовывать подклассы
    virtual void addCondiments() = 0;  // Абстрактный метод, который будут реализовывать подклассы

    // Общие методы, которые не изменяются в подклассах
    void boilWater() {
        std::cout << "Boiling water...\n";
    }

    void pourInCup() {
        std::cout << "Pouring coffee into cup...\n";
    }
};

// Конкретный класс для черного кофе
class BlackCoffeeMaker : public CoffeeMaker {
protected:
    void brewCoffee() override {
        std::cout << "Brewing black coffee...\n";
    }

    void addCondiments() override {
        std::cout << "No condiments added.\n";
    }
};

// Конкретный класс для кофе с молоком
class MilkCoffeeMaker : public CoffeeMaker {
protected:
    void brewCoffee() override {
        std::cout << "Brewing coffee with milk...\n";
    }

    void addCondiments() override {
        std::cout << "Adding milk...\n";
    }
};

int main() {
    std::cout << "Making black coffee:\n";
    BlackCoffeeMaker blackCoffee;
    blackCoffee.makeCoffee();  // Используем общий алгоритм

    std::cout << "\nMaking milk coffee:\n";
    MilkCoffeeMaker milkCoffee;
    milkCoffee.makeCoffee();  // Используем общий алгоритм

    return 0;
}
```
Паттерн **Посетитель (Visitor)** — это поведенческий паттерн, который позволяет добавить новые операции в объекты, не изменяя сами эти объекты. Он используется, когда нужно выполнить операции над объектами различных классов, входящих в одну структуру, и вы хотите отделить эти операции от самих классов.
Основная идея заключается в том, чтобы создать объект "посетителя", который может выполнять различные операции над элементами структуры (например, элементами иерархии классов), не изменяя эти элементы.


```
// Интерфейс посетителя
class Visitor {
public:
    virtual void visit(class TextFile& textFile) = 0;
    virtual void visit(class ImageFile& imageFile) = 0;
    virtual void visit(class SpreadsheetFile& spreadsheetFile) = 0;
    virtual ~Visitor() = default;
};

// Базовый элемент (объект, который будет принимать посетителя)
class File {
public:
    virtual void accept(Visitor& visitor) = 0;
    virtual ~File() = default;
};

// Конкретные элементы

class TextFile : public File {
    std::string content;

public:
    TextFile(const std::string& content) : content(content) {}

    void accept(Visitor& visitor) override {
        visitor.visit(*this);
    }

    const std::string& getContent() const { return content; }
};

class ImageFile : public File {
    int width, height;

public:
    ImageFile(int width, int height) : width(width), height(height) {}

    void accept(Visitor& visitor) override {
        visitor.visit(*this);
    }

    int getWidth() const { return width; }
    int getHeight() const { return height; }
};

class SpreadsheetFile : public File {
    int rows, columns;

public:
    SpreadsheetFile(int rows, int columns) : rows(rows), columns(columns) {}

    void accept(Visitor& visitor) override {
        visitor.visit(*this);
    }

    int getRows() const { return rows; }
    int getColumns() const { return columns; }
};

// Конкретный посетитель

class WordCountVisitor : public Visitor {
public:
    void visit(TextFile& textFile) override {
        int wordCount = 0;
        // Простое разделение текста на слова
        std::string text = textFile.getContent();
        size_t pos = 0;
        while ((pos = text.find(" ", pos)) != std::string::npos) {
            wordCount++;
            pos++;
        }
        wordCount++;  // Последнее слово

        std::cout << "Text file contains " << wordCount << " words." << std::endl;
    }

    void visit(ImageFile& imageFile) override {
        std::cout << "Image file has dimensions " << imageFile.getWidth() << "x" << imageFile.getHeight() << "." << std::endl;
    }

    void visit(SpreadsheetFile& spreadsheetFile) override {
        std::cout << "Spreadsheet has " << spreadsheetFile.getRows() << " rows and " << spreadsheetFile.getColumns() << " columns." << std::endl;
    }
};

int main() {
    // Создаем элементы
    std::vector<std::shared_ptr<File>> files;
    files.push_back(std::make_shared<TextFile>("This is a simple text file with several words."));
    files.push_back(std::make_shared<ImageFile>(1024, 768));
    files.push_back(std::make_shared<SpreadsheetFile>(20, 10));

    // Создаем посетителя
    WordCountVisitor wordCountVisitor;

    // Применяем посетителя ко всем элементам
    for (auto& file : files) {
        file->accept(wordCountVisitor);  // Каждый элемент вызывает метод accept() и передает посетителя
    }

    return 0;
}
```
