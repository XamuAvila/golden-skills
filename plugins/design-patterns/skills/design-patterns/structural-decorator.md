# Decorator

**Also known as:** Wrapper
**Category:** Structural

## Intent

Lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the new behaviors.

## Problem

A notification library initially only sends emails. Users want SMS, Facebook, Slack notifications. Creating subclasses for every combination (SMSFacebook, SMSSlack, etc.) leads to class explosion.

## Solution

Wrap the base notifier in decorator objects. Each decorator adds one behavior. Stack decorators for combinations: `Slack(SMS(Facebook(BaseNotifier)))`.

## Structure

1. **Component** — interface for both wrappers and wrapped objects
2. **ConcreteComponent** — class being wrapped (the base object)
3. **BaseDecorator** — wraps a component, delegates all calls to it
4. **ConcreteDecorator** — overrides methods to add behavior before/after delegating to the wrapped object
5. **Client** — wraps components in layers of decorators

## Pseudocode

```
interface DataSource is
    method writeData(data)
    method readData(): data

class FileDataSource implements DataSource is
    constructor FileDataSource(filename) is // ...

    method writeData(data) is
        // Write data to file

    method readData(): data is
        // Read data from file

class DataSourceDecorator implements DataSource is
    field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    method writeData(data) is
        wrappee.writeData(data)

    method readData(): data is
        return wrappee.readData()

class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Encrypt data
        // 2. Pass encrypted data to wrappee's writeData
        super.writeData(encrypt(data))

    method readData(): data is
        // 1. Get data from wrappee's readData
        // 2. Decrypt and return
        return decrypt(super.readData())

class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Compress data
        // 2. Pass compressed data to wrappee's writeData
        super.writeData(compress(data))

    method readData(): data is
        // 1. Get data from wrappee's readData
        // 2. Decompress and return
        return decompress(super.readData())

// Client code — stack decorators
source = new FileDataSource("somefile.dat")
source = new CompressionDecorator(source)
source = new EncryptionDecorator(source)
// Writing: encrypt(compress(data)) → file
// Reading: file → decompress(decrypt(data))
source.writeData(salaryRecords)
```

## Applicability

- Use when you need to assign extra behaviors to objects at runtime without breaking code that uses those objects
- Use when it's awkward or impossible to extend an object's behavior through inheritance (final class, deep hierarchy)

## Pros and Cons

**Pros:**
- Extend behavior without making new subclass
- Add or remove responsibilities at runtime
- Combine several behaviors by stacking decorators
- Single Responsibility Principle — each decorator = one concern

**Cons:**
- Hard to remove a specific wrapper from the middle of the stack
- Behavior may depend on decorator order
- Initial configuration code can look ugly

## Relations with Other Patterns

- **Adapter** changes the interface; **Decorator** changes behavior keeping interface intact
- **Composite** and Decorator have similar structure but different intents
- **Chain of Responsibility** handlers are independent (each can stop chain); Decorator can't break the flow
- Decorator adds "skin" over an object; **Strategy** changes the "guts"
- Decorator is controlled by the client; **Proxy** manages the service lifecycle independently
