# Facade

**Category:** Structural

## Intent

Provides a simplified interface to a library, a framework, or any other complex set of classes.

## Problem

Your code works with dozens of objects from a sophisticated subsystem. You must initialize them all, manage dependencies, and execute methods in the right order. Business logic becomes tightly coupled to subsystem implementation details.

## Solution

A Facade provides a simple interface to the subsystem's most-used features. It delegates client calls to the appropriate subsystem objects.

## Structure

1. **Facade** — provides convenient access to a subset of the subsystem's functionality. Knows where to direct the client's request
2. **Additional Facade** — optional extra facade to prevent polluting a single facade with unrelated features
3. **Subsystem classes** — the complex library/framework internals. Unaware of the facade
4. **Client** — uses the facade instead of calling subsystem objects directly

## Pseudocode

```
class VideoConverter is
    method convert(filename, format): File is
        file = new VideoFile(filename)
        sourceCodec = new CodecFactory.extract(file)

        if (format == "mp4")
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()

        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = new AudioMixer().fix(result)

        return new File(result)

// Client code — the complex subsystem is hidden behind one method
class Application is
    method main() is
        converter = new VideoConverter()
        mp4 = converter.convert("funny-cats.ogg", "mp4")
        mp4.save()
```

## Applicability

- Use when you need a limited but straightforward interface to a complex subsystem
- Use when you want to structure a subsystem into layers (facade per layer)

## Pros and Cons

**Pros:**
- Isolate your code from subsystem complexity

**Cons:**
- A facade can become a "god object" coupled to all classes of the subsystem

## Relations with Other Patterns

- **Facade** defines a new, simplified interface; **Adapter** makes an existing interface usable
- **Abstract Factory** can serve as an alternative when you only want to hide how subsystem objects are created
- **Flyweight** shows how to share lots of small objects; Facade shows how to make one object that represents an entire subsystem
- **Facade** and **Mediator** both organize collaboration but: Facade = simplified unidirectional entry point; Mediator = centralized bidirectional communication
- A Facade can often be a **Singleton** since typically only one facade is needed
