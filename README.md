# patterns
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


