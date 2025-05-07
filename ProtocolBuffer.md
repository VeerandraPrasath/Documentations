# Protocol Buffer Fundamentals
  
## Introduction
* By google in 2001 as internal project for inter-service communication.
* Network services to communicate quickly and efficiently.
## Version Information
* Protocol Buggers v26.0
* Supports Protocol Buffers v3.0.0 +
## Purpose of Protocol Buffers
*  It's like JSON, except it's smaller and faster
* it can be used in most modern languages, so no matter what language or languages your team is using, be it PHP, C#, Java, or even Visual Basic
* Protocol Buffers is a messaging format.
* gRPC is once again a technology created by Google to allow in inter‑service communication, but at a slightly higher level.
* GRPC is a way for services to communicate with one another using a system called Remote Procedure Call
* Both gRPC and protocol buffer are platform neutral
###  Attributes of Protocol Buffers
* they are very, very fast. They are designed to allow blazingly‑fast communication between network services.
* They're also designed to be extremely efficient
*  the difference between fast and efficient in this example is network bandwidth.
* the fast means that we can normally encode and decode Protocol Buffer messages faster than equivalent technologies such as XML and JSON.
* Efficient means that these messages are small. They are much more compressed and require significantly fewer network resources to send a message using Protocol Buffers versus the competition.
* they are compatible.

&nbsp;With Protocol Buffers, we can actually encode the versioning and version proofing inside of the message definitions themselves, which makes it much easier to maintain our systems as the messaging requirements evolve.

## Protocol Buffers versus JSON
### JSON
```json
{
  "name": "John Doe",
  "age": 30,
}
```
    The schema is simply the structure for the data.
1.Schema is part of message
* JSON message contains the field names, as well as the field values.
* we're defining not only the data that's being transmitted, but the structure of that data with our JSON messages.


2.Transmitted as Text
* The next thing is JSON messages are typically transmitted as plain text, which means that we have to encode numbers as text.

3.Weakly Typed
* here's no definition within the JSON message itself that tells what type of data is being transmitted with each field

4.Cannot transmit binary data without translation to text-based format.
*  we can get binary data transmitted. So if you want to send, for example, an image or a picture through a JSON message, you can do it, but it normally takes some sort of translation to a text‑friendly format such as using Base64 encoding.
* we have further disadvantages by the fact that we need to convert our binary data into text, which makes the message even larger.

### Protocol Buffers
```proto
message Subscriber {
  string username = 1;
  int32 age = 2;
  string email=3;
}
```
     message (sort of!)
     1:123
     2:adent
     3:adent@milliways.com

1.Schema is predefined

2.Transmitted as Binary

3.Strongly Typed

4.Can easily transmit binary data

##  Installing the protocol Buffer

    Website to know about protocol buffer -  protobuf.dev
* Visit the website
* Click download and install -Redirect to github
* Click the latest release on the right side
* Scroll down and download the zip file based on ur OS.
## Comparing Protocol Buffers to JSON
subscribe.proto
```proto

syntax = "proto3";

package CSharpProtoSourceFile;

message Subscriber {
    string username = 1;
    string email = 3;
    Address address = 4;
}

message Address {
    int32 house_number = 1;
    string street = 2;
    string city = 3;
    string county = 4;
}
```
1.Install the Proto Installer in the Proto file path

2.Convert the Proto file to CSharp code by using the command
```bash
protoc --csharp_out=. subscriber.proto
```
3.Then we will get the Subscriber.cs file in the same directory with specified namespace if mentioned (eg:CSharpProtoSourceFile).

Program.cs
```csharp
using Google.Protobuf;
using System.Text.Json;

namespace ProtoVsJson
{
    internal class Program
    {
        static void Main(string[] args)
        {

            var s = new CSharpProtoSourceFile.Subscriber
            {
                Username = "adent",
                Email = "adent@milliways.com",
                Address = new CSharpProtoSourceFile.Address
                {
                    HouseNumber = 155,
                    Street = "Country Lane",
                    City = "Cottington",
                    County = "Cottington"
                }
            };

            var data = s.ToByteArray();

            Console.WriteLine($"Message:\n\t{BitConverter.ToString(data)}\nSize: {data.Length}");

            var s2 = new Subscriber
            {
                Username = "adent",
                Email = "adent@milliways.com",
                Address = new Address
                {
                    HouseNumber = 155,
                    Street = "Country Lane",
                    City = "Cottington",
                    County = "Cottington"
                }
            };

            var data2 = JsonSerializer.Serialize(s2);

            Console.WriteLine($"Message:\n\t{data2}\nSize: {data2.Length}");

        }
    }

    public class Subscriber
    {
        public string Username { get; set; }
        public string Email { get; set; }
        public Address Address { get; set; }
    }

    public class Address
    {
        public float HouseNumber { get; set; }
        public string Street { get; set; }
        public string City { get; set; }
        public string County { get; set; }
    }
}
```
output
```bash
Message:
        0A-05-61-64-65-6E-74-1A-13-61-64-65-6E-74-40-6D-69-6C-6C-69-77-61-79-73-2E-63-6F-6D-22-29-08-9B-01-12-0C-43-6F-75-6E-74-72-79-20-4C-61-6E-65-1A-0A-43-6F-74-74-69-6E-67-74-6F-6E-22-0A-43-6F-74-74-69-6E-67-74-6F-6E
Size: 71
Message:
        {"Username":"adent","Email":"adent@milliways.com","Address":{"HouseNumber":155,"Street":"Country Lane","City":"Cottington","County":"Cottington"}}
Size: 146
```