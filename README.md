# Документация для приложения Todo

Этот проект представляет собой простое приложение для управления задачами (Todo), демонстрирующее использование архитектурного шаблона Model-View-Presenter (MVP). Приложение состоит из различных компонентов, отвечающих за разные задачи, такие как рендеринг форм, обработка взаимодействий пользователя и управление данными приложения.

Стек: HTML, SCSS, TS, Webpack

Структура проекта:

-   src/ — исходные файлы проекта
-   src/components/ — папка с JS компонентами

Важные файлы:

-   src/pages/index.html — HTML-файл главной страницы
-   src/types/index.ts — файл с типами
-   src/index.ts — точка входа приложения
-   src/styles/styles.scss — корневой файл стилей

## Установка и запуск

Для установки и запуска проекта необходимо выполнить команды

```
npm install
npm run start
```

или

```
yarn
yarn start
```

## Сборка

```
npm run build
```

или

```
yarn build
```

### Компоненты (слой View)

#### Форма (Form)

Компонент Form отвечает за отображение и обработку ввода задач в Todo.

**Интерфейс: IForm**

```
export interface IForm extends IEvents {
	buttonText: string;
	placeholder: string;
	render(): HTMLFormElement;
	setValue(data: string): void;
	getValue(): string;
	clearValue(): void;
}
```

**Класс: Form**

```
export class Form extends EventEmitter implements IForm {
	// Свойства класса и конструктор
	constructor(formTemplate: HTMLTemplateElement) {
		super();
		this.formElement = formTemplate.content.querySelector('.todos__form').cloneNode(true) as HTMLFormElement;
		this.inputField = this.formElement.querySelector('.todo-form__input');
		this.submitButton = this.formElement.querySelector('.todo-form__submit-btn');
		this.formElement.addEventListener('submit', (evt) => {
			evt.preventDefault();
			this.emit('submit', { value: this.inputField.value });
		});
	}

	// Методы класса
	render() {
		return this.formElement;
	}

	setValue(data: string) {
		this.inputField.value = data;
	}

	getValue() {
		return this.inputField.value;
	}

	clearValue() {
		this.formElement.reset();
	}

	set buttonText(data: string) {
		this.submitButton.textContent = data;
	}

	set placeholder(data: string) {
		this.inputField.placeholder = data;
	}
}

```

#### Элемент (Item)

Компонент Item отвечает за отображение и управление отдельными задачами Todo.

**Интерфейс: IViewItem**

```
export interface IViewItem extends IEvents {
	id: string;
	name: string;
	render(item: IItem): HTMLElement;
}

```

**Класс: Item**

```
export class Item extends EventEmitter implements IViewItem {
	// Свойства класса и конструктор
	constructor(template: HTMLTemplateElement) {
		super();
		this.itemElement = template.content.querySelector('.todo-item').cloneNode(true) as HTMLElement;
		this.title = this.itemElement.querySelector('.todo-item__text');
		this.deleteButton = this.itemElement.querySelector('.todo-item__del');
		this.copyButton = this.itemElement.querySelector('.todo-item__copy');
		this.editButton = this.itemElement.querySelector('.todo-item__edit');

		this.deleteButton.addEventListener('click', () => this.emit('delete', { id: this._id }));
		this.copyButton.addEventListener('click', () => this.emit('copy', { id: this._id }));
		this.editButton.addEventListener('click', () => this.emit('edit', { id: this._id }));
	}

	// Методы класса
	set id(value: string) {
		this._id = value;
	}

	get id(): string {
		return this._id || '';
	}

	set name(value: string) {
		this.title.textContent = value;
	}

	get name(): string {
		return this.title.textContent || '';
	}

	render(item: IItem) {
		this.name = item.name;
		this.id = item.id;
		return this.itemElement;
	}
}

```

#### Страница (Page)

Компонент Page отвечает за управление структурой и компонентами Todo приложения.

**Интерфейс: IPage**

```
export interface IPage {
	formContainer: HTMLElement;
	todoContainer: HTMLElement[];
}
```

**Класс: Page**

```
export class Page implements IPage {
	// Свойства класса и конструктор
	constructor(protected container: HTMLElement) {
		this._formContainer = this.container.querySelector('.todo-form-container');
		this._todoContainer = this.container.querySelector('.todos__list');
	}

	// Методы класса
	set todoContainer(items: HTMLElement[]) {
		this._todoContainer.replaceChildren(...items);
	}

	set formContainer(formElement: HTMLFormElement | null) {
		if (formElement) {
			this._formContainer.replaceChildren(formElement);
		} else {
			this._formContainer.innerHTML = '';
		}
	}
}
```

#### Всплывающее окно (Popup)

Компонент Popup отвечает за отображение модальных всплывающих окон в приложении.

**Интерфейс: IPopup**

```
export interface IPopup {
	content: HTMLElement;
	open(): void;
	close(): void;
}
```

**Класс: Popup**

```
export class Popup implements IPopup {
	// Свойства класса и конструктор
	constructor(protected container: HTMLElement) {
		this.closeButton = container.querySelector('.popup__close');
		this._content = container.querySelector('.popup__content');

		this.closeButton.addEventListener('click', this.close.bind(this));
		this.container.addEventListener('click', this.close.bind(this));
		this.container.querySelector('.popup__container').addEventListener('click', (event) => event.stopPropagation());
	}

	// Методы класса
	set content(value: HTMLElement) {
		this._content.replaceChildren(value);
	}

	open() {
		this.container.classList.add('popup_is-opened');
	}

	close() {
		this.container.classList.remove('popup_is-opened');
		this.content = null;
	}
}
```

### Модель данных (слой Модели))

Модель ToDoModel отвечает за управление данными приложения Todo.

**Интерфейс: IToDoModel**

```
export interface IToDoModel extends EventEmitter {
	items: IItem[];
	addItem(data: string): IItem;
	removeItem(id: string): void;
	editItem(id: string, name: string): void;
	getItem(id: string): IItem | undefined;
}
```

**Класс: ToDoModel**

```
export class ToDoModel extends EventEmitter implements IToDoModel {
	protected _items: IItem[];

	constructor() {
		super();
		this._items = [];
	}

	set items(data: IItem[]) {
		this._items = data;
		this.emit('changed');
	}

	get items() {
		return this._items;
	}

	addItem(data: string) {
		const uniqueId: number = Math.max(...this._items.map((item) => Number(item.id))) + 1;
		const newItem: IItem = { id: String(uniqueId), name: data };
		this._items.push(newItem);
		this.emit('changed');
		return newItem;
	}

	removeItem(id: string) {
		this._items = this.items.filter((item) => item.id !== id);
		this.emit('changed');
	}

	editItem(id: string, name: string) {
		const editItem = this._items.find((item) => item.id === id);
		if (editItem) {
			editItem.name = name;
			this.emit('changed');
		}
	}

	getItem(id: string) {
		return this._items.find((item) => item.id === id);
	}
}
```

### Слой Презентер (Presenter)

Презентер ItemPresenter отвечает за связь между моделью данных (ToDoModel) и представлением (визуальным интерфейсом) приложения Todo.

**Класс: ItemPresenter**

```
export class ItemPresenter {
	protected itemTemplate: HTMLTemplateElement;
	protected formTemplate: HTMLTemplateElement;
	protected todoForm: IForm;
	protected todoEditForm: IForm;
	protected handleSubmitEditForm: (data: { value: string }) => void;

	constructor(
		protected model: IToDoModel,
		protected formConstructor: IFormConstructor,
		protected viewPageContainer: IPage,
		protected viewItemConstructor: IViewItemConstructor,
		protected modal: IPopup,
	) {
		this.itemTemplate = document.querySelector('#todo-item-template') as HTMLTemplateElement;
		this.formTemplate = document.querySelector('#todo-form-template') as HTMLTemplateElement;
	}

	init() {
		this.todoForm = new this.formConstructor(this.formTemplate);
		this.todoForm.buttonText = 'Добавить';
		this.todoForm.placeholder = 'Следующее дело';
		this.viewPageContainer.formContainer = this.todoForm.render();
		this.todoEditForm = new this.formConstructor(this.formTemplate);
		this.todoEditForm.buttonText = 'Изменить';
		this.todoEditForm.placeholder = 'Новое название';

		this.model.on('changed', () => {
			this.renderView();
		});

		this.todoForm.on('submit', this.handleSubmitForm.bind(this));
		this.todoEditForm.on('submit', (data: { value: string }) =>
			this.handleSubmitEditForm(data),
		);
	}

	handleSubmitForm(data: { value: string }) {
		this.model.addItem(data.value);
		this.todoForm.clearValue();
	}

	handleDeleteItem(item: { id: string }) {
		this.model.removeItem(item.id);
	}

	handleCopyItem(item: { id: string }) {
		const copiedItem = this.model.getItem(item.id);
		this.model.addItem(copiedItem.name);
	}

	handleEditItem(item: { id: string }) {
		const editItem = this.model.getItem(item.id);
		this.todoEditForm.setValue(editItem.name);
		this.modal.content = this.todoEditForm.render();
		this.handleSubmitEditForm = (data: { value: string }) => {
			this.model.editItem(item.id, data.value);
			this.todoEditForm.clearValue();
			this.modal.close();
		};
		this.modal.open();
	}

	renderView() {
		const itemList = this.model.items
			.map((item: IItem) => {
				const todoItem = new this.viewItemConstructor(this.itemTemplate);
				todoItem.on('copy', this.handleCopyItem.bind(this));
				todoItem.on('delete', this.handleDeleteItem.bind(this));
				todoItem.on('edit', this.handleEditItem.bind(this));
				const itemElement = todoItem.render(item);
				return itemElement;
			})
			.reverse();

		this.viewPageContainer.todoContainer = itemList;
	}
}
```

### Класс EventEmitter

Класс для управления событиями и их подписчиками. Реализует паттерн _«Наблюдатель»_.

**Основные типы и интерфейс**

```
type EventName = string | RegExp;
type Subscriber = Function;
type EmitterEvent = {
	eventName: string;
	data: unknown;
};
```

-   EventName: Тип данных для имени события, который может быть строкой или регулярным выражением.
-   Subscriber: Тип данных для функции-подписчика на событие.
-   EmitterEvent: Интерфейс для объекта, содержащего информацию о событии.

**Интерфейс IEvents**

```
export interface IEvents {
	on<T extends object>(event: EventName, callback: (data: T) => void): void;
	emit<T extends object>(event: string, data?: T): void;
	trigger<T extends object>(event: string, context?: Partial<T>): (data: T) => void;
}
```

-   on: Метод для подписки на событие. Принимает имя события и коллбэк-функцию для обработки данных.
-   emit: Метод для генерации события с данными.
-   trigger: Метод для создания коллбэка-триггера, который генерирует событие при вызове.

**Класс EventEmitter**

```
export class EventEmitter implements IEvents {
	_events: Map<EventName, Set<Subscriber>>;

	constructor() {
		this._events = new Map<EventName, Set<Subscriber>>();
	}

	on<T extends object>(eventName: EventName, callback: (event: T) => void) {
		if (!this._events.has(eventName)) {
			this._events.set(eventName, new Set<Subscriber>());
		}
		this._events.get(eventName)?.add(callback);
	}

	off(eventName: EventName, callback: Subscriber) {
		if (this._events.has(eventName)) {
			this._events.get(eventName)!.delete(callback);
			if (this._events.get(eventName)?.size === 0) {
				this._events.delete(eventName);
			}
		}
	}

	emit<T extends object>(eventName: string, data?: T) {
		this._events.forEach((subscribers, name) => {
			if (name === '*') {
				subscribers.forEach((callback) =>
					callback({
						eventName,
						data,
					})
				);
			}
			if (
				(name instanceof RegExp && name.test(eventName)) ||
				name === eventName
			) {
				subscribers.forEach((callback) => callback(data));
			}
		});
	}

	onAll(callback: (event: EmitterEvent) => void) {
		this.on('*', callback);
	}

	offAll() {
		this._events = new Map<string, Set<Subscriber>>();
	}

	trigger<T extends object>(eventName: string, context?: Partial<T>) {
		return (event: object = {}) => {
			this.emit(eventName, {
				...(event || {}),
				...(context || {}),
			});
		};
	}
}
```

-   constructor: Инициализирует объект \_events как Map, где ключами являются имена событий (EventName), а значениями — множества функций-подписчиков (Set<Subscriber>).
-   on: Добавляет функцию-подписчик (callback) на событие (eventName).
-   off: Удаляет функцию-подписчик (callback) с события (eventName).
-   emit: Генерирует событие (eventName) с передачей данных (data) всем подписчикам, зарегистрированным на это событие.
-   onAll: Добавляет функцию-подписчик на все события.
-   offAll: Удаляет все события и их подписчиков.
-   trigger: Возвращает функцию-триггер, которая при вызове генерирует указанное событие с переданными данными.
