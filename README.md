# Jackson Datatype Money

[![Build Status](https://img.shields.io/travis/zalando/jackson-datatype-money.svg)](https://travis-ci.org/zalando/jackson-datatype-money)
[![Coverage Status](https://img.shields.io/coveralls/zalando/jackson-datatype-money.svg)](https://coveralls.io/r/zalando/jackson-datatype-money)
[![Release](https://img.shields.io/github/release/zalando/jackson-datatype-money.svg)](https://github.com/zalando/jackson-datatype-money/releases)
[![Maven Central](https://img.shields.io/maven-central/v/org.zalando/jackson-datatype-money.svg)](https://maven-badges.herokuapp.com/maven-central/org.zalando/jackson-datatype-money)

*Jackson Datatype Money* is a [Jackson](https://github.com/codehaus/jackson) module to support JSON serialization and
deserialization of [Java Money](https://github.com/JavaMoney/jsr354-api) data types. It fills a niche, in that it
connects the Java Money and Jackson libraries so that they work seamlessly together, without requiring additional
developer effort. In doing so, it aims to perform a small but repetitive task — once and for all.

The maintainers of Jackson Datatype Money build APIs, so you might notice that this project reflects an API design
point of view. In developing our work, we sought a reusable library that would enable us to express monetary amounts in
JSON while reflecting our API preferences, as shown in the following example. We couldn't find one, so we created one.

```json
{
  "amount": 29.95, 
  "currency": "EUR"
}
```

## Features
- enables you to express monetary amounts in JSON
- can be used in a REST APIs
- offers customizable serialization by locale
- allows you to implement RESTful API endpoints that format monetary amounts based on the Accept-Language header
- is unique and flexible

## Dependencies
- Java 7 or higher
- Any build tool using Maven Central, or direct download
- Jackson
- Java Money

## Installation

Add the following dependency to your project:

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>jackson-datatype-money</artifactId>
    <version>${jackson-datatype-money.version}</version>
</dependency>
```
For ultimate flexibility, this module is compatible with the official version as well as the backport of Java Money. The actual version will be selected by a profile based on the current JDK version.

## Configuration

Register the module with your `ObjectMapper`:

```java
ObjectMapper mapper = new ObjectMapper()
    .registerModule(new MoneyModule());
```

Alternatively, you can use the SPI capabilities:

```java
ObjectMapper mapper = new ObjectMapper()
    .findAndRegisterModules();
```

### Serialization

For serialization this module currently supports the following data types:

| Input                                                                                                                             | Standard                                          | Output                                 |
|-----------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|----------------------------------------|
| [`java.util.Currency`](https://docs.oracle.com/javase/8/docs/api/java/util/Currency.html)                                         | [ISO-4217](http://en.wikipedia.org/wiki/ISO_4217) | `EUR`                                  |
| [`javax.money.CurrencyUnit`](https://github.com/JavaMoney/jsr354-api/blob/master/src/main/java/javax/money/CurrencyUnit.java)     | [ISO-4217](http://en.wikipedia.org/wiki/ISO_4217) | `EUR`                                  |
| [`javax.money.MonetaryAmount`](https://github.com/JavaMoney/jsr354-api/blob/master/src/main/java/javax/money/MonetaryAmount.java) |                                                   | `{"amount": 99.95, "currency": "EUR"}` |

### Formatting

A special feature for serializing monetary amounts is *formatting*, which is **disabled by default**. To enable it, you
have to pass in a `MonetaryAmountFormatFactory` implementation to the `MoneyModule`:

```java
ObjectMapper mapper = new ObjectMapper()
    .registerModule(new MoneyModule(new DefaultMonetaryAmountFormatFactory()));
```

The `DefaultMonetaryAmountFormatFactory` delegates directly to `MonetaryFormats.getAmountFormat(Locale, String...)`.

Formatting only affects the serialization and can be customized based on the *current* locale, as defined by the
[`SerializationConfig`](http://wiki.fasterxml.com/SerializationConfig). This allows to implement RESTful API endpoints
that format monetary amounts based on the `Accept-Language` header.

The first example serializes a monetary amount using the `de_DE` locale:

```java
ObjectWriter writer = mapper.writer().with(Locale.GERMANY);
writer.writeValueAsString(Money.of(29.95 "EUR"));
```

```json
{
  "amount": 29.95, 
  "currency": "EUR",
  "formatted": "29,95 EUR"
}
```

The following example uses `en_US`:

```java
ObjectWriter writer = mapper.writer().with(Locale.US);
writer.writeValueAsString(Money.of(29.95 "USD"));
```

```json
{
  "amount": 29.95, 
  "currency": "USD",
  "formatted": "USD29.95"
}
```

More sophisticated formatting rules can be supported by implementing `MonetaryAmountFormatFactory` directly. 

### Deserialization

This module will use `org.javamoney.moneta.Money` as an implementation for `javax.money.MonetaryAmount` when
deserializing money values. If you need a different implementation, you can pass an implementation of
`MonetaryAmountFactory` to the `MoneyModule`:

```java
ObjectMapper mapper = new ObjectMapper()
    .registerModule(new MoneyModule(new FastMoneyFactory()));
```

If you're using Java 8, you can also pass in a method reference:

```java
ObjectMapper mapper = new ObjectMapper()
    .registerModule(new MoneyModule(FastMoney::of));
```

*Jackson Datatype Money* comes with the following factories: 

| `MonetaryAmount` Implementation     | Factory                                                                                                                               |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| `org.javamoney.moneta.FastMoney`    | [`org.zalando.jackson.datatype.money.FastMoneyFactory`](src/main/java/org/zalando/jackson/datatype/money/FastMoneyFactory.java)       |
| `org.javamoney.moneta.Money`        | [`org.zalando.jackson.datatype.money.MoneyFactory`](src/main/java/org/zalando/jackson/datatype/money/MoneyFactory.java)               |
| `org.javamoney.moneta.RoundedMoney` | [`org.zalando.jackson.datatype.money.RoundedMoneyFactory`](src/main/java/org/zalando/jackson/datatype/money/RoundedMoneyFactory.java) |                                                                                                                             |

## Getting help

If you have questions, concerns, bug reports, etc, please file an issue in this repository's Issue Tracker.

## Getting involved

To contribute, simply make a pull request and add a brief description (1-2 sentences) of your addition or change. For
more details check the [contribution guidelines](CONTRIBUTING.md).