# Lab Summary

[Back to Main Menu](../../../README.md)

In this lab you will work in pairs to create a chat app. Each partner in the pair will create a message bus queue to which you will send messages to your partner. Likewise you will use your partner's queue to receive message from them.

**Go ahead and pick a partner now!**

## Create a Message Bus namespace

A namespace is the logical container for queues and topics.

1. Sign into the Azure Portal [https://portal.azure.com/](https://portal.azure.com/)
1. From the left-hand menu click on "Create a Resource"
1. In the search bar of the New blade type "Service Bus"
1. From the search results select "Service Bus" and click Create
1. In the Create namespace blade:
    - Give your ServiceBus a unique name, e.g. azurerampup-lab-_nnn_
        - where _nnn_ is replaced by a unique number, e.g. 001
        - Some Azure resources (such as ServiceBus) have globally unique endpoints, therefore the name must be globally unique too.
    - Choose the Basic pricing tier
    - Choose to create a new Resource Group, e.g. azurerampup-lab-rg
        - If you are working in the same subscription as others you may find this resource group already exists
    - Choose the UK South region
1. Click Create

The Service Bus namespace will take approximately 1.5 minutes to be created.

## Create a Queue

Once the Service Bus namespace has been created, navigate to it's blade. You can do this by using the Azure search bar at the top of the screen and searching by resource name (e.g. azurerampup-lab-001) or from the home screen where it should now be listed under "Recent resources".

1. At the top of the main blade select "- Queue".
1. In the pop-up window give you queue a name, e.g. queue1. Leave other options as default and click Create.
1. Just like a database, your new Queue has a connection string which you'll need to make a copy of:
    1. In the Left-hand menu select "Shared access policies"
    1. Click `+ Add` to create a new policy. Give it a name (e.g. `queue1policy`) and select all of `Manage`, `Send` and `Listen`.
    1. Click `Create`.
    1. Click on the policy you have just created.
    1. From the Pop-up copy the "Primary Connection String"
        - This should start with "Endpoint=sb://<_your namespace_>.servicebus.windows.net..."

The connection string will be used by the chat application you are about to build in order to post messages. It currently has Manage, Send and Receive access rights, meaning the application with the connection string has the ability to Send and Receive messages and even to modify the settings of the queue. In practice this would not be case and a connection string would give an application the right to perform just one of these tasks.

## Create a simple chat application

1. Using visual Studio, create a new .NET Console project.
    - File -> New Project
    - From the left menu Visual C# -> .NET Core
    - Choose "Console App (.NET Core)"
    - Provide a name for your project, e.g. ChatApp
    - Click OK
1. Add the Azure ServiceBus NuGet package
    - Right-click the project in the Solution Explorer
    - Manage NuGet Packages...
    - Change to the Browse tab at the top of the window
    - Search for "Microsoft.Azure.ServiceBus"
    - Select the package and click Install
    - Click OK in the Preview Changes window, and "I Accept" in the License Acceptance window.
1. Open the Program.cs file and replace ALL the code with that shown in the example below.
    - Read through the code and ensure you understand the structure of how the application works.
1. Using the ServiceBus namespace connection string and queue name you created earlier (e.g. queue1) you made note of previously, set your queue's connection string and name in the `SendServiceBusConnectionString` and `SendQueueName` variables.
1. Using your partner's connection string and queue name, set the `ReceiverServiceBusConnectionString` and `ReceiverQueueName`.
1. Build the application and run it.

You should now be able to type and send messages to your partner, and receive messages back from them.  Notice that messages sent whilst either you or your partner's application is not running will be received when the application is restarted.

### C# Chat Application

```CSharp
using Microsoft.Azure.ServiceBus;
using System;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace ChatApp
{
    class Program
    {
        const string SendServiceBusConnectionString = "<sender queue connection string>";
        const string SendQueueName = "<sender queue name>";
        static IQueueClient sendQueueClient;

        const string ReceiveServiceBusConnectionString = "<receiver queue connection string>";
        const string ReceiveQueueName = "<receiver queue name>";
        static IQueueClient receiveQueueClient;

        static async Task Main(string[] args)
        {
            InitializeReceiveQueue();

            await MainLoopAsync();
        }


        /****************************************************\
        |- SENDER                                           *|
        \****************************************************/

        /// <summary>
        /// Waits for user input and send the text via message bus.
        /// </summary>
        private static async Task MainLoopAsync()
        {
            sendQueueClient = new QueueClient(SendServiceBusConnectionString, SendQueueName);

            while (true)
            {
                var text = Console.ReadLine();

                await SendMessageAsync(text);
            }

            // Usually we'd have a way to break the while-loop
            await sendQueueClient.CloseAsync();
        }

        /// <summary>
        /// Sends a text message in encoded UTF to the sender queue.
        /// </summary>
        private static async Task SendMessageAsync(string text)
        {
            try
            {
                byte[] encodedText = Encoding.UTF8.GetBytes(text);

                Message message = new Message(encodedText);

                await sendQueueClient.SendAsync(message);
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
            }
        }


        /****************************************************\
        |- RECEIVER                                         *|
        \****************************************************/

        /// <summary>
        /// Create a receiver queue, and setup the message handler
        /// </summary>
        private static void InitializeReceiveQueue()
        {
            receiveQueueClient = new QueueClient(ReceiveServiceBusConnectionString, ReceiveQueueName);

            //Register the message and exception handlers
            receiveQueueClient.RegisterMessageHandler(ProcessMessagesAsync, ExceptionHandlerAsync);
        }

        /// <summary>
        /// Callback method, called when a message is received
        /// </summary>
        private static Task ProcessMessagesAsync(Message message, CancellationToken token)
        {
            byte[] encodedText = message.Body;

            string text = Encoding.UTF8.GetString(encodedText);

            Console.WriteLine();
            Console.WriteLine($">> {text}");
            Console.WriteLine();

            return Task.CompletedTask;
        }

        /// <summary>
        /// Callback method, called when an exception is encountered during message processing
        /// </summary>
        private static Task ExceptionHandlerAsync(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
        {
            Console.WriteLine($"{DateTime.Now} :: Exception: {exceptionReceivedEventArgs.Exception.Message}");

            return Task.CompletedTask;
        }
    }
}
```
