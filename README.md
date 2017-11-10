# By Value Sort
[![Build Status](https://travis-ci.org/matyb/byvalue-sort.png?branch=master)](https://travis-ci.org/matyb/byvalue-sort)

Equivalent functionality to SQL's ORDER BY, but for java. This isn't very performant, if you want that - use a database.

```sql
-- what you would do if have a db available
select tis.* from THING_IM_SORTING tis
join TYPE_OF_REQUEST req on
    req.CODE = tis.REQUEST_CODE
 or (req.CODE is null and tis.REQUEST_CODE is null)
join TYPE_OF_PAYMENT pmt on
    pmt.CODE = tis.PAYMENT_CODE
 or (pmt.CODE is null and tis.PAYMENT_CODE is null)
order by req.ORDER desc, pmt.ORDER desc
```

and what you can do with this library with a lil more ceremony and lousier performance (though you don't need the context switch or infrastructure)

```java
// Java 8 sort by 2 'columns` (values) desc w/ external static definition of value ordering
public static final List<TypeOfRequestEnum> REQUEST_ORDER = ImmutableList.of(
        TypeOfRequestEnum.FIRST, TypeOfRequestEnum.SECOND, TypeOfRequestEnum.THIRD);

public static final List<TypeOfPaymentEnum> PAYMENT_ORDER = ImmutableList.of(
        TypeOfPaymentEnum.CASH, TypeOfPaymentEnum.CHECK, TypeOfPaymentEnum.BOOST_MOBILE);

Function<ThingImSorting, TypeOfRequestEnum> requestExtractor = ThingImSorting::getTypeOfRequest;
Function<ThingImSorting, TypeOfPaymentEnum> paymentExtractor = ThingImSorting::getTypeOfPayment;

// in some method
ByValueSort<ThingImSorting> sorter = new ByValueSort<>(
    new OrderBy<>(requestExtractor, new ListIndexComparator<>(REQUEST_ORDER), OrderBy.DESC),
    new OrderBy<>(paymentExtractor, new ListIndexComparator<>(PAYMENT_ORDER), OrderBy.DESC));

List<ThingImSorting> sorted = sorter.sort(unsorted);
```

or - if you're stuck in java 7 or below

```java
// Java 7 sort by 2 'columns` (values) desc w/ external static definition of value ordering
public static final List<TypeOfRequestEnum> REQUEST_ORDER = ImmutableList.of(
        TypeOfRequestEnum.FIRST, TypeOfRequestEnum.SECOND, TypeOfRequestEnum.THIRD);

public static final List<TypeOfPaymentEnum> PAYMENT_ORDER = ImmutableList.of(
        TypeOfPaymentEnum.CASH, TypeOfPaymentEnum.CHECK, TypeOfPaymentEnum.BOOST_MOBILE);

Extractor<TypeOfRequestEnum, ThingImSorting> requestExtractor = new Extractor<TypeOfRequestEnum, ThingImSorting>(){
    @Override public TypeOfRequestEnum extract(ThingImSorting thing){
        return thing.getTypeOfRequest();
    }
};

Extractor<TypeOfPaymentEnum, ThingImSorting> paymentExtractor = new Extractor<TypeOfPaymentEnum, ThingImSorting>(){
    @Override public TypeOfPaymentEnum extract(ThingImSorting thing){
        return thing.getTypeOfPayment();
    }
};

// in some method
ByValueSort<ThingImSorting> sorter = new ByValueSort<>(
    new OrderBy<>(requestExtractor, new ListIndexComparator<>(REQUEST_ORDER), OrderBy.DESC),
    new OrderBy<>(paymentExtractor, new ListIndexComparator<>(PAYMENT_ORDER), OrderBy.DESC));
    
List<ThingImSorting> sorted = sorter.sort(unsorted);
```

given:
```java
List<ThingImSorting> unsorted = Arrays.asList(
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.CASH),
    new ThingImSorting(TypeOfRequestEnum.THIRD,  TypeOfPaymentEnum.CASH),
    new ThingImSorting(TypeOfRequestEnum.FIRST,  TypeOfPaymentEnum.BOOST_MOBILE),
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.BOOST_MOBILE),
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.CHECK)));
```

when:
```java
List<ThingImSorting> sorted = sorter.sort(unsorted);
sorted.stream().map( ThingImSorting::toJson ).collect(Collectors.toList());
```

given:
```java
List<ThingImSorting> unsorted = Arrays.asList(
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.CASH),
    new ThingImSorting(TypeOfRequestEnum.THIRD,  TypeOfPaymentEnum.CASH),
    new ThingImSorting(TypeOfRequestEnum.FIRST,  TypeOfPaymentEnum.BOOST_MOBILE),
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.BOOST_MOBILE),
    new ThingImSorting(TypeOfRequestEnum.SECOND, TypeOfPaymentEnum.CHECK)));
```

when:
```java
List<ThingImSorting> sorted = sorter.sort(unsorted);
sorted.stream().map( ThingImSorting::toJson ).collect(Collectors.toList());
```

then:
```json
[
    { "typeOfRequest": "THIRD",  "typeOfPayment": "CASH" },
    { "typeOfRequest": "SECOND", "typeOfPayment": "BOOST_MOBILE" },
    { "typeOfRequest": "SECOND", "typeOfPayment": "CHECK" },
    { "typeOfRequest": "SECOND", "typeOfPayment": "CASH" },
    { "typeOfRequest": "FIRST",  "typeOfPayment": "BOOST_MOBILE" }
]
```

Equivalent for group by. Given a collection and key function produces a map of lists containing values grouped by the output of the key function.

```java
@Test
public void testConvertingListWithTwoKeysToAMap() throws Exception {
    assertEquals(ImmutableMap.of("odd", ImmutableList.of(1, 3), "even", ImmutableList.of(2, 4)),
                 CollectionMap.toMap(ImmutableList.of(1, 2, 3, 4), (v) -> v % 2 == 0 ? "even" : "odd"));
}
```
