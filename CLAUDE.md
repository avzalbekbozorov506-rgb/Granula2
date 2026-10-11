# Офис Авзала

Страница «Офис Авзала» показывает, как работает субагент `avzal-investor`:
https://claude.ai/artifact/UVrn1dsS79HDp2L9ZmtK2j
Исходник страницы: `office/index.html`. Состояние хранится в базе страницы (инструмент ArtifactData, коллекция `office`):
- `office/status`: `{state, task, step, summary, updatedAt}`; `state` одно из `idle, search, read, analyze, write, done, error`.
- `office/log`: `{entries: [{t, state, text}]}`, новые записи в начале, не больше 40.

Каждый раз, когда пользователь зовёт avzal-investor:
1. Обнови `office/status` (state `search`, task = вопрос пользователя коротко, step, updatedAt = текущее время UTC) и добавь запись в `office/log`. Читай документы перед записью и передавай `if_version`.
2. Очисти журнал и запусти Monitor на него (timeout 30 мин, при истечении перезапусти, пока агент работает):
   `mkdir -p .office && : > .office/progress.log && tail -n0 -F .office/progress.log`
3. Запусти субагента avzal-investor в фоне (`run_in_background: true`).
4. На каждую строку `время|этап|текст` из Monitor обновляй `office/status` (state = этап, step = текст) и добавляй запись в `office/log`. Несколько строк сразу пиши одним batch.
5. Когда субагент закончил: state `done` (или `error`), step «Отчёт готов!», summary = главный вывод в 1-3 строки; останови Monitor. Затем передай пользователю ответ агента.
6. Если офис недоступен, не задерживай ответ пользователю: работа агента важнее анимации.

## Помощники Авзала
- `revizor` (Ревизор): проверяет отчёты Авзала по первоисточникам и выносит вердикт.
- `portfolio-manager` (Портфельный менеджер): ведёт портфель в `portfolio/portfolio.md`, считает доли и выплаты, строит план покупок.
Они тоже пишут в `.office/progress.log` (текст начинается с «Ревизор:» или «Портфель:»), поэтому при их вызове выполняй те же шаги офиса, что и для avzal-investor.
