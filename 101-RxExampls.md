101 Rx Samples in C#
==================

This was taken from http://rxwiki.wikidot.com/101samples, because I wanted to be able to read it more comfortable with syntax highlighting.

Here's the unedited original, translated to Github Markdown glory:


# 101 Rx Samples - a work in progress

**You!**

**Yes, you, the one who is still scratching their head trying to figure out this Rx thing.
As you learn and explore, please feel free add your own samples here (or tweak existing ones!)
Anyone can (and should!) edit this page. (edit button is at the bottom right of each page)**

*(and sorry for the years of spam pages there - they should be gone now. Thanks for marking them. -Rob)*

## Table of Contents

* [Asynchronous Background Operations](#asynchronous-background-operations)
    * [Start - Run Code Asynchronously](#start---run-code-asynchronously)
    * [Run a method asynchronously on demand](#run-a-method-asynchronously-on-demand)
    * [CombineLatest - Parallel Execution](#combinelatest---parallel-execution)
    * [Create With Disposable & Scheduler - Canceling an asynchronous operation](#create-with-disposable-&-scheduler---canceling-an-asynchronous-operation)
* [Observation Operators](#observation-operators)
    * [Observing an Event - Simple](#observing-an-event---simple)
    * [Observing an Event - Simple (expanded)](#observing-an-event---simple-expanded)
    * [Observing MouseMove in Silverlight](#observing-mousemove-in-silverlight)
    * [Observing an Event - Generic](#observing-an-event---generic)
    * [Observing an Event - Non-Generic](#observing-an-event---non-generic)
    * [Observing an Asynchronous Operation](#observing-an-asynchronous-operation)
    * [Observing a Generic IEnumerable](#observing-a-generic-ienumerable)
    * [Observing a Non-Generic IEnumerable - Single Type](#observing-a-non-generic-ienumerable---single-type)
    * [Observing a Non-Generic IEnumerable - Multiple Types](#observing-a-non-generic-ienumerable---multiple-types)
    * [Observing the Passing of Time](#observing-the-passing-of-time)
* [Restriction Operators](#restriction-operators)
    * [Where - Simple](#where---simple)
    * [Where - Drilldown](#where---drilldown)
* [Projection Operators](#projection-operators)
    * [Select - Simple](#select---simple)
    * [Select - Transformation](#select---transformation)
    * [Select - Indexed](#select---indexed)
* [Grouping](#grouping)
    * [Group By - Simple](#group-by---simple)
* [Time-Related Operators](#time-related-operators)
    * [Buffer - Simple](#buffer---simple)
    * [Delay - Simple](#delay---simple)
    * [Interval - Simple](#interval---simple)
    * [Sample - Simple](#sample---simple)
    * [Throttle - Simple](#throttle---simple)
    * [Interval - With TimeInterval() - Simple](#interval---with-timeinterval---simple)
    * [Interval - With TimeInterval() - Remove](#interval---with-timeinterval---remove)
    * [Timeout - Simple](#timeout---simple)
    * [Timer - Simple](#timer---simple)
    * [Timestamp - Simple](#timestamp---simple)
    * [Timestamp - Remove](#timestamp---remove)
* [Window and Joins](#window-and-joins)
    * [Window](#window)
    * [GroupJoin - Joins two streams matching by one of their attributes](#groupjoin---joins-two-streams-matching-by-one-of-their-attributes)
* [Range](#range)
    * [Range - Prints from 1 to 10.](#range---prints-from-1-to-10.)
* [Generate](#generate)
    * [Generate - simple](#generate---simple)
* [ISubject<T> and ISubject<T1, T2>](#isubjectt-and-isubjectt1,-t2)
    * [Ping Pong Actor Model with ISubject<T1, T2>](#ping-pong-actor-model-with-isubjectt1,-t2)
* [Combination Operators](#combination-operators)
    * [Merge](#merge)
    * [Publish - Sharing a subscription with multiple Observers](#publish---sharing-a-subscription-with-multiple-observers)
    * [Zip](#zip)
    * [CombineLatest](#combinelatest)
    * [Concat - cold observable](#concat---cold-observable)
    * [Concat - hot observable](#concat---hot-observable)
* [Make your class native to IObservable<T>](#make-your-class-native-to-iobservablet)
    * [Use Subject<T> as backend for IObservable<T>](#use-subjectt-as-backend-for-iobservablet)

## Asynchronous Background Operations

### Start - Run Code Asynchronously


```C#
public static void StartBackgroundWork() {
    Console.WriteLine("Shows use of Start to start on a background thread:");
    var o = Observable.Start(() =>
    {
        //This starts on a background thread.
        Console.WriteLine("From background thread. Does not block main thread.");
        Console.WriteLine("Calculating...");
        Thread.Sleep(3000);
        Console.WriteLine("Background work completed.");
    }).Finally(() => Console.WriteLine("Main thread completed."));
    Console.WriteLine("\r\n\t In Main Thread...\r\n");
    o.Wait();   // Wait for completion of background operation.
}
```


### Run a method asynchronously on demand

Execute a long-running method asynchronously. The method does not start running until there is a subscriber. The method is started every time the observable is created and subscribed, so there could be more than one running at once.

```C#
// Synchronous operation
public DataType DoLongRunningOperation(string param)
{
    ...
}

public IObservable<DataType> LongRunningOperationAsync(string param)
{
    return Observable.Create<DataType>(
        o => Observable.ToAsync<string,DataType>(DoLongRunningOperation)(param).Subscribe(o)
    );
}
```


### CombineLatest - Parallel Execution

Merges the specified observable sequences into one observable sequence by emitting a list with the latest source elements whenever any of the observable sequences produces an element.

```C#
public async void ParallelExecutionTest()
{
    var o = Observable.CombineLatest(
        Observable.Start(() => { Console.WriteLine("Executing 1st on Thread: {0}", Thread.CurrentThread.ManagedThreadId); return "Result A"; }),
        Observable.Start(() => { Console.WriteLine("Executing 2nd on Thread: {0}", Thread.CurrentThread.ManagedThreadId); return "Result B"; }),
        Observable.Start(() => { Console.WriteLine("Executing 3rd on Thread: {0}", Thread.CurrentThread.ManagedThreadId); return "Result C"; }) 
    ).Finally(() => Console.WriteLine("Done!"));

    foreach (string r in await o.FirstAsync())
        Console.WriteLine(r);
}
```



<strong>Result</strong><br />
Executing 1st on Thread: 3<br />
Executing 2nd on Thread: 4<br />
Executing 3rd on Thread: 3<br />
Done!<br />
Result A<br />
Result B<br />
Result C

<strong>Note</strong> Was ForkJoin which is no longer supported. CombineLatest gives the same result.)

### Create With Disposable &amp; Scheduler - Canceling an asynchronous operation

<p>This sample starts a background operation that generates a sequence of integers until it is canceled by the main thread. To start the background operation new the Scheduler class is used and a CancellationTokenSource is indirectly created by a Observable.Create.<br />
Please check out the MSDN documentation on <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtokensource%28VS.100%29.aspx">System.Threading.CancellationTokenSource</a> to learn more about cancellation source.</p>

```C#
IObservable<int> ob =
    Observable.Create<int>(o =>
        {
            var cancel = new CancellationDisposable(); // internally creates a new CancellationTokenSource
            NewThreadScheduler.Default.Schedule(() =>
                {
                    int i = 0;
                    for (; ; )
                    {
                        Thread.Sleep(200);  // here we do the long lasting background operation
                        if (!cancel.Token.IsCancellationRequested)    // check cancel token periodically
                            o.OnNext(i++);
                        else
                        {
                            Console.WriteLine("Aborting because cancel event was signaled!");
                            o.OnCompleted();
                            return;
                        }
                    }
                }
            );

            return cancel;
        }
    );

IDisposable subscription = ob.Subscribe(i => Console.WriteLine(i));
Console.WriteLine("Press any key to cancel");
Console.ReadKey();
subscription.Dispose();
Console.WriteLine("Press any key to quit");
Console.ReadKey();  // give background thread chance to write the cancel acknowledge message
```


## Observation Operators

### Observing an Event - Simple


```C#
class ObserveEvent_Simple
{
    public static event EventHandler SimpleEvent;
    static void Main()
    {
        // To consume SimpleEvent as an IObservable:
        var eventAsObservable = Observable.FromEventPattern(
            ev => SimpleEvent += ev,
            ev => SimpleEvent -= ev);
    }
}
```

Alternately, you can use EventArgs:

```C#
public static event EventHandler<EventArgs> SimpleEvent;

private static void Main(string[] args) {
    var eventAsObservable = Observable.FromEventPattern<EventArgs>
        (ev => SimpleEvent += ev,
         ev => SimpleEvent -= ev);
 }
```


### Observing an Event - Simple (expanded)


```C#
class ObserveEvent_Simple
{
    public static event EventHandler SimpleEvent;

    private static void Main()
    {
        Console.WriteLine("Setup observable");
        // To consume SimpleEvent as an IObservable:
        var eventAsObservable = Observable.FromEventPattern(
                ev => SimpleEvent += ev,
                ev => SimpleEvent -= ev);

        // SimpleEvent is null until we subscribe
        Console.WriteLine(SimpleEvent == null ? "SimpleEvent == null" : "SimpleEvent != null");

        Console.WriteLine("Subscribe");
        //Create two event subscribers
        var s = eventAsObservable.Subscribe(args => Console.WriteLine("Received event for s subscriber"));
        var t = eventAsObservable.Subscribe(args => Console.WriteLine("Received event for t subscriber"));

        // After subscribing the event handler has been added
        Console.WriteLine(SimpleEvent == null ? "SimpleEvent == null" : "SimpleEvent != null");

        Console.WriteLine("Raise event");
        if (null != SimpleEvent)
        {
            SimpleEvent(null, EventArgs.Empty);
        }

        // Allow some time before unsubscribing or event may not happen
        Thread.Sleep(100);

        Console.WriteLine("Unsubscribe");
        s.Dispose();
        t.Dispose();

        // After unsubscribing the event handler has been removed
        Console.WriteLine(SimpleEvent == null ? "SimpleEvent == null" : "SimpleEvent != null");

        Console.ReadKey();
    }
}
```


### Observing MouseMove in Silverlight


```C#
var mouseMove = Observable.FromEventPattern<MouseEventArgs>(this, "MouseMove");
mouseMove.ObserveOnDispatcher()
         .Subscribe(args => Debug.WriteLine(args.EventArgs.GetPosition(this)));
```



Note that a reference to System.Reactive.Windows.Threading is required for ObserveOnDispatcher which is in Nuget as Reactive Extensions - Silverlight Helpers.

### Observing an Event - Generic


```C#
class ObserveEvent_Generic
{
    public class SomeEventArgs : EventArgs { }
    public static event EventHandler<SomeEventArgs> GenericEvent;

    static void Main()
    {
        // To consume GenericEvent as an IObservable:
        IObservable<EventPattern<SomeEventArgs>> eventAsObservable = Observable.FromEventPattern<SomeEventArgs>(
            ev => GenericEvent += ev,
            ev => GenericEvent -= ev );
    }
}
```


### Observing an Event - Non-Generic


```C#
class ObserveEvent_NonGeneric
{
    public class SomeEventArgs : EventArgs { }
    public delegate void SomeNonGenericEventHandler(object sender, SomeEventArgs e);
    public static event SomeNonGenericEventHandler NonGenericEvent;

    static void Main()
    {
        // To consume NonGenericEvent as an IObservable, first inspect the type of EventArgs used in the second parameter of the delegate.
        // In this case, it is SomeEventArgs.  Then, use as shown below.
        IObservable<IEvent<SomeEventArgs>> eventAsObservable = Observable.FromEvent(
            (EventHandler<SomeEventArgs> ev) => new SomeNonGenericEventHandler(ev), 
            ev => NonGenericEvent += ev,
            ev => NonGenericEvent -= ev);
    }
}
```


### Observing an Asynchronous Operation


```C#
class Observe_IAsync
{
    static void Main()
    {
        // We will use Stream's BeginRead and EndRead for this sample.
        Stream inputStream = Console.OpenStandardInput();

        // To convert an asynchronous operation that uses the IAsyncResult pattern to a function that returns an IObservable, use the following format.  
        // For the generic arguments, specify the types of the arguments of the Begin* method, up to the AsyncCallback.
        // If the End* method returns a value, append this as your final generic argument.
        var read = Observable.FromAsyncPattern<byte[], int, int, int>(inputStream.BeginRead, inputStream.EndRead);

        // Now, you can get an IObservable instead of an IAsyncResult when calling it.
        byte[] someBytes = new byte[10];
        IObservable<int> observable = read(someBytes, 0, 10);
    }
}
```


<p>Be aware that while the code above formally provides an observable, this is not enough for most intended uses. For more information, see<br />
<a href="http://www.introtorx.com/uat/content/v1.0.10621.0/04_CreatingObservableSequences.html#FromAPM">Creating an observable sequence</a> and<br />
<a href="http://stackoverflow.com/questions/14454766/what-is-the-proper-way-to-create-an-observable-which-reads-a-stream-to-the-end">c# - What is the proper way to create an Observable which reads a stream to the end - Stack Overflow</a>.</p>

### Observing a Generic IEnumerable


```C#
class Observe_GenericIEnumerable
{
    static void Main()
    {
        IEnumerable<int> someInts = new List<int> { 1, 2, 3, 4, 5 };

        // To convert a generic IEnumerable into an IObservable, use the ToObservable extension method.
        IObservable<int> observable = someInts.ToObservable();
    }
}
```


### Observing a Non-Generic IEnumerable - Single Type


```C#
class Observe_NonGenericIEnumerableSingleType
{
    static void Main()
    {
        IEnumerable someInts = new object[] { 1, 2, 3, 4, 5 };

        // To convert a non-generic IEnumerable that contains elements of a single type,
        // first use Cast<> to change the non-generic enumerable into a generic enumerable,
        // then use ToObservable.
        IObservable<int> observable = someInts.Cast<int>().ToObservable();
    }
}
```


### Observing a Non-Generic IEnumerable - Multiple Types

### Observing the Passing of Time


```C#
class Observe_Time
{
    static void Main()
    {
        // To observe time passing, use the Observable.Interval function.
        // It will notify you on a time interval you specify.

        // 0 after 1s, 1 after 2s, 2 after 3s, etc.
        IObservable<long> oneNumberPerSecond = Observable.Interval(TimeSpan.FromSeconds(1));
        IObservable<long> alsoOneNumberPerSecond = Observable.Interval(1000 /* milliseconds */);
    }
}
```


## Restriction Operators

### Where - Simple


```C#
class Where_Simple
{
    static void Main()
    {
        var oneNumberPerSecond = Observable.Interval(TimeSpan.FromSeconds(1));

        var lowNums = from n in oneNumberPerSecond
                      where n < 5
                      select n;

        Console.WriteLine("Numbers < 5:");

        lowNums.Subscribe(lowNum =>
        {
            Console.WriteLine(lowNum);
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
Numbers < 5:<br />
0 <em>(after 1s)</em><br />
1 <em>(after 2s)</em><br />
2 <em>(after 3s)</em><br />
3 <em>(after 4s)</em><br />
4 <em>(after 5s)</em>

### Where - Drilldown


```C#
class Where_DrillDown
{
    class Customer
    {
        public Customer() { Orders = new ObservableCollection<Order>(); }
        public string CustomerName { get; set; }
        public string Region { get; set; }
        public ObservableCollection<Order> Orders { get; private set; }
    }

    class Order
    {
        public int OrderId { get; set; }
        public DateTimeOffset OrderDate { get; set; }
    }

    static void Main()
    {
        var customers = new ObservableCollection<Customer>();

        var customerChanges = Observable.FromEventPattern(
            (EventHandler<NotifyCollectionChangedEventArgs> ev)
               => new NotifyCollectionChangedEventHandler(ev),
            ev => customers.CollectionChanged += ev,
            ev => customers.CollectionChanged -= ev);

        var watchForNewCustomersFromWashington =
            from c in customerChanges
            where c.EventArgs.Action == NotifyCollectionChangedAction.Add
            from cus in c.EventArgs.NewItems.Cast<Customer>().ToObservable()
            where cus.Region == "WA"
            select cus;

        Console.WriteLine("New customers from Washington and their orders:");

        watchForNewCustomersFromWashington.Subscribe(cus =>
        {
            Console.WriteLine("Customer {0}:", cus.CustomerName);

            foreach (var order in cus.Orders)
            {
                Console.WriteLine("Order {0}: {1}", order.OrderId, order.OrderDate);
            }
        });

        customers.Add(new Customer
        {
            CustomerName = "Lazy K Kountry Store",
            Region = "WA",
            Orders = { new Order { OrderDate = DateTimeOffset.Now, OrderId = 1 } }
        });

        Thread.Sleep(1000);
        customers.Add(new Customer
        {
            CustomerName = "Joe's Food Shop",
            Region = "NY",
            Orders = { new Order { OrderDate = DateTimeOffset.Now, OrderId = 2 } }
        });

        Thread.Sleep(1000);
        customers.Add(new Customer
        {
            CustomerName = "Trail's Head Gourmet Provisioners",
            Region = "WA",
            Orders = { new Order { OrderDate = DateTimeOffset.Now, OrderId = 3 } }
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
New customers from Washington and their orders:<br />
Customer Lazy K Kountry Store: <em>(after 0s)</em><br />
Order 1: 11/20/2009&#160;11:52:02 AM -06:00<br />
Customer Trail's Head Gourmet Provisioners: <em>(after 2s)</em><br />
Order 3: 11/20/2009&#160;11:52:04 AM -06:00

## Projection Operators

### Select - Simple


```C#
class Select_Simple
{
    static void Main()
    {
        var oneNumberPerSecond = Observable.Interval(TimeSpan.FromSeconds(1));

        var numbersTimesTwo = from n in oneNumberPerSecond
                              select n * 2;

        Console.WriteLine("Numbers * 2:");

        numbersTimesTwo.Subscribe(num =>
        {
            Console.WriteLine(num);
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
Numbers * 2:<br />
0 <em>(after 1s)</em><br />
2 <em>(after 2s)</em><br />
4 <em>(after 3s)</em><br />
6 <em>(after 4s)</em><br />
8 <em>(after 5s)</em>

### Select - Transformation


```C#
class Select_Transform
{
    static void Main()
    {
        var oneNumberPerSecond = Observable.Interval(TimeSpan.FromSeconds(1));

        var stringsFromNumbers = from n in oneNumberPerSecond
                                 select new string('*', (int)n);

        Console.WriteLine("Strings from numbers:");

        stringsFromNumbers.Subscribe(num =>
        {
            Console.WriteLine(num);
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
Strings from numbers:<br />
<em>(after 0s)</em><br />
<span style="white-space: pre-wrap;">*</span> <em>(after 1s)</em><br />
<span style="white-space: pre-wrap;">**</span> <em>(after 2s)</em><br />
<span style="white-space: pre-wrap;">***</span> <em>(after 3s)</em><br />
<span style="white-space: pre-wrap;">****</span> <em>(after 4s)</em><br />
<span style="white-space: pre-wrap;">*****</span> <em>(after 5s)</em><br />
<span style="white-space: pre-wrap;">******</span> <em>(after 6s)</em>

### Select - Indexed


```C#
class Where_Indexed
{
    class TimeIndex
    {
        public TimeIndex(int index, DateTimeOffset time)
        {
            Index = index;
            Time = time;
        }
        public int Index { get; set; }
        public DateTimeOffset Time { get; set; }
    }

    static void Main()
    {
        var clock = Observable.Interval(TimeSpan.FromSeconds(1))
            .Select((t, index) => new TimeIndex(index, DateTimeOffset.Now));

        clock.Subscribe(timeIndex =>
        {
            Console.WriteLine(
                "Ding dong.  The time is now {0:T}.  This is event number {1}.",
                timeIndex.Time,
                timeIndex.Index);
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
Ding dong. The time is now 1:55:00 PM. This is event number 0. <em>(after 0s)</em><br />
Ding dong. The time is now 1:55:01 PM. This is event number 1. <em>(after 1s)</em><br />
Ding dong. The time is now 1:55:02 PM. This is event number 2. <em>(after 2s)</em><br />
Ding dong. The time is now 1:55:03 PM. This is event number 3. <em>(after 3s)</em><br />
Ding dong. The time is now 1:55:04 PM. This is event number 4. <em>(after 4s)</em><br />
Ding dong. The time is now 1:55:05 PM. This is event number 5. <em>(after 5s)</em>

## Grouping

### Group By - Simple

<p>This example counts how many time you press each key as you furiously hit the keyboard. :)</p>

```C#
class GroupBy_Simple
{
    static IEnumerable<ConsoleKeyInfo> KeyPresses()
    {
        for (; ; )
        {
            var currentKey = Console.ReadKey(true);

            if (currentKey.Key == ConsoleKey.Enter)
                yield break;
            else
                yield return currentKey;
        }
    }
    static void Main()
    {
        var timeToStop = new ManualResetEvent(false);
        var keyPresses = KeyPresses().ToObservable();

        var groupedKeyPresses =
            from k in keyPresses
            group k by k.Key into keyPressGroup
            select keyPressGroup;

        Console.WriteLine("Press Enter to stop.  Now bang that keyboard!");

        groupedKeyPresses.Subscribe(keyPressGroup =>
        {
            int numberPresses = 0;

            keyPressGroup.Subscribe(keyPress =>
            {
                Console.WriteLine(
                    "You pressed the {0} key {1} time(s)!",
                    keyPress.Key,
                    ++numberPresses);
            },
            () => timeToStop.Set());
        });

        timeToStop.WaitOne();
    }
}
```



<strong>Result</strong><br />
<em>Depends on what you press! But something like:</em><br />
Press Enter to stop. Now bang that keyboard!<br />
You pressed the A key 1 time(s)!<br />
You pressed the A key 2 time(s)!<br />
You pressed the B key 1 time(s)!<br />
You pressed the B key 2 time(s)!<br />
You pressed the C key 1 time(s)!<br />
You pressed the C key 2 time(s)!<br />
You pressed the C key 3 time(s)!<br />
You pressed the A key 3 time(s)!<br />
You pressed the B key 3 time(s)!<br />
You pressed the A key 4 time(s)!<br />
You pressed the A key 5 time(s)!<br />
You pressed the C key 4 time(s)!

## Time-Related Operators

### Buffer - Simple

<p>Buffer has a strange name, but a simple concept.<br />
Imagine an email program that checks for new mail every 5 minutes. While you can receive mail at any instant in time, you only get a batch of emails at every five minute mark.<br />
Let's use Buffer to simulate this.</p>

```C#
class Buffer_Simple
{
    static IEnumerable<string> EndlessBarrageOfEmail()
    {
        var random = new Random();
        var emails = new List<String> { "Here is an email!", "Another email!", "Yet another email!" };
        for (; ; )
        {
            // Return some random emails at random intervals.
            yield return emails[random.Next(emails.Count)];
            Thread.Sleep(random.Next(1000));
        }
    }
    static void Main()
    {
        var myInbox = EndlessBarrageOfEmail().ToObservable();

        // Instead of making you wait 5 minutes, we will just check every three seconds instead. :)
        var getMailEveryThreeSeconds = myInbox.Buffer(TimeSpan.FromSeconds(3)); //  Was .BufferWithTime(...

        getMailEveryThreeSeconds.Subscribe(emails =>
        {
            Console.WriteLine("You've got {0} new messages!  Here they are!", emails.Count());
            foreach (var email in emails)
            {
                Console.WriteLine("> {0}", email);
            }
            Console.WriteLine();
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
You've got 5 new messages! Here they are! <em>(after 3s)</em><br />
<span style="white-space: pre-wrap;">></span> Here is an email!<br />
<span style="white-space: pre-wrap;">></span> Another email!<br />
<span style="white-space: pre-wrap;">></span> Here is an email!<br />
<span style="white-space: pre-wrap;">></span> Another email!<br />
<span style="white-space: pre-wrap;">></span> Here is an email!

You've got 6 new messages! Here they are! <em>(after 6s)</em><br />
<span style="white-space: pre-wrap;">></span> Another email!<br />
<span style="white-space: pre-wrap;">></span> Another email!<br />
<span style="white-space: pre-wrap;">></span> Here is an email!<br />
<span style="white-space: pre-wrap;">></span> Here is an email!<br />
<span style="white-space: pre-wrap;">></span> Another email!<br />
<span style="white-space: pre-wrap;">></span> Another email!

### Delay - Simple


```C#
class Delay_Simple
{
    static void Main()
    {
        var oneNumberEveryFiveSeconds = Observable.Interval(TimeSpan.FromSeconds(5));

        // Instant echo
        oneNumberEveryFiveSeconds.Subscribe(num =>
        {
            Console.WriteLine(num);
        });

        // One second delay
        oneNumberEveryFiveSeconds.Delay(TimeSpan.FromSeconds(1)).Subscribe(num =>
        {
            Console.WriteLine("...{0}...", num);
        });

        // Two second delay
        oneNumberEveryFiveSeconds.Delay(TimeSpan.FromSeconds(2)).Subscribe(num =>
        {
            Console.WriteLine("......{0}......", num);
        });

        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0 <em>(after 5s)</em><br />
&#8230;0&#8230; <em>(after 6s)</em><br />
&#8230;&#8230;0&#8230;&#8230; <em>(after 7s)</em><br />
1 <em>(after 10s)</em><br />
&#8230;1&#8230; <em>(after 11s)</em><br />
&#8230;&#8230;1&#8230;&#8230; <em>(after 12s)</em>

### Interval - Simple


```C#
internal class Interval_Simple
{
    private static void Main()
    {
        IObservable<long> observable = Observable.Interval(TimeSpan.FromSeconds(1));

        using (observable.Subscribe(Console.WriteLine))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0 <em>(after 1s)</em><br />
1 <em>(after 2s)</em><br />
2 <em>(after 3s)</em><br />
3 <em>(after 4s)</em><br />
&#8230;

### Sample - Simple


```C#
internal class Sample_Simple
{
    private static void Main()
    {
        // Generate sequence of numbers, (an interval of 50 ms seems to result in approx 16 per second).
        IObservable<long> observable = Observable.Interval(TimeSpan.FromMilliseconds(50));

        // Sample the sequence every second
        using (observable.Sample(TimeSpan.FromSeconds(1)).Timestamp().Subscribe(
            x => Console.WriteLine("{0}: {1}", x.Value, x.Timestamp)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
15: 24/11/2009&#160;15:40:45 <em>(after 1s)</em><br />
31: 24/11/2009&#160;15:40:46 <em>(after 2s)</em><br />
47: 24/11/2009&#160;15:40:47 <em>(after 3s)</em><br />
64: 24/11/2009&#160;15:40:48 <em>(after 4s)</em><br />
&#8230;

### Throttle - Simple

<p>Throttle stops the flow of events until no more events are produced for a specified period of time. For example, if you throttle a TextChanged event of a textbox to .5 seconds, no events will be passed until the user has stopped typing for .5 seconds. This is useful in search boxes where you do not want to start a new search after every keystroke, but want to wait until the user pauses.</p>

```C#
SearchTextChangedObservable = Observable.FromEventPattern<TextChangedEventArgs>(this.textBox, "TextChanged");
_currentSubscription = SearchTextChangedObservable.Throttle(TimeSpan.FromSeconds(.5)).ObserveOnDispatcher().Subscribe(e => this.ListItems.Add(this.textBox.Text));
```


<p>Here is another example:</p>

```C#
internal class Throttle_Simple
{
    // Generates events with interval that alternates between 500ms and 1000ms every 5 events
    static IEnumerable<int> GenerateAlternatingFastAndSlowEvents()
    {
        int i = 0;

        while(true)
        {
            if(i > 1000)
            {
                yield break;
            }
            yield return i;
            Thread.Sleep( i++ % 10 < 5 ? 500 : 1000);
        }
    }

    private static void Main()
    {
        var observable = GenerateAlternatingFastAndSlowEvents().ToObservable().Timestamp();
        var throttled = observable.Throttle(TimeSpan.FromMilliseconds(750));

        using (throttled.Subscribe(x => Console.WriteLine("{0}: {1}", x.Value, x.Timestamp)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
5: <timestamp><br />
6: <timestamp><br />
7: <timestamp><br />
8: <timestamp><br />
9: <timestamp><br />
15: <timestamp><br />
16: <timestamp><br />
17: <timestamp><br />
18: <timestamp><br />
19: <timestamp><br />
&#8230;etc

### Interval - With TimeInterval() - Simple


```C#
internal class TimeInterval_Simple
{
    // Like TimeStamp but gives the time-interval between successive values
    private static void Main()
    {
        var observable = Observable.Interval(TimeSpan.FromMilliseconds(750)).TimeInterval();

        using (observable.Subscribe(
            x => Console.WriteLine("{0}: {1}", x.Value, x.Interval)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0: 00:00:00.8090459 <em>(1st value)</em><br />
1: 00:00:00.7610435 <em>(2nd value)</em><br />
2: 00:00:00.7650438 <em>(3rd value)</em><br />
&#8230;

### Interval - With TimeInterval() - Remove


```C#
internal class TimeInterval_Remove
{
    private static void Main()
    {
        // Add a time interval
        var observable = Observable.Interval(TimeSpan.FromMilliseconds(750)).TimeInterval();

        // Remove it again
        using (observable.RemoveTimeInterval().Subscribe(Console.WriteLine))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0<br />
1<br />
2<br />
&#8230;

### Timeout - Simple


```C#
internal class Timeout_Simple
{
    private static void Main()
    {
        Console.WriteLine(DateTime.Now);

        // create a single event in 10 seconds time
        var observable = Observable.Timer(TimeSpan.FromSeconds(10)).Timestamp();

        // raise exception if no event received within 9 seconds
        var observableWithTimeout = Observable.Timeout(observable, TimeSpan.FromSeconds(9));

        using (observableWithTimeout.Subscribe(
            x => Console.WriteLine("{0}: {1}", x.Value, x.Timestamp), 
            ex => Console.WriteLine("{0} {1}", ex.Message, DateTime.Now)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
02/12/2009&#160;10:13:00<br />
Press any key to unsubscribe<br />
The operation has timed out. 02/12/2009&#160;10:13:09<br />
&#8230;

### Timer - Simple

<p>Observable.Interval is a simple wrapper around Observable.Timer.</p>

```C#
internal class Timer_Simple
{
    private static void Main()
    {
        Console.WriteLine(DateTime.Now);

        var observable = Observable.Timer(TimeSpan.FromSeconds(5), 
                                                       TimeSpan.FromSeconds(1)).Timestamp();

        // or, equivalently
        // var observable = Observable.Timer(DateTime.Now + TimeSpan.FromSeconds(5), 
        //                                                TimeSpan.FromSeconds(1)).Timestamp();

        using (observable.Subscribe(
            x => Console.WriteLine("{0}: {1}", x.Value, x.Timestamp)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
02/12/2009&#160;10:02:29<br />
Press any key to unsubscribe<br />
0: 02/12/2009&#160;10:02:34<em>(after 5s)</em><br />
1: 02/12/2009&#160;10:02:35 <em>(after 6s)</em><br />
2: 02/12/2009&#160;10:02:36 <em>(after 7s)</em><br />
&#8230;

### Timestamp - Simple

<p>Adds a TimeStamp to each element using the system's local time.</p>

```C#
internal class Timestamp_Simple
{
    private static void Main()
    {
        var observable = Observable.Interval(TimeSpan.FromSeconds(1)).Timestamp();

        using (observable.Subscribe(
            x => Console.WriteLine("{0}: {1}", x.Value, x.Timestamp)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0: 24/11/2009&#160;15:40:45 <em>(after 1s)</em><br />
1: 24/11/2009&#160;15:40:46 <em>(after 2s)</em><br />
2: 24/11/2009&#160;15:40:47 <em>(after 3s)</em><br />
3: 24/11/2009&#160;15:40:48 <em>(after 4s)</em><br />
&#8230;

### Timestamp - Remove


```C#
internal class Timestamp_Remove
{
    private static void Main()
    {
        // Add timestamp
        var observable = Observable.Interval(TimeSpan.FromSeconds(1)).Timestamp();

        // Remove it
        using (observable.RemoveTimestamp().Subscribe(Console.WriteLine))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
0 <em>(after 1s)</em><br />
1 <em>(after 2s)</em><br />
2 <em>(after 3s)</em><br />
3 <em>(after 4s)</em><br />
&#8230;

## Window and Joins

### Window

<p>Divides a stream into "Windows" of time. For example, 5 five second window would contain all elements pushed in that five second interval.</p>

```C#
IObservable<long> mainSequence = Observable.Interval(TimeSpan.FromSeconds(1));
IObservable<IObservable<long>> seqWindowed = mainSequence.Window(() =>
    {
        IObservable<long> seqWindowControl = Observable.Interval(TimeSpan.FromSeconds(6));
        return seqWindowControl;
    });

seqWindowed.Subscribe(seqWindow =>
    {
        Console.WriteLine("\nA new window into the main sequence has opened: {0}\n",
                            DateTime.Now.ToString());
        seqWindow.Subscribe(x => { Console.WriteLine("Integer : {0}", x); });
    });

Console.ReadLine();
```


### GroupJoin - Joins two streams matching by one of their attributes


```C#
var leftList = new List<string[]>();
leftList.Add(new string[] { "2013-01-01 02:00:00", "Batch1" });
leftList.Add(new string[] { "2013-01-01 03:00:00", "Batch2" });
leftList.Add(new string[] { "2013-01-01 04:00:00", "Batch3" });

var rightList = new List<string[]>();
rightList.Add(new string[] { "2013-01-01 01:00:00", "Production=2" });
rightList.Add(new string[] { "2013-01-01 02:00:00", "Production=0" });
rightList.Add(new string[] { "2013-01-01 03:00:00", "Production=3" });

var l = leftList.ToObservable();
var r = rightList.ToObservable();

var q = l.GroupJoin(r,
    _ => Observable.Never<Unit>(), // windows from each left event going on forever
    _ => Observable.Never<Unit>(), // windows from each right event going on forever
    (left, obsOfRight) => Tuple.Create(left, obsOfRight)); // create tuple of left event with observable of right events

// e is a tuple with two items, left and obsOfRight
q.Subscribe(e =>
{
    var xs = e.Item2;
    xs.Where(
     x => x[0] == e.Item1[0]) // filter only when datetime matches
     .Subscribe(
     v =>
     {
        Console.WriteLine(
           string.Format("{0},{1} and {2},{3} occur at the same time",
           e.Item1[0],
           e.Item1[1],
           v[0],
           v[1]
        ));
     });
});
```


## Range

<p>Generates a Range of values. Useful for testing purposes.</p>

### Range - Prints from 1 to 10.


```C#
IObservable<int> source = Observable.Range(1, 10);
IDisposable subscription = source.Subscribe(
x => Console.WriteLine("OnNext: {0}", x),
ex => Console.WriteLine("OnError: {0}", ex.Message),
() => Console.WriteLine("OnCompleted"));
Console.WriteLine("Press ENTER to unsubscribe...");
Console.ReadLine();
subscription.Dispose();
```


## Generate

<p>There are several overloads for Generate.</p>

### Generate - simple

<p>A simple use is to replicate Interval but have the sequence stop.</p>

```C#
internal class Generate_Simple
{
    private static void Main()
    {
        var observable =
            Observable.Generate(1, x => x < 6, x => x + 1, x => x, 
                                         x=>TimeSpan.FromSeconds(1)).Timestamp();

        using (observable.Subscribe(x => Console.WriteLine("{0}, {1}", x.Value, x.Timestamp)))
        {
            Console.WriteLine("Press any key to unsubscribe");
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>Result</strong><br />
1: 24/11/2009&#160;15:40:45 (after 1s)<br />
2: 24/11/2009&#160;15:40:46 (after 2s)<br />
3: 24/11/2009&#160;15:40:47 (after 3s)<br />
4: 24/11/2009&#160;15:40:48 (after 4s)<br />
5: 24/11/2009&#160;15:40:49 (after 5s)

## ISubject<T> and ISubject<T1, T2>

<p>There are several implementations for ISubject.</p>

### Ping Pong Actor Model with ISubject<T1, T2>


```C#
using System;
using System.Collections.Generic;
using System.Linq;

namespace RxPingPong
{
    /// <summary>Simple Ping Pong Actor model using Rx </summary>
    /// <remarks>
    /// You'll need to install the Reactive Extensions (Rx) for this to work.
    /// You can get the installer from <see href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx"/>
    /// </remarks>
    class Program
    {
        static void Main(string[] args)
        {
            var ping = new Ping();
            var pong = new Pong();

            Console.WriteLine("Press any key to stop ...");

            var pongSubscription = ping.Subscribe(pong);
            var pingSubscription = pong.Subscribe(ping);

            Console.ReadKey();

            pongSubscription.Dispose();
            pingSubscription.Dispose();

            Console.WriteLine("Ping Pong has completed.");
        }
    }

    class Ping : ISubject<Pong, Ping>
    {
        #region Implementation of IObserver<Pong>

        /// <summary>
        /// Notifies the observer of a new value in the sequence.
        /// </summary>
        public void OnNext(Pong value)
        {
            Console.WriteLine("Ping received Pong.");
        }

        /// <summary>
        /// Notifies the observer that an exception has occurred.
        /// </summary>
        public void OnError(Exception exception)
        {
            Console.WriteLine("Ping experienced an exception and had to quit playing.");
        }

        /// <summary>
        /// Notifies the observer of the end of the sequence.
        /// </summary>
        public void OnCompleted()
        {
            Console.WriteLine("Ping finished.");
        }

        #endregion

        #region Implementation of IObservable<Ping>

        /// <summary>
        /// Subscribes an observer to the observable sequence.
        /// </summary>
        public IDisposable Subscribe(IObserver<Ping> observer)
        {
            return Observable.Interval(TimeSpan.FromSeconds(2))
                .Where(n => n < 10)
                .Select(n => this)
                .Subscribe(observer);
        }

        #endregion

        #region Implementation of IDisposable

        /// <summary>
        /// Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.
        /// </summary>
        /// <filterpriority>2</filterpriority>
        public void Dispose()
        {
            OnCompleted();
        }

        #endregion
    }

    class Pong : ISubject<Ping, Pong>
    {
        #region Implementation of IObserver<Ping>

        /// <summary>
        /// Notifies the observer of a new value in the sequence.
        /// </summary>
        public void OnNext(Ping value)
        {
            Console.WriteLine("Pong received Ping.");
        }

        /// <summary>
        /// Notifies the observer that an exception has occurred.
        /// </summary>
        public void OnError(Exception exception)
        {
            Console.WriteLine("Pong experienced an exception and had to quit playing.");
        }

        /// <summary>
        /// Notifies the observer of the end of the sequence.
        /// </summary>
        public void OnCompleted()
        {
            Console.WriteLine("Pong finished.");
        }

        #endregion

        #region Implementation of IObservable<Pong>

        /// <summary>
        /// Subscribes an observer to the observable sequence.
        /// </summary>
        public IDisposable Subscribe(IObserver<Pong> observer)
        {
            return Observable.Interval(TimeSpan.FromSeconds(1.5))
                .Where(n => n < 10)
                .Select(n => this)
                .Subscribe(observer);
        }

        #endregion

        #region Implementation of IDisposable

        /// <summary>
        /// Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.
        /// </summary>
        /// <filterpriority>2</filterpriority>
        public void Dispose()
        {
            OnCompleted();
        }

        #endregion
    }
}
```



<strong>Result</strong><br />
1: Ping received Pong.<br />
2: Pong received Ping.<br />
3: Ping received Pong.<br />
4: Pong received Ping.<br />
5: Ping received Pong.

## Combination Operators

### Merge

<p>The Merge operator combine two or more sequences. In the following example, the two streams are merged into one so that both are printed with one subscription. Also note the use of "using" to wrap the Observable, thus ensuring the subscription is Disposed.</p>

```C#
class Merge
{
    private static IObservable<int> Xs
    {
        get { return Generate(0, new List<int> {1, 2, 2, 2, 2}); }
    }

    private static IObservable<int> Ys
    {
        get { return Generate(100, new List<int> {2, 2, 2, 2, 2}); }
    }

    private static IObservable<int> Generate(int initialValue, IList<int> intervals)
    {
        // work-around for Observable.Generate calling timeInterval before resultSelector
        intervals.Add(0); 

        return Observable.Generate(initialValue,
                                   x => x < initialValue + intervals.Count - 1,
                                   x => x + 1,
                                   x => x,
                                   x => TimeSpan.FromSeconds(intervals[x - initialValue]));
    }

    private static void Main()
    {
        Console.WriteLine("Press any key to unsubscribe");

        using (Xs.Merge(Ys).Timestamp().Subscribe(
            z => Console.WriteLine("{0,3}: {1}", z.Value, z.Timestamp),
            () => Console.WriteLine("Completed, press a key")))
        {
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
0: 11/12/2009&#160;12:17:44<br />
100: 11/12/2009&#160;12:17:45<br />
1: 11/12/2009&#160;12:17:46<br />
101: 11/12/2009&#160;12:17:47<br />
2: 11/12/2009&#160;12:17:48<br />
102: 11/12/2009&#160;12:17:49<br />
3: 11/12/2009&#160;12:17:50<br />
103: 11/12/2009&#160;12:17:51<br />
4: 11/12/2009&#160;12:17:52<br />
104: 11/12/2009&#160;12:17:53

### Publish - Sharing a subscription with multiple Observers


```C#
class Publish
{
    private static void Main()
    {
        var unshared = Observable.Range(1, 4);

        // Each subscription starts a new sequence
        unshared.Subscribe(i => Console.WriteLine("Unshared Subscription #1: " + i));
        unshared.Subscribe(i => Console.WriteLine("Unshared Subscription #2: " + i));

        Console.WriteLine();

        // By using publish the subscriptions are shared, but the sequence doesn't start until Connect() is called.
        var shared = unshared.Publish();
        shared.Subscribe(i => Console.WriteLine("Shared Subscription #1: " + i));
        shared.Subscribe(i => Console.WriteLine("Shared Subscription #2: " + i));
        shared.Connect();

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
Unshared Subscription #1: 1<br />
Unshared Subscription #1: 2<br />
Unshared Subscription #1: 3<br />
Unshared Subscription #1: 4<br />
Unshared Subscription #2: 1<br />
Unshared Subscription #2: 2<br />
Unshared Subscription #2: 3<br />
Unshared Subscription #2: 4

Shared Subscription #1: 1<br />
Shared Subscription #2: 1<br />
Shared Subscription #1: 2<br />
Shared Subscription #2: 2<br />
Shared Subscription #1: 3<br />
Shared Subscription #2: 3<br />
Shared Subscription #1: 4<br />
Shared Subscription #2: 4

### Zip


```C#
class Zip
{
    // same code as above for Merge...

    private static void Main()
    {
        Console.WriteLine("Press any key to unsubscribe");

        using (Xs.Zip(Ys, (x, y) => x + y).Timestamp().Subscribe(
            z => Console.WriteLine("{0,3}: {1}", z.Value, z.Timestamp),
            () => Console.WriteLine("Completed, press a key")))
        {
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
100: 11/12/2009&#160;12:17:45<br />
102: 11/12/2009&#160;12:17:47<br />
104: 11/12/2009&#160;12:17:49<br />
106: 11/12/2009&#160;12:17:51<br />
108: 11/12/2009&#160;12:17:53

### CombineLatest


```C#
class CombineLatest
{
    // same code as above for Merge...

    private static void Main()
    {
        Console.WriteLine("Press any key to unsubscribe");

        using (Xs.CombineLatest(Ys, (x, y) => x + y).Timestamp().Subscribe(
            z => Console.WriteLine("{0,3}: {1}", z.Value, z.Timestamp),
            () => Console.WriteLine("Completed, press a key")))
        {
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
100: 11/12/2009&#160;12:17:45<br />
101: 11/12/2009&#160;12:17:46<br />
102: 11/12/2009&#160;12:17:47<br />
103: 11/12/2009&#160;12:17:48<br />
104: 11/12/2009&#160;12:17:49<br />
105: 11/12/2009&#160;12:17:50<br />
106: 11/12/2009&#160;12:17:51<br />
107: 11/12/2009&#160;12:17:52<br />
108: 11/12/2009&#160;12:17:53

### Concat - cold observable


```C#
class ConcatCold
{
    private static IObservable<int> Xs
    {
        get { return Generate(0, new List<int> {0, 1, 1}); }
    }

    private static IObservable<int> Ys
    {
        get { return Generate(100, new List<int> {1, 1, 1}); }
    }

    // same Generate() method as above for Merge...

    private static void Main()
    {
        Console.WriteLine("Press any key to unsubscribe");

        Console.WriteLine(DateTime.Now);

        using (Xs.Concat(Ys).Timestamp().Subscribe(
            z => Console.WriteLine("{0,3}: {1}", z.Value, z.Timestamp),
            () => Console.WriteLine("Completed, press a key")))
        {
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
0: 11/12/2009&#160;12:17:45<br />
1: 11/12/2009&#160;12:17:46<br />
2: 11/12/2009&#160;12:17:47<br />
100: 11/12/2009&#160;12:17:48<br />
101: 11/12/2009&#160;12:17:49<br />
102: 11/12/2009&#160;12:17:50

### Concat - hot observable


```C#
class ConcatHot
{
    private static IObservable<int> Xs
    {
        get { return Generate(0, new List<int> {0, 1, 1}); }
    }

    private static IObservable<int> Ys
    {
        get { return Generate(100, new List<int> {1, 1, 1}).Publish(); }
    }

    // same Generate() method as above for Merge...

    private static void Main()
    {
        Console.WriteLine("Press any key to unsubscribe");

        Console.WriteLine(DateTime.Now);

        using (Xs.Concat(Ys).Timestamp().Subscribe(
            z => Console.WriteLine("{0,3}: {1}", z.Value, z.Timestamp),
            () => Console.WriteLine("Completed, press a key")))
        {
            Console.ReadKey();
        }

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();
    }
}
```



<strong>result</strong><br />
0: 11/12/2009&#160;12:17:45<br />
1: 11/12/2009&#160;12:17:46<br />
2: 11/12/2009&#160;12:17:47<br />
102: 11/12/2009&#160;12:17:48

## Make your class native to IObservable<T>

If you are about to build new system, you could consider using just IObservable<T>.

### Use Subject<T> as backend for IObservable<T>


```C#
class UseSubject
{
    public class Order
    {            
        private DateTime? _paidDate;

        private readonly Subject<Order> _paidSubj = new Subject<Order>();
        public IObservable<Order> Paid { get { return _paidSubj.AsObservable(); } }

        public void MarkPaid(DateTime paidDate)
        {
            _paidDate = paidDate;                
            _paidSubj.OnNext(this); // Raise PAID event
        }
    }

    private static void Main()
    {
        var order = new Order();
        order.Paid.Subscribe(_ => Console.WriteLine("Paid")); // Subscribe

        order.MarkPaid(DateTime.Now);
    }
}
```