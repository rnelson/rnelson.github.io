---
layout: post
title: "Java 8 Time Zone Conversion"
description: ""
category: 
tags: kotlin, java, time zone, timezone, conversion, joda-time
---
{% include JB/setup %}

## Off-by-one

Work has an API that we use internally quite a bit. We needed to make a large 
number of changes to the data and could not use the API to do it. The developers 
of the API provided information on doing the work at the database level, so all 
we had to do was ensure the large batch of records we were changing were the 
right ones.

The Data Engineering team built a process to compare some input files to the 
database and provided a set of records along with a date from the database. To 
ensure we were going to alter the correct records, I wrote a small validation 
program in [Kotlin](https://kotlinlang.org). The program compares the results 
of the API to an entry in the DE output files and makes sure that everything 
looks accurate.

From a sample set of around 150k records, over half were _almost_ correct. The 
date the API provided and the date Data Engineering provided were a day off. In 
every case, the API returned a date one day after what DE gave me.

The developers who wrote the API explained what was happening: while the API 
returned timestamps in UTC, the data was stored in the database in a different 
time zone.

## Learning Opportunity

Whether because they were the current versions at the time of development or the 
available JVM being older, I haven't done a whole lot with anything newer than 
Java 7.

While [Joda-Time](http://www.joda.org/joda-time/) has existed for quite some time 
and is designed to make date and time processing in older versions of Java less 
painful, I've only used it on a couple of small projects. It's not available in 
all of the projects I do at work, so I'm stuck with the built-in Java functionality.

This validation program, however, is meant as a short lived utility that can run 
on my computer instead of having to run within our existing products. Since I was 
already building the code on Java 8, I took this as a chance to play with the "new" 
date and time classes.

## Time Zone Conversion

To convert a date/time from one time zone to another, there are three things I ended 
up doing.

First, I needed to get a 
[ZoneOffset](https://docs.oracle.com/javase/8/docs/api/java/time/ZoneOffset.html) 
object for my target time zone:

```kotlin
val instant = Instant.now()
val zoneId = ZoneId.of("America/Chicago")
val offset = zoneId.rules.getOffset(instant)
```

Once I had an offset, I needed to parse the timestamp the API was returning. For 
this particular problem, I needed to iterate through multiple timestamps and find 
the latest one, so my setup involved both creating a 
[DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html) 
object that specifies my input string and an arbitrary date that predates anything 
in the database.

```kotlin
val pattern = DateTimeFormatter.ofPattern("u-MM-dd HH:mm:ss z")
var latestDate = ZonedDateTime.of(1989, 12, 31, 0, 0, 0, 0, ZoneOffset.of("-05:00"))
```

Finally, the actual conversion:

```kotlin
val dateTime = LocalDateTime.parse(inputString, pattern)
val utcDateTime = ZonedDateTime.of(dateTime, ZoneOffset.UTC)
val date = utcDateTime.withZoneSameInstant(offset)
```

The `LocalDateTime.parse()` call will attempt to parse `inputString` with the 
`DateTimeFormatter` and create a `LocalDateTime` object. While my pattern specifies 
that time zone information is included (`z`), the `LocalDateTime` does not contain 
any time zone details -- it is just a date and time.

`ZonedDateTime.of()` will convert a `LocalDateTime` into a `ZonedDateTime`, which 
does contain time zone information, using whatever offset is specified. Since the 
API is returning timestamps in UTC, my second argument was `ZoneOffset.UTC`.

Finally, the 
[withZoneSameInstant](https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html#withZoneSameInstant-java.time.ZoneId-) 
method will create a new `ZonedDateTime` that's set to a different time zone offset. 
In the example above, the new object is set to `America/Chicago`.

There was one additional tweak I had to make to my date object once I had it set to 
the correct time zone. Since Data Engineering was only providing the date, all time 
elements were set to their default value, `0`.

Fortunately, `ZonedDateTime` has a method to help me do the same to my converted value: 
[truncatedTo](https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html#truncatedTo-java.time.temporal.TemporalUnit-). 
Any value _after_ the specified argument, in terms of accuracy, is initialized 0. 
Since I am comparing just the day, `ChronoUnit.DAYS` is specified and hour, minute, 
second, milliseconds, and nanoseconds are all set to 0.

```kotlin
val zeroDate = date.truncatedTo(ChronoUnit.DAYS)
```

My date now matches what Data Engineering provided!
