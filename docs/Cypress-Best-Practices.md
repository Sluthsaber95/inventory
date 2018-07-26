---
id: Cypress-Best-Practices
title: Best Practices
---

###### Last Updated: 10/7/2018

##### Please note that these optimisation techniques are focused on `Cypress.io` and may not work for other UI Testing libraries such `Test Cafe` or `Selenium` with counterparts.

## What should I test?
Recommend watching the Brian Mann (Cypress Founder) video: [I see your point, but...(3:43 - 7:34)](https://www.youtube.com/watch?v=5XQOK0v_YRE#t=3m43s); Brian goes each common cases and what to for and some cases for what not to test for as well.


## Organising Tests
Each Spec should describe an individual page, this provides each spec to test in isolation other others

__One Step Further__  - decompose each spec down to smaller files to group the primary data model objects, between the pages.

Example below
```
/intergration
|- /articles
|   |- article_details_spec.js
|   |- article_new_spec.js
|   |- article_list_spec.js
|
|- /author
|   |- author_details_spec.js
|
|- /shared
|   |- header_spec.js
|
|- /user
    |- login_spec.js
    |- register_spec.js
    |- settings_spec.js
```

To log

Watch the Brian Mann (Founder of Cypress.io) Video called [I see your point, but...(7:35 - 8:30)](https://www.youtube.com/watch?v=5XQOK0v_YRE#t=7m35s)


## General Rules for Testing

### Singular Test Units

- __SRP (Single Responsibility Principle)__ - iterate over one test at a time, meaning that test should be testing only one thing
- __Small Testable Units__ - each unit should be isolated from each other, with the aim of decoupling test code as much as possible

Breaking tests up into the smallest testable unit; has the added bonus of making tests easier to readable; more maintainable.
<br>

```
// Do
...
it('greets with Sign in', () => {
  
})
it('links to #/register', () => {

})
...
```
```
// Don't
...
it('greets with Sign in, which should link to #/register ', () => {
  
})
...
```

<br>

### Use the /support directory
- __DRY (Don't Repeat Yourself)__ - Code should not be duplicated across different specs
- __Avoid Brittle Selectors__ - the code under test, may have HTML selectors, that may change very often, thus use selectors that will change less often

With the advice above, is there any easier way of applying what's been learnt? 

Well as it suggested in the title, the `/support` directory is an abstraction that aids in the 'decoupling' and 'reusability' in the code; scaffolded structure of cypress environment that is initially receive upon installation; should look like the structure below

```
/cypress
|- /integration
|   |- login_spec.js
|   |- settings_spec.js
|
|- /support
    |- index.js
    |- commands.js
```


<br>

### commands.js
Allow for actions to coded < Continue Here>
Really simple utilise the `Cypress.Commands.add()` method, and include a set of Cypress commands that you have mostly previously used for a file.

```
// support/commands.js

Cypress.Commands.add('login', () => {
  cy.visit('/#/login')
  cy.get('[data-test=email]').type('joe@example.com')
  cy.get('[data-test=password]').type('joe{enter}')
  cy.hash().should('eq', '#/')
})
```

```
// integration/settings_spec.js

describe('/settings', () => {
  beforeEach(() => {
    cy.login()
    cy.visit('./#/settings')
  })
  it('greets with Your Settings', () => {
    cy.contains('h1', 'Your Settings')
  })
})

```

cy.request uses the browser agent and cookie store
```
Cypress.Commands.add('login', () => {
  cy.request({
    method: 'POST', 
    url: 'http://localhost:3000/api/usrs/login',
    body: {
      user: {
        email: 'joe@example.com,
        password: 'joe' 
      }
    }
  })
  .then((response) => {
    window.localStorage.setItem('jwt', response.body.user.token) 
  })
})
```

Cypress can do this as it is bound to the local storage of you application

Benefits:
  - Tests run a lot faster
  - Less noise given with BEFORE EACH section of each test ran


The aim here is reduce the test feedback loop to a bare minimum, 

- Don't use the UI to build up state, unless that UI is under test - login example
- Set the state directly / programmatically

- Don't use page objects to share UI knowledge
- Write specs in isolation, avoid coupling

- Don't limit yourself trying to act like a use
- You have native access to everything


Cypress you can
- Control Time: cy.clock()
- Stub Objects: cy.stub() - you can stub objects that you have reference too
- Modify Stores: cy.window - utilise our application state and source directly to utilse event and force it to respond programmatically

```
// client/src/store.js
function createStoreWithState( state = {} ){
  const store = createStore(reducer, state)

  // snippet below 
  if (window.Cypress) {
    window.__store__ = store
  }
  return store
}
```

```
// cypress/integration/settings_spec.js
it('can log out programmatically', () => {
  cy
  .window()
  .its('store')
  .then((store) => {
    store.dispatch({ type: 'LOGOUT' })
  })

  cy
  .window()
  .its('localStorage')
  .invoke('getItem', 'jwt')
  .should('not.exist')
})
```

## Optimisations 
[Other Best Practices](https://docs.cypress.io/guides/references/best-practices.html)


