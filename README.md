# Learn Beam

## MapReduce
* Almost every data processing can be done with {Map, Shuffle, Reduce} steps.
* Map: Each input element(<K1, V1>) is mapped to output element(s) (<K1, V1>)
* Shuffle: Output elements(<K2,V2>) are distributed so that elements with the same key end up in the same node/worker.
* Reduce: Each node/worker processes the received data

## Dataflow
A pipeline escribes a job graph of data processing transformations. Developer will design/define a pipeline by writing Java code using Beam SDK

## PCollection<T> 
is a collection of data of type T (must be serializable),  
PCollection (PC) represents “data”

The PCollection abstraction represents a potentially distributed, multi-element data set. You can think of a PCollection as “pipeline” data; Beam transforms use PCollection objects as inputs and outputs. As such, if you want to work with data in your pipeline, it must be in the form of a PCollection.

After you’ve created your Pipeline, you’ll need to begin by creating at least one PCollection in some form. The PCollection you create serves as the input for the first operation in your pipeline.

### Creation
1. Reading from an extternal source
2. Creating a PCollection from in-memory data

### PCollection characteristics
A PCollection is owned by the specific Pipeline object for which it is created; multiple pipelines cannot share a PCollection. In some respects, a PCollection functions like a collection class. However, a PCollection can differ in a few key ways:

1. Element type: Elements of a PCollection may be of any type.Beam need to be able to encode each individual element as a byte string (so elements can be passed around to distributed workers).
2. Immutability: A PCollection is immutable. Once created, you cannot add, remove, or change individual elements. A Beam Transform might process each element of a PCollection and generate new pipeline data (as a new PCollection), but it does not consume or modify the original input collection.
3. Random access: A PCollection does not support random access to individual elements. Instead, Beam Transforms consider every element in a PCollection individually.
4. Size and boundedness: A PCollection is a large, immutable “bag” of elements. There is no upper limit on how many elements a PCollection can contain; any given PCollection might fit in memory on a single machine, or it might represent a very large distributed data set backed by a persistent data store. 

A PCollection can be either bounded or unbounded in size. A bounded PCollection represents a data set of a known, fixed size, while an unbounded PCollection represents a data set of unlimited size. Whether a PCollection is bounded or unbounded depends on the source of the data set that it represents. Reading from a batch data source, such as a file or a database, creates a bounded PCollection. Reading from a streaming or continously-updating data source, such as Pub/Sub or Kafka, creates an unbounded PCollection (unless you explicitly tell it not to).

The bounded (or unbounded) nature of your PCollection affects how Beam processes your data. A bounded PCollection can be processed using a batch job, which might read the entire data set once, and perform processing in a job of finite length. An unbounded PCollection must be processed using a streaming job that runs continuously, as the entire collection can never be available for processing at any one time.

Beam uses windowing to divide a continuously updating unbounded PCollection into logical windows of finite size. These logical windows are determined by some characteristic associated with a data element, such as a timestamp. Aggregation transforms (such as GroupByKey and Combine) work on a per-window basis — as the data set is generated, they process each PCollection as a succession of these finite windows.

5. Element timestamps
Each element in a PCollection has an associated intrinsic timestamp. The timestamp for each element is initially assigned by the Source that creates the PCollection. Sources that create an unbounded PCollection often assign each new element a timestamp that corresponds to when the element was read or added.

## PTransform  
represents “manipulation/transformation” of data (PC): "mapped", "aggregated", "filtered"

Transforms are the operations in your pipeline, and provide a generic processing framework. You provide processing logic in the form of a function object (colloquially referred to as “user code”), and your user code is applied to each element of an input PCollection (or more than one PCollection). Depending on the pipeline runner and back-end that you choose, many different workers across a cluster may execute instances of your user code in parallel. The user code running on each worker generates the output elements that are ultimately added to the final output PCollection that the transform produces.

The Beam SDKs contain a number of different transforms that you can apply to your pipeline’s PCollections. These include general-purpose core transforms, such as ParDo or Combine. There are also pre-written composite transforms included in the SDKs, which combine one or more of the core transforms in a useful processing pattern, such as counting or combining elements in a collection. You can also define your own more complex composite transforms to fit your pipeline’s exact use case.

### Applying transforms
To invoke a transform, you must apply it to the input PCollection. Each transform in the Beam SDKs has a generic apply method . Invoking multiple Beam transforms is similar to method chaining, but with one slight difference: You apply the transform to the input PCollection, passing the transform itself as an argument, and the operation returns the output PCollection. This takes the general form:

```java
[Output PCollection] = [Input PCollection].apply([Transform])
```

Because Beam uses a generic apply method for PCollection, you can both chain transforms sequentially and also apply transforms that contain other transforms nested within (called composite transforms in the Beam SDKs).

How you apply your pipeline’s transforms determines the structure of your pipeline. The best way to think of your pipeline is as a directed acyclic graph, where PTransform nodes are subroutines that accept PCollection nodes as inputs and emit PCollection nodes as outputs. For example, you can chain together transforms to create a pipeline that successively modifies input data:

```java
[Final Output PCollection] = [Initial Input PCollection].apply([First Transform])
  .apply([Second Transform])
  .apply([Third Transform])
```

However, note that a transform does not consume or otherwise alter the input collection–remember that a PCollection is immutable by definition. This means that you can apply multiple transforms to the same input PCollection to create a branching pipeline.

### Core Beam Transforms

Beam provides the following core transforms, each of which represents a different processing paradigm:

ParDo   
GroupByKey   
CoGroupByKey   
Combine   
Flatten  
Partition   

#### ParDo
ParDo is a Beam transform for generic parallel processing. The ParDo processing paradigm is similar to the “Map” phase of a Map/Shuffle/Reduce-style algorithm: a ParDo transform considers each element in the input PCollection, performs some processing function (your user code) on that element, and emits zero, one, or multiple elements to an output

ParDo is useful for:
1. Filtering a data set. You can use ParDo to consider each element in a PCollection and either output that element to a new collection, or discard it.
2. Formatting or type-converting each element in a data set. If your input PCollection contains elements that are of a different type or format than you want, you can use ParDo to perform a conversion on each element and output the result to a new PCollection.
3. Extracting parts of each element in a data set. If you have a PCollection of records with multiple fields, for example, you can use a ParDo to parse out just the fields you want to consider into a new PCollection.
4. Performing computations on each element in a data set. You can use ParDo to perform simple or complex computations on every element, or certain elements, of a PCollection and output the results as a new PCollection.

##### Applying ParDo: 
Like all Beam transforms, you apply ParDo by calling the apply method on the input PCollection and passing ParDo as an argument, as shown in the following example code:

```Java
// The input PCollection of Strings.
PCollection<String> words = ...;

// The DoFn to perform on each element in the input PCollection.
static class ComputeWordLengthFn extends DoFn<String, Integer> { ... }

// Apply a ParDo to the PCollection "words" to compute lengths for each word.
PCollection<Integer> wordLengths = words.apply(
    ParDo
    .of(new ComputeWordLengthFn()));        // The DoFn to perform on each element, which
                                            // we define above.
```

In the example, our input PCollection contains String values. We apply a ParDo transform that specifies a function (ComputeWordLengthFn) to compute the length of each string, and outputs the result to a new PCollection of Integer values that stores the length of each word.

##### Creating a DoFn

The DoFn object that you pass to ParDo contains the processing logic that gets applied to the elements in the input collection. When you use Beam, often the most important pieces of code you’ll write are these DoFns–they’re what define your pipeline’s exact data processing tasks.

```Java
static class ComputeWordLengthFn extends DoFn<String, Integer> { ... } // <String: input type, Integer: output type>
```

Inside your DoFn subclass, you’ll write a method annotated with @ProcessElement where you provide the actual processing logic. You don’t need to manually extract the elements from the input collection; the Beam SDKs handle that for you. Your @ProcessElement method should accept a parameter tagged with @Element, which will be populated with the input element. In order to output elements, the method can also take a parameter of type OutputReceiver which provides a method for emitting elements. The parameter types must match the input and output types of your DoFn or the framework will raise an error. Note: @Element and OutputReceiver were introduced in Beam 2.5.0; if using an earlier release of Beam, a ProcessContext parameter should be used instead.

```Java
static class ComputeWordLengthFn extends DoFn<String, Integer> {
  @ProcessElement
  public void processElement(@Element String word, OutputReceiver<Integer> out) {
    // Use OutputReceiver.output to emit the output element.
    out.output(word.length());
  }
}
```

A given DoFn instance generally gets invoked one or more times to process some arbitrary bundle of elements. However, Beam doesn’t guarantee an exact number of invocations; it may be invoked multiple times on a given worker node to account for failures and retries. As such, you can cache information across multiple calls to your processing method, but if you do so, make sure the implementation does not depend on the number of invocations.

In your processing method, you’ll also need to meet some immutability requirements to ensure that Beam and the processing back-end can safely serialize and cache the values in your pipeline. Your method should meet the following requirements:

You should not in any way modify an element returned by the @Element annotation or ProcessContext.sideInput() (the incoming elements from the input collection).
Once you output a value using OutputReceiver.output() you should not modify that value in any way.

#### MapElements
If your ParDo performs a one-to-one mapping of input elements to output elements–that is, for each input element, it applies a function that produces exactly one output element, you can use the higher-level MapElements transform. MapElements can accept an anonymous Java 8 lambda function for additional brevity.

Here’s the previous example using MapElements :

```Java
// The input PCollection.
PCollection<String> words = ...;

// Apply a MapElements with an anonymous lambda function to the PCollection words.
// Save the result as the PCollection wordLengths.
PCollection<Integer> wordLengths = words.apply(
  MapElements.into(TypeDescriptors.integers())
             .via((String word) -> word.length()));
```

#### GroupByKey

GroupByKey is a Beam transform for processing collections of key/value pairs. It’s a parallel reduction operation, analogous to the Shuffle phase of a Map/Shuffle/Reduce-style algorithm. The input to GroupByKey is a collection of key/value pairs that represents a multimap, where the collection contains multiple pairs that have the same key, but different values. Given such a collection, you use GroupByKey to collect all of the values associated with each unique key.

GroupByKey is a good way to aggregate data that has something in common. For example, if you have a collection that stores records of customer orders, you might want to group together all the orders from the same postal code (wherein the “key” of the key/value pair is the postal code field, and the “value” is the remainder of the record).

Applying groupByKey:

From:
```
cat, 1
dog, 5
and, 1
jump, 3
tree, 2
cat, 5
dog, 2
and, 2
cat, 9
and, 6
...
```

To:
```
cat, [1,5,9]
dog, [5,2]
and, [1,2,6]
jump, [3]
tree, [2]
...
```

Thus, GroupByKey represents a transform from a multimap (multiple keys to individual values) to a uni-map (unique keys to collections of values)

##### GroupByKey and unbounded PCollections

If you are using unbounded PCollections, you must use either non-global windowing or an aggregation trigger in order to perform a GroupByKey or CoGroupByKey. This is because a bounded GroupByKey or CoGroupByKey must wait for all the data with a certain key to be collected, but with unbounded collections, the data is unlimited. Windowing and/or triggers allow grouping to operate on logical, finite bundles of data within the unbounded data streams.

If you do apply GroupByKey or CoGroupByKey to a group of unbounded PCollections without setting either a non-global windowing strategy, a trigger strategy, or both for each collection, Beam generates an IllegalStateException error at pipeline construction time.

When using GroupByKey or CoGroupByKey to group PCollections that have a windowing strategy applied, all of the PCollections you want to group must use the same windowing strategy and window sizing. For example, all of the collections you are merging must use (hypothetically) identical 5-minute fixed windows, or 4-minute sliding windows starting every 30 seconds.

If your pipeline attempts to use GroupByKey or CoGroupByKey to merge PCollections with incompatible windows, Beam generates an IllegalStateException error at pipeline construction time.

#### CoGroupByKey

#### Flatten

Flatten is a Beam transform for PCollection objects that store the same data type. Flatten merges multiple PCollection objects into a single logical PCollection.

The following example shows how to apply a Flatten transform to merge multiple PCollection objects.

```Java
// Flatten takes a PCollectionList of PCollection objects of a given type.
// Returns a single PCollection that contains all of the elements in the PCollection objects in that list.
PCollection<String> pc1 = ...;
PCollection<String> pc2 = ...;
PCollection<String> pc3 = ...;
PCollectionList<String> collections = PCollectionList.of(pc1).and(pc2).and(pc3);

PCollection<String> merged = collections.apply(Flatten.<String>pCollections());

```

##### Data encoding in merged collections
By default, the coder for the output PCollection is the same as the coder for the first PCollection in the input PCollectionList. However, the input PCollection objects can each use different coders, as long as they all contain the same data type in your chosen language.

##### Merging windowed collections
When using Flatten to merge PCollection objects that have a windowing strategy applied, all of the PCollection objects you want to merge must use a compatible windowing strategy and window sizing. For example, all the collections you’re merging must all use (hypothetically) identical 5-minute fixed windows or 4-minute sliding windows starting every 30 seconds.

If your pipeline attempts to use Flatten to merge PCollection objects with incompatible windows, Beam generates an IllegalStateException error when your pipeline is constructed.

#### Requirements for writing user code for Beam transforms

When you build user code for a Beam transform, you should keep in mind the distributed nature of execution. For example, there might be many copies of your function running on a lot of different machines in parallel, and those copies function independently, without communicating or sharing state with any of the other copies. Depending on the Pipeline Runner and processing back-end you choose for your pipeline, each copy of your user code function may be retried or run multiple times. As such, you should be cautious about including things like state dependency in your user code.

In general, your user code must fulfill at least these requirements:
1. Your function object must be serializable.
2. Your function object must be thread-compatible, and be aware that the Beam SDKs are not thread-safe.

* Seializability
Any function object you provide to a transform must be fully serializable. This is because a copy of the function needs to be serialized and transmitted to a remote worker in your processing cluster. The base classes for user code, such as DoFn, CombineFn, and WindowFn, already implement Serializable; however, your subclass must not add any non-serializable members.

  * Tips:
    1. Transient fields in your function object are not transmitted to worker instances, because they are not automatically serialized.
    2. Avoid loading a field with a large amount of data before serialization.
    3. Individual instances of your function object cannot share data.
    4. Mutating a function object after it gets applied will have no effect.
    5. Take care when declaring your function object inline by using an anonymous inner class instance. In a non-static context, your inner class instance will implicitly contain a pointer to the enclosing class and that class’ state. That enclosing class will also be serialized, and thus the same considerations that apply to the function object itself also apply to this outer class.

* Thread-compatibility
Your function object should be thread-compatible. Each instance of your function object is accessed by a single thread at a time on a worker instance, unless you explicitly create your own threads. Note, however, that the Beam SDKs are not thread-safe. If you create your own threads in your user code, you must provide your own synchronization. Note that static members in your function object are not passed to worker instances and that multiple instances of your function may be accessed from different threads.

