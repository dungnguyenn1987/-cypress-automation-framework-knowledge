# Differences between Cypress and Selenium
<details>
  
- Selenium supports all major languages like C#, Java, Python, JavaScript, Ruby etc.Cypress Supports only JavaScript/Typescript languages
- Selenium Commands are executed through web drivers, Cypress Commands Executed directly on the browser
- Selenium Supports all major browsers chrome,Edge, Internet Explorer, Safari, Firefox. Cypress supports only Chrome, Edge and Firefox
- With Selenium Configuration of drivers and language biding should be done by our own. With Cypress we get ready framework available we just need to install.
- Selenium Appium can we used to test native mobile applications. Cypress doesn’t support any native mobile application testing

</details>

# Advantages of Cypress
<details>
  
- Cypress is NodeJS modern framework so it works seamlessly with Single Page applications and internal Ajax calls.
- Cypress provides Time Travel options, so It takes snapshots of each tests, after execution we can see what happened exactly in each step. We don’t have to do any configuration for this this comes by default with cypress.
- Cypress Debuggability feature provides access to developer tools on the browser so we can debug directly in the browser.
- Cypress Automatically waits for commands and assertions or any animation to complete so most of the time we don’t have to put additional sleep in our tests.
- Since Cypress directly executes commands on browser it is quite faster compared to Selenium based tools.
- Cypress runs tests and executes commands directly on browser so it is less flaky.
- Cypress provides Video Capture option we can record whole set of tests execution.

</details>
  
# Cypress structure
<details>
  
**cypress** folder is main folder located in the project root folder that contains:
- **cypress.json**: specify any custom configuration
- **Fixtures** folder can be used to store our external Json data files and we can use these files in our tests using the command `cy.fixture()`.
- **Integration** folder mainly consists of our actual spec/test files
- **Plugins** folder contains special files that executes in Node before project is loaded, before the browser launches, and during/before/after your test execution (pre processers and post processors)
- **Support** folder contains the special file `index.js` which will be run before each and every test. Support folder can also be used to create utility methods,  Custom commands or global overrides.

</details>
  
# Installation
<details>
  
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

</details>
  
# Core Concepts
<details>
  
* Cypress commands do not **return** their subjects, they **yield** them. Remember: Cypress commands are asynchronous and commands don't do anything at the moment they are invoked, but rather enqueue themselves to be run later. Use aliases and closures to access and store the returned values what Commands yield you. During execution, subjects are yielded from one command to the next, and a lot of helpful Cypress code runs between each command to ensure everything is in order.

* Cypress selectors:
     * CSS Selectors is the selectors supported by Cypress. For using xpath locator, need to install cypress-xpath plugin.
     * Cypress prefer elements with attribute: data-cy, data-test, data-testid
     * Access shadow DOM
 
```
Shadow DOM allows hidden DOM trees to be attached to elements in the regular DOM tree — this shadow DOM tree starts with a shadow root.

<div class="shadow-host">
#shadow-root
<button class="my-button">Click me</button>
</div>

We can access the above shadow dom using below code in cypress
cy.get('.shadow-host').shadow().find('.my-button').click()
```

* Element Interaction
     * Text Content
 
```
<div>Hello&nbsp;world</div>

// Assert on an element's text content:
cy.get('div').should('have.text', 'Hello\u00a0world')

// Get element by/filter by text content
cy.contains('div', 'Hello world')
cy.get('li').filter(':contains("Services")').should('have.length', 2)

// Work with text content
cy.get('div').should(($div) => {
  const text = $div.text()
   ...
})
cy.get('div').invoke('text').then(parseFloat).should('be.gt', 10)
cy.get('div')
  .invoke('text')
  .then((text1) => {
    // do more work here
  })
``` 

   * In some cases, your DOM element will not be actionable. Cypress gives you a powerful `{force:true}` option you can pass to most action commands.
```
cy.get('.checkbox').check({ force: true })
```
     
   * Iterate table and list of rows and data
```
cy.get('#customers').get('tbody tr td').each(($ele)=>{
   cy.log($ele.text());
})
```
     
* Debug commands
    * cy.pause()
    * cy.debug()
    * debugger

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
* Cypress by default doesn’t support, working with new windows so we need to follow different approach by disable attribute to open new tab.
```
cy.get('a[href*="google"]').invoke('removeAttr', 'target').click()
```

* Preserve cookies in Cypress: Cypress by default clear the cookies after every test. For preserve cookies
```
beforeEach(() => {
      Cypress.Cookies.preserveOnce('session_id', 'remember_token')
});
```

*  Custom commands in Cypress
```
cypress/support/command.js
Cypress.Commands.add("login", (username, password) => {
  cy.get("#username").type(username);
  cy.get("#password").type(password);
  cy.get("#login").click();
});

test file
cy.visit("http://mysite.com/login.html");
cy.login('Myusername','My Passowrd');
```

* Handle window alert in cypress
```
Cypress.on("uncaught:exception", (err, runnable) => {
// returning false here prevents Cypress from failing the test
return false;
});

cy.visit("https://test.com/popups");
cy.get('[name="alert"]').click(); //This will handle the pop up internally
```

# Variables and Aliases
* You CANNOT assign or work with the return values of any Cypress command. To access what each Cypress command yields you use `.then()`.

```js
cy
  // Find the el with id 'some-link'
  .get('#some-link')

  .then(($myElement) => { //When the previous command cy.get() resolves, it will call your callback function with the yielded subject as the first argument.

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

</details>

# API Test
<details>
  
* Make HTTP request

```
// GET request
cy.request('http://localhost:8080/db/seed')

// POST request
cy.request('POST', 'http://localhost:8888/users/admin', { name: 'Jane' }).then(
  (response) => {
    // response.body is automatically serialized into JSON
    expect(response.body).to.have.property('name', 'Jane') // true
  }
)
```

* Request Polling
Call cy.request() over and over again
  
```
// a regular ol' function folks
function req () {
  cy
    .request(...)
    .then((resp) => {
      // if we got what we wanted

      if (resp.status === 200 && resp.body.ok === true)
        // break out of the recursive loop
        return

      // else recurse
      req()
    })
}

cy
  // do the thing causing the side effect
  .get('button').click()

  // now start the requests
  .then(req)

```

* Download a PDF file

```
cy.request({
  url: 'http://localhost:8080/some-document.pdf',
  encoding: 'binary',
}).then((response) => {
  cy.writeFile('path/to/save/document.pdf', response.body, 'binary')
})
```

* Get Data URL of an image

```
cy.request({
  url: 'https://docs.cypress.io/img/logo.png',
  encoding: 'base64',
}).then((response) => {
  const base64Content = response.body
  const mime = response.headers['content-type'] // or 'image/png'
  // see https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs
  const imageDataUrl = `data:${mime};base64,${base64Content}`
})
```

</details>

# Cucumber test
<details>
  
1. Install plugin:
`npm i @badeball/cypress-cucumber-preprocessor`
`npm i @bahmutov/cypress-esbuild-preprocessor`

2. Configure Cypress to use the plugins

```
// cypress/plugins/index.js
const createEsbuildPlugin = require('@badeball/cypress-cucumber-preprocessor/esbuild').createEsbuildPlugin
const createBundler = require('@bahmutov/cypress-esbuild-preprocessor')
const addCucumberPreprocessorPlugin = require('@badeball/cypress-cucumber-preprocessor').addCucumberPreprocessorPlugin
module.exports = async (on, config) => {
  
  const bundler = createBundler({
    plugins: [createEsbuildPlugin(config)],
  })
  await addCucumberPreprocessorPlugin(on, config)
  on('file:preprocessor', bundler)

  return config
}
```

3. Modify the default configuration of the Cucumber preprocessor

```
// package.json
"cypress-cucumber-preprocessor": {
  "stepDefinitions": [
    "cypress/e2e/[filepath]/**/*.{js,ts}",
    "cypress/e2e/[filepath].{js,ts}",
    "cypress/support/step_definitions/**/*.{js,ts}",
  ]
}
```


4. Feature test

```
# cypress/e2e/board.feature
Feature: Board functionality

  Scenario: Creating a <listName> list within a board
    Given I am on empty home page
    And There are the existing boards
            | id    | name          | status |
            | SL001 | Shopping list | ACTIVE |
    When I type in "<boardName>" and submit
    And Create a list with the name "<listName>"
    Then I should be redirected to the board detail

  Examples:
      | boardName     | listName         |
      | Shopping list | Groceries        |
      | Rocket launch | Preflight checks |
```

5. Step definitions

```
# cypress/e2e/board.js (same folder of feature or in the config path)
import { When, Then, Given } from "@badeball/cypress-cucumber-preprocessor";
import homePage from "../../pageObjects/home-page"

Given("I am on empty home page", () => {
  homePage.open();
});

Given(/^There are the existing boards$/, function (datatable) {
  datatable.hashes().forEach(data => {
    // call funtion to create board by data.id, data.name, data.status
  })
})
When("I type in {string} and submit", (boardName) => {
  homePage.typeName(`${boardName}{enter}`);
});

Then("I should be redirected to the board detail", () => {
  cy.location("pathname").should('match', /\/board\/\d/);
});

```

6. Page Objects

```
# cypress/pageObjects/homepage.js
import basePage from "base-page"

class HomePage {

  elements = {
    txtId: () => cy.getBySel('id'),
    txtName: () => cy.getBySel('name'),
    lblBoard:() => cy.getBySel('boardName')
  }
  open() {
     cy.visit("/")
  }
  getId() {
    this.elements.txtId().invoke('attr', 'value').as('id')
  }

  typeName(value) {
    basePage.clearAndType(this.elements.txtName(), value);
  }
}
const homePage = new HomePage()
export default homePage

```

</details>

# Troubleshoot
<details>
  
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

</details>
