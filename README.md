# Medea - Low-memory-overhead JSON tokenizer

Medea can tokenize an arbitrary length of JSON with only a single byte of buffering, generating ([SAX-style](https://en.wikipedia.org/wiki/Simple_API_for_XML)) events to notify each structural element and its containing data.

## Examples

Two sample JSON API results from Twitter and from OpenWeatherMap have been included in the repository and can be successfully processed into tokens by the library even on an ESP8266 as demonstrated by, for example...

```
import examples.scripts.twitterTimelineTokenizeCached
```

...or...

```
import examples.scripts.forecastTokenizeCached
```

To process Twitter API or OpenWeatherMap JSON documents, you will have to register your application and get a bearerId or appId, and fill in the details in medea/auth.py, for the URLs to be properly authenticated.

Although Medea runs on CPython and ESP32 as regular python modules loaded from the filesystem, on ESP8266 it needs to be distributed as [frozen modules](http://docs.micropython.org/en/v1.9.3/unix/reference/constrained.html). Otherwise there is not enough memory for the SSL socket handshake to complete. By default the ESP8266 TLS buffer is only large enough to handle a Twitter timeline API call requesting a single Tweet (count=1). However, the buffer can [be increased](https://github.com/micropython/micropython/commit/a47b8711316a4901bc81e1c46ce50de00207c47f) on ESP8266 to be able to handle larger payloads, for example increasing to 8192 bytes enabled the handling of at least 10 tweets in testing.


## Motivation

If JSON is parsed in the conventional Micropython way ( see [ujson.loads()](https://docs.micropython.org/en/latest/esp8266/library/ujson.html#ujson.loads) ) then an in-memory structure is created. A root ancestor dict or list is the parent of contained child elements. Those children can then be parents of further data structures. All these descendants accumulate in memory as the JSON is decoded.

On an internet-of-things device like the ESP8266 (Cockle) this means complex online API structures such as the OpenWeatherMap API or Twitter are impossible to decode because the all the descendants cannot be stored in memory. For some verbose JSON services, even constructing a single string to pass into the ```json.loads()``` call would exceed the memory, before creating any children at all.

## Limitations

The biggest limitation Medea faces is recursion depth from nested items inside items. The ESP8266 interpreter cannot handle recursion beyond 19 stack levels, so very deeply nested JSON data still cannot be handled without further optimising this library.

# Why the name?

[Medea](https://en.wikipedia.org/wiki/Medea) was the sorcerer wife of [Jason](https://en.wikipedia.org/wiki/Jason) who was famed for killing off her own children. It is a fitting name for a low-overhead JSON tokenizer for Micropython. In this implementation, no children survive at all. Only a series of SAX-style notifications are made as the JSON family 'tree' is traversed.
