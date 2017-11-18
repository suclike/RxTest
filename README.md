# RxKotlinTest

[![Build Status](https://travis-ci.org/RubyLichtenstein/RxKotlinTest.svg?branch=master)](https://travis-ci.org/RubyLichtenstein/RxKotlinTest)
[![codecov](https://codecov.io/gh/RubyLichtenstein/RxKotlinTest/branch/master/graph/badge.svg)](https://codecov.io/gh/RubyLichtenstein/RxKotlinTest)

# Introduction

RxKotlinTest is an easy to use extendable kotlin DSL for testing RxJava2 code.
The library based on [KotlinTest](https://github.com/kotlintest/kotlintest).

# Example

### Before
```kotlin
        val values = listOf("Hello", "Rx", "Kotlin", "Test")
        
        Observable.fromIterable(values)
                .test()
                .assertValueSequence(values)
                .assertValueCount(values.size)
                .assertComplete()
                .assertNoErrors();

        Observable.never<Unit>()
                .test()
                .assertValueCount(0)
                .assertNotComplete()
                .assertNoErrors();
```

### After
```kotlin
        val values = listOf("Hello", "Rx", "Kotlin", "Test")

        Observable.fromIterable(values)
                .test {
                    it shouldHave valueSequence(values)
                    it shouldHave valueCount(values.size)
                    it shouldHave noErrors()
                    it should complete()
                }

        Observable.never<Unit>()
                .test {
                    it shouldHave noValues()
                    it shouldHave noErrors()
                    it should notComplete()
                }
                
Given("list of values"){
            val values = listOf("Hello", "Rx", "Kotlin", "Test")
            val publishSubject = PublishSubject.create<String>()
            val publishSubjectTest = publishSubject.test()

            When("subject emit values"){
                values.forEach {
                    publishSubject.onNext(it)
                }

                Then("values emitted"){
                    publishSubjectTest shouldHave valueSequence(values)
                }
            }

            When("call subject onComplete"){
                publishSubject.onComplete()

                Then("subject complete with no errors"){
                    publishSubject.test {
                        it should complete()
                    }
                }
            }
        }                

```

### Composing Assertions
```kotlin
fun <T> noValues() = valueCount<T>(0)

fun <T> errorOrComplete(error: Throwable) = error<T>(error) or complete()

fun <T> moreValuesThen(count: Int)
        = createAssertion<T>({ it.values().size > count }, "Should have more values then $count")
```

# Usage
### Assertions
```kotlin
it should complete() 

it should notComplete()

it shouldHave error(error: Throwable)

it shouldHave error(errorClass: Class<out Throwable>)

it shouldHave error(errorPredicate: (Throwable) -> Boolean)

it shouldHave noErrors(): TestObserverMatcher<T>

it shouldHave value(value: T): TestObserverMatcher<T>

it shouldHave value(valuePredicate: (T) -> Boolean)

it shouldHave never(value: T)

it shouldHave never(neverPredicate: (T) -> Boolean)

it shouldHave valueAt(index: Int, value: T)

it shouldHave valueAt(index: Int, valuePredicate: (T) -> Boolean)

it shouldHave values(vararg values: T)

it shouldBy empty()

it shouldHave noTimeout()

it shouldHave timeout()

it should subscribed()

it should notSubscribed()

it shouldHave failure(errorPredicate: (Throwable) -> Boolean, vararg values: T)

it shouldHave failure(error: Class<out Throwable>, vararg values: T)

it shouldHave failureAndMessage(error: Class<out Throwable>, message: String, vararg values: T)

it shouldHave result(vararg values: T)

it should terminate()

it shouldHave valueCount(count: Int)

it shouldHave valueSequence(sequence: Iterable<T>)

it shouldHave valueSet(expected: Collection<T>)

it shouldHave valueOnly(vararg values: T)

```
### Composing assertions
```kotlin
createAssertion(action: (TestObserver<T>) -> Boolean, message: String)
```

# Create your own assertions

### Example
```kotlin
fun <T> noValues() = valueCount<T>(0)

fun <T> errorOrComplete(error: Throwable) = error<T>(error) or complete()

fun <T> moreValuesThen(count: Int)
        = createAssertion<T>({ it.values().size > count }, "Should have more values then $count")

fun <T> lessValuesThen(count: Int)
        = createAssertion<T>({ it.values().size < count }, "Should have less values then $count")

fun <T> valueCountBetween(min: Int, max: Int) = moreValuesThen<T>(min) and lessValuesThen<T>(max)

@Test
fun composeTest() {
    val values = listOf<String>("Rx", "Kotlin", "Test")
    Observable.fromIterable(values)
            .test {
                it shouldHave moreValuesThen(2)
                it shouldHave noErrors()
                it shouldHave valueSequence(values)
            }

    Observable.empty<String>()
            .test {
                it shouldHave noValues()
            }

    Observable.just("")
            .test {
                it shouldHave errorOrComplete(Throwable())
            }

    Observable.just("","")
            .test {
                it shouldHave valueCountBetween(1, 3)
            }
}
```

# Download
Gradle
```groovy
testCompile 'com.rubylichtenstein:rxkotlintest:1.0.2'
```

Maven

```xml
<dependency>
    <groupId>com.rubylichtenstein</groupId>
    <artifactId>rxkotlintest</artifactId>
    <version>1.0.2</version>
    <type>pom</type>
</dependency>
```

# Contribute

Contact me - ruby.lichtenstein@gmail.com

