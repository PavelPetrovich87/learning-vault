---
status: stable
tags: [ai, perceptron, neural-networks, javascript]
created: 2026-10-14
updated: 2026-10-14
---

# Перцептрон и XOR-сеть на JavaScript

Related: [[Perceptron-Basics]] · [[Hidden-Layer-New-Coordinates]]

> [!summary] TL;DR
> Весь перцептрон — класс на 15 строк. Сеть для XOR — два детектора углов плюс выходной нейрон поверх их ответов. Запускается в консоли браузера или `node`.

## 1. Один перцептрон

```javascript
class Perceptron {
  constructor(nInputs, lr = 0.1) {
    this.weights = new Array(nInputs).fill(0); // веса, стартуют с нулей
    this.bias = 0;
    this.lr = lr; // learning rate — размер шага
  }

  predict(inputs) {
    const total = this.weights.reduce((s, w, i) => s + w * inputs[i], this.bias);
    return total >= 0 ? 1 : 0; // ступенька: сумма ≥ 0 → «да»
  }

  train(data, epochs = 100) {
    for (let epoch = 0; epoch < epochs; epoch++) {
      let errors = 0;
      for (const [inputs, target] of data) {
        const error = target - this.predict(inputs); // −1, 0 или +1
        if (error !== 0) errors++;
        // вини только голосовавших: × inputs[i] обнуляет шаг для «молчавших» входов
        this.weights = this.weights.map((w, i) => w + this.lr * error * inputs[i]);
        this.bias += this.lr * error; // bias = вес фиктивного входа, всегда равного 1
      }
      if (errors === 0) return epoch + 1; // сошёлся
    }
    return null; // не сошёлся
  }
}
```

## 2. Обучаем на AND / OR / NOT

```javascript
const andData = [[[0,0],0], [[0,1],0], [[1,0],0], [[1,1],1]];
const orData  = [[[0,0],0], [[0,1],1], [[1,0],1], [[1,1],1]];
const notData = [[[0],1],  [[1],0]];

const pAnd = new Perceptron(2);
console.log('AND сошёлся на эпохе', pAnd.train(andData)); // 4
console.log('веса:', pAnd.weights, 'bias:', pAnd.bias);   // [0.2, 0.1] −0.2

console.log('OR сошёлся на эпохе', new Perceptron(2).train(orData));  // 4
console.log('NOT сошёлся на эпохе', new Perceptron(1).train(notData)); // 3
```

Веса получаются «на грани» и не уникальны — учится поведение, а не конкретные числа.

## 3. XOR: один перцептрон бессилен

```javascript
const xorData = [[[0,0],0], [[0,1],1], [[1,0],1], [[1,1],0]];
console.log(new Perceptron(2).train(xorData)); // null — кружит вечно
```

Причина — математическая невозможность одной прямой: [[XOR-Impossibility]].

## 4. XOR-сеть: два детектора углов + выходной нейрон

```javascript
// скрытые нейроны: тройка чисел = вопрос про вход
const h1 = ([x1, x2]) => (-0.3 * x1 + 0.3 * x2 - 0.15 >= 0) ? 1 : 0; // «верхне-левый угол?»
const h2 = ([x1, x2]) => ( 0.3 * x1 - 0.3 * x2 - 0.15 >= 0) ? 1 : 0; // «нижне-правый угол?»

// выходной: обычный перцептрон поверх ответов h1/h2
const output = (s1, s2) => (s1 + s2 - 0.5 >= 0) ? 1 : 0;

const xor = ([x1, x2]) => output(h1([x1, x2]), h2([x1, x2]));

[[0,0],[0,1],[1,0],[1,1]].forEach(p => console.log(p.join(','), '->', xor(p)));
// 0,0 -> 0 ; 0,1 -> 1 ; 1,0 -> 1 ; 1,1 -> 0  ✓
```

Здесь веса скрытых нейронов подобраны руками для наглядности. Настоящее обучение всех трёх нейронов сразу — это backpropagation: ошибка выхода «разливается» назад по слоям. См. шаг дальше в курсе.

## 5. Шпаргалка соответствий Python → JS

| Python | JavaScript |
|--------|------------|
| `zip(a, b)` | пара по индексу: `a[i]` и `b[i]` внутри `reduce`/цикла |
| `sum(w*x for ...)` | `arr.reduce((s, w, i) => s + w * b[i], 0)` |
| `x if cond else y` | `cond ? x : y` |
| `[0.0] * n` | `new Array(n).fill(0)` |
| `for a, b in data:` | `for (const [a, b] of data)` |
| `range(epochs)` | `for (let e = 0; e < epochs; e++)` |
