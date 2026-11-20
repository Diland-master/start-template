# Библиотека образцов `_library/`

Проверенные компоненты, layout-блоки и JS-модули из проекта Stream Vibe (`_stream-vibe-9.1`), переведённые на табы. Папка **не участвует в сборке**: это справочник.

## Как пользоваться

1. Найти компонент в таблице ниже.
2. Скопировать папку в такое же место `src/` проекта (`_library/components/Button` → `src/components/Button`) **вместе с зависимостями** из колонки «Зависит от».
3. Скопировать нужные ассеты (`_library/assets/...` → `src/assets/...`).
4. **Перекрасить**: заменить токены Stream Vibe (колонка «Токены») на токены нового дизайна, подогнать `fluid()`-значения, радиусы и отступы под макет.
5. Если в модуле есть JS, скопировать модуль из `_library/modules` (+ `generic/BaseComponent.js`, `utils/*`, `constants/MatchMedia.js` при необходимости) и подключить его в `src/main.js`.
6. Не нужные по дизайну варианты (`mode`) и пропсы **удалить**: в проекте остаётся только то, что используется.

## Компоненты (`components/`)

| Компонент | Назначение и ключевые пропсы | Зависит от | Токены Stream Vibe |
|---|---|---|---|
| `Icon` | SVG из спрайта. `name`, `hasFill`, `ariaLabel`. Размер `--iconSize` | `assets/icons/*.svg` | — |
| `Button` | Кнопка или ссылка (`href`). `mode: '' \| transparent \| black-10 \| black-08 \| black-06`, `label`, `isLabelHidden`, `iconName`, `iconPosition: before \| after`, `hasFillIcon`, `type`, `target`, `extraAttrs`. Квадратная кнопка-иконка через `:has(.button__icon:only-child)` | Icon | red-45, red-60, black-06/08/10, gray-60, white; radius s/xs |
| `Logo` | Ссылка на главную с `<img>`. `loading` | `assets/images/logo.svg` (свой) | — |
| `BurgerButton` | Анимированный бургер (SVG-штрихи), состояние `is-active`. `extraAttrs` | — | black-10, black-15, red-60, white |
| `Badge` | Плашка/чип. `mode: '' \| accent`, `isBig`, `iconName`, `hasFillIcon`, `iconAriaLabel` | Icon | black-08, gray-60, red-45 |
| `Tags` | Список тегов. `items: string[]` | — | black-08, white |
| `Grid` | Сетка `<ul>`. `columns: 1 \| 2 \| 3`. Адаптив 3 → 2 → 1, последний нечётный элемент растягивается | — | — |
| `Field` | Поле `input` / `textarea`. `label`, `type`, `placeholder`, `isRequired`, `inputMode`, `mask`, `renderBefore(cls)` | utils/getIdFromTitle, модуль InputMask (для `mask`) | black-08, gray-60, red-45, white, `--field-padding-x` |
| `Checkbox` | Кастомный чекбокс на нативном `input`. `label`, `isRequired` | utils/getIdFromTitle | black-06, red-45, white |
| `Select` | Доступный кастомный селект. На mobile показывается нативный. `label`, `isLabelHidden`, `options: {value, isSelected}[]`, `buttonClassName` | utils/getIdFromTitle, модуль SelectCollection, constants/MatchMedia, `assets/images/select/arrow-down_gray.svg` | black-06, white |
| `Slider` | Обёртка Swiper. `sliderParams`, `navigationTargetElementId`, `hasScrollbarOnMobile`, `navigationPosition: '' \| abs-bottom`, `navigationMode`, `isNavigationHiddenMobile`, `isBeyondTheViewportOnMobileS` | `components/SliderNavigation`, Button, модуль SliderCollection, `swiper` | black-06, black-20, red-45 |
| `Slider/components/SliderNavigation` | Стрелки + пагинация. `mode: '' \| tile \| rounded`, `position`, `justifyContent`, `hasPagination`, `buttonMode`, `id` | Button, иконки `arrow-left` / `arrow-right` | — |
| `Tabs` | ARIA-табы. `title`, `items: {title, isActive, children}[]`, `navigationTargetElementId`, `isEnableOnlyOnMobile` (на desktop показываются все панели) | `components/TabsNavigation`, `utils/getTabsElementsIdsFromTitle`, модуль TabsCollection, BaseComponent | black-12, white |
| `Accordion` | Аккордеон на `<details>` без JS. `title`, `titleLevelClassName`, `subtitle`, `id`, `name` (эксклюзивная группа), `isOpen`, `isArrowButton` | Icon (`arrow-down`) | black-08, gray-60, red-60, white |
| `AccordionGroup` | Список аккордеонов. `columns: 1 \| 2`, `mode: '' \| dark`, `isOrderedList` (нумерация 01, 02…) | — | black-06, black-12, red-45 |
| `Table` | Таблица. `headCells: {children, width}[]`, `rows: {cells}[]` | — | black-06, black-15, white |
| `RatingView` | Звёзды с дробным значением через `clip-path`. `value`, `label` | `assets/images/rating/*` | — |
| `Socials` | Список соц-кнопок. `links: {label, iconName}[]` | Button, иконки соцсетей | — |
| `DeviceCard` | Пример карточки «иконка + заголовок + текст» с градиентным фоном | — | black-06, black-08, black-12 |

## Layout (`layouts/`)

| Блок | Что внутри |
|---|---|
| `Header` | Сетка `1fr auto 1fr`, меню с активным пунктом по `url`, кнопки-иконки, `<dialog>` с бургер-меню на tablet, фон при скролле через `animation-timeline: scroll()`, модификатор `is-fixed`. Нужны Logo, Button, BurgerButton, модуль OverlayMenu. |
| `Footer` | Колонки меню (grid 6 → 3 → 2 через `--footerMenuColumns`), соцсети, нижняя строка с копирайтом и ссылками-разделителями. |
| `Content` | `<main>` с верхним отступом под шапку, модификатор `--reset-padding-top`. |
| `Section` | Универсальная секция: заголовок (`h2.h3`), описание, `actions` справа (на mobile переносятся вниз или скрываются через `isActionsHiddenOnMobile`), `children`. |

## Примеры секций (`sections/`)

Даны как образцы композиции, а не для прямого копирования (картинки к ним не приложены).

| Секция | Что показывает |
|---|---|
| `Hero` | Первый экран на `100vh`, фон с двумя градиентами + картинка, центральная кнопка-плей |
| `Categories` | `Section` + `Slider`, навигация вынесена в `actions` по id |
| `Devices` | `Section` + `Grid columns={3}` + карточки из массива |
| `Plans` | `Section` + `TabsNavigation` в `actions` + `Tabs` с `Grid` внутри, данные в `planGroups.js` |
| `Questions` | `Section` + `AccordionGroup columns={2}` + `Accordion name="questions"` |
| `Support` | Двухколоночная секция с формой: Field, Select-префикс телефона, маска, Checkbox, submit |
| `Banner` | CTA-баннер с градиентом поверх фона и отдельным мобильным фоном |

## JS (`modules/`, `utils/`, `constants/`)

См. [`docs/05-js-modules.md`](../docs/05-js-modules.md). `generic/BaseComponent.js` — база с Proxy-состоянием. Утилиты: `getParams`, `getAttrNameFromSelector`, `getIdFromTitle`, `pxToRem`. Константы: `MatchMedia`.

Если используется Slider, в `globals.scss` проекта нужно добавить:

```scss
:has(.swiper) {
	min-width: 0;
}
```
