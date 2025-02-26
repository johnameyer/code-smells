# Generic Smells

## Importance of Readability

Readability is the hallmark of well structured and thought-through code. Readable code helps not only your reviewer but anyone who ends up visiting your code in the future - potentially you in a few years when you don't remember the particulars!

Long classes, functions, and parameter lists all increase not only the mental load to understand the logic but also increase the complexity and number of tests needed to ensure that component works appropriately.

Aside from the general best object oriented practices, the following help to make code readable.

## Don't Repeat Yourself

A.k.a. DRY

Whenever code is repeated in multiple places, this normally can be a maintainance issue as both places will need updating in the future and therefore can end up drifting in implementation due to oversight.

A normal heuristic is that two times is normally fine but the third time means it is time to refactor out to a common place.

* Introduce variables to avoid duplication
* Use destructuring to avoid accessing multiple fields on same object

```ts
const link = my[deeply].nested.field?.link;
const linkText = my[deeply].nested.field?.linkText;
```

refactored to

```ts
const field = my[deeply].nested.field;
const link = field?.link;
const linkText = field?.linkText;
```

refactored to

```ts
const {link, linkText} = my[deeply].nested.field ?? {};
```

This goes not only for exact duplicates but for more abstract concepts. One example might be retry logic:

```java
var value = null;
for(int i = 0; i < 3; i++) {
  value = tryGetValue();
  if(value != null) {
    break;
  }
}
```

The particulars of the type of the value, the number of retries, the backoff logic, the predicate could all be different in different usage sites. However, the core logic of the function could be broken out using generics:

```java
<T> Optional<T> retry(Supplier<Optional<T>> function, Predicate<T> criteria, RetryConfig config) {
  for(int i = 0; i < 3; i++) {
    var value = tryGetValue();
    if(value.isPresent() && criteria.test(value.get())) {
      return value;
    }
  }
  return Optional.empty();
}
```

## Single Level of Abstraction

Functions should ideally be written at a single level of abstraction. This means that functions should generally read at a high level almost similar to as if you were to write out psuedocode.

Writing functions that operate at different levels of granularity increases the burden to understand the logic in a function at a glance - in no small part due to the length of the function.

Prime examples often show up in tests:

```java
void testCanUpdateField() {
  Car car = Car.builder()
    .withMake("honda")
    .withModel("crv")
    .withColor(Color.BLUE)
    .withFeatures(
      List.of(
        Feature.AWD
      )
    )
    .build();
  database.store(car);

  UpdateRequest request = UpdateRequest.builder()
    .withMake("honda")
    .withModel("crv")
    .withColor("red")
    .build();
  var result = handler.update(request);

  assertEquals(request.color, result.color);

  var storedCar = database.get(car.make, car.model);

  assertEquals(request.color, storedCar.color);
  assertEquals(car.features, storedCar.features);
}
```

Instead we should have helper functions that we could then re-use for other tests as well:

```java
void testCanUpdateField() {
  Car car = buildTestCar();
  database.store(car);

  var newColor = "red";
  var result = handler.update(buildUpdateColorRequestForCar(car, newColor));

  assertEquals(newColor, result.color);

  var storedCar = database.get(car.make, car.model);

  assertEquals(newColor, storedCar.color);
  assertEquals(car.features, storedCar.features);
}

Car buildTestCar() {
  return Car.builder()
    .withMake("honda")
    .withModel("crv")
    .withColor(Color.BLUE)
    .withFeatures(
      List.of(
        Feature.AWD
      )
    )
    .build();
}

UpdateRequest buildUpdateColorRequestForCar(Car car, Color color) {
  return UpdateRequest.builder()
    .withMake(car.make)
    .withModel(car.color)
    .withColor(color)
    .build();
}
```

## Meaningful Naming

Giving names to values is one of the easiest ways to communicate logic without needing to resort to comments.

(Note that in Java we would normally use BigDecimal instead of integers for monetary values)

Example:

```java
int calculateOrderCost() {
  int totalCost = shoppingCart.stream().mapToInt(Item::price).sum();

  // Apply large order discount
  if(shoppingCart.size() > 5) {
    totalCost = (int) Math.ceil(totalCost * 4F / 5);
  }

  // Apply free shipping
  if(totalCost < 50) {
    totalCost += 10;
  }
}

```

could be re-written as

```java
int calculateOrderCost(List<Item> shoppingCart) {
  int totalCost = shoppingCart.stream().mapToInt(Item::price).sum();

  boolean hasLargeOrderDiscount = shoppingCart.size() > 5;
  if(hasLargeOrderDiscount) {
    totalCost = (int) Math.ceil(totalCost * 4F / 5);
  }

  boolean hasFreeShipping = totalCost > 50;
  if(!hasFreeShipping) {
    totalCost += 10;
  }
}
```
