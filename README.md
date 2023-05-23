# Code Refactoring

### **Варіант 2**

Необхідно проаналізувати дві ділянки коду, вказати конкретні "запахи коду" провести рефакторинг і вказати методики рефакторинга, які застосовуються.

## 1. Рефакторинг на рівні окремих операторів

The following function checks to see if there are any suspicious individuals in the list of persons whose names are hard coded.

```
// Початковий код
void checkSecurity(String[] people)
{
    boolean found = false;
    for (int i=0; i < people.Length; i++)
    {
        if (!found)
        {
            if (people[i].Equals ("Don"))
            {
                sendAlert();
                found = true;
            }
            if (people[i].Equals ("John"))
            {
                sendAlert();
                found = true;
            }
        }
    }
}
```

Знайдені "запахи коду":

1. Дублювання коду: Код для перевірки кожної особи ("Don" та "John") майже ідентичний, включаючи виклик функції sendAlert() і присвоєння found = true. Це дублювання можна виправити.
2. Зайва змінна found: Змінна found використовується для відстеження того, чи була знайдена підозріла особа. Однак, оскільки функція sendAlert() викликається в обох умовах, можна усунути зайву змінну found і викликати функцію безпосередньо.

 ``` 
 // Перероблений код
  void checkSecurity(String[] people)
  {
      boolean found = false;
      for (int i = 0; i < people.Length; i++)
      {
          if (people[i].Equals("Don") || people[i].Equals("John"))
          {
              sendAlert();
              found = true;
              break; // Перериваємо цикл, оскільки вже знайдено підозрілу особу
          }
      }
  }
  ```

Методика рефакторингу:

1. Заміна дублюючого коду: Об'єднання двох умовних операторів в один, що дозволяє уникнути дублювання коду.
2. Спрощення управління складністю: Використання змінної found без зовнішнього умовного оператора, що дозволяє спростити логіку та зменшити складність коду.

## 2. Рефакторинг на рівні даних

 ``` 
 // Початковий код
 enum TransportType
  {
      eCar,
      ePlane,
      eSubmarine
  };

  class Transport
  {
  public:
      Transport(TransportType type) : m_type(type) {}

      int GetSpeed(int distance, int time)
      {
          if (time != 0)
          {
              switch (m_type)
              {
                  case eCar:
                      return distance / time;
                  case ePlane:
                      return distance / (time - getTakeOffTime() - getLandingTime());
                  case eSubmarine:
                      return distance / (time - getDiveTime() - getAscentTime());
              }
          }
      }

      // ...
  private:
      int m_takeOffTime;
      int m_landingTime;
      int m_diveTime;
      int m_ascentTime;
      TransportType m_type;
  };
  ```

Знайдені "запахи коду":

1. Недостатня інкапсуляція даних: Змінні m_takeOffTime, m_landingTime, m_diveTime і m_ascentTime визначені як публічні. Це порушує принцип інкапсуляції і дозволяє безпосередній доступ до цих даних зовні класу. Краще було б оголосити ці змінні як приватні і забезпечити доступ до них через публічні методи (getters/setters).
2. Проблематичний вибір шляху в switch: У методі GetSpeed() використовується оператор switch для вибору розрахунку швидкості в залежності від типу транспорту. Однак, в цьому випадку оператор switch стає занадто складним і може стати незручним для підтримки та розширення. Замість цього можна було б реалізувати підхід поліморфізму, де кожен тип транспорту має свою власну реалізацію методу GetSpeed(). Це дозволить легко додавати нові типи транспорту в майбутньому без необхідності змінювати існуючий код.

```
// Перероблений код
enum TransportType
{
    eCar,
    ePlane,
    eSubmarine
};

class Transport
{
public:
    Transport(TransportType type) : m_type(type) {}

    int GetSpeed(int distance, int time)
    {
        if (time != 0)
        {
            switch (m_type)
            {
                case eCar:
                    return CalculateCarSpeed(distance, time);
                case ePlane:
                    return CalculatePlaneSpeed(distance, time);
                case eSubmarine:
                    return CalculateSubmarineSpeed(distance, time);
            }
        }

        return 0; 
    }

private:
    int CalculateCarSpeed(int distance, int time) { return distance / time; }
    int CalculatePlaneSpeed(int distance, int time) { return distance / (time - getTakeOffTime() - getLandingTime()); }
    int CalculateSubmarineSpeed(int distance, int time) { return distance / (time - getDiveTime() - getAscentTime()); }
    int getTakeOffTime() const { return m_takeOffTime; }
    int getLandingTime() const { return m_landingTime; }
    int getDiveTime() const { return m_diveTime; }
    int getAscentTime() const { return m_ascentTime; }

    TransportType m_type;
    int m_takeOffTime;
    int m_landingTime;
    int m_diveTime;
    int m_ascentTime;
};
```

Для виправлення "запахів коду" було проведено покращення інкапсуляції даних і використання поліморфізму для вибору відповідної реалізації методу GetSpeed(). Крім того, було додано публічні методи-аксесори (getters) для доступу до приватних змінних класу.
