+++
date = '2026-02-08T08:57:18-04:00'
draft = false 
title = 'Event Driven: RabbitMQ for Dummies'
tags = ['RabbitMQ', '.NET', 'Docker', 'Event-Driven Architecture', 'Message Brokers', 'Distributed Systems']
categories = ['tech']
+++

In the world of development of high-impact software you need reliability, past are the arquitectures where you rely on a single server's threads, you find yourself having to manage X amount of clients interacting with the system or each other. The term "event-driven" has become inevitable, just like the need for traffic lights in a busy intersection. It refers to a programming paradigm where the flow of the program is determined by events, could be user actions or messages from other systems, these events are produced by clients, need to be consumed by other clients, and in the middle you have a message broker that gives you that sweet sweet reliability that messages will indeed be delivered.

One of the most popular tools for implementing event-driven architectures is RabbitMQ, a message broker that allows applications to communicate with each other asynchronously, making it easier to build scalable and decoupled systems. The mental model to have in mind is that of a post office, you have senders (producers) that send letters (messages) to the post office (RabbitMQ), and then you have receivers (consumers) that come to the post office to pick up their letters. The post office ensures that the letters are delivered to the right recipients, even if they are not available at the time of sending. This is the essence of event-driven architecture. 

We can talk about its cousin Apache Kafka another day. This weekend my focus was on RabbitMQ, the mental model required to understand it, and how to apply it in the most minimal way in order to understand the core code concepts that can be expanded upon later. So, let's see how to actually code a simple event-driven system using RabbitMQ. 

## Baby's first message broker

The first step I took was visiting the RabbitMQ website and immediately learning something interesting: the actual [protocols](https://www.rabbitmq.com/docs/protocols) and open standards powering the technology, those being **Advanced Message Queuing Protocol (AMQP)** and others, even the familiar [websocket](https://datatracker.ietf.org/doc/html/rfc6455) is supported for transport. Followed by the [protocol-specific differences](https://www.rabbitmq.com/docs/publishers#protocols) section, all in all, documentation is very well written and pleasant to read.

I was also delighted to confirm one of my assumptions, that you could very well setup clusters of RabbitMQ server nodes, and have a truly distributed, asyncronous system for producing and consuming events reliably accross a global scale, even, genuinely cool stuff.

Lets take a visual look at the architecture of what we are going to be building, especially so we can get a sense of the scale of distributed event driven systems.

{{< video src="/videos/scaling_msg_queue.mp4" >}}

thanks tim corey, very cool.

You can see how we can independently scale producers, consumers and even the message broker cluster itself, and how the message broker is the glue that holds everything together. 
Imagine the possibilities of doing this dynamically, to where you can spin up more producers or consumers based on demand, and the message broker decouples the system so that applications can communicate with each other without needing to be directly connected or even aware of each other's existence at all.

Cool. Now to actually install RabbitMQ, the broker, I went with the [Docker image](https://hub.docker.com/_/rabbitmq). I'm going to be setting up a docker-compose file so I can bring up all my services at once.

```yaml
 services:
  rabbitmq:
    image: rabbitmq:management
    ports:
      - "15671:15671"
      - "8081:15672"
    networks:
      - rabbitmq_network
...
```

## Time to get to work: .NET producers and consumers
Creating a producer and consumer in .NET is pretty straightforward using the [RabbitMQ.Client](https://www.nuget.org/packages/RabbitMQ.Client/) library.

I followed the syntax in this [tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet.html) to create a simple producer and consumer in no time. The producer will send messages to a queue, and the consumer is setup with an event handler that will eventfully receive messages from that queue, finally messages are acknowledged (acked) by the consumer to signal successful processing.

```csharp
app.MapPost("/message", async (string msg, string  client_id) =>
{
    var message = new Message(msg, client_id);
    app.Logger.LogInformation("API: Received message: {Message}", message.ToString());
    await channel.BasicPublishAsync(exchange: exchangeName, routingKey: routingKey, body: System.Text.Encoding.UTF8.GetBytes(message.ToString()));
    app.Logger.LogInformation("API: Published message to RabbitMQ exchange: {Exchange} with routing key: {RoutingKey}", exchangeName, routingKey);
})
.WithName("PostMessage");
```
Straightforward, as you noticed I setup an API endpoint to receive messages from clients, and then publish those messages to the RabbitMQ exchange. I have a simple groupchat application in mind.

### Consumer, plus non RabbitMQ, project overhead shenanigans

An interesting decision came up when I was setting up the consumer, I had my event handler setup to receive messages from the queue, but I had to decide how to finally deliver those messages to the clients of _my_ API, lets say, a React Native chat application. Messages could be polled by the clients, but that would be inefficient and not real-time, even though polling might be fine for some applications, RabbitMQ is designed for real-time messaging, so having clients poll the API for new messages, is not ideal. 

I decided to use SignalR, a library for ASP.NET I have used before, that allows for real-time web functionality, to push messages to subscribed online clients as soon as they are received by the consumer.

Perfect, almost, what if my clients are offline when the message is received? messages could get lost during the last leg of the journey. To solve this, the obvious solution is a simple, good old database. 

I added an in-memory store to hold messages for offline clients, and when they come back online, they can retrieve their missed messages since a timestamp. This way, we ensure that messages are not lost even if clients are temporarily unavailable.

Here is your consumer code dump, with the SignalR and in-memory buffer logic included:
```csharp
#region consumer
//--- Consumer setup: listens to RabbitMQ, stores in buffer, pushes to SignalR clients ---//

// Get SignalR hub context and message buffer for real-time + offline delivery
var hubContext = app.Services.GetRequiredService<IHubContext<MessageHub>>();
var messageBuffer = app.Services.GetRequiredService<MessageBuffer>();
// SignalR hub endpoint
app.MapHub<MessageHub>("/messageHub");

var consumer = new AsyncEventingBasicConsumer(channel);

//interesting syntax: the event handler is a function 'added' or extended with our own lambda expression
consumer.ReceivedAsync += async (model, ea) =>
{
    var body = ea.Body.ToArray();
    var messageJson = System.Text.Encoding.UTF8.GetString(body);
    app.Logger.LogInformation("Consumer: Received message from RabbitMQ: {Message}", messageJson);

    // Deserialize JSON to Message object
    var message = JsonSerializer.Deserialize<Message>(messageJson);
    if (message == null)
    {
        app.Logger.LogWarning("Consumer: Failed to deserialize message");
        return;
    }
    
    // signalr + buffer logic:
    // Step 1: Store in buffer first (ensures no message loss)
    messageBuffer.Add(messageJson, message.Client_id);
    app.Logger.LogInformation("Consumer: Message stored in buffer for client: {ClientId}", message.Client_id);

    // Step 2: Push to online SignalR clients
    try
    {
        await hubContext.Clients.All.SendAsync("ReceiveMessage", message);
        app.Logger.LogInformation("Consumer: Message pushed to SignalR clients");
    }
    catch (Exception ex)
    {
        app.Logger.LogWarning(ex, "Consumer: Failed to push via SignalR, but message is in buffer");
    }
};

await channel.BasicConsumeAsync(queue: queueName, autoAck: true, consumer: consumer);

// Endpoint for clients to retrieve missed messages after reconnection
app.MapGet("/messages/since", (DateTime since) =>
{
    return messageBuffer.GetSince(since);
}).WithName("GetMissedMessages");
#endregion
```

A couple out of scope extras, but it makes for a better demonstration of real time communication, and also shows how RabbitMQ can be integrated with other technologies to build a complete event-driven system.

Finally time for a lil video demo of the whole thing in action, I have a simple React Native app that connects to the SignalR hub and displays messages in real time, while also allowing users to post new messages via the API endpoint.

{{< video src="/videos/screenrecording-2026-02-09_10-10-46.mp4" >}}