# hair-mini.js

Minimal single file version.

## Overview
A Javascript library for creation and update of DOM elements via component function composition (but not purely FRP style)

Taking a few of the things I think people like about FRP libraries and removing others. Less concerned with managing a functional reactive component state isolated from the DOM, and more with responding to a separate model/world state. Some of these choices make more sense from the point of view of games, simulations, and long lived state, rather than more transactional websites with fresh queries per page.

hair.js presupposes an app state or model, that is maintained and updated over the life of the program in response to user actions or network driven updates. The developer creates composable functions that take the current state of the app, or a part of that state, and recursively produce "component specifications" describing the "view" produced for that state. The main functions for producing this view specification allow the developer to mix text, HTML elements, and child elements including functions, arrays, or explicitly composed sub-components, allowing a fluid and natural shape to the view functions. A render pass then creates or updates DOM elements to match that specification, hooking DOM listeners and other callbacks directly to the rendered elements. 

UI state that is more transient than the underlying app state or model should remain the DOM's business where possible, and a "render context" representing the instantiation of a component specifications into the DOM provides a scoped object to handle the live connection between the view specifications and DOM.

hair.js is a single file module, that can be included in a project without any required build steps.

### Goals

 * Functions as composeable components that generate and update DOM state
 * The same component code can create or update the DOM
 * Direct access to the dom elements is supported by callbacks
 * Update via re-render of components OR via incremental change to DOM elements (ie. perhaps depending whether it is model state that has updated, or UI state)
 * Maintain a single file, minimal, version of hair.js focused only on HTML render

### Trade offs / non-goals

 * Using a convention whereby the identity of state objects matters (ie. works better if updates to the base state or model are incremental updates to a mutable object, rather than a fresh state object returned from a query each time)
 * Not attempting to be a full or perfect implementation of a virtual dom or provide a genuine functional reactive paradigm
 * Additional integrations will not be added directly to this minimal version
 
### Efficiency or reducing re-renders

 * If a component does not directly use the state passed to it to determine its output (perhaps only sub-components of that component use it), then execute that component function directly when including it (rather than add it by reference to be executed later).
 * Prefer to re-use state or model objects across updates to help the library re-use DOM elements, especially lists of objects
 
## Usage

The main usage is to create functions that describe an HTML layout using the functions provided by the library. These functions perform the basic work of creating and updating the DOM.

The output of the function can consist of single elements, or arrays of elements. Whenever elements are provided, they can be provided directly or as function references to create other components. The render process recursively resolves arrays and functions into direct elements.

All element functions allow three arguments in any order, a string for any text content, an object to set properties on the created element, and an array or single child element.

 * Objects arguments are assumed to be properties to apply to element eg { class: 'parent', disabled: true }
 * Number or string arguments are assumed to be text content of the element
 * Arrays or any single recognised component spec is assumed to be a child (recursively forming the full component spec)

Event listeners are instantiated as a special kind of child element (hair.listen), and there is special handling for lists of objects (hair.compose).

See the included example TODO list to see examples of all of these in use.

Snippet:

	// bring the hair library in accessed as h.*
	import * as h from './js/hair.js';

	export default function app(model) {
		return [
			h.h1(model.name),
			
			h.div([
				
				// add this component in by immediately calling a function to return 
				itemCount(model.items.length),
				
				// use the compose function to request a managed render of a list of objects with a component
				h.ol({ _id: 'list' }, h.compose(model.items, (item) => displayItem(model, item))),
			]),
			
			h.div({ _class: 'add_item_area' }, [
				h.p('Add new items below'),
				
				// add this component in by providing a function to be called with the same state object
				addItem,
			]),
		];
	}

### Viewing the example project

Serve the root of repo using any local web server, eg.

	cd hair
	python3 -m http.server

Open the example folder in your browser

	open http://localhost:8000/example_todo_list/

	open http://localhost:8000/example_quiz/
 
### Special property handling

During instantiation or update of DOM elements, recognised property names are given special treatment when applied to the element.

 * \_id, this property sets a reference to the element on the context object, eg. \_id = "textbox" => sets a reference to this element at context.textbox
 * class, this property when given a name, or an array of names, will update the classList of the element to match
 * style, when an object value is applied to this property, the values of that object will be merged into the element style object, rather than replacing it

*setPropertyHandler* can be used to provide your own property handlers globally for your project.

## Reference

	state/model object ----------------*-----------------*----------------*-->
	                                   |                 |                |
	DOM parent -------------- -|       |                 |                |
	                           |--> RenderContext -------------------------------------->
	[ComponentSpecifications] -|    <render>         <update>         <update>
	                               RenderPhase      RenderPhase      RenderPhase


### Component Specifications

Strings, numbers, arrays and functions are all treated transparently as component specifications. Arrays and functions are iterated or executed recursively to produce explicit component specifications.

Otherwise any object that inherits from ComponentSpecification is treated as specification for creating or updating the DOM during render.

"element" specifies an HTML element, and many common HTML tags have pre-created factory functions to make this read better, eg. hair.div, hair.p

Some "special" components interact with the render process or the created DOM elements to provide event listeners, time based functionality, or to compose sub-components for objects within the state, or lists of objects, eg. compose, listen, onDelay, or onAttach.

#### Reuse keys

Many components allow single or multiple "reuse keys" to provided in their spec, these keys can be any value, including strings and objects. When updating an already rendered view, DOM elements will only be reused if reuse keys match, if no reuse keys are provided then DOM elements will be re-used optimistically.

### RenderContext

A RenderContext tracks the rendered output of a component in the DOM, allowing the rendering to be updated with reuse. Where sub-components are composed within the rendering they have their own render context. Parameters can be set on the context to allow sharing of information between components at the same level. Sub components can read values inherited from their parent context. Arbitrary events can be broadcast to all components attached to a render context tree.

The RenderPhase tracks objects created or updated during the latest render or update, facilitating reuse and managing a list of current RenderAttachments, anything attached from the rendering process into 

By default the render context will "watch" the state or model object provided to it, and if this object is "signalled" it will automatically re-render on the next animation frame.

### Signals
 
 * watch
 * signal
 * removeWatcher
 
Watchers are held by weak reference only, and disposed objects are ignored.

### Timers

 * delay (seconds, action, owner)
 * timer (seconds, action, owner)
 * onNextFrame (action, owner)
 * onEveryFrame (action, owner)
 * cancel (owner)
 
Timer events are held by weak reference only, and disposed objects are ignored.

### Disposal

 * markObjectAsDisposed (obj)
 * isObjectDisposed (obj)

## License

MIT License, Copyright (c) 2024 Samuel Baird