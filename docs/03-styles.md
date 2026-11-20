# 03. Стили (SCSS)

## 1. Токены — `styles/variables.scss`

Всё повторяющееся хранится в CSS-переменных в `:root`. Имена в **kebab-case**.

| Группа | Правило | Пример |
|---|---|---|
| Палитра | Имена как в дизайне. Схема: `--color-<оттенок>-<светлота HSL>` | `--color-red-45`, `--color-black-08`, `--color-gray-60` |
| Роли | Алиасы на палитру для `globals.scss` | `--color-bg`, `--color-text`, `--color-heading`, `--color-accent`, `--color-border` |
| Бордер | Единый бордер + толщины | `--border: var(--border-width-s) solid var(--color-border)` |
| Радиусы | Шкала `l / m / s / xs / xxs / xxxs` | `--border-radius-l: 12px` |
| Шрифт | `--font-family-base` | `'Manrope', sans-serif` |
| Контейнер | `--container-width` (fluid), `--container-padding-x` | |
| Ритм | `--section-padding-y: #{fluid(75, 40)}`, `--field-padding-x` | |
| Анимации | `--transition-duration` (0.2s), `--transition-duration-longer` (0.4s) | |

В компонентах **нельзя** писать hex/rgb напрямую. Исключения: полупрозрачные стопы градиентов (`rgba(20, 20, 20, 0.00)`), которые есть только в одном месте дизайна.

Компонент из `_library/` использует палитру Stream Vibe (`--color-red-45`, `--color-black-10`…). При копировании эти имена **заменяются на токены нового дизайна**.

## 2. Адаптив значений — `fluid()`

Файл `helpers/functions.scss`. Функция возвращает `clamp(min, линейная функция от 100vw, max)` в rem.

| Функция | Интерполяция | Когда использовать |
|---|---|---|
| `fluid(max, min)` | от вьюпорта **390** (min) до **1600** (max) | значение отличается между desktop и mobile макетами |
| `fluid-to-laptop(max, min)` | от **1280** (min) до **1600** (max), ниже 1280 остаётся min | значение отличается между desktop и laptop, а на mobile такое же, как на laptop |
| просто `px` | — | значение одинаково на всех макетах (`gap: 4px`) |
| `px` внутри медиа-миксина | — | резкая смена на брейкпоинте (`@include mobile { padding-inline: 8px }`) |

- Шрифты: `@include fluid-text(max, min)` (это `font-size: fluid(max, min)`).
- Аргументы — **числа без единиц**, в px из макета: `fluid(50, 24)`.
- Внутри CSS-переменной функция оборачивается в интерполяцию: `--iconSize: #{fluid-to-laptop(28, 24)};`.
- `max` — значение из desktop-макета, `min` — из mobile (для `fluid`) или laptop (для `fluid-to-laptop`).

Если у дизайна другие ширины макетов, это меняется один раз на старте проекта:
- `$container-desktop-width` / `$container-laptop-width` в `helpers/constants.scss` — ширины контейнера на desktop и laptop;
- `$minViewportWidth: 390` в `fluid()` — ширина mobile-макета.

## 3. Брейкпоинты — `helpers/media.scss`

Подход **desktop-first**, синтаксис диапазонов (`width <= …`).

| Миксин | Условие | Обратный |
|---|---|---|
| `laptop` | `width <= 1440px` | `laptop-above` |
| `tablet` | `width <= 1023px` | `tablet-above` |
| `mobile` | `width <= 767px` | `mobile-above` |
| `mobile-s` | `width <= 480px` | `mobile-s-above` |

```scss
.footer__menu {
	--footerMenuColumns: 6;

	@include laptop {
		--footerMenuColumns: 3;
	}

	@include mobile {
		--footerMenuColumns: 2;
	}
}
```

- **Нельзя** писать `@media` руками. Исключения: `@supports` и `prefers-*`.
- Медиа-миксины вкладываются **внутрь селектора**, а не наоборот.
- Ховер — только `@include hover { … }`: срабатывает на `:hover` у устройств с мышью и на `:active` у тач-устройств. Второй параметр `false` отключает `:active` на таче.
- Показ и скрытие по брейкпоинтам — утилитами `.visible-tablet`, `.hidden-tablet`, `.visible-mobile`, `.hidden-mobile`.
- В JS брейкпоинты берутся из `constants/MatchMedia.js` (`window.matchMedia` в rem через `pxToRem`).

## 4. Миксины — `helpers/mixins.scss`

`fluid-text`, `flex-center($isInline)`, `abs-cover`, `abs-center`, `abs-x`, `abs-y`, `square($size)`, `hide` / `show` (opacity + visibility — для анимируемого скрытия), `visually-hidden`, `focus-visible`, `disabled`.

Если стиль повторяется в 3 и более местах, под него заводится миксин. Если только внутри одного блока, миксин объявляется прямо внутри селектора (как `scrolling-header-styles` в Header).

## 5. БЭМ

- Блок = компонент: `.movie-card`. Элементы: `.movie-card__image`. Модификаторы: `.movie-card--big`, `.button--transparent`.
- Элементы всегда плоские: `header__menu-list`, **а не** `header__menu__list`. В SCSS вложенность пишется через `&-`:

```scss
.header {
	&__menu {
		&-list { … }   // .header__menu-list
		&-link { … }   // .header__menu-link
	}
}
```

- Состояния, которые переключает JS или задаёт разметка, — глобальные классы `is-*`: `is-active`, `is-expanded`, `is-selected`, `is-current`, `is-fixed`, `is-lock` (на `<html>` при открытом меню). Пишутся как `&.is-active`.
- В JSX у корня компонента внешний `className` идёт **первым**: `classNames(className, 'button', {...})`. Так родитель задаёт миксы: `<Button className="header__button" />`.
- Миксы с `container`: `className="support container"`, `className="header__inner container"`.

## 6. Локальные CSS-переменные компонента

Имена в **camelCase** с префиксом блока: `--iconSize`, `--gridColumns`, `--tabsNavigationPadding`, `--categoryCardImagesGradientOpacity`.

Используются, чтобы:
- менять значение на брейкпоинте одной строкой (`--gridColumns: 2` внутри `@include tablet`);
- перенастраивать дочерний компонент снаружи без перебивания свойств:

```scss
.category-card__icon {
	&.icon {
		--iconSize: #{fluid(30, 20)};
	}
}
```

- передавать значение из JS (`style.setProperty('--tabsNavigationActiveButtonWidth', …)`) или из JSX (`style={{ '--ratingViewValue': value }}`).

Чтобы перебить чужой блок, используется составной селектор `&.section { padding: 0 }`. `!important` допускается только в утилитах.

## 7. Порядок внутри селектора

```scss
.block {
	--localVar: …;                 // 1. локальные переменные

	@include flex-center;          // 2. миксины-свойства
	@include square(40px);
	                               // (пустая строка)
	position: …; z-index; inset/top…  // 3. позиционирование
	display; flex/grid-свойства; gap  // 4. раскладка
	width; height; min/max-*          // 5. размеры
	padding; margin                   // 6. отступы
	font-*; line-height; color; text-*  // 7. типографика
	background; border; border-radius; outline  // 8. оформление
	opacity; overflow; user-select; cursor; transition-*; translate/rotate/scale  // 9. прочее

	@include laptop { … }          // 10. медиа (от большего к меньшему)
	@include hover { … }           // 11. ховер
	&:active, &:disabled, &:focus-visible, &::before …  // 12. псевдоклассы и псевдоэлементы
	&:has(…) { … }
	&.is-active { … }              // 13. состояния
	&--modifier { … }              // 14. модификаторы
	&__element { … }               // 15. элементы
}
```

## 8. Типографика и глобальные стили — `globals.scss`

- `body`: `fluid-text` базового размера, `font-family`, цвет текста, фон, `display: flex; flex-direction: column` и `main { flex-grow: 1 }` (футер прижат к низу).
- `h1–h6` и классы `.h1–.h6` описываются **одинаково**. Семантика отдельно от внешнего вида: `<h2 className="section__title h3">`. Эталонная шкала Stream Vibe (desktop / mobile): h1 58/28, h2 48/24, h3 38/20, h4 24/18, h5 22/18, h6 20/16, body 18/14. Под новый дизайн значения меняются.
- `a, button, label, input, textarea, select, svg *` получают `transition-duration: var(--transition-duration)` глобально. Поэтому в компонентах обычно **не пишется** `transition`, достаточно поменять свойство в ховере. Для `div` и псевдоэлементов `transition-duration` указывается явно.
- `a[class]` — без подчёркивания. Ховер ссылок окрашивает их в акцентный цвет.
- `:focus-visible` — общий `@include focus-visible`. Если внутри карточки обводка обрезается, используется `outline-offset: -2px`.
- Если в проекте есть Swiper: `:has(.swiper) { min-width: 0; }`, иначе flex/grid-родитель растянет слайдер.

## 9. Контейнер и ритм

- `.container`: `max-width: calc(var(--container-width) + padding * 2)`, `padding-inline: var(--container-padding-x)`.
- Каждая секция получает `padding-block: var(--section-padding-y)`. Отступы между секциями отдельно не задаются.
- `Content` (`<main>`) задаёт верхний отступ под шапку. Если шапка лежит поверх hero, его убирает модификатор `--reset-padding-top`.

## 10. Современный CSS, принятый в проекте

Разрешено и используется: `:has()`, `inset`, `translate` / `rotate` / `scale` отдельными свойствами, `@starting-style` + `transition-behavior: allow-discrete` (появление `<dialog>`), `animation-timeline: scroll()` с фолбэком через `@supports not`, `:user-invalid`, `<details name>` для эксклюзивного аккордеона, анимация высоты через `grid-template-rows: 0fr → 1fr`, `clip-path`, CSS-счётчики, `column-count`.

## 11. Чего не делать

- Не задавать фиксированные `height` текстовым блокам. Высоту фиксировать только у контролов (полей, кнопок-иконок) и медиа.
- Не использовать `px` в `font-size`, если размер шрифта меняется между макетами.
- Не писать `margin` между соседями, если можно задать `gap` у родителя. Раскладка делается через flex/grid + `gap`.
- Не использовать id-селекторы и стили по тегам внутри компонентов. Исключения: `svg` внутри `.icon` и `Table.scss`, где теги являются частью компонента.
