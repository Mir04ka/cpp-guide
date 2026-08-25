C++ лаба 6(переопределение операторов)
Во-первых, все переопределенные операторы нужно расположить в самом конце класса:
```cpp
class PurchaseCollection {
  private:
    Purchase** arr;
    int size;
    int maxSize;

  public:
    PurchaseCollection(int s);
    PurchaseCollection(const PurchaseCollection& pc);
    ~PurchaseCollection();

    void add(Purchase* p);
    void remove(int i);
    int getSize() const;
    Purchase* get(int i) const;
    void swap(int i, int j);
    void sort();
    void print();
    
    friend void operator+= (PurchaseCollection& pc, Purchase* p);
    void operator+= (const Purchase& p);
    void operator-= (int index);
    void operator-= (string productName);
    Purchase* operator[](int index);
    Purchase* operator[](string productName);
    int operator< (const Purchase& p);
};
```
Все операторы изначально методы класса.

Задания, которые она мне дала:
1. Один оператор из метода класса переделать в дружественную функцию:
h файл:
```cpp
//было:
void operator+= (Purchase* p);

//стало:
friend void operator+= (PurchaseCollection& pc, Purchase* p);
```
cpp файл:
```cpp
//было:
void PurchaseCollection::operator+= (Purchase* p) {
  add(p);
}

//стало:
void operator+= (PurchaseCollection& pc, Purchase* p) {
    pc.add(p);
}
```

2. Написали в main.cpp сравнение двух объектов класса:
```cpp
Purchase pur3("Meat", 12.50, 1);
Purchase pur4("Bread", 12.50, 2);
if (pur3 < pur4) { ... }
```
Это задание на переопределение оператора сравнения в нашем базовом классе:
```cpp
// в .h файл нашего базового класса добавлям оператор:
bool operator< (const Purchase& p);
```
```cpp
// в .cpp файл базового класса реализуем сравнение:
bool Purchase::operator< (const Purchase& p) {
// логику сравнения она вам скажет, мой говнокод вам тут не нужен
// обязательно возвращаем true/false
}
```

3. Пишем в main.cpp следующее:
```cpp
// это у нас было, но нужно для контекста
PurchaseCollection pc(1);
Purchase pur3("Meat", 12.50, 1);

// а вот это само задание
int k = pc < pur3;
```
По заданию k должно принять значение количества элементов коллекции, которые меньше pur3. Это задание основано на прошлом, так как нам понадобится оператор сравнения базового класса. Так как первый операнд — коллекция, мы переопределим оператор "меньше" в классе коллекции:
```cpp
// в Collection.h добавляем заголовок:
int operator< (const Purchase& p);

// в Collection.cpp логику:
int PurchaseCollection::operator< (const Purchase& p) {
  int result = 0;
  
  int pcSize = size;
  for (int i = 0; i < pcSize; i++) {
    if (*arr[i] < p) {
      result++;
    }
  }
  
  return result;
}
```
Алгоритм может отличаться в зависимости от задания + у меня динамическая коллекция в отличии от вас, но суть, я думаю, понятна.
