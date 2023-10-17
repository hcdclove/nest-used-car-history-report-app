Nest is a powerful web framework for Node.js, which helps you effortlessly build efficient, scalable applications. It uses modern JavaScript, is built with TypeScript and combines best concepts of both OOP (Object Oriented Progamming) and FP (Functional Programming).

It is not just another framework. You do not have to wait for a large community, because Nest is built with awesome, popular well-known libraries – Express and socket.io! It means, that you can quickly start using framework without worrying about a third party plugins.

**Core Concept**
The core concept of Nest is to provide an architecture, which helps developers to accomplish maximum separation of layers and increase abstraction in their applications.

**Setup Application**
Nest is built with features from both ES6 and ES7 (decorators, async / await). It means, that the easiest way to start adventure with it is to use Babel or TypeScript.

In this article I will use TypeScript (it is not required!) and I recommend everyone to choose this way too.

**Sample tsconfig.json file:**

{

"compilerOptions":
{
"module": "commonjs",
"declaration": false,
"noImplicitAny": false,
"noLib": false,
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
"target": "es6"
},
"exclude":
[
"node_modules"
]
}

So let’s start from scratch. Firstly, we have to create entry module of our application (app.module.ts):

import { Module } from 'nest.js';

@Module({})
export class ApplicationModule {}

At this moment module metadata is empty ({}), because we only want to run application (we do not have any controlles or components right now).

Second step – make file index.ts (or whatever) and use NestFactory to create Nest application instance based on our module class.

import { NestFactory } from 'nest.js';
import { ApplicationModule } from './app.module';

const app = NestFactory.create(ApplicationModule);
app.listen(3000, () => console.log('Application is listening on port 3000'));

**Express instance**

If you want to have a full control of express instance lifecycle, you can simply pass already created object as a second argument of NestFactory.create() method,

**just like this:**

import express from 'express';
import { NestFactory } from 'nest.js';
import { ApplicationModule } from './modules/app.module';

const instance = express();
const app = NestFactory.create(ApplicationModule, instance);
app.listen(3000, () => console.log('Application is listening on port 3000'));

It means, that you can directly add some custom configuration (e.g. setup some plugins such as morgan or body-parser).

**First Controller**

The Controllers layer is responsible for handling incoming HTTP requests.

In Nest, Controller is a simple class with @Controller() decorator.

In previuos section we set up entry point for an application. Now, let’s build our first endpoint /users:

import { Controller, Get, Post } from 'nest.js';

@Controller()
export class UsersController {

    @Get('users')
    getAllUsers(req, res, next) {}

    @Get('users/:id')
    getUser(req, res, next) {}

    @Post('users')
    addUser(req, res, next) {}

}

As you can guess, we created an endpoint with 3 different paths:

GET: users
GET: users/:id
POST: users

Look at our class again. Is it necessary to repeat ‚users’ in each path? Fortunately – not.

Nest allows us to pass additional metadata to @Controller() decorator { path } which is a prefix for each route. Let’s rewrite our controller:

@Controller({ path: 'users' })
export class UsersController {
@Get()
getAllUsers(req, res, next) {}

    @Get('/:id')
    getUser(req, res, next) {}

    @Post()
    addUser(req, res, next) {}

}
As you can see, methods in Nest controllers have the same list of arguments and behaviour as a simple routes in Express.

If you want to learn more about req (request), res (response) and next you should read short Routing Documentation. In Nest, they work equivalently. In fact, Nest controller methods are some kind of encapsulated express routes.

UsersController is ready to use, but our module doesn’t know about it yet. Let’s open ApplicationModule and add some metadata.

import { Module } from 'nest.js';
import { UsersController } from "./users.controller";

@Module({
controllers: [ UsersController ]
})

export class ApplicationModule {}

As you can see – we only have to insert our controller into controllers array. It’s everything.

**Components**

Almost everything is a component – Service, Repository, Provider etc. and they might be injected to controllers or to another component by constructor.

In previous section, we built a simple controller – UsersController. This controller has an access to our data (I know, it’s a fake data, but it doesn’t really matter here). It’s not a good solution. Our controllers should only handle HTTP requests and delegate more complex tasks to services.

This is why we are going to create UsersService component.

In real world, UsersService should call appropriate method from persistence layer e.g. UsersRepository component. We don’t have any kind of database, so again – we will use fake data.

import { Component, HttpException } from 'nest.js';

@Component()
export class UsersService {

        private users = [
            { id: 1, name: "Johny Fool" },
            { id: 2, name: "Hernan P" },
            { id: 3, name: "Other uses" },
        ];

    getAllUsers() {
        return Promise.resolve(this.users);
    }

    getUser(id: number) {
        const user = this.users.find((user) => user.id === id);
        if (!user) {
            throw new HttpException("User not found", 404);
        }
            return Promise.resolve(user);
        }

        addUser(user) {
            this.users.push(user);
            return Promise.resolve();
            }

}

**Nest Component is a simple class, with @Component() annotation.**

As might be seen in getUser() method we used HttpException. It is a Nest built-in Exception, which takes two parameters – error message and status code. It is a good practice to create domain exceptions, which should extend HttpException (more about it in „Error Handling” section).

Good, our service is prepared, let’s use it in UsersController.

@Controller({ path: 'users' })
export class UsersController {
constructor(private usersService: UsersService) {}

    @Get()
    getAllUsers(req, res) {
        this.usersService.getAllUsers()
            .then((users) => res.status(200).json(users));
    }

    @Get('/:id')
    getUser(req, res) {
        this.usersService.getUser(+req.params.id)
            .then((user) => res.status(200).json(user));
    }

    @Post()
    addUser(req, res) {
        this.usersService.addUser(req.body.user)
            .then((msg) => res.status(201).json(msg));
    }

}

Notice: I removed next argument from each method, because we don’t need it here.

As shown, UsersService will be injected into constructor.

It is incredibly easy to manage dependencies with TypeScript, because Nest will recognize your dependencies just by type! So this:

constructor(private usersService: UsersService)

Is everything what you have to do. There is one important thing to know — you must have emitDecoratorMetadata option set to true in your tsconfig.json.

If you are not TypeScript enthusiast and you work with plain JavaScript, you have to do it in this way:

import { Dependencies, Controller, Post, Get } from 'nest.js';

@Controller({ path: 'users' })
@Dependencies([ UsersService ])
export class UsersController {
constructor(usersService) {
this.usersService = usersService;
}

    @Get()
    getAllUsers(req, res, next) {
        this.usersService.getAllUsers()
            .then((users) => res.status(200).json(users));
    }

    @Get('/:id')
    getUser(req, res, next) {
        this.usersService.getUser(+req.params.id)
            .then((user) => res.status(200).json(user));
    }

    @Post()
    addUser(req, res, next) {
        this.usersService.addUser(req.body.user)
            .then((msg) => res.status(201).json(msg));
    }

}

Simple, right?

In this moment, application will not even start working.

Why? Because Nest doesn’t know anything about UsersService. This component is not a part of ApplicationModule yet. We have to add it there:

import { Module } from 'nest.js';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
controllers: [ UsersController ],
components: [ UsersService ],
})
export class ApplicationModule {}
That’s it! Now, our application will run, but still one of routes doesn’t work properly – addUser. Why? Because we are trying to extract request body (req.body.user) without body-parser express middleware. As you should already know, it is possible to pass express instance as a second argument of NestFactory.create() method.

Let’s install the body-parser plugin:

$ npm install --save body-parser
Then setup it in our express instance.

import express from 'express';
import \* as bodyParser from 'body-parser';
import { NestFactory } from 'nest.js';
import { ApplicationModule } from './modules/app.module';

const instance = express();
instance.use(bodyParser.json());

const app = NestFactory.create(ApplicationModule, instance);
app.listen(3000, () => console.log('Application is listening on port 3000'));

**Async / await**

Nest is compatible with async / await feature from ES7, so we can quickly rewrite our UsersController:

@Controller({ path: 'users' })
export class UsersController {
constructor(private usersService: UsersService) {}

    @Get()
    async getAllUsers(req, res) {
        const users = await this.usersService.getAllUsers();
        res.status(200).json(users);
    }

    @Get('/:id')
    async getUser(req, res) {
        const user = await this.usersService.getUser(+req.params.id);
        res.status(200).json(user);
    }

    @Post()
    async addUser(req, res) {
        const msg = await this.usersService.getUser(req.body.user);
        res.status(201).json(msg);
    }

}

Looks better right? There you can read more about async / await.

**Modules**

Module is a class with @Module({}) decorator. This decorator provides metadata, which framework uses to organize application structure.

Right now, it is our ApplicationModule:

import { Module } from 'nest.js';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
controllers: [ UsersController ],
components: [ UsersService ],
})

export class ApplicationModule {}

By default, modules encapsulate each dependency. It means, that it is not possible to use its components / controllers outside module.

Each module can also import another modules. In fact, you should think about Nest Modules as a tree of modules.

Let’s move UsersController and UsersService to UsersModule. Simply create new file e.g. users.module.ts with below content:

import { Module } from 'nest.js';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
controllers: [ UsersController ],
components: [ UsersService ],
})
export class UsersModule {}
Then import UsersModule into ApplicationModule (our main application module):

import { Module } from 'nest.js';
import { UsersModule } from './users/users.module';

@Module({
modules: [ UsersModule ]
})
export class ApplicationModule {}

It’s everything.

As might be seen, with Nest you can naturally split your code into separated and reusable modules!

**Middlewares**

Middleware is a function, which is called before route handler. Middleware functions have access to request and response objects, so they can modify them. They can also be something like a barrier – if middleware function does not call next(), the request will never be handled by route handler.

Let’s build a dummy authorization middleware (for explanation purposes – just by username). We will use X-Access-Token HTTP header to provide username (weird idea, but it doesn’t matter here).

import { UsersService } from './users.service';
import { HttpException, Middleware, NestMiddleware } from 'nest.js';

@Middleware()
export class AuthMiddleware implements NestMiddleware {
constructor(private usersService: UsersService) {}

    resolve() {
        return (req, res, next) => {
            const userName = req.headers["x-access-token"];
            const users = this.usersService.getUsers();

            const user = users.find((user) => user.name === userName);
            if (!user) {
                throw new HttpException('User not found.', 401);
            }
            req.user = user;
            next();
        }
    }

}

**Some facts about middlewares:**

you should use @Middleware() annotation to tell Nest, that this class is a middleware,

you can use NestMiddleware interface, which forces on you to implement resolve() method,

middlewares (same as components) can inject dependencies through their constructor (dependencies have to be a part of module),

middlewares must have resolve() method, which must return another function (higher order function). Why? Because there is a lot of third-party,

ready to use express middlewares, which you could simply – thanks to this solution – use in Nest.
Notice: I added getUsers method to UsersService, which returns encapsulated list of stored users.

Okey, we already have prepared middleware, but we are not using it anywhere. Let’s set it up:

import { MiddlewareBuilder } from 'nest.js';

@Module({
controllers: [ UsersController ],
components: [ UsersService ],
exports: [ UsersService ],
})
export class UsersModule {
configure(builder: MiddlewareBuilder) {
builder.apply(AuthMiddleware).forRoutes(UsersController);
}
}

As shown, Modules can have additional method – configure(). This method receives as a parameter MiddlewareBuilder, an object, which helps us to configure middlewares.

This object has apply() method, which receives infinite count of middlewares (this method uses spread operator, so it is possible to pass multiple classes separated by comma). apply() method returns object, which has two methods:

forRoutes() – we use this method to pass infinite count of Controllers or objects (with path and method properties) separated by comma,
with – we use this method to pass custom arguments to resolve()method of middleware.
How does it works?
When you pass UsersController in forRoutes method, Nest will setup middleware for each route in controller:

GET: users
GET: users/:id
POST: users
But it is also possible to directly define for which path middleware should be used, just like that:

builder.apply(AuthMiddleware)
.forRoutes({ path: '\*', method: RequestMethod.ALL });

**Chaining**

MiddlewareBuilder is a builder in fact. It means, that you can simply chain apply() calls:

builder.apply(AuthMiddleware, PassportMidleware)
.forRoutes(UsersController, OrdersController, ClientController);
.apply(...)
.forRoutes(...);

**Ordering**

Middlewares are called in the same order as they are placed in array. Middlewares configured in sub-module are called after the parent module configuration.

Real-time applications with Gateways
There are special components in Nest called Gateways. Gateways help us to create real-time web apps. They are some kind of encapsulated socket.io features adjusted to framework architecture.

**Gateways**

Gateway is a Component, so it can inject dependencies through constructor. Gateway also can be injected into another component.

import { WebSocketGateway, NestGateway } from 'nest.js/websockets';

@WebSocketGateway()
export class UsersGateway implements NestGateway {
afterInit(server) {}
handleConnection(client) {}
handleDisconnect(client) {}
}

By default – server runs on port 80 and with default namespace. We can easily change those settings:

@WebSocketGateway({ port: 81, namespace: 'users' })
Of course – server will run only if UsersGateway component is listed in module components array, so we have to place it there:

@Module({
controllers: [ UsersController ],
components: [ UsersService, UsersGateway ],
exports: [ UsersService ],
})

By the way, there are three useful events of

**Gateway:**

afterInit, which gets as an argument native server socket.io object,

handleConnection and handleDisconnect, which gets as an argument native client socket.io object.

**What with messages?**

In Gateway, we can simply subscribe to emitted messages:

import { WebSocketGateway, NestGateway, SubscribeMessage } from 'nest.js/websockets';

@WebSocketGateway({ port: 81, namespace: 'users' })
export class UsersGateway implements NestGateway {
afterInit(server) {}
handleConnection(client) {}
handleDisconnect(client) {}

    @SubscribeMessage({ value: 'drop' })
    handleDropMessage(sender, data) {
        // sender is a native socket.io client object
    }

}

And from client side:

import \* as io from 'socket.io-client';
const socket = io('http://URL:PORT/');
socket.emit('drop', { msg: 'test' });
@WebSocketServer()
If you want to assign to chosen property socket.io native server instance, you could simply decorate it with @WebSocketServer() decorator.

import { WebSocketGateway, WebSocketServer, NestGateway } from 'nest.js/websockets';

@WebSocketGateway({ port: 81, namespace: 'users' })
export class UsersGateway implements NestGateway {
@WebSocketServer() server;

    afterInit(server) {}
    handleConnection(client) {}
    handleDisconnect(client) {}

    @SubscribeMessage({ value: 'drop' })
    handleDropMessage(sender, data) {
        // sender is a native socket.io client object
    }

}

Value will be assigned after server initialization.

**Reactive Streams**

As you already know, Nest Gateway is a simple component, which can be injected into another components. This feature gives us possibility to choose how we are going to react on messages.

Of course – we can just inject components to Gateway and call their appropriate methods, when it is necessary.

But there is another solution – Gateway Reactive Streams. You can read more about them here.

**Microservices Support**

It is unbelievably simple to transform Nest application into Nest microservice.

Take a look – this is how you create web application:

const app = NestFactory.create(ApplicationModule);
app.listen(3000, () => console.log('Application is listening on port 3000'));
Now, switch it to a microservice:

const app = NestFactory.createMicroservice(ApplicationModule, { port: 3000 });
app.listen() => console.log('Microservice is listening on port 3000'));

It’s everything!

**Communication via TCP**

Microservices by default Nest microservice is listening for messages via TCP protocol. It means that right now @RequestMapping() (and @Post(), @Get() etc. too) will not be useful, because it is mapping HTTP requests. So, how microservice will recognize messages? Just by patterns.

**What is pattern?**

It is nothing special. It could be an object, string or even number (but it is not a good idea).

Let’s create MathController:

import { MessagePattern } from 'nest.js/microservices';

@Controller()
export class MathController {
@MessagePattern({ cmd: 'add' })
add(data, respond) {
const numbers = data || [];
respond(null, numbers.reduce((a, b) => a + b));
}
}

As you might seen – if you want to create message handler, you have to decorate it wih @MessagePattern(pattern).

In this example, choose { cmd: 'add' } as a pattern.

The handler method receives two arguments:

data, which is a variable with data sent from another microservice (or just web application),

respond a function which accepts two arguments (error, response).
Client
You already know how to listen for messages. Now, let’s check how to send them from another microservice or web application.

Before you can start, Nest has to know, where you are exactly going to send messages.

It’s easy – you only have to create @Client object.

import { Controller } from 'nest.js';
import { Client, ClientProxy, Transport } from 'nest.js/microservices';

@Controller()
export class ClientController {
@Client({ transport: Transport.TCP, port: 5667 })
client: ClientProxy;
}

@Client() decorator receives object as a parameter.

This object can have 3 properties:

transport – with this you can decide which method you are going to use – TCP or Redis (TCP by default),

url – only for Redis purposes (default – redis://localhost:6379),

port (default 3000).

Use client

Let’s create custom endpoint to test our communication.

import { Controller, Get } from 'nest.js';
import { Client, ClientProxy, Transport } from 'nest.js/microservices';

@Controller()
export class ClientController {
@Client({ transport: Transport.TCP, port: 5667 })
client: ClientProxy;

    @Get('client')
    sendMessage(req, res) {
        const pattern = { command: 'add' };
        const data = [ 1, 2, 3, 4, 5 ];

        this.client.send(pattern, data)
            .catch((err) => Observable.empty())
            .subscribe((result) => res.status(200).json({ result }));
    }

}

As you might seen, in order to send message you have to use send method, which receives message pattern and data as an arguments. This method returns an Observable from Rxjs package.

It is very important feature, because reactive Observables provide set of amazing operators to deal with, e.g. combine, zip, retryWhen, timeout and more…

Of course, if you want to use Promises instead of Observables, you could simply use toPromise() method.

That’s all.

Now, when someone will make /test request (GET), that’s how response should looks like (if both microservice and web app are available):

{
"result": 15
}

**Redis**

There is another way to work with Nest microservices. Instead of direct TCP communication, we could use amazing Redis feature – publish / subscribe. Of course before you can use it, it is obligatory to install Redis.

Create Redis microservice

To create Redis Microservice, you have to pass additional configuration in NestFactory.createMicroservice() method.

const app = NestFactory.createMicroservice(
MicroserviceModule,
{
transport: Transport.REDIS,
url: 'redis://localhost:6379'
}
);

app.listen(() => console.log('Microservice listen on port:', 5667 ));

And that’s all. Now your microservice will subscribe to messages published via Redis. The rest works same – patterns, error handling, etc.

**Redis Client**

Now, let’s see how to create client. Previously, your client instance configuration looks like that:

@Client({ transport: Transport.TCP, port: 5667 })
client: ClientProxy;

We want to use Redis instead of TCP, so we have to change those settings:

@Client({ transport: Transport.REDIS, url: 'redis://localhost:6379' })
client: ClientProxy;

Easy, right? That’s all. Other functionalities works same as in TCP communication.

**Shared Module**

Nest Modules can export their components. It means, that we can easily share component instance between them.

The best way to share an instance between two or more modules is to create Shared Module.

For example – if we want to share ChatGateway component across entire application, we could do it in this way:

@Module({
components: [ ChatGateway ],
exports: [ ChatGateway ]
})
export class SharedModule {}
Then, we only have to import this module into another modules, which should share component instance:

@Module({
modules: [ SharedModule ]
})
export class FeatureModule {}

That’s all.

**Dependency Injection**

Dependency Injection is a strong mechanism, which helps us easily manage dependencies of our classes. It is very popular pattern in strongly typed languages like C# and Java.

In Node.js it is not such important to use those kind of features, because we already have amazing module loading system and e.g. sharing instance between files is effortless.

The module loading system is sufficient for small and medium size applications. When amount of code grows, it is harder and harder to smoothly organize dependencies between layers. Someday everything may just blow up.

**giphy**

It is also less intuitive than DI by constructor.

This is the reason, why Nest has its own DI system.

**Custom components**

You have already learnt, that it is incredibly easy to add component to chosen module:

@Module({
controllers: [ UsersController ],
components: [ UsersService ]
})
But there is some other scenarios, which Nest allows you to take advantages of.

**Use value:**

const customObject = {};
@Module({
controllers: [ UsersController ],
components: [
{ provide: UsersService, useValue: customObject }
],
})

**When:**

you want to use specific value. Now, in this module Nest will associate customObject with UsersService metatype,
you want to use test doubles (unit testing).

Use class:

@Component()
class CustomUsersService {}

@Module({
controllers: [ UsersController ],
components: [
{ provide: UsersService, useClass: CustomUsersService }
],
})

**When:**

you want to use chosen, more specific class only in this module.
Use factory:

@Module({
controllers: [ UsersController ],
components: [
ChatService,
{
provide: UsersService,
useFactory: (chatService) => {
return Observable.of('customValue');
},
inject: [ ChatService ]
}
],
})

**When:**

you want to provide a value, which has to be calculated using other components (or custom packages features),
you want to provide async value (just return Observable or Promise), e.g. database connection.
Remember:

if you want to use components from module, you have to pass them in inject array. Nest will pass instances as a arguments of factory in the same order.
Custom providers

@Module({
controllers: [ UsersController ],
components: [
{ provide: 'isProductionMode', useValue: false }
],
})

**When:**

you want to provide value with a chosen key.
Remember:

it is possible to use each types useValue, useClass and useFactory.
How to use?

To inject custom provided component / value with chosen key, you have to tell Nest about it, just like that:

import { Component, Inject } from 'nest.js';

@Component()
class SampleComponent {
constructor(@Inject('isProductionMode') private isProductionMode: boolean) {
console.log(isProductionMode); // false
}
}

**ModuleRef**

Sometimes you might want to directly get component instance from module reference. It not a big thing with Nest – just inject ModuleRef in your class:

export class UsersController {
constructor(
private usersService: UsersService,
private moduleRef: ModuleRef) {}
}

ModuleRef provides one method:

get<T>(key), which returns instance for equivalent key (mainly metatype). Example moduleRef.get<UsersService>(UsersService) returns instance of UsersService component from current module.
In addition to this, ModuleRef is combined with real instance of your module. What it means? If your UsersModule looks like that:

export class UsersModule {
getContext() {
return 'Test';
}
}

Then, you can simply call this method from moduleRef:

moduleRef.getContext() === 'Test' // true

Of course – if you are using TypeScript – you have to cast instance to appropriate type.

**Testing**

Nest gives you a set of test utilities, which boost application testing process. There are two different approaches to test your components and controllers – isolated tests or with dedicated Nest test utilities.

**Unit Tests**

Both Nest controllers and components are a simple JavaScript classes. Itmeans, that you could easily create them by yourself:

const instance = new SimpleComponent();

If your class has any dependency, you could use test doubles, for example from such libraries as Jasmine or Sinon:

const stub = sinon.createStubInstance(DependencyComponent);
const instance = new SimpleComponent(stub);

**Nest Test Utilities**

The another way to test your applications building block is to use dedicated Nest Test Utilities.

Those Test Utilities are placed in static Test class (nest.js/testing module).

import { Test } from 'nest.js/testing';
This class provide two methods:

createTestingModule(metadata: ModuleMetadata), which receives as an parameter simple module metadata (the same as Module() class). This method creates a Test Module (the same as in real Nest Application) and stores it in memory.
get<T>(metatype: Metatype<T>), which returns instance of chosen (metatype passed as parameter) controller / component (or null if it is not a part of module).
Example:

Test.createTestingModule({
controllers: [ UsersController ],
components: [ UsersService ]
});
const usersService = Test.get<UsersService>(UsersService);
Mocks, spies, stubs
Sometimes you might not want to use a real instance of component / controller. Instead of this – you can use test doubles or custom values / objects.

const mock = {};
Test.createTestingModule({
controllers: [ UsersController ],
components: [
{ provide: UsersService, useValue: mock }
]
});

const usersService = Test.get<UsersService>(UsersService); // mock

**Error Handling**

Notice: It is mainly for REST applications.

Nest has error handling layer, which catches all unhandled exceptions.

If – somewhere – in your application, you will throw an Exception, which is not HttpException (or inherited one), Nest will handle it and return to user below json object (500 status code):

{
"message": "Unkown exception"
}

**Exception Hierarchy**

In your application, you should create your own Exceptions Hierarchy. All HTTP exceptions should inherit from built-in HttpException.

For example, you can create NotFoundException and UserNotFoundException classes:

import { HttpException } from 'nest.js';

export class NotFoundException extends HttpException {
constructor(msg: string) {
super(msg, 404);
}
}

export class UserNotFoundException extends NotFoundException {
constructor() {
super('User not found.');
}
}

Then – if you somewhere in your application throw UserNotFoundException, Nest will response to user with status

code 404 and above json object:

{
"message": "User not found."
}

It allows you to focus on logic and make your code much easier to read.
