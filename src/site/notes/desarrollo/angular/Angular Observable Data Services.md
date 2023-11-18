---
{"dg-publish":true,"permalink":"/desarrollo/angular/angular-observable-data-services/","tags":["gardenEntry"]}
---


# [Angular Observable Data Services - Angular 12 | 11](https://coryrylan.com/blog/angular-observable-data-services)

> ## Excerpt
> A tutorial on how to use Observables and how they can improve your Angular data services and managing state in your applications.

---
This article has been updated to the latest version [Angular](https://angular.io/) 12 and tested with Angular 11. The content is likely still applicable for all Angular 2 + versions.

Angular brings many new concepts that can can improve our JavaScript applications. The first new concept to Angular is the use of Observables. Observables are a proposed feature coming to the JavaScript specification. I wont go in depth into Observables but will just cover some of the high level concepts. If you want a introduction to Observables check out my screen cast.

[Intro to RxJS Observables and Angular](https://coryrylan.com/blog/reactive-programming-with-rxjs-and-angular-ng-houston)

The rest of this post will cover more data and application state management in a Angular application.

Observables can help manage async data and a few other useful patterns. Observables are similar to Promises but with a few key differences. The first is Observables emit multiple values over time. For example a Promise once called will always return one value or one error. This is great until you have multiple values over time. Web socket/real-time based data or event handlers can emit multiple values over any given time. This is where Observables really shine. Observables are used extensively in Angular. The new HTTP service and EventEmitter system are all Observable based.
Lets look at an example where we subscribe to an Observable.

```ts
todosService.todos.subscribe(updatedTodos => {
	this.componentTodos = updatedTodos;
});
this.todos = todosService.todos;
```

In this snippet our `todos` property on our data service is an Observable. We can subscribe to this Observable in our component. Each time there is a new value emitted from our Observable Angular updates the view.

```html
<div *ngFor="let todo of todos | async">
	<button (click)="deleteTodo(todo.id)">
	x
	</button>
</div>
```

Observables are treated like arrays. Each value over time is one item in the array. This allows us to use array like methods called operators on our Observable such as `map`, `flatmap`, `reduce`, ect. In our service we will be using a special type of an Observable called a BehaviorSubject. A BehaviorSubject allows us to push and pull values to the underlying Observable. We will see how this will help us construct our service.

In Angular we use RxJS a polyfill/util library for the proposed Observables primitive in the next new version JavaScript. RxJS version 5 is a peer dependency with Angular. A slim Observable is used in Angular core. The slim Observable does not have many of the useful operators that makes RxJS so productive. The Observable in Angular is slim to keep the byte site of the library down. To use extra operators we import them like so: 
```ts
import { map } from 'rxjs/operators';
```

In our example we will have a `TodosService`. Our todos service will have basic CRUD operations and a Observable stream to subscribe to. This example we will use a REST based API but it could be converted to a real-time socket based API with little effort. Next lets take a look at a the constructor of our service.

```ts
@Injectable()export class TodoService {
	private _todos = new BehaviorSubject<Todo[]>([]);
	private baseUrl = 'https://56e05c3213da80110013eba3.mockapi.io/api';
	private dataStore: { todos: Todo[] } = { todos: [] };
	readonly todos = this._todos.asObservable(); 
	
	constructor(private http: HttpClient) {}
	
	get todos() {
		return this._todos.asObservable();  
	}
}
```

In our todo service we have a few moving parts. First we have a private data store. This is where we store our list of todos in memory. We can return this list immediately for faster rendering or when off-line. For now its simply just holds onto our list of todos. Since services in Angular are singletons we can use them to hold our data model/state we want to share. 

### [Angular Form Essentials](https://angularforms.dev/)

Forms can be complicated. Become an expert using **Angular Reactive Forms** and **RxJS**. Learn to manage **async validation**, build **accessible**, and **reusable custom inputs**. Get a jump start on building Angular Forms today!

[Get the e-Book now!](https://angularforms.dev/)

Next is our `todos` BehaviorSubject. Our BehaviorSubject can recieve and emit new Todo lists. We don't want subscribers of our service to be able to push new values to our subject without going through our CRUD methods. To prevent the data from being altered ouside the service we expose the BehaviorSubject through a public property and cast it to an Observable using the `asObservable` operator. This will allow components to receive updates but not push new values. We will see how that is done in further in the example. Note a `BehaviorSubject` is like the `Subject` class but requires a inital starting value.

Now in our public methods we can load, create, update, and remove todos. Lets start off with loading the todos.

```ts
@Injectable() export class TodoService {
	private _todos = new BehaviorSubject<Todo[]>([]);
	private baseUrl = 'https://56e05c3213da80110013eba3.mockapi.io/api';
	private dataStore: { todos: Todo[] } = { todos: [] };
	readonly todos = this._todos.asObservable(); 
	
	constructor(private http: HttpClient) {}
	
	get todos() {
		return this._todos.asObservable();
	}
	
	loadAll() {
		this.http.get(`${this.baseUrl}/todos`).subscribe(
			data => {
				this.dataStore.todos = data;
				this._todos.next(Object.assign({}, this.dataStore).todos);
			},
			error => console.log('Could not load todos.')
		);  
	}
}
```

Notice instead of these methods returning new values of our todos list they update our internal data store. We make a copy using `Object.assign()` so we pass back a new copy of todos and don't accidentally pass a reference to the original data store.

```ts
this._todos.next(Object.assign({}, this.dataStore).todos);
```

Once the data store of todos is updated we push the new list of todos with our private `_todos` BehaviorSubject. Now anytime we call one of these methods any component subscribed to our public `todos` Observable stream will get a value pushed down and always have the latest version of the data. This helps keeps our data consistent across our application.

In our component's [`ngOnInit`](https://angular.io/docs/ts/latest/tutorial/toh-pt4.html) method we subscribe to the `todos` data stream then call `load()` to load the latest into the stream. The `ngOnInit` is the ideal place for loading in data. You can read into the docs and the various reasons why this is a best practice.

```ts
ngOnInit() {
	this.todos = this.todoService.todos;
	this.singleTodo$ = this.todoService.todos.pipe(
		map(todos => todos.find(item => item.id === '1'))
	);
	this.todoService.loadAll();
	this.todoService.load('1');
}
```

We could subsequently call remove and our stream will get a new list with one less todo. So lets add the rest of the code to add CRUD operations to our todos.  
Now lets look at the todos service in its entirety.

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Todo {
	id?: any;
	createdAt?: number;
	value: string;
}

@Injectable()
export class TodoService {
	
	private _todos = new BehaviorSubject<Todo[]>([]);
	private baseUrl = 'https://56e05c3213da80110013eba3.mockapi.io/api';
	private dataStore: { todos: Todo[] } = { todos: [] };
	readonly todos = this._todos.asObservable();

	constructor(private http: HttpClient) {}

	loadAll() {
		this.http.get(`${this.baseUrl}/todos`).subscribe(
			(data) => {
				this.dataStore.todos = data;
				this._todos.next(Object.assign({}, this.dataStore).todos);
			},
			(error) => console.log('Could not load todos.')
		);
	}

	load(id: number | string) {
		this.http.get<Todo>(`${this.baseUrl}/todos/${id}`).subscribe(
			(data) => {
				let notFound = true;
				this.dataStore.todos.forEach((item, index) => {
				if (item.id === data.id) {
					this.dataStore.todos[index] = data;
					notFound = false;
				}
			});
			if (notFound) {
				this.dataStore.todos.push(data);
			}
			this._todos.next(Object.assign({}, this.dataStore).todos);
			},
			(error) => console.log('Could not load todo.')

		);
	}

	create(todo: Todo) {
		this.http
			.post<Todo>(`${this.baseUrl}/todos`, JSON.stringify(todo))
			.subscribe(
				(data) => {
					this.dataStore.todos.push(data);
					this._todos.next(Object.assign({}, this.dataStore).todos);
				},
				(error) => console.log('Could not create todo.')
			);
	}

	update(todo: Todo) {
	this.http
		.put<Todo>(`${this.baseUrl}/todos/${todo.id}`, JSON.stringify(todo))
		.subscribe(
			(data) => {
				this.dataStore.todos.forEach((t, i) => {
					if (t.id === data.id) {
						this.dataStore.todos[i] = data;
					}
				});
				this._todos.next(Object.assign({}, this.dataStore).todos);
			},
			(error) => console.log('Could not update todo.')
		);
	}

	remove(todoId: number) {
		this.http.delete(`${this.baseUrl}/todos/${todoId}`).subscribe(
			(response) => {
				this.dataStore.todos.forEach((t, i) => {
					if (t.id === todoId) {
						this.dataStore.todos.splice(i, 1);
					}
				});
				this._todos.next(Object.assign({}, this.dataStore).todos);
			},
			(error) => console.log('Could not delete todo.')
		);
	}
}
```

## Overview[](https://coryrylan.com/blog/angular-observable-data-services#overview)

This pattern can also be used in Angular 1. RxJS and Observables are not just an Angular feature. This may seem like a lot of work for a simple todo app but scale this up to a very large app and Observables can really help manage our data and application state. This pattern follows the idea of unidirectional data flow. Meaning data flow is predictable and consistently comes from one source. If you have worked with [Flux](https://facebook.github.io/flux/docs/overview.html)/[Redux](http://redux.js.org/) based architectures this may seem very familiar.

![Diagram of Angular Data flow with Observables](https://coryrylan.com/assets/images/posts/2015-11-17-angular-observable-data-services/observable-service-data-flow.svg)

Angular data flow with Observables

This pattern can ensure data is coming from one place in our application and that every component receives the latest version of that data through our data streams. Our component logic simple by just subscribing to public data streams on our data services. A link to the working demo of a Observable data service can be found below. This service conforms to a REST based backend but could easily translate to a socket based service like [Firebase](https://www.firebase.com/) without having to change any components.

Want to learn more about Observables? Check out my video tutorial, [Reactive Programming with RxJS and Angular](https://coryrylan.com/blog/reactive-programming-with-rxjs-and-angular-ng-houston).
