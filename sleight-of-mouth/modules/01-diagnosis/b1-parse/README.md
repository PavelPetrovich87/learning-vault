---
id: block-b1-parse
type: block
parent: module-01-diagnosis
contract:
  input: {ref: contracts/phrase}
  output: {ref: contracts/map, fills: [тип, A, B]}
---

# b1-parse — разбор

Находит во фразе конструкцию CEq или C-E, вычленяет звенья A и B, определяет тип — заполняет шапку карты. Чтение мишеней I–II зависит от результата (CEq = определение, C-E = причина/следствие — канон §4).

Грамматика конструкций: [two-forms.drawio](two-forms.drawio) (две формы, слои) · [examples.md](examples.md) (референс, 34 примера) · [parse.md](parse.md) (заметка блока: вход, две формы, норма полноты, алгоритм с примерами). Алгоритм: [alg-parse.drawio](alg-parse.drawio) (блок-схема, шаги 1–4) · [implicit/implicit.md](implicit/implicit.md) + [implicit/implicit.drawio](implicit/implicit.drawio) (проба скрытой связки, шаг 3: признаки, подстановки, прецеденты) · [normalize/normalize.md](normalize/normalize.md) + [normalize/normalize.drawio](normalize/normalize.drawio) (нормализация звена: контракт, правила, прецеденты) · [outcomes.drawio](outcomes.drawio) (результаты и маршруты). Материал: гл. 1, 5–6 (с. 24–26, 90–96, 113–127).

Отказ — только не-утверждение (вопрос/императив) или несколько утверждений в одной фразе (единица входа). Неполная формулировка без связки CEq/C-E (гл. 6, с. 125) — не отказ: маршрут в b3-экспликацию, четыре вопроса-причины (см. [parse.md](parse.md)).

Алгоритмы: по наполнению (`alg-*.md`).
