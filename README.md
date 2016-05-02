Jsouper
=====

No API, no problem. Jsouper helps you parse HTML into Java objects. It is largely based on Square's [Moshi][moshi] and [jsoup][jsoup]:

```java
Document document = Jsoup.connect("https://play.google.com/store").get();

Jsouper jsouper = new Jsouper.Builder()
  .add(CoverAdapter.class, new CoverAdapter())
  .build();
ElementAdapter<Cover> elementAdapter = jsouper.adapter(CoverAdapter.class);

Cover cover = elementAdapter.fromElement(document);
System.out.println(cover);
```

### Sample

See the [sample][sample] module for a Java and Android example which uses Jsouper to build a Google Play Movies UI just from hitting the website:

<img src="https://github.com/ekchang/jsouper/blob/master/screenshot.png" height="400">

### Custom Element Adapters

Unlike Moshi or Gson, there is a lot more work required to parse HTML to your Java objects. This is because HTML tags and attributes rarely map directly to how you want to structure your objects.

At minimum, you will need to define a custom `ElementAdapter` for your most primitive classes. `query()` is used to define the primary key for identifying the top-most `Element` that maps to your Java object - it gets called using jsoup's `Element.select(query)`. You will probably need to familiarize yourself with [how to extract attributes with jsoup][jsoup-attributes-text] before proceeding. 

From each valid `Element` from this query, define how the object is constructed in `fromElement`:

```java
public class CoverAdapter extends ElementAdapter<Cover> {
  @Override
  public String query() {
    return "cover";
  }

  @Override
  public Cover fromElement(Element element) throws IOException {
    final String imageUrl =
        element.select("div.cover-inner-align").select("img").first().attr("data-cover-large");
    final String targetUrl = element.select("a.card-click-target").attr("href");
    return new Cover(imageUrl, targetUrl);
  }
}
```

Objects composed by objects you've already defined `ElementAdapter`'s for can be generated by Jsouper. You still need to define the `query` parameter, but this can be done with the `@ElementQuery` annotation in your model class declaration:

```java
@ElementQuery(query = "div.card.no-rationale.tall-cover.movies.small")
public class Movie {
  public final Cover cover;
  public final Detail detail;
  public final Rating rating;

  public Movie(Cover cover, Detail detail, Rating rating) {
    this.cover = cover;
    this.detail = detail;
    this.rating = rating;
  }
}

...

ElementAdapter<Movie> movieAdapter = jsouper.adapter(Movie.class);
Movie movie = movieAdapter.fromElement(document);
```

There is no serialization support at the moment.

### Built-in Type Adapters

Jsouper currently has built-in support for 

 * Collections, Lists, Sets

Contributions to add support for Maps and Arrays (and anything else that is currently missing) are welcome.

#### Retrofit Converter

Add the converter dependency below and configure your Retrofit. Be sure to add it before any of your JSON parsers - JsoupConverterFactory will fail gracefully and allow your other parsers to step in so you can mix JSON and HTML parsing:

```java
Jsouper jsouper = ...;

Retrofit retrofit = new Retrofit.Builder()
        .addConverterFactory(JsoupConverterFactory.create(jsouper))
        .build();
```


Download
--------

Download [the latest JAR][dl] or depend via Maven:
```xml
<dependency>
  <groupId>com.ekc.jsouper</groupId>
  <artifactId>jsouper</artifactId>
  <version>0.0.1</version>
</dependency>
```
or Gradle:
```groovy
compile 'com.ekc.jsouper:jsouper:0.0.1'
```

Retrofit2 converter is also available:
```groovy
compile 'com.ekc.jsouper:retrofit-converter-jsoup:0.0.1'
```

License
--------

    Copyright 2016 Erick Chang
    Copyright 2015 Square, Inc.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


 [dl]: https://search.maven.org/remote_content?g=com.ekc.jsouper&a=jsouper&v=LATEST
 [moshi]: https://github.com/square/moshi/
 [jsoup]: https://jsoup.org/
 [jsoup-attributes-text]: https://jsoup.org/cookbook/extracting-data/attributes-text-html
 [sample]: https://github.com/ekchang/jsouper/tree/master/sample