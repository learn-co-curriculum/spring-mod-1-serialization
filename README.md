# Serialization

## Learning Goals

- Explain serialization.
- Elaborate on the process of serializing a Java object.
- Discuss how to de-serialize a byte stream into a Java object again.

## Introduction

In Java Module 3, we learned about File IO. In that section, we learned about
JSON files. As a refresher, JSON is a file format that offers more flexibility
than CSV files and allows us to map the data back to an object in Java.

In this lesson, we will learn about a process called **serialization**, which is
the process of converting an object to a stream of bytes so that it can be stored.

## What is Serialization in Java?

Java offers its own built-in mechanism for serialization, which is optimized for
transferring objects over a network or saving them to a file or a database.
Unlike our CSV or JSON formats, a serialized Java object is not meant to be
human-readable.

Making a Java class serializable is as easy as implementing the `Serializable`
interface. This tells the JDK that we intend for the instances of the class to be
serializable by the JVM and makes them acceptable as arguments to Java's
built-in serialization methods.

Let's look at our `Student` class we worked with in Java Module 3 again:

```java
import java.util.List;

public class Student {
   private String firstName;
   private String lastName;
   private char grade;
   private List<String> classes;

   public Student() {}

   // Add the list of classes to the constructor
   public Student(String firstName, String lastName, char grade, List<String> classes) {
      this.firstName = firstName;
      this.lastName = lastName;
      this.grade = grade;
      this.classes = classes;
   }

   public String getFirstName() {
      return firstName;
   }

   public void setFirstName(String firstName) {
      this.firstName = firstName;
   }

   public String getLastName() {
      return lastName;
   }

   public void setLastName(String lastName) {
      this.lastName = lastName;
   }

   public char getGrade() {
      return grade;
   }

   public void setGrade(char grade) {
      this.grade = grade;
   }

   // Add a getter method for the list of classes
   public List<String> getClasses() {
      return classes;
   }

   // Add a setter method for the list of classes
   public void setClasses(List<String> classes) {
      this.classes = classes;
   }

   // Modify the toString() method to include the list of classes
   @Override
   public String toString() {
      String studentString = String.format("%s %s has the letter grade %s",
              firstName,
              lastName,
              grade);
      studentString = studentString + " and is in the following classes: ";
      for (String c : classes) {
         studentString = studentString.concat(c).concat(" ");
      }
      return studentString;
   }
}
```

We can make our `Student` class serializable like this:

```java
public class Student implements Serializable {
    // same implementation as before
}
```

Now that `Student` is `Serializable`, we can use the methods the JDK offers for
serialization and de-serialization.

> - **Serialization** is the process of turning an object into a byte
>   representation.
> - **De-serialization** is the process of turning an object's byte
>   representation into an in-memory object.

## Serializing an Object

To serialize an object, we need 2 classes:

1. `FileOutputStream`: this class will let us write the byte stream we get from
   serializing the object to a file in the file system.
2. `ObjectOutputStream`: this class will let us create a byte stream from an
   object.

The code to serialize our `Person` object looks like this:

```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class SerializeStudentDriver {
   public static void main(String[] args) {
       // Create the Student to serialize
      List<String> classes = new ArrayList<String>(Arrays.asList("Computer Science"));
      Student suzie = new Student("Suzie", "Bingham", 'A', classes);

      try (
              FileOutputStream studentFile = new FileOutputStream("student.data");
              ObjectOutputStream studentObjectStream = new ObjectOutputStream(studentFile)
      ) {
         studentObjectStream.writeObject(suzie);
      } catch (Exception exception) {
         exception.printStackTrace();
      }
   }
}
```

Here is a breakdown of this code:

1. As before with other IO resources, we need to make sure we release these
   resources when we're done with them, so we will use the try-with-resources
   statement to open and close the `FileOutputStream` object and the
   `ObjectOutputStream` object. Notice that we can open multiple resources within
   the try-with-resources statement and that each will be closed appropriately in
   the reverse order they are instantiated in. So in this example, the code will
   close the `studentObjectStream` before it closes the `studentFile` object.
2. We will create a `FileOutputStream` with the name of the file we want to save
   the object to. In this case, we'll name the file `student.data`.
3. We will create an `ObjectOutputStream` with the `studentFile`, indicating
   that we want this object's byte stream to be saved to a file.
4. Then we will write the `Student` output stream to a file:
   `studentObjectStream.writeObject(suzie);`

If we were to run the code above, we'd see that it creates a file called
`student.data`; however, this file is not human-readable. This is because
serialized files are not meant to be human-readable since it is converting the
Java object into a stream of bytes.

## De-serializing an Object

To de-serialize an object, we need 2 classes:

1. `FileInputStream`: this class will let us read the byte stream from the
   serialized version of the object from a file.
2. `ObjectInputStream`: this class will let us convert the byte stream version of
   the object into an in-memory object.

The code to de-serialize our `Person` object looks like this:

```java
import java.io.FileInputStream;
import java.io.ObjectInputStream;

public class DeserializeStudentDriver {
    public static void main(String[] args) {
       try (
               FileInputStream studentFile = new FileInputStream("student.data");
               ObjectInputStream studentInputStream = new ObjectInputStream(studentFile)
       ) {
          Student newStudent = (Student) studentInputStream.readObject();
          System.out.println(newStudent);
       } catch (
               Exception exception) {
          exception.printStackTrace();
       }
    }
}
```

Here is a breakdown of this code:

1. Like before, we'll add instantiate both the `FileInputStream` and
   `ObjectInputStream` resources within the try-with-resources statement so that
   the resources can be properly closed when we finish executing the code.
2. We'll create a `FileInputStream` with the name of the file we want to read the
   object's bytes from. In this case, we want to read in the binary data we wrote
   out, so we'll pass in the `student.data` file as an argument.
3. Next we'll create an `ObjectInputStream` with the `studentFile`, indicating that
   we want to read this object's byte representation from the file we specified.
4. Then we ask our object input stream object to read the `Student` input
   stream from the file:
   `Student newStudent = (Student) studentInputStream.readObject();`
5. Note that the `readObject()` method returns an instance of the `Object`
   class, which must then be explicitly cast to an instance of our `Student`
   class

## Conclusion

Serialization is a powerful tool for sending objects to other systems through
network connections and saving Java-based representations of object in
persistent systems like databases or file systems.

Note that the main difference between JSON and serialization is that JSON is a
cross-platform representation of an object, whereas the serialized version of a
Java object can only be read by a Java program. Other programming languages do
not understand the way that Java represents its objects and therefore cannot
consume a serialized version of those objects.

In the next lesson, we'll learn about data transfer objects and turn our focus
more on sending and receiving JSON objects back from a Spring application.
