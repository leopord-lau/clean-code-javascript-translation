# 编写更简洁的JavaScript
[clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)的中文翻译

@[toc]





# 变量

------

## 使用有意义并且可读的变量名称

Bad:

```javascript
const yyyymmdstr = moment().format("YYYY/MM/DD");
```

Good:

```javascript
const currentDate = moment().format("YYYY/MM/DD");
```



## 为相同类型的变量使用相同的词汇

Bad:

```javascript
getUserInfo();
getClientData();
getCustomerRecord();
```

Good:

```javascript
getUser();
```



## 使用可搜索的名称

我们要阅读的代码比要写的代码多得多， 所以我们写出的代码的可读性和可搜索性是很重要的。 使用没有 意义的变量名将会导致我们的程序难于理解， 将会伤害我们的读者， 所以请使用可搜索的变量名。 类似 [buddy.js](https://github.com/danielstjules/buddy.js) 和 [ESLint](https://github.com/eslint/eslint/blob/660e0918933e6e7fede26bc675a0763a6b357c94/docs/rules/no-magic-numbers.md) 的工具可以帮助我们找到未命名的常量。

Bad:

```javascript
// 86400000 是什么东西？
setTimeout(blastOff, 86400000);
```

Good:

```javascript
// 将它们声明为全局常量 `const` 。
const MILLISECONDS_IN_A_DAY = 86400000;

setTimeout(blastOff, MILLISECONDS_IN_A_DAY);
```



## 使用解释性的变量

Bad:

```javascript
const address = 'One Infinite Loop, Cupertino 95014';
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/;
saveCityZipCode(address.match(cityZipCodeRegex)[1], address.match(cityZipCodeRegex)[2]);
```

Good:

```javascript
const address = 'One Infinite Loop, Cupertino 95014';
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/;
const [, city, zipCode] = address.match(cityZipCodeRegex) || [];
saveCityZipCode(city, zipCode);
```



## 避免心理映射

显示比隐式更好

Bad:

```javascript
const locations = ['Austin', 'New York', 'San Francisco'];
locations.forEach((l) => {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  // l代表什么意思
  dispatch(l);
});
```

Good:

```javascript
const locations = ['Austin', 'New York', 'San Francisco'];
locations.forEach((location) => {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  dispatch(location);
});
```



## 不添加不必要的上下文

如果你的类名/对象名有意义， 不要在变量名上再重复。

Bad:

```javascript
const Car = {
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue'
};

function paintCar(car) {
  car.carColor = 'Red';
}
```

Good:

```javascript
const Car = {
  make: 'Honda',
  model: 'Accord',
  color: 'Blue'
};

function paintCar(car) {
  car.color = 'Red';
}
```



## 使用默认变量替代短路运算或条件

Bad:

```javascript
function createMicrobrewery(name) {
  const breweryName = name || 'Hipster Brew Co.';
  // ...
}
```

Good:

```javascript
function createMicrobrewery(breweryName = 'Hipster Brew Co.') {
  // ...
}
```

# 函数

------

## 函数参数 (两个以下最理想)

限制函数参数的个数是非常重要的， 因为这样将使你的函数容易进行测试。 一旦超过三个参数将会导致组 合爆炸， 因为你不得不编写大量针对每个参数的测试用例。

没有参数是最理想的， 一个或者两个参数也是可以的， 三个参数应该避免， 超过三个应该被重构。 通常， 如果你有一个超过两个函数的参数， 那就意味着你的函数尝试做太多的事情。 如果不是， 多数情况下一个 更高级对象可能会满足需求。

由于 JavaScript 允许我们不定义类型/模板就可以创建对象， 当你发现你自己需要大量的参数时， 你 可以使用一个对象。

Bad：

```javascript
function createMenu(title, body, buttonText, cancellable) {
  // ...
}
```

Good:

```javascript
const menuConfig = {
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
};

function createMenu(config) {
  // ...
}
```



## 函数应当只做一件事情

这是软件工程中最重要的一条规则， 当函数需要做更多的事情时， 它们将会更难进行编写、 测试和推理。 当你能将一个函数隔离到只有一个动作， 他们将能够被容易的进行重构并且你的代码将会更容易阅读。 如 果你严格遵守本指南中的这一条， 你将会领先于许多开发者。

Bad:

```javascript
function emailClients(clients) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

Good:

```javascript
function emailClients(clients) {
  clients
    .filter(isClientActive)
    .forEach(email);
}

function isClientActive(client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```



## 函数名称应该说明它要做什么

Bad:

```javascript
function addToDate(date, month) {
  // ...
}

const date = new Date();

// 很难从函数名看出加了什么
addToDate(date, 1);
```

Good:

```javascript
function addMonthToDate(month, date) {
  // ...
}

const date = new Date();
addMonthToDate(1, date);
```



## 函数应该只有一个抽象级别

当在你的函数中有多于一个抽象级别时， 你的函数通常做了太多事情。 拆分函数将会提升重用性和测试性。

Bad:

```javascript
function parseBetterJSAlternative(code) {
  const REGEXES = [
    // ...
  ];

  const statements = code.split(' ');
  const tokens = [];
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}
```

Good:

```javascript
function tokenize(code) {
  const REGEXES = [
    // ...
  ];

  const statements = code.split(' ');
  const tokens = [];
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ );
    });
  });

  return tokens;
}

function lexer(tokens) {
  const ast = [];
  tokens.forEach((token) => {
    ast.push( /* ... */ );
  });

  return ast;
}

function parseBetterJSAlternative(code) {
  const tokens = tokenize(code);
  const ast = lexer(tokens);
  ast.forEach((node) => {
    // parse...
  });
}
```



## 移除冗余代码

竭尽你的全力去避免冗余代码。 冗余代码是不好的， 因为它意味着当你需要修改一些逻辑时会有多个地方 需要修改。

想象一下你在经营一家餐馆， 你需要记录所有的库存西红柿， 洋葱， 大蒜， 各种香料等等。 如果你有多 个记录列表， 当你用西红柿做一道菜时你得更新多个列表。 如果你只有一个列表， 就只有一个地方需要更 新！

你有冗余代码通常是因为你有两个或多个稍微不同的东西， 它们共享大部分， 但是它们的不同之处迫使你使 用两个或更多独立的函数来处理大部分相同的东西。 移除冗余代码意味着创建一个可以处理这些不同之处的 抽象的函数/模块/类。

让这个抽象正确是关键的， 这是为什么要你遵循 *Classes* 那一章的 SOLID 的原因。 不好的抽象比冗 余代码更差，所以要谨慎行事。既然已经这么说了，如果你能够做出一个好的抽象，才去做。不要重复你自己，否则你会发现当你要修改一个东西时时刻需要修改多个地方。

Bad:

```javascript
function showDeveloperList(developers) {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary();
    const experience = developer.getExperience();
    const githubLink = developer.getGithubLink();
    const data = {
      expectedSalary,
      experience,
      githubLink
    };

    render(data);
  });
}

function showManagerList(managers) {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();
    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```

Good:

```javascript
function showList(employees) {
  employees.forEach((employee) => {
    const expectedSalary = employee.calculateExpectedSalary();
    const experience = employee.getExperience();

    let portfolio = employee.getGithubLink();

    if (employee.type === 'manager') {
      portfolio = employee.getMBAProjects();
    }

    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```



## 使用 Object.assign 设置默认对象

Bad:

```javascript
const menuConfig = {
  title: null,
  body: 'Bar',
  buttonText: null,
  cancellable: true
};

function createMenu(config) {
  config.title = config.title || 'Foo';
  config.body = config.body || 'Bar';
  config.buttonText = config.buttonText || 'Baz';
  config.cancellable = config.cancellable === undefined ? config.cancellable : true;
}

createMenu(menuConfig);
```

Good:

```javascript
const menuConfig = {
  title: 'Order',
  // User did not include 'body' key
  buttonText: 'Send',
  cancellable: true
};

function createMenu(config) {
  config = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config);

  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig);
```



## 不要使用标记位做为函数参数

标记位是告诉你的用户这个函数做了不只一件事情。 函数应该只做一件事情。 如果你的函数因为一个布尔值 出现不同的代码路径， 请拆分它们。

Bad:

```javascript
function createFile(name, temp) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

Good:

```javascript
function createFile(name) {
  fs.create(name);
}

function createTempFile(name) {
  createFile(`./temp/${name}`);
}
```



## 避免副作用 (第一部分)

如果函数进行不是接收参数然后返回值这种操作，就会产生副作用。副作用可能是修改了某些全局变量或者是写入到一个文件中。

Bad:

```javascript
let name = "xiao ming";
function splitName() {
    name = name.split(" ");
}
splitName();
console.log(name)
// ["xiao", "ming"]
```



Good:

``````javascript
function splitName(name) {
    return name.split(" ")
};
const name = "xiao ming";
const newName = splitName(name);
console.log(name); // xiao ming
console.log(newName) ["xiao", "ming"]
``````



## 避免副作用（第二部分）

在JavaScript中，一些值是不会变的（immutable）而一些则是可以变动的（mutable）。对象（object）和数组（arrays）就是两种可变的值，因此在作为函数参数进行传递时要非常小心。函数可以更改对象的值或者是修改一个数组中的内容，这些很容易造成bug。

假设有一个函数接收一个数组参数（可以把这个数组想象成一个购物车）。如果函数中对这个数组进行更改——添加多一个商品。其它有用到该数组的函数也会受影响。这不是挺不错的嘛，怎么会不好呢？让我们想象一个坏情形：

用户点击购买按钮后调用`purchase`函数发送网络请求，把购物车中的数据传递给服务端。由于网络连接不好，`purchase`函数一直重试请求。如果这个时候用户不小心将某件他们不想要的商品添加至购物车会怎么办？如果真的出现这种情形而且网络请求发送成功了，那么`purchase`函数就会将不小心添加商品后的数据发送至服务端。

一个好的解决办法就是在调用`addItemToCart`函数前克隆一份购物车数据，对克隆的数据进行操作后再返回。这样可以保证其他使用购物车数据的函数不会受到影响。

对于这个方法有两个注意事项：

- 也可能存在你真的想要改变传入的对象这种情形。当你接受了这个语法训练以后你会发现那些情形非常少见。大多数都是可以改写成无副作用的形式。
- 克隆大对象在性能方面可能非常昂贵。幸运的是，这在实践中并不是一个大问题，因为有一些很棒的库允许这种编程方法速度更快，并且不像手动克隆对象和数组那样占用内存。

Bad:

```javascript
const addItemToCart = (cart, item) => {
    cart.push({item, date: Date.now()})
}
```

Good:

```javascript
const addItemToCart = (cart, item) => {
    return [...cart, {item, date: Date.now()}]
}
```



## 不写全局函数

污染全局在JavaScript中是一个坏操作，因为这可能跟你引入的其他库发生冲突，导致你的API的调用者无法判断最终在生产环境中出现异常。我们想想一个例子：如果你想要在JavaScript的原生Array方法中写入一个`diff`方法，用于展示两个数组间的区别，要怎么做呢？你可以在`Array.prorotype`中添加一个新方法，但是也可能跟其它有相同想法的库发生冲突。假如其它的库只是用`diff`去对数组的第一个和最后一个元素进行判断呢？这就是为什么使用`ES2015/ES6`中的classes 会更好而且更容易全局扩展`Array`。

Bad:

```javascript
Array.prototype.diff = function diff(comparisonArray) {
    const hash = new Set(comparisonArray);
    return this.filter(elem => !hash.has(elem))
}
```



Good:

```javascript
class SuperArray entends Array {
    diff(comparisonArray) {
        const hash = new Set(comparisonArray);
        return this.filter(elem => !hash.has(elem))
    }
}
```



## 支持函数式编程而非命令式编程

JavaScript不像Haskell那样是一种函数式语言，但是它有一种函数式的风格。函数式语言更加简洁且容易测试。尽量使用函数式编程风格。

Bad:

```javascript
const programmerOutput = [
    {
        name: 'xiao ming',
        linesOfCode: 100
    },
    {
        name: 'xiao hong',
        linesOfCode: 200
    },
    {
        name: 'xiao pang',
        linesOfCode: 300
    }
];
let totalOutput = 0;
for(let i = 0; i < programmerOutput.length; i++) {
    totalOutput += programmerOutput[i].linesOfCode;
}
```



Good:

```javascript
const programmerOutput = [
    {
        name: 'xiao ming',
        linesOfCode: 100
    },
    {
        name: 'xiao hong',
        linesOfCode: 200
    },
    {
        name: 'xiao pang',
        linesOfCode: 300
    }
];

const totalOutput = programmerOutput.reduce(
	(totalLines, output) => totalLines + output.linesOfCode, 0
)
```



## 封装条件

Bad:

```javascript
if(fsm.state === 'fetching' && isEmpty(listNode)) {
    // ..
}
```



Good:

```javascript
function shouldShowSpinner(fsm, listNode) {
    return fsm.state === 'fetching' && isEmpty(listNode)
}
if(shouldShowSpinner(fsmInstance, listNodeInstance)) {
    // ..
}
```



## 避免使用否定条件

Bad:

```javascript
function isDomNotPresent(node) {
    //..
}
if(!isDomNotPresent(node)) {
    //..
}
```



Good:

```javascript
function isDomPresent(node) {
    //..
}
if(isDomPresent) {
    //..
}
```



## 避免条件句

这看起来像一个根本不可能的任务。当第一次听到这个的时候，大多数人都会说，"没有 if 判断我什么也做不了啊"。答案就是你可以用多态在不同的例子中实现相同的任务。第二个问题通常会是, "那挺好的但是我为什么要那样做"。答案是我们之前学到的一个干净的代码概念：一个函数应该只做一件事。如果你有很多类（classes）和函数（functions）中都存在 if 判断，你告诉你的用户你的函数做了不止一件事。记住，只要做一件事。

Bad:

```javascript
class Airplane {
    getCruisingAltitude() {
        switch (this.type) {
            case "777":
                return this.getMaxAltitude() - this.getPassengerCount();
            case "Air Force One":
                return this.getMaxAltitude();
        }
    }
}
```

Good:

```javascript
class Airplane {
    // ..
}
class Boeing777 extends Airplane {
    // ..
    getCruisingAltitude() {
        return this.getMaxAltitude() - this.getPassengerCount();
    }
}
class AirForceOne extends Airplane {
    // ..
    getCruisingAltitude() {
        return this.getMaxAltitude()
    }
}
```



## 避免类型检验（第一部分）

JavaScript是无类型的，也就意味着你的函数可以接受任意类型参数。有时候你会受这种自由折磨而且在函数中添加类型检验看起来非常不错。有很多方法可以避免这样做。首先要考虑的是一致的API。

Bad:

```javascript
function travelToTexas(vehicle) {
    if(vehicle instanceof Bicycle) {
        vehicle.pedal(this.currentLocation, new Location("texas"))
    } else if (vehicle instanceof Car) {
        vehicle.drive(this.currentLocation, new Location("texas"))
    }
}
```

Good:

```javascript
function travelToTexas(vehicle) {
    vehicle.move(this.currentLocation, new Location("texas"))
}
```



## 避免类型检验（第二部分）

如果你使用的是字符串和整数之类的原始值类型，虽然不能使用多态性，但仍然需要进行类型检查，那么应该考虑使用TypeScript。它是普通JavaScript的优秀替代品，因为它在标准JavaScript语法的基础上为您提供了静态类型。手动检查普通JavaScript的问题在于，要想把它做好，需要太多额外的赘言，以至于你得到的“类型安全”并不能弥补失去的可读性。保持JavaScript干净，编写良好的测试，并进行良好的代码评审。除了使用TypeScript之外，不然所有这些工作都要做。（就像我说的，这是一个很好的选择！）

Bad:

```javascript
function combine(val1, val2) {
    if(
    	(typeof val1 === 'number' && typeof val2 === 'number') ||
        (typeof val1 === 'string' && typeof val2 === 'string')
    ) {
        return val1 + val2;
    }
    throw new Error("Must be of type String ot Number")
}
```



Good:

```javascript
function combine(val1, val2) {
    return val1 + val2;
}
```



## 不要过度优化

现代浏览器在运行时进行了大量的优化。很多时候，如果你在优化，那就是在浪费时间。[这个资源](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)可以看到哪里缺少优化。主要针对那些进行优化除非它们已经被解决了。

Bad:

```javascript
// 在旧浏览器上，每次迭代都使用未缓存的`列表长度`会很昂贵
// 因为`列表长度`重新计算。在现代浏览器中，这是经过优化的。
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

Good:

```javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```



## 删除死代码

死代码和重复代码一样糟糕。没有理由把它放在你的代码库里。如果没有被调用，就把它删了！如果你仍然需要它，它在你的版本历史记录中仍然是安全的。

Bad:

```javascript
function oldRequestModule(url) {
    //..
}
function newRequestModule(url) {
  // ...
}

const req = newRequestModule;
inventoryTracker("apples", req, "www.inventory-awesome.io");
```

Good:

```javascript
function newRequestModule(url) {
  // ...
}

const req = newRequestModule;
inventoryTracker("apples", req, "www.inventory-awesome.io");
```



# 对象和数据结构

------

## 使用getters和setters

使用getters和setters来操作对象上的数据可能会比简单的在对象上查看值更好。"为什么呢？"你可能会问。以下是一些原因：

- 当你想要在获取对象值的时候做更多的操作，你不需要在你的代码库中查找然后改变所有的引用。
- 在赋值（`set`）操作时可以添加校验。
- 封装内部表达式。
- 在获取值和设置值的时候可以更简单的输出日志跟异常处理。
- 你可以懒加载你对象的值，就好比说从服务器中获取的。

Bad:

```javascript
function makeBankAccount() {
    //..
   	return {
        balance: 0
    }
}
const account = makeBankAccount();
account.balance = 100;
```

Good:

```javascript
function makeBankAccount() {
  // 这个是私有的
  let balance = 0;

  // 一个getter，让外部访问返回下边的对象
  function getBalance() {
    return balance;
  }

  // 一个setter，让外部访问返回下边的对象
  // a "setter", made public via the returned object below
  function setBalance(amount) {
    // ... 可以在赋值前添加校验
    balance = amount;
  }

  return {
    // ...
    getBalance,
    setBalance
  };
}

const account = makeBankAccount();
account.setBalance(100);
```



## 让对象包含私有成员

这个可以通过闭包实现（ES5及以下）

Bad:

```javascript
const Employee = function(name) {
    this.name = name;
}

Employee.prototype.getName = function getName(){
    return this.name;
}

const employee = new Employee("xiao ming");
console.log(`Employee name: ${employee.getName()}`) // Employee name： xiao ming
delete employee.name;
console.log(`Employee name: ${employee.getName()}`) // Employee name：undefined
```



Good:

```javascript
function makeEmployee(name) {
    return {
        getName() {
            return name;
        }
    }
}
const employee = makeEmployee("xiao ming");
console.log(`Employee name: ${employee.getName()}`); // Employee name: xiao ming
delete employee.name;
console.log(`Employee name: ${employee.getName()}`); // Employee name: xiao ming
```



# 类

------

## 优先考虑 ES2015 / ES6 的 classes 而不是ES5的普通函数

对于传统的ES5类来说，很难获得可读的类继承、构造和方法定义。

Bad:

```javascript
const Animal = function(age) {
    if(!(this instanceof Animal)) {
        throw new Error("instantiate Animal with `new`")
    }
    this.age = age
}

Animal.prototype.move = function move() {}

const Mammal = function(age, furColor) {
    if(!(this instanceof Mammal)) {
        throw new Error("instantiate Mammal with `new`")
    }
    Animal.call(this, age);
    this.furColor = furColor;
}

Mammal.prototype = Object.create(Animal.prototype);
Mammal.prototype.constructor = Mammal;
Mammal.prototype.liveBirth = function liveBirth(){};

const Human = function(age, furColor, languageSpoken) {
    if(!(this instanceof Human)) {
        throw new Error("instantiate Human with `new`")
    }
}

Human.prototype = Object.create(Mammal.prototype);
Human.prototype.constructor = Human;
Human.prototype.speak = function speak() {};
```



Good:

```javascript
class Animal {
    constructor(age) {
        this.age = age;
    }
    move(){}
}
class Mammal extends Animal {
    constructor(age, furColor) {
        super(age);
        this.furColor = furColor;
    }
    liveBirth() {}
}
class Human extends Mammal {
    constructor(age, furColor, languageSpoken) {
        super(age, furColor);
        this.languageSpoken = languageSpoken;
    }
    speak(){}
}
```



## 使用方法链

这种模式在JavaScript中是非常有帮助的而且你可以在jQuery和Lodash这样的许许多多的库里面看到。它允许你的代码具有表现力，并且不那么冗长。因此，我说，使用方法链然后可以看看你的代码有多简洁。在你的类里边的每个函数末尾都简单返回`this`，然后你就可以连接更多的类方法。

Bad:

```javascript
class Car {
  constructor(make, model, color) {
    this.make = make;
    this.model = model;
    this.color = color;
  }

  setMake(make) {
    this.make = make;
  }

  setModel(model) {
    this.model = model;
  }

  setColor(color) {
    this.color = color;
  }

  save() {
    console.log(this.make, this.model, this.color);
  }
}

const car = new Car("Ford", "F-150", "red");
car.setColor("pink");
car.save();
```



Good:

```javascript
class Car {
  constructor(make, model, color) {
    this.make = make;
    this.model = model;
    this.color = color;
  }

  setMake(make) {
    this.make = make;
    // 返回this
    return this;
  }

  setModel(model) {
    this.model = model;
    // 返回this
    return this;
  }

  setColor(color) {
    this.color = color;
    // 返回this
    return this;
  }

  save() {
    console.log(this.make, this.model, this.color);
    // 返回this
    return this;
  }
}

const car = new Car("Ford", "F-150", "red").setColor("pink").save();
```



## 倾向合成而非继承

正如 Gang of Four在 Design Patterns（设计模式）中所说的那样，在可能的情况下，您应该更喜欢组合而不是继承。使用继承和组合有很多很好的理由。这条格言的要点是，如果你的思维本能地倾向于继承，那么试着思考组合是否能更好地模拟你的问题。在某些情况下，它是可以的。

你可能会想，“我什么时候应该使用继承？”这取决于你手头的问题，但这是一个很好的列表，列出了什么时候继承比组合更有意义：

1. 你的继承代表一种“是一种”关系，而不是一种“有一种”关系。（人类 -> 动物， 用户 -> 用户详情）
2. 你可以重用来自基类的代码（人类可以像所有动物一样移动）。
3. 你希望通过更改基类来对派生类进行全局更改。（改变所有动物移动时的热量消耗）。

Bad:

```javascript
class Employee {
    constructor(name, email) {
        this.name = name;
        this.email = email;
    }
}
// 这种方法不好是因为 Employee 存在税务数据， EmployeeTaxData不是Employee的一种类型
class EmployeeTaxData extends Employee {
  constructor(ssn, salary) {
    super();
    this.ssn = ssn;
    this.salary = salary;
  }

  // ...
}
```



Good:

```javascript
class EmployeeTaxData {
  constructor(ssn, salary) {
    this.ssn = ssn;
    this.salary = salary;
  }

  // ...
}

class Employee {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  setTaxData(ssn, salary) {
    this.taxData = new EmployeeTaxData(ssn, salary);
  }
  // ...
}
```



# solid

------

## 单一责任原则（Single Responsibility Principle (SRP)）

正如在 Clean Code 中所述，“一个类改变的原因不应该超过一个”。在一个类中塞满很多函数是很有诱惑力的，就好像你在飞机上只能带一个手提箱。这样做的问题是，你的类在概念上没有连贯性，它会给它很多改变的理由。尽量减少更改类的次数是很重要的。这一点很重要，因为如果一个类中有太多的功能，而您修改了其中的一部分，则很难理解这将如何影响代码库中的其他依赖模块。

Bad:

```javascript
class UserSettings {
    constructor(user) {
        this.user = user;
    }
    changeSettings(settings) {
        if(this.verifyCredentials()) {
            // ..
        }
    }
    verifyCredentials() {
        // ..
    }
}
```



Good:

```javascript
class UserAuth {
    construtor(user) {
        this.user =user;
    }
    verifyCredentials() {
        // ...
    }
}

class UserSettings {
    constructor(user) {
        this.user = user;
        this.auth = new UserAuth(user);
    }
    changeSettings(settings) {
        if(this.auth.verifyCredentials()) {
            // ...
        }
    }
}
```



## 开闭原则

正如 Bertrand Meyer所说，“软件实体（类、模块、函数等）应该为扩展而打开，但为修改而关闭。”这意味着什么呢？这个原则基本上是说，你应该允许用户添加新的功能，而不必更改原有的代码。

Bad:

```javascript
class AjaxAdapter extends Adapter {
	constructor() {
        super();
        this.name = "ajaxAdapter";
    }
}

class NodeAdapter extends Adapter {
    constructor() {
        super();
        this.name = "nodeAdapter";
    }
}

class HttpRequester {
    constructor(adapter) {
        this.adapter = adapter;
    }
    fetch(url) {
        if(this.adapter.name === "ajaxAdapter") {
            return makeAjaxCall(url).then(response => {
                // ...
            })
        } else if (this.adapter.name === "nodeAdapter") {
            return makeHttpCall(url).then(response => {
                // ...
            })
        }
    }
}

function makeAjaxCall(url) {
    // 请求并返回promise
}
function makeHttpCall(url) {
    // 请求并返回promise
}
```



Good:

```javascript
class AjaxAdapter extends Adapter {
    constructor() {
        super();
        this.name = "ajaxAdapter";
    }
    request(url) {
        // 请求并返回promise
    }
}

class NodeAdapter extends Adapter {
    constructor() {
        super();
        this.name = "nodeAdapter";
    }
    request(url) {
        // 请求并返回promise
    }
}

class HttpRequester {
    constructor(adapter) {
        this.adapter = adapter;
    }
    fetch(url) {
        return this.adapter.request(url).then(response => {
            // ...
        })
    }
}
```





## 里氏替换原则

对于一个非常简单的概念来说，这是一个可怕的术语。它被正式定义为“如果S是T的子类型，那么T类型的对象可以被S类型的对象替换（即S类型的对象可以替换T类型的对象），而不改变该程序的任何期望属性（正确性、执行的任务等）。”这是一个更可怕的定义。

最好的解释是，如果有父类和子类，那么基类和子类可以互换使用，而不会得到错误的结果。这可能仍然令人困惑，所以让我们看一看经典的正方形-矩形示例。从数学上讲，正方形是矩形，但如果通过继承使用“is-a”关系对其进行建模，你很快就会遇到麻烦。

Bad:

```javascript
class Rectangle {
    constructor() {
        this.width = 0;
        this.height = 0;
    }
    setColor(color) {
        // ...
    }
    render(area) {
        // ...
    }
    setWidth(width) {
        this.width = width;
    }
    setHeight(height) {
        this.height = height;
    }
    getArea() {
        return this.width * this.height;
    }
}

class Square extends Rectangle {
    setWidth(width) {
        this.width = width;
        this.height = width;
    }
    setHeight(height) {
        this.width = height;
        this.height = height;
    }
}

function renderLargeRectangles(rectangles) {
    rectangles.forEach(rectangle => {
        rectangle.setWidth(4);
        rectangle.setHeight(5);
        const area = rectangle.getArea();
        // Square返回25， 但实际应该是20
        rectangle.render(area);
    })
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles(rectangles);
```



Good:

```javascript
class Shape {
    setColor(color) {
        // ...
    }
    render(area) {
        // ...
    }
}

class Rectangle extends Shape {
    constructor(width, height) {
        super();
        this.width = width;
        this.height = height;
    }
    getArea() {
        return this.width * this.height;
    }
}

class Square extends Shape {
    constructor(length) {
        super();
        this.length = length
    }
    getArea() {
        return this.length * this.length;
    }
}

function renderLargeShapes(shapes) {
    shapes.forEach(shape => {
        const area = shape.getArea();
        shape.render(area);
    })
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeShapes(shapes);
```



## 接口隔离原则

JavaScript没有接口，所以这个原则没有其他原则那么严格。然而，即使JavaScript缺少类型系统，它也很重要。

接口隔离原则声明“不应该强迫客户依赖于他们不使用的接口。” 由于是鸭子类型，接口在JavaScript中是隐式契约。

在JavaScript中演示这一原理的一个很好的例子是针对需要大量设置对象的类。不要求客户端设置大量的选项是有益的，因为大多数时候他们不需要所有的设置。让它们可选有助于防止出现“胖接口”。

Bad:

```javascript
class DOMTraverser {
    constructor(settings) {
        this.settings = settings;
        this.setup();
    }
    setup() {
        this.rootNode = this.settings.rootNode;
        this.settings.animationModule.setup();
    }
    traverse() {
        // ...
    }
}

const $ = new DOMTraverser({
    rootNode: document.getElementsByTagName("body"),
    animationModule(){} // 在大多数情况下我们在遍历时都不需要动画
})
```



Good:

```javascript
class DOMTraverser {
    constructor(settings) {
        this.settings = settings;
        this.options = settings.options;
        this.setup();
    }
    setup() {
        this.rootNode = this.settings.rootNode;
        this.setupOptions();
    }
    setupOptions() {
        if(this.options.animationModule) {
            // ...
        }
    }
    traverse() {
        // ...
    }
}

const $ = new DOMTraverser({
    rootNode: document.getElementsByTagName("body"),
    options: {
        animationModule(){}
    }
})
```



## 依赖反转原理

这一原则规定了两个基本事项：

1. 高级模块不应依赖于低级模块。两者都应该依赖于抽象。

2. 抽象不应该依赖于细节。细节应该取决于抽象。

这一点一开始可能很难理解，但是如果你使用过AngularJS，你就可能已经看到了依赖注入（DI）形式的这一原则的实现。虽然它们不是完全相同的概念，但是依赖倒置原理阻止高级模块了解其低级模块的细节并设置它们。它可以通过DI来实现这一点。这样做的一个巨大好处是减少了模块之间的耦合。耦合是一种非常糟糕的开发模式，因为它使代码很难重构。

如前所述，JavaScript没有接口，因此依赖的抽象是隐式契约。也就是说，一个对象/类向另一个对象/类公开的方法和属性。在下面的示例中，隐式约定是`InventoryTracker`的任何请求模块都将具有`requestItems`方法。

Bad:

```javascript
class InventoryRequester {
    constructor() {
        this.REQ_METHODS = ["HTTP"];
    }
    requstItem(item) {
        // ...
    }
}

class InventoryTracker {
    constructor(items) {
        this.items = items;
        
        // 我们已经创建了对特定请求实现的依赖关系。
	   // 我们应该让requestItems依赖于一个请求方法：`request`
        this.requester = new InventoryRequester();
    }
    requestItems() {
        this.items.forEach(item => {
            this.requester.requestItem(item);
        })
    }
}

const inventoryTracker = new InventoryTracker(["apples", "bananas"]);
inventoryTracker.requestItems();
```



Good:

```javascript
class InventoryTracker {
    constructor(items, requester) {
        this.items = items;
        this.requester = requester;
    }
    requestItems() {
        this.items.forEach(item => {
            this.requester.requestItem(item)
        })
    }
}

class InventoryRequesterV1 {
    constructor() {
        this.REQ_METHODS = ["HTTP"];
    }
    requestItem(item) {
        // ...
    }
}

class InventoryRequesterV2 {
    constructor() {
        this.REQ_METHODS = ["WS"];
    }
    requestItem(item) {
        // ...
    }
}

// 通过在外部构造依赖项并注入它们，我们可以很容易地
// 将我们的请求模块替换为使用WebSockets的新模块。
const inventoryTracker = new InventoryTracker (
	["apples", "bananas"],
    new InventoryRequesterV2()
)
inventoryTracker.requestItems();
```

# 测试

------



测试比运输更重要。如果您没有测试或者测试量不足，那么每次发布代码时，你都不能确定自己没有破坏任何东西。决定什么是足够的数量取决于您的团队，但是拥有100%的覆盖率（所有语句和分支）是您获得非常高的信心和开发人员的安心的方式。这意味着除了拥有一个优秀的测试框架外，还需要使用一个[好的覆盖工具](https://gotwarlost.github.io/istanbul/)。

没有理由不写测试。有很多好的JS测试框架，所以找一个你的团队更喜欢的。当你找到一个对你的团队有效的方法时，就要始终为你引入的每一个新特性/模块编写测试。如果您首选的方法是测试驱动开发（Test-Driven Development，TDD），那就很好了，但主要的一点是在启动任何特性或重构现有特性之前，确保你达到了覆盖率目标。

## 一个测试一个概念

Bad:

```javascript
import assert from "assert";

describe("MomentJS", () => {
  it("handles date boundaries", () => {
    let date;

    date = new MomentJS("1/1/2015");
    date.addDays(30);
    assert.equal("1/31/2015", date);

    date = new MomentJS("2/1/2016");
    date.addDays(28);
    assert.equal("02/29/2016", date);

    date = new MomentJS("2/1/2015");
    date.addDays(28);
    assert.equal("03/01/2015", date);
  });
});
```

Good:

```javascript
import assert from "assert";

describe("MomentJS", () => {
  it("handles 30-day months", () => {
    const date = new MomentJS("1/1/2015");
    date.addDays(30);
    assert.equal("1/31/2015", date);
  });

  it("handles leap year", () => {
    const date = new MomentJS("2/1/2016");
    date.addDays(28);
    assert.equal("02/29/2016", date);
  });

  it("handles non-leap year", () => {
    const date = new MomentJS("2/1/2015");
    date.addDays(28);
    assert.equal("03/01/2015", date);
  });
});
```



# 并发

------

## 使用 Promises 而不是回调函数

回调看起来不简洁，它们会导致过多的嵌套。对于ES2015/ES6，Promises是一种内置的全局类型。使用它们！

Bad:

```javascript
import { get } from "request";
import { writeFile } from "fs";

get(
  "https://en.wikipedia.org/wiki/Robert_Cecil_Martin",
  (requestErr, response, body) => {
    if (requestErr) {
      console.error(requestErr);
    } else {
      writeFile("article.html", body, writeErr => {
        if (writeErr) {
          console.error(writeErr);
        } else {
          console.log("File written");
        }
      });
    }
  }
);
```



Good:

```javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(body => {
    return writeFile("article.html", body);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```



## Async / Await 比Promises更简洁

Promises是回调函数的一个非常简洁的替代方法，但是ES2017/ES8带来了async和await，这提供了一个更干净的解决方案。你所需要的只是一个以`async`关键字为前缀的函数，然后就可以在不使用`then`函数链的情况下强制编写逻辑了。如果你现在可以利用ES2017/ES8的功能，请使用此功能！

Bad:

```javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(body => {
    return writeFile("article.html", body);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```



Good:

```javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

async function getCleanCodeArticle() {
  try {
    const body = await get(
      "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
    );
    await writeFile("article.html", body);
    console.log("File written");
  } catch (err) {
    console.error(err);
  }
}

getCleanCodeArticle()
```





# 错误处理

------

抛出错误是一件好事情！它们意味着运行时已经成功地识别了程序中的某些错误，它通过停止当前堆栈上的函数执行、终止进程（在Node中）并在控制台中用堆栈跟踪来通知你。



## 不要忽视异常捕获

对捕获到的错误不采取任何措施都不会使你有能力修复或对所述错误作出反应。将错误记录到控制台(`console.log`)不是很好，因此它容易迷失在控制台输出的一大堆信息中。如果你使用`try/catch`包装任何代码，这意味着你认为可能会在那里发生错误，因此你应该为此制定计划或创建代码路径。

Bad:

```javascript
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}
```

Good:

```javascript
try {
  functionThatMightThrow();
} catch (error) {
  // 一种选择 (比console.log更明显):
  console.error(error);
  // 另一个选择:
  notifyUserOfError(error);
  // 其他选择:
  reportErrorToService(error);
  // 或者执行以上三种
}
```



## 不要忽视 rejected 状态的 promises

跟你不应该忽略try/catch中捕获的错误一样的原因。

Bad:

```javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
    console.log(error);
  });
```



Good:

```javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
   	// 一种选择 (比console.log更明显):
  	console.error(error);
  	// 另一个选择:
  	notifyUserOfError(error);
  	// 其他选择:
  	reportErrorToService(error);
  	// 或者执行以上三种
  });
```





# 格式化

------

格式化是主观的。像这里的许多规则一样，没有硬性规定你必须遵守。要点是不要争论格式问题。有[很多工具](https://standardjs.com/rules.html)可以实现自动化。用一个就行了！工程师就格式问题争论是浪费时间和金钱。

## 统一使用大写字母

JavaScript是非类型化的，所以大写告诉你很多关于变量、函数等的信息。这些规则是主观的，所以你的团队可以选择他们想要的任何东西。关键是，无论你们选择什么，都要始终如一。

Bad:

```javascript
const DAYS_IN_WEEK = 7;
const daysInMonth = 30;

const songs = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
const Artists = ["ACDC", "Led Zeppelin", "The Beatles"];

function eraseDatabase() {}
function restore_database() {}

class animal {}
class Alpaca {}
```

Good:

```javascript
const DAYS_IN_WEEK = 7;
const DAYS_IN_MONTH = 30;

const SONGS = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
const ARTISTS = ["ACDC", "Led Zeppelin", "The Beatles"];

function eraseDatabase() {}
function restoreDatabase() {}

class Animal {}
class Alpaca {}
```



## 函数调用者和被调用者应该是接近的

如果一个函数调用另一个函数，在源文件中垂直关闭这些函数。理想情况下，让调用者在被调用者的正上方。我们倾向于自上而下地阅读代码，就像读报纸一样。因此，让你的代码以这种方式阅读。

Bad:

```javascript
class PerformanceReview {
  constructor(employee) {
    this.employee = employee;
  }

  lookupPeers() {
    return db.lookup(this.employee, "peers");
  }

  lookupManager() {
    return db.lookup(this.employee, "manager");
  }

  getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  perfReview() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();
  }

  getManagerReview() {
    const manager = this.lookupManager();
  }

  getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.perfReview();
```



Good:

```javascript
class PerformanceReview {
  constructor(employee) {
    this.employee = employee;
  }

  perfReview() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();
  }

  getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  lookupPeers() {
    return db.lookup(this.employee, "peers");
  }

  getManagerReview() {
    const manager = this.lookupManager();
  }

  lookupManager() {
    return db.lookup(this.employee, "manager");
  }

  getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.perfReview();
```



# 注释

------

## 仅对具有业务逻辑复杂性的内容进行注释。

评论是道歉，不是要求。好的代码主要是文档本身。

Bad:

```javascript
function hashIt(data) {
  // hash
  let hash = 0;

  // string 的长度
  const length = data.length;

  // 遍历data中的元素
  for (let i = 0; i < length; i++) {
    // 获取元素的charCode
    const char = data.charCodeAt(i);
    // 获取hash
    hash = (hash << 5) - hash + char;
    // 转成32位整数
    hash &= hash;
  }
}
```

Good:

```javascript
function hashIt(data) {
  let hash = 0;
  const length = data.length;

  for (let i = 0; i < length; i++) {
    const char = data.charCodeAt(i);
    hash = (hash << 5) - hash + char;

    // 转成32位整数
    hash &= hash;
  }
}
```



## 不要在代码库中留下注释掉的代码

版本控制的存在是有一定道理的，就让旧代码存在在你的历史中。

Bad:

```javascript
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

Good:

```javascript
doStuff();
```



## 不要有日志注释

记住，使用版本控制！不需要死代码、注释代码，尤其是日志注释。使用`git log`获取历史记录！

Bad:

```javascript
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
function combine(a, b) {
  return a + b;
}
```

Good:

```javascript
function combine(a, b) {
  return a + b;
}
```



## 避免位置标记

它们通常只会让人厌恶。让函数和变量名以及适当的缩进和格式为代码提供可视化结构。

Bad:

```javascript
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
$scope.model = {
  menu: "foo",
  nav: "bar"
};

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
const actions = function() {
  // ...
};
```

Good:

```javascript
$scope.model = {
  menu: "foo",
  nav: "bar"
};

const actions = function() {
  // ...
};
```

