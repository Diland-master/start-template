# 04. JSX-компоненты и секции

## 1. Шаблон компонента

```jsx
import './Badge.scss'
import classNames from 'classnames'
import Icon from '@/components/Icon'

const Badge = (props) => {
	const {
		className,
		/**
		 * '' (default) | 'accent'
		 */
		mode = '',
		isBig = false,
		iconName,
		children,
	} = props

	return (
		<div
			className={classNames(className, 'badge', {
				[`badge--${mode}`]: mode,
				'badge--big': isBig,
			})}
		>
			{iconName && <Icon className="badge__icon" name={iconName} />}
			<span>{children}</span>
		</div>
	)
}

export default Badge
```

Правила:
1. Компонент — **стрелочная функция** `const Name = (props) => {}` и `export default Name` в конце. Для страниц и `global.jsx` допускается `export default () => {}`.
2. Импорты идут в порядке: свой `.scss` → внешние библиотеки (`classnames`, `minista`) → проектные через `@/`.
3. Пропсы деструктурируются **внутри** тела функции из `props`, каждый с новой строки, со значениями по умолчанию.
4. Перечислимые пропсы описываются JSDoc-комментарием с вариантами: `/** '' (default) | 'transparent' | 'black-10' */`.
5. Названия пропсов:
   - `className` — внешний микс, есть у каждого компонента;
   - `mode` — визуальный вариант, превращается в модификатор `--${mode}`;
   - `is*` / `has*` — булевы (`isLabelHidden`, `hasFillIcon`, `isActionsHiddenOnMobile`);
   - `*Id` — id для связи с ARIA (`titleId`, `navigationTargetElementId`);
   - `extraAttrs` — объект произвольных атрибутов, разворачивается на корень (сюда кладутся `data-js-*`);
   - `children` — содержимое, `render*` — функция-слот (`renderBefore={(cls) => <Select … />}`);
   - `TitleTag = 'h2'` — когда уровень заголовка зависит от места использования.
6. Условный рендер записывается как `{value && (…)}`, списки — как `{items.map((item, index) => (<X {...item} key={index} />))}`.
7. Модификаторы собираются только через `classNames` с объектом `{ 'block--mod': условие }`.
8. Логика вне разметки (id, вычисления) описывается константами над `return`.

## 2. Полиморфные компоненты

`Button` рендерит `<a>`, если передан `href`, иначе `<button type="button">`:

```jsx
const isLink = href !== undefined
const Component = isLink ? 'a' : 'button'
```

Кнопка-иконка без текста получает `isLabelHidden`: `label` уходит в `title` и `aria-label`.

## 3. Секции

Если блок выглядит как «заголовок + описание + действия справа + контент», он оборачивается в `layouts/Section`:

```jsx
<Section
	title="Frequently Asked Questions"
	titleId="questions-title"
	description="Got questions? We've got answers!"
	actions={<Button label="Ask a Question" href="/support" />}
	isActionsHiddenOnMobile
>
	{/* контент */}
</Section>
```

Нестандартная секция (Hero, Support, Banner) пишется вручную по каркасу:

```jsx
const Support = () => {
	const titleId = 'support-title'

	return (
		<section className="support container" aria-labelledby={titleId}>
			<h2 className="support__title" id={titleId}>…</h2>
			…
		</section>
	)
}
```

- Корень `<section>` с `aria-labelledby` указывает на id заголовка.
- `container` вешается на корень или на `__inner`. Если фон секции на всю ширину, `container` ставится на внутренний элемент.
- Если в дизайне у блока нет видимого заголовка, всё равно добавляется `<h2 className="visually-hidden" id={titleId}>`.

## 4. Данные

- Контент, который повторяется (карточки, пункты меню, FAQ), описывается **массивом объектов**, разметка строится через `map`.
- Короткий массив объявляется внутри компонента. Длинный или общий для нескольких секций выносится в соседний файл `xxxItems.js` / `xxxGroups.js` с `export default`.
- Поля объекта совпадают с пропсами карточки, поэтому карточка рендерится через `{...item}`.
- Пути к растровым картинкам в данных записываются строкой от корня: `'/src/assets/images/movies/1.jpg'`. Их обработает `<Image>` из minista.

## 5. Связь секций, навигации и модулей

Если навигация слайдера или табов в дизайне стоит **в шапке секции**, а не рядом с контентом, она рендерится отдельно в `actions` и связывается по id:

```jsx
const sliderNavigationId = 'categories-slider-navigation'

<Section actions={<SliderNavigation mode="tile" id={sliderNavigationId} />} …>
	<Slider navigationTargetElementId={sliderNavigationId}>…</Slider>
</Section>
```

С табами то же самое: `<TabsNavigation id=… />` + `<Tabs navigationTargetElementId=… />`.

## 6. Формы

- `Field` (input/textarea), `Checkbox`, `Select` берутся из библиотеки. Если `id` не передан, он вычисляется из `label` через `getIdFromTitle`.
- Обязательное поле: `isRequired` → нативный `required` + звёздочка `aria-hidden`.
- Валидация нативная: `type="email"`, `required`, стили `:user-invalid`. Маска задаётся через `mask="(000) 000-00-00"` → `data-js-input-mask`.
- Сетка формы — grid, ячейки `__form-cell`, широкая ячейка — модификатор `--wide` (`grid-column: -1 / 1`).

## 7. Шапка и меню

Образец лежит в `_library/layouts/Header`:
- меню — `<nav>` с `<ul>`. Активный пункт определяется сравнением `href === url` (`url` приходит из `global.jsx`) и получает класс `is-active`;
- на tablet меню и действия переносятся в `<dialog>`, его открывает `BurgerButton` через модуль `OverlayMenu`. На desktop у `<dialog>` стоит `display: contents`;
- поведение при скролле — `animation-timeline: scroll()` с фолбэком;
- шапка поверх hero: флаг страницы `isHeaderFixed` → `Header isFixed` + `Content isResetPaddingTop`.
