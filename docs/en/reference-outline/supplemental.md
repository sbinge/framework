# Crafting Enterprise Web Applications

## Introduction - Application Lifecycle

[incl. A simple diagram to highlight major application lifecycle stages - development, testing, delivery, feedback, iterating back. Just to illustrate the big picture & set the context of the lifecycle terminology used in the rest of the documentation.]

In an era of agile delivery, small pieces of functionality are constantly shipped to users. The software industry began favouring this approach as it helps minimise risk and instead maximises user engagement and satisfaction.

Even with modern delivery methodologies, some risk is still inevitable. Complexity is one such risk, and can become a major concern for mature applications. Regardless of which system architectures an application may follow, over time, many small pieces of functionality can build up into a large and daunting codebase that requires several teams to oversee.

## Managing Complexity

### Minimizing the Potential for Errors

[Incl. a diagram that expands upon the simple lifecycle above & highlights scale of complexity of the pipeline - e.g. many developers feeding multiple development streams, merging into continuous integration, progressing through several quality control streams & ending up across multiple servers in multiple live environments]

The opportunities to implement new cleanly-designed features become rarer the longer an application is in production. Instead, existing features are more likely to be tweaked, bug-fixed or extended. Successful applications - and the features that comprise them - spend the majority of their lifecycle under maintenance.

Maintaining a complex application requires great discipline. It is far too easy for teams to get overwhelmed and spend their time clashing with codebase and colleagues instead of delivering value to users. Mitigating this risk involves many different approaches, covering areas of standards, patterns, technology choices and tooling, amongst others.

Errors are best caught as early as possible in the software delivery lifecycle. It is far quicker and cheaper to fix an error that has just been introduced in a single development stream than it is once the error has progressed through the entire delivery pipeline and is live in production where users may be negatively impacted by it.

### Typing

A good way of catching errors early is favoring strong typing in the application development phase. Logical errors caused by mismatched data types can be avoided if type information is made explicit in application code. Compilers and static type checkers can validate against the type information and fail a build when such a type mismatch occurs. Software can only progress past an individual developer’s workspace to the rest of the delivery pipeline once all such errors are resolved.

[link to dependencies] Dojo builds upon TypeScript to provide explicit typing and static compile-time type checking. Applications built using Dojo can benefit from using TypeScript over vanilla JavaScript.

[link to cli docs] When using the Dojo CLI to scaffold applications, a TypeScript compilation phase is included by default in the application build process. Developers can simply start writing type-safe application code from the offset.

### Modularization

#### Single Responsibility Principle

A component should ideally be small enough for it to only implement a single responsibility. The simpler and more encapsulated a component is, the easier it becomes to understand and maintain between any number of developers over long periods of time. Complex applications with large codebases are built up through combinations of many such smaller, more well-understood components.

Isolating responsibilities within individual components has many benefits when trying to reduce complexity:

-   Scope is limited. Assuming a component maintains a consistent API, internal changes can be made without affecting external users of the component. Conversely, details of the component are kept internal to its definition module, meaning its definition will not conflict with that of other components that may overlap certain naming conventions.
-   Testing requirements are simplified, as unit tests only need to focus on a single responsibility rather than exponential combinations of application flows through multiple conditionals.
-   Components can be reused in multiple locations, avoiding repetition. Bug fixes need only be made to a single component instead of several independent instances.

For web applications, isolation comes with additional benefits to end users. Applications can be partitioned into multiple layers, allowing users to load only the layer they are interested in at a given point in time. This reduces resource size and associated network transfer requirements, resulting in shorter load times for users before they can become productive.

#### Components of a Dojo application

##### Index HTML file

HTML pages are the foundation of every web application and Dojo applications are no different. Traditionally a single index.html file serves the role of representing both the entry point to the application, as well as the root container for the application’s overall structure within the DOM.

Dojo applications are typically injected into a single DOM element. This allows a Dojo application to easily coexist with other content on a page - static assets, a legacy application or even another Dojo application.

##### Widgets

Widgets are the Dojo analogy of DOM elements, and are the central concept of encapsulation within a Dojo application. Just as a traditional website is built up through a hierarchy of DOM elements, a Dojo application is constructed through a hierarchy of widgets.

Widgets represent everything from individual UI elements - such as a label or a textbox - to more complex containers that may represent a form, a page, or an entire application.

Similarly as not all elements within a DOM are visible to users, Dojo widgets are not exclusively focused on providing a user interface, but can also serve any behind-the-scenes requirements to implement the full range of application functionality.

[link to ref]See the Creating Widgets reference section for information on how to create widgets within your application.

##### TypeScript Modules

[link to performant rendering section] A Dojo widget is represented as a TypeScript class that inherits from WidgetBase, contained within a single TypeScript module. This class/module encapsulates most of what constitutes the widget, including its behaviour and semantic representation within relevant DOM elements.

Widgets provide an API to external consumers via a combination of:

-   An exported class, inheriting from WidgetBase, that defines the widget
-   Optional public methods on the exported class, to expose additional functionality
-   [link to state management] An exported Properties interface, inheriting from WidgetProperties, that defines the widget’s state requirements. This interface serves as both a list of individual data elements that comprise the widget’s state, as well as functions that can be called when state changes are made within the widget.

A widget that acts as a parent container for a number of child widgets will import the class definitions that the child widgets export. If the parent widget needs to inject state into the children, the parent will also import the Properties interfaces that the child widgets export. The parent will then pass in relevant data structures that match the Properties interfaces when constructing the child widgets.

##### CSS Modules

[link to performant rendering section] Widgets typically control their semantic structure via VDOM element declarations in their TypeScript module code. However, presentational styling of the widget to end users is handled via CSS, similar to styling of regular HTML elements.

[link to css modules] Dojo’s build system makes use of CSS Modules to allow encapsulated CSS fragments that represent the presentational concerns of a single widget. The scope of a single CSS module is limited to that module alone, similar to scoping of single TypeScript modules. This avoids clashing between widgets that may use similar CSS class names.

A widget’s TypeScript module can import its corresponding CSS module as per any other TypeScript import statement. This allows the widget to refer to its CSS class names via properties on an object, which can be enumerated and autocompleted within a developer’s IDE. These property names can then be used to specify styling classes when declaring the widget’s semantic element structure. CSS class name mismatches between a widget and its styling can therefore be identified at build time.

While a widget can entirely encapsulate its own styling via its corresponding CSS module, usually some flexibility is required. A widget may be used in different configurations across an application, each with their own unique presentational needs.

For optimal user experience, applications are typically presented to users in a consistent theme, such as a corporate color scheme or within a consistent font family. To accommodate this, Dojo’s build pipeline allows using cssnext features such as custom properties that can be referenced across CSS modules.

[link to Theming section]See the Theming section for more details on consistent presentation within an application.

### State Management

Enterprise applications typically require their state to be persisted over time, and allow users to view and manipulate this data in a variety of ways. State management can become one of the most complex areas of a large application when given the need to access and edit the same data, concurrently, across multiple locations while maintaining consistency.

State is usually persisted in a data store or database that is external to the web application components, meaning some state management complexity needs to be solved outside of the application. However for instances where data is flowing between the application and its users, several paradigms can help minimize the risks of complex state management.

#### Reactive Data Modification

Applications written in an imperative manner describe what data should be changed, how the change should occur, as well as specifying when and where the changes must occur. If several pieces of data are logically connected via some form of computation or assignment, their connection is only represented at a discrete point in time. Outside of this point in time, any of the data values may be changed in a way that violates their intended logical connection.

Applications written in a reactive manner instead try to elevate the logical connections between data, and relinquish control of specifying exactly when and where changes are made in favor of having the logical data connections kept consistent over time.

Complex applications with multiple service layers may have even more representations of the same data point as it flows around various locations in the application - a common pattern for this is the use of data transfer objects. Maintaining integrity of application state becomes exponentially complex the more representations a given piece of data may have.

Any application with a UI that presents dynamic state - including web applications - will encounter the problem of maintaining logical data connection consistency. A piece of data in these applications will always have at least two representations.

[incl. data flow diagram illustrating this concept of multiple representations of the same data point & the consistency issues that arise]
[clarify - highlight instances where consistency problems are outside the scope that dojo solves]

##### Example Problem Illustration

Given a todo list application that stores a set of tasks, a single task will have the following two data representations when shown to a user:

-   The task’s current description (its “source of truth”, such as what its value is in a data store)
-   A copy of the task’s description that is presented to a user via a UI element, such as a label or textbox.

If users can only view tasks, there are several issues related to how changes to a task’s description can be made visible to the users.

If a task is changed in the underlying data store, its new description needs to be propagated up through the UI so users aren’t viewing stale data. If the task is displayed in more than one location in the UI, all instances need to be updated so that users aren’t viewing inconsistent data between locations.

If users can also modify tasks (such as changing their description), there are additional issues that need solving.

A task description now has two sources of truth: the old value in the datastore, and the new value that a user has entered in a textbox UI element.

A change request then needs to be propagated back down to the underlying data store so the old value can be replaced with the new. Once the change is made, the new task description needs to be sent back up to the user so they see the correct value that includes their change. Any errors that may occur when trying to change the task description also need to be factored into this data exchange.

#### State Management in Dojo

[link to ‘basic’ ref]For the most basic requirements, a widget can manage its own state via internal class properties. While this approach favours isolation and encapsulation, it is only appropriate for very simple use cases such as widgets that appear in a single location within an application, or are disconnected from all other state an application deals with.

[link to ‘intermediate’ ref]As the need to share state between widgets increases, Dojo favours Reactive Inversion of Control. State can be lifted up into parent container widgets and injected into contained child widgets via the child’s WidgetProperties interface. This lifting of state can traverse the entire widget hierarchy if needed, where state is centralised within the root application widget and portions of it are then injected into relevant child branches.

[link to typescript module section]Dojo widgets typically export a WidgetProperties interface that describes the widget’s state requirements. This interface will list both the data fields that make up the widget’s state, as well as callback functions that the widget can invoke when state is changed, for example if the widget deals with providing a data collection form to users.

When instantiating a child, a parent widget passes in an implementation of the child widget’s WidgetProperties interface. This allows the parent to pass in state to the child, as well as handle any callbacks the child may invoke if state changes are made.

[probably a good place for a data flow diagram to illustrate the below]

[link to injectors]State changes traverse from leaf to root of the widget hierarchy via change event functions on WidgetProperties interfaces.

For applications with some form of data store, Injectors can be used to link externalized state with application widgets. State changes can then traverse from root to leaf of the widget hierarchy, initiated by an external state change invalidation event. 

Applications wired in this way can benefit from reactive state change propagation, for both changes made by users in the application, as well as changes made externally within a data store. Dojo will automatically and efficiently take care of re-rendering portions of the application that are affected by any state changes.

For more complex requirements, an externalized data store may be the best approach to simplifying application state requirements. A central data store can help applications that deal with substantial amounts of state, allow complex state edit operations, or require the same subsets of state in many locations. 

[link to ‘advanced’ ref]Dojo also provides an implementation of a centralized data store through its Stores module. This supports a variety of advanced requirements such as:

-   Inherent support for asynchronous commands, such as making calls to remote services for data management.
-   Deterministic sequencing of state manipulation operations.
-   State operation recording, allowing operation rollbacks/undo
-   Middleware wrapping of data manipulation processes, for cross-cutting concerns such as authorization or logging.
-   Built-in support for a localStorage-based data store, aiding PWAs.
-   Support for optimistic data updates, with automatic rollback on failure

### Application Composition

As web applications grow in size, it becomes inefficient for users to load all application resources when only a subset may be required for a given task. Every application resource has a cost associated with its size: memory storage requirements, data transfer over the network; all impacting the time a user needs to wait for before they can begin their work. It is in the users’ best interests to keep this cost to a minimum by only loading what is required.

#### Registry

One way Dojo solves the problem of only loading resources that are actually needed is through its Registry. Widgets can be added to the Registry with deferred loading, then referenced by a name tag elsewhere in the application. Widgets accessed in this way via the registry are only loaded on demand if and when the execution path reaches their usage.

Dojo supports both a global registry, accessible throughout the application, as well locally-scoped registries that are only accessible within the widgets that use them.

#### Externalizing State: Injectors

[link to state management]Dojo’s Registry also allows registration of injector functions, which provide a way to externalize state.

Once an injector has been registered, widgets can be associated with it via a Container wrapper. The Container deals with mapping state returned from an injector to the wrapped widget’s properties. Containers then act as a proxy to the wrapped widget, while making fields within the widget’s Properties interface optional, as state can now be provided via the injector.

Injectors can also signal when a state change occurs via an invalidation callback. If state is changed and the updates need to be reflected across an application, the invalidation callback can be invoked after which Dojo will determine which widgets are affected and only re-render the affected subset of the application.

### Testing Strategies

Not all errors can be caught through compilers or static type checkers. Features can be written that are syntactically and logically valid, but either don’t anticipate problems at runtime, or do not perform functionality in the intended way. To mitigate this risk, additional testing needs to be performed.

[link to cli & intern docs]When using the Dojo CLI to scaffold applications, a test runner for the Intern test library is included by default. This allows developers to begin writing test code immediately alongside application functionality.

[link to relevant Intern subsections for different testing types] Typical testing concerns for enterprise web applications are:

-   Validating isolated functionality of single modules, classes and methods - Unit testing.
-   Validating interoperation between several modules or classes - Integration testing.
-   Validating that an isolated but fully-integrated subsystem is working as per its specification - System testing.
-   Benchmarking application performance (with the added complexities of network latency) and ensuring it is within acceptable boundaries - Performance testing
-   Checking that the application functions as expected for end users - which for web applications typically needs to be conducted through a web browser - Acceptance/Functional/Compatibility testing
-   Ensuring the application works for the widest range of users - Usability/Accessibility testing
-   Ensuring restricted application functionality only works for the intended users - and conversely, that the application does not expose nor manipulate data it should not when accessed by unintended users, or worse, malicious parties - Security testing

[link to testing harness docs]Intern provides solutions for many of the above testing concerns but may not be sufficient for all testing needs of a project. Dojo also provides a simple testing harness that allows application test code to validate use of the framework and widgets at the VDOM abstraction level. This harness can be used from any test runner such as Intern, Jest, or any others that an application’s testing strategy necessitates.

### Feature Branching

[TODO]
[runtime, build time elision]

## Presentation Concerns

### Theming

[link to Material]One way applications provide optimal user experiences is via a consistent presentation to end users. This may be as simple as using a consistent font family across similar elements, but often extends to presenting the application in a corporate color palette, or even implementing an entire design language such as Material Design.

[link to PostCSS docs]TODO:cssnext & describe using alternatives
[link to CSS modules section]Dojo’s styling pipeline makes use of CSS modules and PostCSS processing. The combination of these allows for fully encapsulated CSS fragments for individual widgets, as well as centralised CSS variables to define common theme attributes that can be shared between all widgets within an application. Custom theming can also be provided for Dojo’s widget suite.

[link to ref]See the Creating Themes reference section for information on how to create a theme for your application.

### Navigational Routing

While some applications provide users a primary view in which to conduct the majority of their work, many applications contain more areas that a user can access. Help pages, settings panels or multi-step workflows are examples of where an application may have several different interfaces that a user could access at any given time.

Sections of an application need to be uniquely identifiable in order for a user to access them. These identifiers are also required to support bookmarking & sharing of links to a particular section of an application. Users also need a way to navigate between sections in order to access all functionality an application may provide. Navigation could simply be going forward to the next step of a process, backwards to a previous step, or ad-hoc jumping between several options depending on what a user chooses.

Traditional websites that use static files naturally have separately-identifiable sections, in that each static file within the site can be individually accessed. HTML files can use anchor elements to allow users to navigate between files by clicking on links, rather than having to manually change the URI in their browser’s address bar.

Single-page web applications, as their name suggests, only contain a primary file through which a user accesses the entire application. These applications can however make use of URIs (together with all their inherent benefits) to identify each subsection.

A Router component provides navigation options across a hierarchy of routes, and handles dispatching to relevant application subsections that correspond to identifiable routes. A router will also handle any error conditions, such as navigation to non-existent routes.

#### Routing in Dojo

The @dojo/framework/routing module allows applications to register URL subpaths (routes) that link to a specific type of widget, called an Outlet. When a user navigates to a particular route, the outlet widget registered against the route will be rendered.

While outlets are ‘rendered’ when users navigate to them, they seldom deal directly with the rendering of application functionality. Outlets are primarily wrappers that handle navigational concerns - passing of query parameters, or handling error fallbacks - and instead delegate to other widgets within an application for functional rendering.

Applications can provide navigation options to users via Link widgets which are associated to Outlets, in a similar manner to using anchors in traditional HTML pages.

[link to layering/bundling section]When using routing, Dojo’s build system can automatically generate separate bundles for each top level route within the application. Each bundle can then be independently delivered to users as they are needed. 

[link to ref]See the Routing reference section for details on how to implement routing within your application.

### Efficiency and Performance

#### Application Delivery: Layering and Bundling

[link to registry]Dojo’s Registry allows applications to load module dependencies on demand, only if and when they are required. This helps cut down the amount of data that users need to download before their application becomes responsive, but can become problematic in large applications containing many independent modules.

Fetching an application resource incurs additional overhead around HTTP resource negotiation. Data needs to be requested by the client, after which the client has to wait before the server finishes sending the last byte of the resource. In more severe cases, the overhead can also include DNS resolution, full TCP connection re-establishment and TLS cipher/certificate negotiation.

Browsers are efficient in minimizing this overhead, but they cannot eliminate it entirely - a web application still has its own part in the responsibility of minimizing resource transfer overhead.

The overhead associated with fetching application resources is relatively static when compared to the size of a given resource. Fetching a 1 KB file incurs similar overhead to fetching a 100 KB file.

Overhead can therefore be minimized in two ways: by decreasing the total number of resources, and by increasing the size of a single resource. Web applications can achieve both by layering and bundling related resources.

A single layer should contain the set of resources related to particular functionality within an application. When a user accesses the functionality, all resources in the layer are likely to be loaded around the same time. Everything comprising a single layer can then be bundled together into a single file for more efficient delivery to the user.

[link to dependencies, cli docs] Dojo’s build pipeline makes use of webpack for the process of bundling layered resources, and supports a level of automation to simplify webpack configuration requirements.

##### Automated Layering

[link to routing]When using Dojo’s Routing system, applications can benefit from automatic layering and bundling. An application’s top-level routes each become a separate layer, and Dojo’s build system will automatically generate bundles for each subsection.

Dojo applications can therefore benefit from layer separation and resource bundling without needing any additional tool chain configuration. There is a tradeoff with this automation in that common dependencies shared across multiple layers are inlined and therefore duplicated within each bundle.

##### Declarative Layering

[link to app dev lifecycle]Complex applications may require more fine-grained control over their layer/bundle definitions. For example, if an application has a set of shared dependencies that are used across multiple routes - rather than inlining/duplicating the dependencies within each route’s bundle, it may be desirable to extract the shared dependencies to their own bundle which can be lazily-loaded on first reference.

An application can designate its own set of bundles, together with a list of all resources that comprise each bundle. This is accomplished via the bundles section within an application’s .dojorc build configuration file. Dojo’s build pipeline will behind-the-scenes replace any direct references to widgets contained within a designated bundle into local registry entries that will lazily-load the bundle. This means developers can achieve efficient lazy resource loading exclusively via bundle definitions, without needing to deal directly with a Registry instance.

#### Performant Rendering

[TODO - renderer, diffing/partial re-rendering, vdom, v(), w()]
Dynamic website content - including JavaScript - has been a part of the web for many years. Websites have long been able to include scripts that manipulate the DOM to add, update or remove content. The origin of the web, however - and what remains one of its key features today - is a foundation on static content. Browsers’ DOM implementations have been optimised over time to render static document content as efficiently and quickly as possible to end users.

As more complex web applications have appeared in recent years, browsers have answered with DOM performance optimizations that favor more dynamic content. However, in order to render their user interfaces, web applications still need to interact with an imperative API that has remained mostly unchanged for decades. Modern web applications designed around reactive data propagation need a more efficient way of translating their user interfaces into a webpage’s DOM.

##### Intersection Observer

[TODO]

##### Resize Observer

[TODO]

### Internationalization (i18n)

[TODO]
-   Describe need for global apps
-   Managing message bundles - how does this relate to layering/bundling? Are msg bundles also layered?
-   Link to i18n ref section

### Accessibility (a11y)

[TODO]
-   Link to WCAG/WAI-ARIA
-   Describe dojo’s implementation of attributes, etc
-   Does Dojo’s impl relate to its theming at all?

### Progressive Applications

[TODO]
-   Benefits: Closer to native application experience for users without the cost of writing device-native application code
-   PWA “for free” with dojorc entry
-   notifications/push/sync/messaging APIs & service workers - stuff outside the main app code

## Security

[TODO]
-   Expand on the tenet of ‘fix errors as early as possible’ - malicious actors could exploit errors & gain access to functionality or data that they should not have access to
-   Briefly describe what Dojo does around security & what other concepts users will need to ensure they cover in their own apps
-   Highlight areas the framework can’t address, & where app devs will need to think about their own security implementations

## Application Development Lifecycle

[TODO]
-   [mostly the ‘setting up a dev environment’ advanced tut]
-   Dev vs prod builds
-   .dojorc config
-   Watching
-   Testing? Should the above section be here rather? Or just link to it when describing the test phase.
