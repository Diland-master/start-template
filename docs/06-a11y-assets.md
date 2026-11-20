# 06. Доступность и ресурсы

## Доступность (обязательно)

- Один `<h1>` на страницу. Уровни заголовков не пропускаются. Как заголовок выглядит, задают классы `.h1–.h6`.
- Каждая `<section>` получает `aria-labelledby={titleId}`. Если заголовок не виден, он всё равно есть в разметке с классом `visually-hidden`.
- Кнопка без текста получает `aria-label` и `title` (в `Button` это делает `isLabelHidden`).
- Декоративные картинки: `alt=""`. Содержательные: осмысленный `alt`. Иконка со смыслом получает `ariaLabel` у `<Icon>`.
- Списки карточек и меню размечаются как `<ul>/<ol>` + `<li>`. Пронумерованные пункты: `<ol>` с CSS-счётчиком (`AccordionGroup`).
- Интерактив повторяет ARIA-паттерны: табы (`role="tablist/tab"`, `aria-selected`, `aria-controls`, роуминг `tabIndex`, стрелки), селект (`role="combobox/listbox/option"`, `aria-expanded`, `aria-activedescendant`).
- Даты размечаются через `<time dateTime="2023-04-14">`.
- Фокус: глобальный `:focus-visible`. Свой стиль фокуса допускается, но убирать фокус без замены нельзя.
- `lang` у `<html>` берётся из `constants/site.js`.
- Звёздочка обязательного поля скрывается от скринридеров (`aria-hidden`), а у поля стоит нативный `required`.

## Картинки

| Случай | Как |
|---|---|
| Растровая контентная картинка (фото, постер) | `import { Image } from 'minista'` → `<Image className="x__image" src="/src/assets/images/…/1.jpg" />`. Minista сама строит `srcset` по брейкпоинтам и `@2x`. |
| SVG или логотип в `<img>` | импорт файла: `import logoImgSrc from '@/assets/images/logo.svg'` → `<img src={logoImgSrc} width={199} height={60} alt="" loading="lazy" />`. Для того, что видно на первом экране, `loading="eager"`. |
| Фон | В SCSS: `url('@/assets/images/hero/bg.png') 50%/cover`. Для мобильной версии отдельный файл `bg_mobile.png` в медиа-миксине. Градиентные затемнения пишутся слоями `background`. |
| Видео | `src/assets/videos`, постер в `images/video-posters`. |

- Картинки складываются по папкам секций: `images/<section>/…`.
- У `<img>` всегда указаны `width` и `height` (размеры из макета, против CLS).
- Подписи к картинкам не дублируются в `alt`, если рядом есть текст.

## Иконки

- SVG кладутся в `src/assets/icons/<name>.svg`. Minista собирает из них спрайт.
- Подключение: `<Icon name="arrow-left" />` (компонент из `_library/components/Icon`).
- В `minista.config.js` `cleanSymbols` вычищает `fill` и `stroke`, поэтому цвет иконки = `currentColor`:
  - иконка-контур: по умолчанию (`stroke="currentColor"`);
  - иконка-заливка: `hasFill` / `hasFillIcon` (`fill="currentColor"`).
- Размер задаётся через `--iconSize` (по умолчанию `fluid-to-laptop(28, 24)`) и перебивается снаружи: `&.icon { --iconSize: 20px }`.
- Многоцветные SVG (иллюстрации, логотипы) в спрайт не кладутся, они идут в `images/` и подключаются через `<img>`.

## Шрифты

- Только `.woff2`, в `src/assets/fonts`. На каждое начертание отдельный `@font-face` в `styles/fonts.scss` с `font-display: swap`. Путь записывается через алиас: `url('@/assets/fonts/Name-Regular.woff2')`.
- Семейство задаётся в `--font-family-base`.
- Если файлов шрифта нет (а только название в Figma), это фиксируется в отчёте. Подключать Google Fonts по ссылке нельзя без согласования.

## Фавиконки

Лежат в `public/` (`favicon.ico`, `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png`, `android-chrome-*.png`, `site.webmanifest`). В каждом проекте заменяются на свои.
