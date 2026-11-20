# 05. JS-модули

JSX выполняется только на сборке, поэтому весь интерактив пишется **ванильным JS** в `src/modules` и подключается в `src/main.js`.

## 1. Связь разметки и JS — только через `data-js-*`

- Классы отвечают за стили, `data-js-*` — за поведение. JS **никогда** не ищет элементы по классам.
- Имена атрибутов: `data-js-<модуль>` на корне, `data-js-<модуль>-<элемент>` на частях: `data-js-slider`, `data-js-slider-swiper`, `data-js-slider-next-button`.
- В JSX: `data-js-tabs-content=""` или через `extraAttrs={{ 'data-js-slider-previous-button': '' }}`.
- Параметры передаются **JSON-строкой в атрибуте корня** и читаются через `getParams`:

```jsx
<div data-js-slider={JSON.stringify({ sliderParams, navigationTargetElementId })}>
```

```js
this.params = getParams(this.rootElement, this.selectors.root)
```

## 2. Структура модуля

```js
import BaseComponent from '@/modules/generic/BaseComponent'
import getParams from '@/utils/getParams'

const rootSelector = '[data-js-tabs]'

class Tabs extends BaseComponent {
	selectors = {
		root: rootSelector,
		button: '[data-js-tabs-button]',
		content: '[data-js-tabs-content]',
	}

	stateClasses = {
		isActive: 'is-active',
	}

	stateCSSVariables = {
		activeButtonWidth: '--tabsNavigationActiveButtonWidth',
	}

	constructor(rootElement) {
		super()
		this.rootElement = rootElement
		this.buttonElements = [...this.rootElement.querySelectorAll(this.selectors.button)]
		this.state = this.getProxyState({ activeTabIndex: 0 })
		this.bindEvents()
	}

	updateUI() {
		// перерисовка по this.state: классы is-*, aria-*, tabIndex, CSS-переменные
	}

	onButtonClick = (index) => {
		this.state.activeTabIndex = index
	}

	bindEvents() {
		this.buttonElements.forEach((buttonElement, index) => {
			buttonElement.addEventListener('click', () => this.onButtonClick(index))
		})
	}
}

class TabsCollection {
	constructor() {
		this.init()
	}

	init() {
		document.querySelectorAll(rootSelector).forEach((element) => {
			new Tabs(element)
		})
	}
}

export default TabsCollection
```

Правила:
1. В классе есть поля `selectors`, `stateClasses`, при необходимости `stateCSSVariables` и `a11yAttributes`. Строки селекторов и классов в методах напрямую не пишутся.
2. DOM-элементы хранятся в полях с суффиксом `Element` / `Elements`: `this.rootElement`, `this.buttonElements`.
3. Обработчики — стрелочные поля класса `onXxxClick = () => {}`, навешиваются в `bindEvents()`.
4. Если у компонента есть состояние, он наследуется от `BaseComponent`: `this.state = this.getProxyState({...})`. Любое изменение `this.state.x` вызывает `updateUI()`. UI меняется **только** в `updateUI()`.
5. Компонент, который встречается много раз, оборачивается в класс `XxxCollection`, а он делает `querySelectorAll(rootSelector)`. Компонент, который на странице один (`OverlayMenu`), создаётся без коллекции, но с проверкой, что корень найден.
6. Клавиатура описывается картой `{ ArrowLeft: this.previousTab, … }[event.code]`. Если действие найдено, вызывается `event.preventDefault()`.
7. Состояние, которое нужно стилям, передаётся через CSS-переменные в rem: `` `${pxToRem(width)}rem` ``.
8. Подключение в `main.js`:

```js
import TabsCollection from '@/modules/TabsCollection'

new TabsCollection()
```

## 3. Готовые модули (в `_library/modules`)

| Модуль | Разметка | Что делает |
|---|---|---|
| `OverlayMenu` | `data-js-overlay-menu`, `-dialog`, `-burger-button` | Бургер-меню на `<dialog>`, блокировка скролла через `html.is-lock` |
| `SliderCollection` | `data-js-slider='{"sliderParams":…}'`, `-swiper`, `-navigation`, `-previous-button`, `-next-button`, `-pagination`, `-scrollbar` | Swiper с Navigation, Pagination, Scrollbar; навигация может быть вынесена по id |
| `TabsCollection` | `data-js-tabs`, `-navigation`, `-button`, `-content` | ARIA-табы, стрелки, Home/End, «ползунок» активного таба через CSS-переменные |
| `SelectCollection` | `data-js-select`, `-original-control`, `-button`, `-dropdown`, `-option` | Кастомный селект (combobox/listbox). На mobile показывается нативный `<select>` |
| `InputMaskCollection` | `data-js-input-mask="(000) 000-00-00"` | Маска через IMask |
| `VideoPlayerCollection` | `data-js-video-player`, `-video`, `-play-button` | Запуск видео в fullscreen по кнопке |

Аккордеон работает **без JS**: `<details name="group">` + CSS.

Если нужного модуля нет (модалка, счётчик, ленивая загрузка и т.п.), новый пишется по той же структуре.
