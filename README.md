# Выполнили Погребняк Степан Сиваселвамович и Кирилин Геннадий Дмитривевич

# Цель работы

![image](https://github.com/user-attachments/assets/35b35477-0faf-4c65-ac2e-6989fd009cde)




```
#include <stdint.h>
#include <stdbool.h>

#define GPIOA_BASE  0x48000000
#define GPIOB_BASE  0x48000400
#define RCC_BASE    0x40021000

#define GPIOA_MODER     (*(volatile uint32_t*)(GPIOA_BASE + 0x00))
#define GPIOA_IDR       (*(volatile uint32_t*)(GPIOA_BASE + 0x10))
#define GPIOA_ODR       (*(volatile uint32_t*)(GPIOA_BASE + 0x14))

#define GPIOB_MODER     (*(volatile uint32_t*)(GPIOB_BASE + 0x00))
#define GPIOB_ODR       (*(volatile uint32_t*)(GPIOB_BASE + 0x14))

#define RCC_AHBENR      (*(volatile uint32_t*)(RCC_BASE + 0x14))

// Битовые маски сегментов для цифр 0-9
uint8_t segment_map[] = {
    0b00111111, // 0
    0b00000110, // 1
    0b01011011, // 2
    0b01001111, // 3
    0b01100110, // 4
    0b01101101, // 5
    0b01111101, // 6
    0b00000111, // 7
    0b01111111, // 8
    0b01101111  // 9
};

// Переменные
volatile uint32_t counter = 0; // Счётчик импульсов
volatile bool button_pressed = false;

// Задержка (для подавления дребезга контактов)
void simpleDelay(volatile uint32_t count) {
    while (count--);
}

// Инициализация GPIO
void initGPIO() {
    // Включение тактирования GPIOA и GPIOB
    RCC_AHBENR |= (1 << 17) | (1 << 18);

    // Настройка GPIOA: PA0-PA7 как Output (для сегментов)
    GPIOA_MODER &= ~(0xFFFF);  
    GPIOA_MODER |= 0x5555;

    // Настройка GPIOB: PB0 как Output (для светодиода)
    GPIOB_MODER &= ~(0x3);
    GPIOB_MODER |= 0x1;

    // Настройка GPIOA: PA8 как Input (для генератора логических уровней)
    GPIOA_MODER &= ~(0x3 << 16);
}

// Отображение цифры на семисегментном индикаторе
void displayDigit(uint8_t digit) {
    GPIOA_ODR = segment_map[digit]; // Установка состояния выходов
}

// Обработка входного сигнала с подавлением дребезга
void processInput() {
    static bool last_state = false;
    bool current_state = (GPIOA_IDR & (1 << 8)) ? true : false;

    if (current_state && !last_state) {
        simpleDelay(10000); // Небольшая задержка для подавления дребезга
        if ((GPIOA_IDR & (1 << 8)) != 0) {
            counter++;
            button_pressed = true;
        }
    }
    last_state = current_state;
}

int main() {
    initGPIO();

    while (1) {
        // Обработка входного сигнала
        processInput();

        // Обновление индикации на семисегментном индикаторе
        displayDigit(counter % 10);

        // Управление светодиодом
        if (button_pressed) {
            GPIOB_ODR |= (1 << 0);  // Включить светодиод
            button_pressed = false;
        } else {
            GPIOB_ODR &= ~(1 << 0); // Выключить светодиод
        }

        simpleDelay(100000); // Задержка между обновлениями
    }

    return 0;
}

```
