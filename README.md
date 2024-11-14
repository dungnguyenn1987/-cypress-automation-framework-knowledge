# Installation
* Nodejs 18.x, 20.x, 22.x and above 
    * Download https://nodejs.org/en
    * Check installed `node -v`
* Cypress Project
    * Create folder for your Cypress Project
    * Install `npm install cypress --save-dev`
    * Open `npx cypress open` or call config in package.json `npm run cy:open`
    * Select Browser > Create New Spec >  Run
* Test Runner
![image](https://github.com/user-attachments/assets/923c12f8-66cd-4c49-b4b0-155e8719f098)

    We can see Cypress output additional information in the console when opening Dev Tools:

    * Command (that was issued)
    * Yielded (what was returned by this command)
    * Elements (the number of elements found)
    * Selector (the argument we used)

![image](https://github.com/user-attachments/assets/a56acb8c-c49d-4390-a53a-98e7e8fbdd0e)

# Core Concepts
* Cypress commands do not **return** their subjects, they **yield** them. Remember: Cypress commands are asynchronous and commands don't do anything at the moment they are invoked, but rather enqueue themselves to be run later. Use aliases and closures to access and store the returned values what Commands yield you. During execution, subjects are yielded from one command to the next, and a lot of helpful Cypress code runs between each command to ensure everything is in order.

* Debug commands
    * cy.pause()
    * cy.debug()
    * debugger
* Cypress prefer elements with:
    * data-cy
    * data-test
    * data-testid

* Cypress wraps all DOM queries with robust **retry-and-timeout (automatical wait)**  logic that better suits how real web apps work in default timeout
    * Custom global timeout https://docs.cypress.io/app/references/configuration#Timeouts
        * in cypress.config.js: the `defaultCommandTimeout` setting.
        * in command line: `cypress run --config defaultCommandTimeout=10000`
    * Custom specific timeout `cy.get('.my-slow-selector', { timeout: 10000 })`
    * Disable retry: override { timeout: 0 }

* Types in Cypress
    * **Queries** link up, retrying the entire chain together. **Only queries can be retried**
    * **Assertions** are a type of query that's specially displayed in the command log.
    * **Commands** 
        * are Non-queries and only execute once. Because they could potentially change the state of the application under test (e.g: .click() action command). 
        * should be at the end of chains (after alias as well)
        * They do NOT form the subject for later commands. 
        * The implicit assertions built into every command (https://docs.cypress.io/app/core-concepts/interacting-with-elements)

```js
it('creates an item', () => {
  // Non-query commands only execute once.
  cy.visit('/')

  // The queries .get() and .find() link together, forming the subject for
  // the non-query `.type()`.
  cy.get('.header').find('.new-todo').type('todo A{enter}')

  // Two queries and an assertion chained together (*)
  cy.get('.todoapp').find('.todo-list li').should('have.length', 1)
})
```
* Retry-ability
Consider the above script (*):  `cy.get()` queries the application's DOM, finds the elements that match the selector, and then passes them to .find('.todo-list li'). `.find()` locates a new set of elements, and passes them to .should(). `.should()` then asserts on the list of found elements 

    * ✅ If the assertion passes, then .should() finishes successfully.

    * 🚨 If the assertion fails, then Cypress will requery the application's DOM again - starting from the top of the chain of linked queries. It will look for elements that match .get().find() again, and re-run the assertion. If the assertion still fails, Cypress continues retrying until the timeout is reached.

# Variables and Aliases
* You CANNOT assign or work with the return values of any Cypress command. To access what each Cypress command yields you use `.then()`.

```js
cy
  // Find the el with id 'some-link'
  .get('#some-link')

  .then(($myElement) => { //When the previous command cy.get('button') resolves, it will call your callback function with the yielded subject as the first argument.

    // grab its href property
    const href = $myElement.prop('href')

    // strip out the 'hash' character and everything after it
    return href.replace(/(#.*)/, '')
  }) 

// if you wish to continue chaining commands after your .then(), you'll need to specify the subject you want to yield to those commands, which you can achieve with a return value

  .then((href) => { 

    // href is now the new subject
    // which we can work with now
  })


```

* Chains of Commands: Cypress uses to chain commands together. It manages a Promise chain on your behalf, with each command yielding a 'subject' to the next command, until the chain ends or an error is encountered. If you perform an action, like navigating the page, clicking a button or scrolling the viewport, **end the chain** of commands there and start fresh from cy. NOTE: the `.then()` command breaks the chain of queries - nothing before it re-runs.
``` js
cy.get('.main-container') // Yields an array of matching DOM elements
  .contains('Headlines') // Yields the first DOM element containing content
  .click() // Yields same DOM element from previous command.
```

```js
// WRONG: this test will not work as intended
cy.get('[data-testid="random-number"]') // <div>🎁</div>
  .invoke('text') // "🎁"
  .then(parseFloat) // NaN and not retry
  .should('be.gte', 1) // fails
  .and('be.lte', 10) // never evaluates

// SHOULD BE
cy.get('[data-testid="random-number"]').should(($div) => {
  // all the code inside here will retry
  // until it passes or times out
  const n = parseFloat($div.text())
  expect(n).to.be.gte(1).and.be.lte(10)
})
```

* Sharing Context with aliases `.as()` command
```js
beforeEach(() => {
  // alias the $btn.text() as 'text'
  cy.get('button').invoke('text').as('text')

  // alias the users fixtures
  cy.fixture('users.json').as('users')

  // alias intercept, request
  cy.request('https://jsonplaceholder.cypress.io/comments').as('comments')
})

it('has access to text', function () { // NOTE:  Accessing aliases as properties with this.* will not work if you use arrow functions (the lambda "fat arrow" syntax () => {}) for your tests or hooks.

  // 2 WAYS of accessing aliases
  // 1. using the this.* syntax. Value was set when the alias was first stored
  this.text // is now available
  const user = this.users[0] // access the users property
  cy.get('header').should('contain', user.name)

  // 2. use  cy.get() command with the special '@' syntax. Any queries are re-evaluated every time the alias is accessed (latest stored). Need to use this way to work with alias DOM elements to re-query the DOM
  cy.get('@comments').should((response) => {
  if (response.status === 200) {
      expect(response).to.have.property('duration')
    }
  })
})
```

# Troubleshoot
## Click but no action
* Likely Cause
    * The application was slow to respond, while Cypress was fast to act
    * Example when the framework shows the calendar modal, it starts attaching the event listeners to the DOM elements to process the actual click on a date element (e.g: show the selected date to textbox). However before the event listeners are attached, Cypress manages to find the DOM element with the required date and click on it 
* Possible Solutions
    * slow down the tests `.wait(500)` before action
    * using an 3rd party Cypress plugin called `cypress-pipe` with a custom command `.pipe(click)`. Be careful to keep clicking on the element will only work for some applications. If our application saved records into the database on each click, retrying click would not be a good option

    ```js
    const click = $el => $el.click()

    cy.get('.owl-dt-popup')
    .should('be.visible')
    .contains('.owl-dt-calendar-cell-content', dayRegex)
    .pipe(click) // keep clicking until the calendar modal is closed and the particular date element we are clicking on is no longer visible.
    .should($el => {
        expect($el).to.not.be.visible
    })
    ```

