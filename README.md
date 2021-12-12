# TypeScript cheatsheet
## TOC
- [TypeScript cheatsheet](#typescript-cheatsheet)
  - [TOC](#toc)
  - [Types](#types)
    - [Type operations](#type-operations)
    - [Mapped types](#mapped-types)
    - [Narrowing](#narrowing)
      - [Basic type guards](#basic-type-guards)
      - [Custom guard function](#custom-guard-function)
      - [Discriminated unions](#discriminated-unions)
      - [`this` type guards](#this-type-guards)
  - [Objects and interfaces](#objects-and-interfaces)
  - [Functions](#functions)
    - [`this` type](#this-type)
    - [Generic functions](#generic-functions)
    - [Overloading](#overloading)
  - [Enums](#enums)
  - [Classes](#classes)
    - [Abstract classes](#abstract-classes)
    - [Generic classes](#generic-classes)
    - [Constructor signatures](#constructor-signatures)
  - [Extras](#extras)

## Types

```ts
// primitives - string, number, boolean, undefined, null, bigint, symbol
const hello: string = 'hello'
const halo = 'halo' // inferencing

// any
let v: any
// type-checker is 'turned off', no error
return v.toString()

// unknown
let y: unknown
if (typeof y == 'number') {
  // y is <number>
  return y.toString()
}

// never
const fails = (): never => {
  throw new Error('Error')
}

// void
const print = (value: string): void => {
  // void prevents accidental usage of return value (in contrast to any)
  console.log(value)
}

// literal types
type color = 'R' | 'G' | 'B' // type unions

// tuples
type coords = readonly [number, number?, ...boolean[]] // optional?, ...restType[]

// arrays
let primes: number[] = [2, 3, 5, 7, 13]

// type assertion
let greeting: unknown = 'Hello, world!'
let stringGreeting: string = later as string
// let stringGreeting: string = <string>later

// non-null assertion
const toString(v?: number | null) {
  v!.toString() // <number>
}

// type alias
type ID = string

type Person = {
  name: string
}

// const - infer narrowest type
const args = [8, 5] as const // array might have 0 els without const
const angle = Math.atan2(...args)

```

### Type operations

```ts
// template literal types
type World = 'world'
type Greeting = `hello ${World}`

// unions are cross products
type Dialects = 'en' | 'ar'
type Countries = 'us' | 'uk'
type Languages = `${Dialects}_${Countries}` // <en_us, en_uk, ...>
type Entities = `${Dialects | Countries}_1` // <en_1, us_1...>

// types with generics
type Maybe<T> = T | null
type OneOrMany<T> = T | T[]
type MaybeOneOrMany<T> = Maybe<OneOrMany<T>>

type Person = {
  firstName: string
}

type Citizen = {
  id: number
}

// type intersection
type Resident = Person & Citizen // <{ firstName: string, id: number }>

const jake: Resident = {
  firstName: 'Jake',
  id: 1,
}

// keyof
type K = keyof typeof jake // <'firstName' | 'id'>

type List = { [n: number]: unknown }
type Item = keyof List // <number> == <0 | 1 | ...>

// indexed access type
type FirstName = Person['firstName'] // <string>
type ResidentTypes = Resident[keyof Resident] // <string | number>
// type ResidentTypes = Resident['firstName' | 'id'] // <string | number>

// conditional types - T extends U ? V : W
type Flatten<T> = T extends any[] ? T[number] : T // Flatten<string[]> -> <string>

// infer
type GetArrayType<T> = T extends Array<infer R> ? R : T // infer R[]
type GetReturnType<T> = T extends (...args: never[]) => infer R ? R : never
type GetObjectTypes<T> = T extends { x: infer R  } ? R : unknown

// distributive law
type ToArray<T> = T extends any ? T[] : never
type StrArrOrNumArr = ToArray<string | number> // <string[] | number[]>
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never // [avoid distribution]
type StrOrNumArr = ToArrayNonDist<string | number> // <string | number>[]
```

### Mapped types
Create new types by mapping over existing types  

```ts
// '-' - remove attribute, '+' - add
type CreateMutable<T> = {
  -readonly [K in keyof T]: T[K]
}

// remove optionals
type Concrete<T> = {
  [K in keyof T]-?: T[K]
}

// nullable
type MakeNullable<T> = {
  [P in keyof T]: T[P] | null
}

// map key type
type N = string

type Remap<T> = {
  [K in keyof T as N]: T[K]
}

// rename keys
type Getters<T> = {
  // <string & K> ?
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] // <getKey: () => T[K]>
}

// remove key
type RemoveName<T> = {
  [K in keyof T as Exclude<K, 'name'>]: T[K]
}

// extract key
type ExtractId<T> = {
  [K in keyof T]: T[K] extends { id: true } ? true : false // all props with { id: true } -> true
}

interface Device {
  x: { id: true }
  y: { id: false }
}

type WithId = ExtractId<Device> // { x: true; y: false}

// use object props
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    event: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void
}
```

### Narrowing
#### Basic type guards
```ts
// typeof
function printId(id?: number | string) {
  if (id && typeof id === 'string') {
    // id<string>
    console.log(id.toUpperCase())
  } else if (id && typeof id === 'number') {
    // id<number>
  } else {
    // id<undefined>
  }
}

// equality
function add(a: string | number, b: string | boolean) {
  if (a === b) {
    // a<number>, b<number>
  }
}

// in, instanceof
interface Printer {
  pId: number
}

interface Screen {}

function print(p: Printer | Screen) {
  if ('pId' in p) {
    // p<Printer>
  }
}
```

#### Custom guard function
```ts
// type predicate using 'is'
function isString(test: any): test is string {
  return typeof test === 'string'
}

function print(value: string | undefined): void {
  if (isSting(value)) {
    // value is <string>
  }
}
```

#### Discriminated unions
```ts
interface Dog {
  kind: 'dog' // common 'singleton' property
}

interface Cat {
  kind: 'cat'
}

function animalNoise(animal: Cat | Dog): void {
  if (animal.kind == 'dog') {
    // animal: Dog
  }
}
```
#### `this` type guards
```ts
class FS {
  isDirectory(): this is Directory {
    return this instanceof Directory
  }

  // this is <Networked | FS> ?
  isNetworked(): this is Networked & this {
    return this.networked
  }
}

interface Networked {
  host: string
}

class Directory extends FS {
  children!: FS[]
}
```

## Objects and interfaces
> Literals must not specify unknown props

```ts
let card: { title: string; valid: boolean } = {
  title: 'Volunteer',
  valid: true,
}

// destructuring
let { a: aAlias = 'x', b: bAlias = 9 }: { a: string; b: number } = { a: 'a', b: 2 }

interface Author {
  readonly age: number
  firstName: string
  // optional property - check for undefined before use
  middleInitial?: string 
}

// index signature
interface List {
  readonly [index: number]: string
  length: number
}

// interface generics
interface Box<T> {
  contents: T
}

// interfaces are extensible
interface Student extends Person {
  studentId: string
}

// declaration merging
interface Student {
  module: string // { name, studentId, module }
}

// generic interface
interface Identity {
  <T>(v: T): T
}

```
Interfaces vs type aliases:
* Interface declaration with the same name are allowed, and are merged
* Type aliases can be used to rename primitives

## Functions
```ts
// optionalParam?: T
function greet(person: string, greeting?: string): string {
  return `${greeting || 'Hello'} ${person}`
}

// destructuring
function sum({ a, b }: { a: number; b: number }) {
  console.log(a + b)
}

// arrow functions
const identity = (v: unknown): unknown => v

// contextual typing - anonymous functions
const names = ["Alice", "Bob", "Eve"];
names.forEach(function (s) {
  // s is <string>
  console.log(s.toUppercase())
}
```

### `this` type
Signals the type of `this` to the type checker   
Erased at compile time 
```ts
interface User {}
interface DB {
  users: User[]
  filterUsers(filter: (this: User) => boolean): User[]
}

const myDB: DB = {
  users: []
  filterUsers() {
    /* ... */
    // this is type <User>
    return this.users
  }
}
```

### Generic functions
```ts
function map<T, U>(a: T[], fn: (v: T) => U): U[] {
  const res: U[] = []
  return res
}

// generic type constraint
function longest<T extends { length: number }>(a: T, b: T) {}

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```
### Overloading
Only overload signatures are callable (implementation signature isn't)  
Implementation signature type should general enough to *include* the overload signatures  
There must be at least two overload signatures
```ts
type OrArray<T> = T | T[]

function greet(person: string): string
function greet(persons: string[]): string[]
function greet(person: OrArray<string>): OrArray<string> {
  if (typeof person == 'string') {
    return `Hello, ${person}!`
  } else if (Array.isArray(person)) {
    return person.map((name) => `Hello, ${name}!`)
  }
}
```

## Enums
```ts
// numeric enum
enum Direction {
  Up = 1, // default is 0
  Down,
  Left,
  Right,
}

// string enum
enum Day {
  M = 'Monday',
  T = 'Tuesday',
  W = 'Wednesday',
}

// using enums
interface Task {
  date: Day
}

const task1: Task = {
  date: Day.M // 'Monday'
}

// return key, rather than value
Day[Day.M] // 'M'
```

## Classes

Member visibility - `public`, `private`, `protected`  
Member modifiers - `readonly`, `static`  
Derived classes can changed visibility of inherited members  
TypeScript uses *structural typing*  
Empty classes are supertypes of every class

```ts
interface Drives {
  driver() {}
}

class Car implements Drives {
  drive() {}
}

class Golfer implements Drives {
  drive() {}
}

let w: Car = new Golfer()

class Book {
  #hashCode: string // hard private
  private ID: number // soft private

  // parameter properties - auto initialized
  constructor(public author: string) {}
}
```

### Abstract classes
```ts
abstract class Base {
  abstract getName(): string
  printName() {}
}

class Derived extends Base {
  getName() {} // has to fulfill abstract contract
}
```

### Generic classes

```ts
class Box<T> {
  // static members cannot reference type parameter T
  contents: T

  constructor(contents: T) {
    this.contents = contents
  }

  set(value: T) {
    this.contents = value
    return this
  }

  // 'this' type - works with derived classes
  equal(other: this) {
    return other.contents === this.contents
  }
}
```
### Constructor signatures
>  Useful for defining existing APIs that define a 'new'-able function
```ts
interface Point {}

interface PointConstructor {
  new (x: number, y: number): Point
}

function instantiate(c: PointConstructor): Point {
  return new c(x, y)
}

// class types generics
function create<T>(c: { new (): T }): T {
  return new c()
}
```

## Extras

Use `declare` to specify the type for existing variables
```ts
declare function forEach<T>(arr: T[], callback: (el: T) => void): void

// DOM manipulation
const divEl = document.getElementById('div') // <HTMLElement | null>
const children = divEl.children // <HTMLCollection>
// const children = divEl.childNodes // <NodeList>

```