# 02. Архитектура проекта

## Стек

| Что | Зачем |
|---|---|
| `minista` 3 | Статический генератор: JSX → HTML на этапе сборки. Страницы = файлы в `src/pages`. |
| `sass` | SCSS. Хелперы `@/styles/helpers` автоматически подключаются в **каждый** `.scss` (`additionalData` в `minista.config.js`), импортировать их вручную не нужно. |
| `postcss-pxtorem` | Только в `build`: все `px` → `rem` (включая медиазапросы). В исходниках пишем **px**. |
| `classnames` | Сборка `className` с модификаторами. |
| `swiper` | Слайдеры (через модуль `SliderCollection`). |
| `imask` | Маски полей (через модуль `InputMaskCollection`). |
| `@a1rth/css-normalize` | Сброс стилей. |

Необязательно: `postcss-preset-env` (в Stream Vibe подключён в prod-сборке после pxtorem) — добавлять, если нужен автопрефиксинг и полифилы современного CSS.

Алиас `@/` → `src/`. Импорты всегда через `@/`, кроме импорта стилей и подкомпонентов из своей же папки (`./Button.scss`, `./components/SliderNavigation`).

## Папки

```
src/
├── pages/            # страницы: index.jsx → /, support.jsx → /support
├── global.jsx        # общая обёртка: <Head>, Header, Content, Footer
├── main.js           # точка входа JS: new XxxCollection() для каждого модуля
├── layouts/          # каркас: Header, Footer, Content, Section
├── sections/         # крупные блоки страниц (Hero, Plans, Questions…)
│   └── Plans/
│       ├── Plans.jsx
│       ├── Plans.scss       # только если нужны свои стили
│       ├── planGroups.js    # данные секции
│       └── index.js
├── components/       # переиспользуемые UI-компоненты
│   └── Slider/
│       ├── Slider.jsx
│       ├── Slider.scss
│       ├── index.js
│       └── components/SliderNavigation/   # подкомпоненты — внутри родителя
├── modules/          # ванильный JS: классы на data-js-*
│   └── generic/BaseComponent.js
├── utils/            # чистые функции (getParams, pxToRem, getIdFromTitle)
├── constants/        # константы (site.js, MatchMedia.js)
├── styles/
│   ├── index.js      # порядок подключения: normalize → fonts → variables → utils → globals
│   ├── fonts.scss
│   ├── variables.scss
│   ├── utils.scss    # .container, .visually-hidden, .visible-*/.hidden-*
│   ├── globals.scss  # body, заголовки, ссылки, focus
│   └── helpers/      # constants, functions (fluid), media, mixins
└── assets/
    ├── fonts/        # .woff2
    ├── icons/        # .svg → спрайт для <Icon name="..." />
    ├── images/       # картинки, сгруппированные по секциям: images/hero/bg.png
    └── videos/
```

## Именование

| Сущность | Правило | Пример |
|---|---|---|
| Папка и файлы компонента | PascalCase | `MovieCard/MovieCard.jsx` |
| `index.js` | только реэкспорт | `import X from './X'` + `export default X` |
| CSS-блок | kebab-case от имени компонента | `MovieCard` → `.movie-card` |
| Файл данных | camelCase, множественное число + `Items` / `Groups` | `categoryItems.js`, `planGroups.js` |
| Модуль JS | `XxxCollection.js` (много экземпляров) или `Xxx.js` (единственный на странице) | `TabsCollection.js`, `OverlayMenu.js` |
| Иконка | kebab-case, имя файла = `name` у `<Icon>` | `arrow-left.svg` → `<Icon name="arrow-left" />` |
| Картинки | папка по секции, файлы нумеруются | `images/categories/action/1.jpg` |

Сниппеты для VS Code (расширение «Create Component Folder») лежат в `.vscode/cch-template/`: «JSX Component Folder» и «… with styles».

## Страницы

```jsx
import Hero from '@/sections/Hero'
import Questions from '@/sections/Questions'

export const metadata = {
	title: 'Home',          // попадёт в <title>: "Site Name | Home"
	isHeaderFixed: true,    // пример своего флага: доступен в global.jsx через props
}

export default () => {
	return (
		<>
			<Hero />
			<Questions />
		</>
	)
}
```

- Всё, что лежит в `metadata`, а также `url` текущей страницы, приходит пропсами в `global.jsx`. Так передаются флаги в layout: `url` нужен для активного пункта меню, `isHeaderFixed` — для прозрачной шапки поверх hero.
- Страница содержит **только секции**, без собственной разметки и стилей.
- Блок, который повторяется на всех страницах перед футером (CTA-баннер), подключается в `global.jsx` внутри `<Content>`.
