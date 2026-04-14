# Гайд по 8 лабе(исключения)
Мне опять не дали дополнительных задач, поэтому вот, что нужно сделать

### Класс исключения
Заголовочный файл
```cpp
class Exception {
  private:
    string message;
    float badArgument; // или int в зависимости от вашего числового поля
  public:
    Exception(string message, float badArgument);
    ~Exception();

    string getMessage() const;
    float getBadArgument() const;
};
```
Реализация
```cpp
#include "Exception.h"

Exception::Exception(string message, float badArgument) {
    this->message = message;
    this->badArgument = badArgument;
}

Exception::~Exception() {}

string Exception::getMessage() const {
    return message;
}

float Exception::getBadArgument() const {
    return badArgument;
}
```

### Изменения базового класса
Теперь нужно, чтобы сеттер выбрасывал исключение при неверном значении(в моем случае это отрицательная цена)
```cpp
void Purchase::setPrice(float price) {
    if (price < 0) {
        throw Exception("Purchase: got wrong price.", price);
    }

    this->price = price;
}
```
Чтобы выполнить задание нужно отлавливать это исключение в конструкторе и методе ввода:
```cpp
Purchase::Purchase(string productName, float price, int quantity) {
    this->productName = productName;
    this->quantity = quantity;
    
    try {
    	setPrice(price);
	}
	catch (const Exception& e) {
		this->price = 0;
	}
}
```
В методе ввода создаем цикл:
```cpp
float input_price;
	while (true) {
		cout << "\nPrice: ";
    	cin >> input_price;
		
		try {
			setPrice(input_price);
			break;
		}
		catch (const Exception& e) {
			cout << "\nWrong price!";
		}
	}
```

### Изменение коллекции:
```cpp
// вместо
if (!f) {
  cout << "\nFile not found\n";
  return;
}
// выбрасываем исключение:
if (!f) {
  throw string("PurchaseCollection::saveToFile: error opening file: ") + fileName;
}
```
То же самое делам в методе загрузки из файла

### Демонстрация в main:
```cpp
// логи в файл
freopen("logs.txt", "w", stderr);

// демонстрируем работу конструктора с неверным значением
Purchase* badPur = new Purchase("Cola", -3, 2);
badPur->output();

// используем сеттер с неверным значением
try {
	pur->setPrice(-1);
}
catch (const Exception& e) {
	cerr << e.getMessage() << " Got: " << e.getBadArgument() << endl;
}
pur->output();

PurchaseCollection pc(0);
pc.add(pur);

// сохраняем коллекцию на диск С, который заблокирован на стационарных ПК
try {
	pc.saveToFile("C:\Collection.txt");
}
catch (const string& e) {
	cerr << e << endl;
}

PurchaseCollection pc2(0);
// читаем несуществующий файл
try {
	pc2.loadFromFile("Collection.txt");
}
catch (const string& e) {
	cerr << e << endl;
}
```

