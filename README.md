jasmine-beforeAll
=================

Tested against [Jasmine](http://pivotal.github.io/jasmine/) 1.3.1.

The [Jasmine test framework](http://pivotal.github.io/jasmine/) test framework provides `beforeEach()` and `afterEach()` hooks
(equivalent to RSpec's `before(:each)` and `after(:each)`)
to allow executing setup and teardown code before executing a test (`it(...)`).
For tests where setup/teardown is an expensive operation (for example populating a database with test data) this
can lead to terribly inneficient tests.
It can also lead to test design that that encourages you to minimimze
the number of `it(...)` tests which obviates the descriptive benefits of BDD tests.
Finally it can lead to bloated testuites that take so long to execute that they aren't run frequently enough...

This script patches [Jasmine JS](http://pivotal.github.io/jasmine/) to add support for beforeAll() and afterAll() hooks,
(equivalent to RSpec's `before(:all)` and `after(:all)`).

**Caveats:**

  * Monkey-patches internal Jasmine methods (`jasmine.Spec.prototype.addBeforesAndAftersToQueue`,
`jasmine.Suite.prototype.finish` and `jasmine.Runner.prototype.finishCallback`),
so it'll possibly break in future Jasmine releases.
  * Not yet tested with Node.
  * Not yet tested with async setup/teardown.
  * Not yet tested when setup/teardown callback throws.

### Why aren't they built in into Jasmin?

Excellent question!
I'm not sure what the official reason is, but this  [Jasmine pull request](https://github.com/pivotal/jasmine/pull/56)
has a good discussion thread on the need for these hooks.
I suspect that [this comment](https://github.com/pivotal/jasmine/pull/56#issuecomment-774091) might be the
justification for not including it in Jasmine.  While I agree that using `before/afterEach` nicely isolates
tests from each other, it's simply not pragmatic in many situtations.

Using `before/afterAll` hooks takes more discipline because you have to be careful to ensure that your
test blocks are idempotent.  That is, executing a test block shouldn't affect other test blocks that
depend on the same `before/afterAll` setups.

### Using in Browser:

Include `jasmine-beforeAll` *after* including the main `jasmine.js` script:
```
  ...
  <link rel="stylesheet" type="text/css" href="jasmine.css">
  <script type="text/javascript" src="jasmine.js"></script>
  <script type="text/javascript" src="jasmine-beforeAll.js"></script>
  <script type="text/javascript" src="jasmine-html.js"></script>

  <!-- include source files here... -->
  ...

  <!-- include spec files here... -->
  ...
```

### Using in Tests

The `beforeAll` and `afterAll` hooks are similar to Jasmine's `beforeEach` and `afterEach` hooks,
except that they only execute once.  Use them for doing (expensive) setup/cleanup operations that
can be shared between (idempotent) tests.  They can be used at the top-level or nested within test
suites (`describe()` blocks).  They are executed in the order they are declared.

When running a test, the `beforeAll` callbacks it depends are executed (once) *before* any `beforeEach`
callbacks for the test.
Conversely, the `afterAll` callbacks are executed (once) *after* any `afterEach` callbacks for the *last*
test that depends on them.

The `before/afterAll` callbacks are executed lazily.  That is, if you don't execut a test that depends on them,
they will not be executed, either.

### Example

Running the following spec:

```
describe("Top", function() {
  beforeEach(function() { console.log("Top Setup"); });
  beforeAll(function() { console.log("Top Before All"); });
  afterAll(function() { console.log("Top Cleanup"); });

  describe("Example 1", function() {
    beforeEach(function() { console.log("Example 1 Before Each"); });
    afterEach(function() { console.log("Example 1 After Each"); });
    beforeAll(function() { console.log("Example 1 Setup"); });
    afterAll(function() { console.log("Example 1 Cleanup"); });

    it("Test 1", function() {
      console.log("Test 1");
    });

    it("Test 2", function() {
      console.log("Test 2");
    });
  }); // Example 1

  describe("Example 2", function() {
    beforeAll(function() { console.log("Example 2 Setup"); });
    afterAll(function() { console.log("Example 2 Cleanup"); });

    it("Test 3", function() {
      console.log("Test 3");
    });

    it("Test 4", function() {
      console.log("Test 4");
    });
  }); // Example 2

});
```

will genegerate the following console output:
```
Top Setup
Example 1 Setup
Top Before Each
Example 1 Before Each
Test 1
Example 1 After Each
Top Before Each
Example 1 Before Each
Test 2
Example 1 After Each
Example 1 Cleanup
Example 2 Setup
Test 3
Test 4
Example 2 Cleanup
Top Cleanup
```

### License

MIT
