jasmine-beforeAll
=================

Patches [Jasmine JS](http://pivotal.github.io/jasmine/) to support beforeAll() and afterAll() hooks,
providing functionality suggested by this  [Jasmine pull request](https://github.com/pivotal/jasmine/pull/56).
Tested with Jasmine 1.3.1.

Caveat: Monkey-patches internal Jasmine methods, so it'll possibly break in future Jasmine releases.

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

This script adds `beforeAll` and `afterAll` hooks to your tests,
equivalent to RSpec's `before(:all)` and `after(:all)` hooks.

These hooks are similar to Jasmine's `beforeEach` and `afterEach` hooks, except that they only execute once.
They are useful for doing expensive setup/cleanup operations that can be shared between tests.  They are also
useful when specs in a test suite contain invariant assertions that don't require automatic cleanup after each
spec.

When running a test, `beforeAll` callbacks will be executed (once) *before* `beforeEach` callbacks.
Conversely, `afterAll` callbacks are executed (once) *after* `afterEach` callbacks.

### Example

Running the following spec:

```
describe("Top", function() {
  beforEach(function() { console.log("Top Setup"); });
  beforAll(function() { console.log("Top Before All"); });
  afterAll(function() { console.log("Top Cleanup"); });
  
  describe("Example", function() {
    beforEach(function() { console.log("Example Before Each"); });
    afterEach(function() { console.log("Example After Each"); });
    beforAll(function() { console.log("Example Setup"); });
    afterAll(function() { console.log("Example Cleanup"); });
    
    it("Test 1", function() {
      console.log("Test 1");
    });

    it("Test 2", function() {
      console.log("Test 2");
    });

  });
  
});
```

will genegerate the following console output:
```
Top Setup
Example Setup
Top Before Each
Example Before Each
Test 1
Example After Each
Top Before Each
Example Before Each
Test 2
Example After Each
Example Cleanup
Top Cleanup
```

### License

MIT
