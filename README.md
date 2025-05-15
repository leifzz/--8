# programirowaniya
#include <SFML/Graphics.hpp>
#include <iostream>
#include <vector>
#include <ctime>
#include <sstream>
#include <iomanip>

using namespace sf;
using namespace std;

// Класс для автомобиля
class Car {
public:
    string licensePlate;
    time_t entryTime;
    double costPerHour;

    Car(string plate, time_t time, double rate) : 
        licensePlate(plate), entryTime(time), costPerHour(rate) {}

    double calculateCost() const {
        time_t now = time(nullptr);
        double hours = difftime(now, entryTime) / 3600.0;
        return hours * costPerHour;
    }

    string getParkingDuration() const {
        time_t now = time(nullptr);
        int seconds = difftime(now, entryTime);
        int hours = seconds / 3600;
        int mins = (seconds % 3600) / 60;
        int secs = seconds % 60;
        
        stringstream ss;
        ss << setfill('0') << setw(2) << hours << ":" 
           << setw(2) << mins << ":" << setw(2) << secs;
        return ss.str();
    }
};

// Класс парковки
class ParkingLot {
private:
    vector<Car> cars;
    int maxSpots;
    double hourlyRate;
    double totalIncome;

public:
    ParkingLot(int spots, double rate) : 
        maxSpots(spots), hourlyRate(rate), totalIncome(0) {}

    bool addCar(const string& plate) {
        if (cars.size() >= maxSpots) return false;
        
        cars.emplace_back(plate, time(nullptr), hourlyRate);
        return true;
    }

    double removeCar(int index) {
        if (index < 0 || index >= cars.size()) return -1;
        
        double cost = cars[index].calculateCost();
        totalIncome += cost;
        cars.erase(cars.begin() + index);
        return cost;
    }

    void setHourlyRate(double rate) {
        hourlyRate = rate;
    }

    int getAvailableSpots() const {
        return maxSpots - cars.size();
    }

    int getOccupiedSpots() const {
        return cars.size();
    }

    double getTotalIncome() const {
        return totalIncome;
    }

    const vector<Car>& getCars() const {
        return cars;
    }

    double getHourlyRate() const {
        return hourlyRate;
    }
};

// Генератор случайных номеров автомобилей
string generateLicensePlate() {
    static const string letters = "ABEKMHOPCTYX";
    string plate;
    
    plate += letters[rand() % letters.size()];
    plate += to_string(rand() % 900 + 100);
    plate += letters[rand() % letters.size()];
    plate += letters[rand() % letters.size()];
    
    return plate;
}

int main() {
    srand(time(nullptr));
    
    // Создание окна
    RenderWindow window(VideoMode(800, 600), "Платная парковка");
    window.setFramerateLimit(60);

    // Шрифты
    Font font;
    if (!font.loadFromFile("arial.ttf")) {
        cerr << "Ошибка загрузки шрифта!" << endl;
        return 1;
    }

    // Создание парковки
    ParkingLot parking(20, 50.0); // 20 мест, 50 руб/час

    // Элементы интерфейса
    Text title("Платная парковка", font, 30);
    title.setPosition(20, 10);
    title.setFillColor(Color::Black);

    Text infoText("", font, 18);
    infoText.setPosition(20, 50);
    infoText.setFillColor(Color::Black);

    Text rateText("", font, 18);
    rateText.setPosition(20, 80);
    rateText.setFillColor(Color::Black);

    Text spotsText("", font, 18);
    spotsText.setPosition(20, 110);
    spotsText.setFillColor(Color::Black);

    Text incomeText("", font, 18);
    incomeText.setPosition(20, 140);
    incomeText.setFillColor(Color::Black);

    // Кнопки
    RectangleShape addCarButton(Vector2f(200, 40));
    addCarButton.setPosition(20, 180);
    addCarButton.setFillColor(Color(100, 200, 100));

    Text addCarText("Добавить автомобиль", font, 18);
    addCarText.setPosition(30, 185);
    addCarText.setFillColor(Color::White);

    RectangleShape removeCarButton(Vector2f(200, 40));
    removeCarButton.setPosition(240, 180);
    removeCarButton.setFillColor(Color(200, 100, 100));

    Text removeCarText("Удалить автомобиль", font, 18);
    removeCarText.setPosition(250, 185);
    removeCarText.setFillColor(Color::White);

    RectangleShape changeRateButton(Vector2f(200, 40));
    changeRateButton.setPosition(460, 180);
    changeRateButton.setFillColor(Color(100, 100, 200));

    Text changeRateText("Изменить тариф", font, 18);
    changeRateText.setPosition(470, 185);
    changeRateText.setFillColor(Color::White);

    // Таблица автомобилей
    RectangleShape tableHeader(Vector2f(760, 30));
    tableHeader.setPosition(20, 240);
    tableHeader.setFillColor(Color(70, 70, 70));

    Text header1("Номер", font, 18);
    header1.setPosition(30, 240);
    header1.setFillColor(Color::White);

    Text header2("Время въезда", font, 18);
    header2.setPosition(180, 240);
    header2.setFillColor(Color::White);

    Text header3("Время парковки", font, 18);
    header3.setPosition(380, 240);
    header3.setFillColor(Color::White);

    Text header4("Стоимость", font, 18);
    header4.setPosition(580, 240);
    header4.setFillColor(Color::White);

    vector<Text> carTexts;
    vector<RectangleShape> carRows;

    // Статус бар
    RectangleShape statusBar(Vector2f(760, 30));
    statusBar.setPosition(20, 550);
    statusBar.setFillColor(Color(200, 200, 200));

    Text statusText("Готов к работе", font, 18);
    statusText.setPosition(30, 550);
    statusText.setFillColor(Color::Black);

    // Основной цикл программы
    while (window.isOpen()) {
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }

            // Обработка кликов на кнопки
            if (event.type == Event::MouseButtonReleased && 
                event.mouseButton.button == Mouse::Left) {
                Vector2f mousePos = window.mapPixelToCoords(
                    Vector2i(event.mouseButton.x, event.mouseButton.y));

                // Добавить автомобиль
                if (addCarButton.getGlobalBounds().contains(mousePos)) {
                    if (parking.getAvailableSpots() > 0) {
                        parking.addCar(generateLicensePlate());
                        statusText.setString("Автомобиль добавлен на парковку");
                    } else {
                        statusText.setString("Ошибка: нет свободных мест!");
                    }
                }

                // Удалить автомобиль
                if (removeCarButton.getGlobalBounds().contains(mousePos)) {
                    if (parking.getOccupiedSpots() > 0) {
                        // В реальной программе здесь должен быть выбор автомобиля
                        double cost = parking.removeCar(0);
                        stringstream ss;
                        ss << "Автомобиль удален. Стоимость: " << fixed << setprecision(2) << cost << " руб";
                        statusText.setString(ss.str());
                    } else {
                        statusText.setString("Ошибка: нет автомобилей на парковке!");
                    }
                }

                // Изменить тариф
                if (changeRateButton.getGlobalBounds().contains(mousePos)) {
                    // В реальной программе здесь должно быть диалоговое окно
                    parking.setHourlyRate(parking.getHourlyRate() + 10);
                    statusText.setString("Тариф увеличен на 10 руб/час");
                }
            }
        }

        // Обновление информации
        time_t now = time(nullptr);
        tm* localTime = localtime(&now);
        char timeStr[100];
        strftime(timeStr, sizeof(timeStr), "%d.%m.%Y %H:%M:%S", localTime);

        stringstream infoSS;
        infoSS << "Текущее время: " << timeStr;
        infoText.setString(infoSS.str());

        stringstream rateSS;
        rateSS << "Текущий тариф: " << fixed << setprecision(2) << parking.getHourlyRate() << " руб/час";
        rateText.setString(rateSS.str());

        stringstream spotsSS;
        spotsSS << "Мест: " << parking.getAvailableSpots() + parking.getOccupiedSpots() 
                << " | Занято: " << parking.getOccupiedSpots() 
                << " | Свободно: " << parking.getAvailableSpots();
        spotsText.setString(spotsSS.str());

        stringstream incomeSS;
        incomeSS << "Общий доход: " << fixed << setprecision(2) << parking.getTotalIncome() << " руб";
        incomeText.setString(incomeSS.str());

        // Обновление таблицы автомобилей
        carTexts.clear();
        carRows.clear();

        const vector<Car>& cars = parking.getCars();
        for (size_t i = 0; i < cars.size(); ++i) {
            // Строка таблицы
            RectangleShape row(Vector2f(760, 25));
            row.setPosition(20, 280 + i * 30);
            row.setFillColor(i % 2 == 0 ? Color(220, 220, 220) : Color(240, 240, 240));
            carRows.push_back(row);

            // Номер автомобиля
            Text plateText(cars[i].licensePlate, font, 16);
            plateText.setPosition(30, 280 + i * 30);
            plateText.setFillColor(Color::Black);
            carTexts.push_back(plateText);

            // Время въезда
            char entryTimeStr[100];
            strftime(entryTimeStr, sizeof(entryTimeStr), "%d.%m.%Y %H:%M:%S", localtime(&cars[i].entryTime));
            Text entryText(entryTimeStr, font, 16);
            entryText.setPosition(180, 280 + i * 30);
            entryText.setFillColor(Color::Black);
            carTexts.push_back(entryText);

            // Время парковки
            Text durationText(cars[i].getParkingDuration(), font, 16);
            durationText.setPosition(380, 280 + i * 30);
            durationText.setFillColor(Color::Black);
            carTexts.push_back(durationText);

            // Стоимость
            stringstream costSS;
            costSS << fixed << setprecision(2) << cars[i].calculateCost() << " руб";
            Text costText(costSS.str(), font, 16);
            costText.setPosition(580, 280 + i * 30);
            costText.setFillColor(Color::Black);
            carTexts.push_back(costText);
        }

        // Отрисовка
        window.clear(Color::White);

        // Рисуем элементы интерфейса
        window.draw(title);
        window.draw(infoText);
        window.draw(rateText);
        window.draw(spotsText);
        window.draw(incomeText);

        window.draw(addCarButton);
        window.draw(addCarText);
        window.draw(removeCarButton);
        window.draw(removeCarText);
        window.draw(changeRateButton);
        window.draw(changeRateText);

        // Рисуем таблицу
        window.draw(tableHeader);
        window.draw(header1);
        window.draw(header2);
        window.draw(header3);
        window.draw(header4);

        for (auto& row : carRows) window.draw(row);
        for (auto& text : carTexts) window.draw(text);

        window.draw(statusBar);
        window.draw(statusText);

        window.display();
    }

    return 0;
}
