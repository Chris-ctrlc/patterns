# patterns
## Порождающие(creational)


**Фабричный метод** — это порождающий паттерн проектирования, который делегирует создание объектов подклассам. То есть он предоставляет интерфейс для создания объектов, но позволяет подклассам решать, какой класс инстанцировать.

🧠 Зачем он нужен?
✅ Разделение ответственности — логика создания объекта отделяется от его использования.
✅ Расширяемость — легко добавлять новые типы продуктов, не меняя существующий код.
✅ Инкапсуляция — клиентский код не знает, какие именно классы используются, и работает через абстракции.


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

🎯 Зачем он нужен?
Когда у тебя:
есть группы объектов, которые должны сотрудничать друг с другом (например, кнопка и окно из одного GUI),
и ты хочешь переключаться между ними динамически (например, между Windows и macOS),
и при этом ты не хочешь писать кучу if и switch, и жёстко привязываться к конкретным классам.

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

🎯 Зачем нужен?
Когда конструктор с 10 параметрами становится неудобным;
Когда нужен гибкий способ настройки объекта перед его созданием;
Когда у объекта есть разные "варианты сборки", но он должен быть одним и тем же классом в итоге.
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
🎯 Зачем он нужен?
Когда тебе нужно, чтобы:
был только один объект какого-то класса (например, конфигурация, логгер, подключение к БД);
к нему можно было получить доступ из любой части программы.


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

✅ Что такое паттерн Прототип?
Прототип — это паттерн, который позволяет:
создавать новые объекты через клонирование уже существующих (прототипов),
избегать повторного создания дорогих объектов «с нуля»,
скрыть детали создания объекта от клиента.
🎯 Когда он нужен?
Когда создание объекта — дорогостоящая операция (много полей, вложенные объекты);
Когда нужно копировать объекты, не зная их точный тип во время компиляции;
Когда используется абстрактная фабрика и нужно клонировать объекты из семейства.

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

## Структурные(structural)


**Адаптер (Adapter)** — это структурный паттерн, который позволяет объектам с несовместимыми интерфейсами работать вместе.
Он оборачивает один класс в другой, чтобы преобразовать интерфейс существующего класса в тот, который ожидает клиент.
🎯 Зачем нужен?
Когда:
ты хочешь использовать существующий класс, но его интерфейс не совпадает с тем, который ожидает твой код;
переопределить чужой код ты не можешь (например, сторонняя библиотека);
нужно интегрировать старый и новый код, не переписывая всё.

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

✅ Что делает паттерн Мост?
Он позволяет:

Избежать жёсткой связи между абстрактным интерфейсом и его реализацией;

Изменять абстракцию и реализацию независимо (например, без перекомпиляции);

Упростить код при росте количества комбинаций подклассов (вместо множественного наследования).

🎯 Когда использовать?
Когда есть разные виды объектов и множество реализаций поведения, и их комбинации могут взрывать количество подклассов;

Когда ты хочешь переключать реализацию во время выполнения;

Когда хочешь разделить интерфейс и реализацию по разным иерархиям.


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

✅ Что делает паттерн Компоновщик?
Он позволяет строить иерархии объектов в виде дерева, где:
Листья — это простые объекты (например, кнопка),
Узлы (композитные объекты) — содержат другие объекты (например, окно, содержащее кнопки и текст).
🎯 Зачем нужен?
Чтобы упростить работу с иерархией: клиент не должен отличать, работает он с отдельным объектом или с их группой.
Чтобы унифицировать интерфейс для всех компонентов.
Особенно полезен в GUI, файловых системах, иерархиях сцен и т.д.

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
✅ Что делает паттерн Декоратор?
Он:
Оборачивает объект в другой объект с таким же интерфейсом,
Добавляет новое поведение до/после вызова оригинального метода,
Позволяет комбинировать разные "дополнения" гибко и независимо.
🎯 Когда использовать?
Когда нельзя или неудобно использовать наследование;
Когда нужно гибко добавлять поведение к объектам во время выполнения;
Когда хочется разделить обязанности без создания множества подклассов.

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

✅ Что делает фасад?
Он:
Скрывает сложность системы за простым интерфейсом;
Делает код удобнее и понятнее для пользователя;
Помогает изолировать зависимости между клиентом и внутренними компонентами.
🎯 Когда использовать?
Когда система слишком сложная или запутанная, и ты хочешь скрыть детали реализации;
Когда есть много взаимосвязанных классов, но клиенту нужно только простое взаимодействие;
Когда нужно облегчить миграцию, тестирование или разделение ответственности.

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

✅ Зачем нужен паттерн Приспособленец?
Он нужен, когда:
В программе много объектов, и они занимают слишком много памяти;
Многие из этих объектов повторяются по содержанию (например, символы, части дерева, плитки, враги и т.п.);
Повторяющееся состояние можно вынести в общее хранилище (внешнее состояние);
Остаётся только уникальное состояние (внутреннее), которое не занимает много памяти.

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

✅ Зачем нужен паттерн Прокси?
Паттерн прокси нужен, когда:
Нужно контролировать доступ к объекту (например, ограничения доступа или логирование).
Требуется ленивая загрузка (например, объект слишком тяжёлый и создаётся только по мере необходимости).
Требуется кэширование результатов работы объекта.
Нужно обеспечить удалённый доступ (например, к удалённым ресурсам).
Безопасность: прокси может проверять права доступа к объекту.
🎯 Когда использовать?
Когда объект слишком тяжёлый или дорогой в создании (например, загрузка большого изображения или доступ к базе данных).
Когда нужно ограничить доступ к какому-либо ресурсу (например, только авторизованные пользователи могут выполнять определённые операции).
Когда объект должен быть заменён или защищён другими объектами с дополнительным поведением (например, кэширование, логирование).

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
