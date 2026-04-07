# Гайд по 7 лабе по С++(полиморфизм)
Задание, которое я получил: продемонстрировать полиморфизм. Это можно делать по-разному, но от вас ожидают следующее:
```cpp
Purchase* newPur;
newPur = new CreditPurchase("TEST2", 99.0, 3, 3, "Belagro");
newPur->output(); //output - виртуальный метод
```
Здесь мы сначала объявляем ссылку на объект базового класса, но заносим туда адрес объекта класса наследника. output выводит данные класса-наследника именно благодаря полиморфизму.
На этом задания кончились, но это скорее из-за того, что я много переделывал код. 
### Вот пару советов, которые помогут написать код, который нужен
Виртуальными методами нужно сделать: 
- деструктор;
- метод вывода данных класса(print/output);
- метод ввода данных(input);
- метод сохранения данных класса в файл;
- метод загрузки из файла;
- метод получения символьного ключа для сортировки. 
Кроме этого для работоспособности потребуется еще 2 виртуальных метода: getType(), который возвращает уникальное для это класса значения(например string, в котором будет название класса) и метод clone(), который возвращает ссылку на копию этого объекта. Первый метод нужен для сохранения в файл так как в С++ нет функции получения типа класса. Второй нужен чтобы избежать "обрезки" объекта класса наследника при добавлении через перегруженный оператор +=:
```cpp
//Базовый класс
string Purchase::getType() {
    return "Purchase";
}

Purchase* Purchase::clone() const {
    return new Purchase(*this);
}
```

```cpp
//Класс наследник
string CreditPurchase::getType() {
    return "CreditPurchase";
}

CreditPurchase* CreditPurchase::clone() const {
    return new CreditPurchase(*this);
}
```
clone используется в операторе += &Purchase:
```cpp
void PurchaseCollection::operator+= (const Purchase& p) {
    *this += p.clone();
}
```

### Символьный ключ для сортировки
У меня базовый класс нужно сортировать по названию продукта, а класс-наследник по названию банка. Если есть и те и другие классы, то сначала идет базовый класс. Для этого мы пишем такой виртуальный метод:
```cpp
//Базовый класс
string Purchase::getKey() {
    return "A" + productName;
}
```

```cpp
//Класс наследник
string CreditPurchase::getKey() {
    return "B" + bank;
}
```
Этими буквами перед сортируемыми полями мы делаем так, чтобы объекты базового класса всегда шли первые.

### Методы сохранения в файл
Для дэбилов(в том числе меня): в классах методы для сохранения в файл должны быть вызваны только из коллекции, не нужно пытаться использовать их самостоятельно в main.cpp.
```cpp
//Базовый класс
void Purchase::saveToFile(FILE* file) {
    fprintf(file, "%s\n%s %f %d ", getType().c_str(), productName.c_str(), price, quantity);
}

void Purchase::loadFromFile(FILE* file) {
    char buffer[100];
    float p;
    int q;

    if (fscanf(file, "%99s %f %d ", buffer, &p, &q) == 3) {
        setProductName(buffer);
        setPrice(p);
        setQuantity(q);
        return;
    }

    cout << "\nFailed to load from file\n";
}
```

```cpp
//Класс наследник
void CreditPurchase::saveToFile(FILE *file) {
	Purchase::saveToFile(file); //Здесь вызовется getType, но так как это виртуальный метод подставится тип класса-наследника, а не базового класса
    fprintf(file, "%d %s ", monthCount, bank.c_str());
}

void CreditPurchase::loadFromFile(FILE *file) {
    int m;
    char bankBuffer[100];
    
	Purchase::loadFromFile(file);

    if (fscanf(file, "%d %99s ", &m, bankBuffer) == 2) {
        setMonthCount(m);
        setBank(bankBuffer);
        return;
    }

    cout << "\nFailed to load from file\n";
}
```

```cpp
//Коллекция
void PurchaseCollection::saveToFile(string fileName) {
    FILE* f = fopen(fileName.c_str(), "wt");
    if (!f) {
        cout << "\nFile not found\n";
        return;
    }
    
    fprintf(f, "%d\n", size); //Записываем количество элементов коллекции

	for (int i = 0; i < size; i++) {
	    arr[i]->saveToFile(f); //Для каждого элемента коллекции вызываем метод сохранения
	    fprintf(f, "\n");
    }

    fclose(f);
}

void PurchaseCollection::loadFromFile(string fileName) {
    FILE* f = fopen(fileName.c_str(), "rt");
    if (!f) {
        cout << "\nFile not found\n";
        return;
    }
    
    //Очищаем коллекцию перед загрузкой из файла
    int pcSize = size;
    for (int i = 0; i < pcSize; i++) {
    	remove(0);
	}
    
    int collectionSize;
    fscanf(f, "%d\n", &collectionSize);
    
    char type[100];
    
    for (int i = 0; i < collectionSize; i++) {
    	fscanf(f, "%99s", type);
    	Purchase* curPur;

        if (string(type) == "Purchase") {
            curPur = new Purchase;
        }
        else {
            curPur = new CreditPurchase;
        }

        curPur->loadFromFile(f);
        add(curPur);
	}

    fclose(f);
}
```
Моя реализация может отличаться от того, что будет у вас потому что у меня динамическая коллекция.
