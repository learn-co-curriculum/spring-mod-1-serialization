# Serialization

## Learning Goals

- Understand serialization
- Serialize files in Java

## Introduction

With CSV and JSON, we have explored ways to translate a Java object into a
(somewhat) readable string representation. We then took that string
representation, saved it to file, and used it later to restore our object to its
previous state. The process of saving an object's state to file is called
"serialization".

## Serialization in Java

Java offers its own built-in mechanism for serialization, which is optimized for
transferring objects over a network or saving them to a file or a database.
Unlike our CSV or JSON formats, a serialized Java object is not meant to be
human-readable.

Making a Java class serializable is as easy as implementing the `Serializable`
interface. This tells the JDK that you intend for instances of your class to be
serializable by the JVM and makes them acceptable as arguments to Java's
built-in serialization methods.

We can do this to our Person class like this:

```java
public class Person implements Serializable {
    // same implementation as before
}
```

Now that `Person` is `Serializable`, we can use the methods the JDK offers for
serialization and de-serialization.

> - Serialization is the process of turning an object into a byte representation
> - De-serialization is the process of turning an object's byte representation
>   into an in-memory object

To serialize an object, we need 2 classes:

1. `FileOutputStream`: this class will let us write the byte stream we get from
   serializing the object to a file in the file system
2. `ObjectOutputStream`: this class will let us create a byte stream from an
   object

The code to serialize our `Person` object looks like this:

```java
    static void serializePerson(Person person) throws IOException {
        ObjectOutputStream personObjectStream = null;
        try {
            FileOutputStream personFile = new FileOutputStream("person.dat");
            personObjectStream = new ObjectOutputStream(personFile);
            personObjectStream.writeObject(person);
        } catch (Exception exception) {
            exception.printStackTrace();
        } finally {
            if (personObjectStream != null) {
                personObjectStream.flush();
                personObjectStream.close();
            }
        }
    }
```

Here is a breakdown of this code:

1. As before with other IO resources, we need to make sure we release these
   resources when we're done with them, so we do so in the `finally` block,
   which means we need to define our object output stream outside of the `try`
   and `catch` block
2. We create a file output stream with the name of the file we want to save the
   object to
3. We create an object output stream with the file output stream, indicating
   that we want this object's byte stream to be saved to a file
4. Then we ask our object output stream object to write the `person`'s output
   stream to file: `personObjectStream.writeObject(person)`

To de-serialize an object, we need 2 classes:

1. `FileInputStream`: this class will let us read the byte stream from the
   serialized version of the object from a file
2. `ObjectInputStream`: this clas will let us convert the byte stream version of
   the object into an in-memory object

The code to de-serialize our `Person` object looks like this:

```java
    static Person deserializePerson() throws IOException {
        ObjectInputStream personInputStream = null;
        Person newPerson = null;
        try {
            FileInputStream personFile = new FileInputStream("person.dat");
            personInputStream = new ObjectInputStream(personFile);
            newPerson = (Person) personInputStream.readObject();
        } catch (Exception exception) {
            exception.printStackTrace();
        } finally {
            if (personInputStream != null) {
                personInputStream.close();
            }
        }

        return newPerson;
    }
```

Here is a breakdown of this code:

1. As always, we define our stream that we need to clean up outside the `try`
   and `catch` block, so we can clean up after ourselves in the `finally` block
2. We create a file input stream with the name of the file we want to read the
   object's bytes from
3. We create an object input stream with the file input stream, indicating that
   we want to read this object's byte representation from the file we specified
   earlier
4. Then we ask our object input stream object to read the `person`'s input
   stream from the file: `newPerson = (Person) personInputStream.readObject();`
5. Note that the `readObject()` method returns an instance of the `Object`
   class, which must then be explicitly cast to an instance of our `Person`
   class

Serialization is a powerful tool for sending objects to other systems through
network connections and saving Java-based representations of object in
persistent systems like databases or file systems.

Note that the main difference between JSON and serialization is that JSON is a
cross-platform representation of an object, whereas the serialized version of a
Java object can only be read by a Java program. Other programming languages do
not understand the way that Java represents its objects and therefore cannot
consume a serialized version of those objects.

> All data from an object gets saved when it gets serialized, except for fields
> that are explicitely marked `transient`. The definition of a `transient` field
> is a field that does not get serialized. This is used for fields that
> represent data that cannot be restored in the future. For example, a network
> connection that exists at moment T will not exist at moment T + 1 when the
> object is restored from its persistent state.
