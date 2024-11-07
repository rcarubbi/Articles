# 🚀 Accelerating Large File Reading with Parallel Processing in Carubbi.TextFile 📂

Carubbi.TextFile is a .NET library focused on text and CSV file manipulation. One of its most powerful features is **parallel reading processing**, which allows efficient and high-performance reading of large files. This article explores the implementation and benefits of this feature, ideal for scenarios where data volume is high and processing speed is crucial.

## Motivation for Parallel Processing

When dealing with large text files, sequential reading can become a bottleneck. With parallel processing, you can split the file into parts (batches) and distribute these parts across multiple threads, maximizing CPU resources. This approach significantly improves performance, especially on multi-core machines.

## Implementation Overview

Parallel reading in `Carubbi.TextFile` is implemented in the `ReadFileInParallel` method of the `FlatTextFileReader` class. Below is an overview of the main components that make this feature possible:

1. **Splitting the File into Batches**: The file is divided into equal parts based on the total size and the number of threads.
2. **Synchronization to Maintain Order**: At the end of the process, batches are organized into a list to preserve the original order of the file lines.
3. **Calculating Header Offset**: If the file has a header, it is calculated so that subsequent threads skip it.

### Key Implementation Components

Let's explore each of these components and their significance in parallel reading.

### Splitting the File into Batches

The file is divided into equal parts, called batches, so each thread processes a specific section. The size of each batch is calculated by dividing the total file size by the number of available threads:

```csharp
int numberOfThreads = Environment.ProcessorCount;
long fileSize = new FileInfo(filePath).Length;
long bytesPerBatch = fileSize / numberOfThreads;
```

### Handling the Header

If the file has a header and the `SkipHeader` option is enabled, the header offset must be calculated so it can be skipped in subsequent batch reads. The `CalculateHeaderOffset` method reads and calculates the size of the first line, adding the line size and the newline character size:

```csharp
private async Task<long> CalculateHeaderOffset(StreamReader streamReader)
{
    long headerOffset = 0;

    if (readingOptions.SkipHeader)
    {
        string? headerLine = await streamReader.ReadLineAsync();
        if (headerLine != null)
        {
            headerOffset = Encoding.UTF8.GetByteCount(headerLine) + Encoding.UTF8.GetByteCount(Environment.NewLine);
        }
    }
    return headerOffset;
}
```

### Processing the Batches

The `ProcessBatch` method handles each batch, ensuring each thread starts and ends at the correct point. For the first batch, the `headerOffset` is applied to skip the header. The method continuously checks if the batch's end has been reached:

```csharp
private async Task<Batch<T>> ProcessBatch(string filePath, int numberOfThreads, long bytesPerBatch, long headerOffset, int batchIndex)
{
    var batch = new Batch<T>(batchIndex);
    await using FileStream fs = new(filePath, FileMode.Open, FileAccess.Read);
    using StreamReader reader = new(fs, Encoding.UTF8);

    long startPosition = batchIndex * bytesPerBatch;
    long endPosition = (batchIndex + 1) * bytesPerBatch;

    if (batchIndex == 0)
    {
        startPosition += headerOffset;
    }

    fs.Seek(startPosition, SeekOrigin.Begin);

    long bytesRead = startPosition;
    while (!IsEndOfBatch(reader, endPosition, bytesRead))
    {
        string? line = await reader.ReadLineAsync();
        bytesRead += Encoding.UTF8.GetByteCount(line ?? string.Empty) + Encoding.UTF8.GetByteCount(Environment.NewLine);

        if (line != null)
        {
            T model = new T();
            ProcessLine(line, model, readingOptions.Mode);
            batch.Add(model);
        }
    }

    return batch;
}
```

### Identifying the End of Each Batch

The `IsEndOfBatch` method checks if the current batch has reached its end. It compares the number of bytes read so far with the designated batch limit (`endPosition`) or if the reader has reached the end of the file:

```csharp
private bool IsEndOfBatch(StreamReader reader, long endPosition, long bytesRead)
{
    return bytesRead > endPosition || reader.EndOfStream;
}
```

### Aggregating Batches and Synchronization

After all batches are processed, they are aggregated into a final list to maintain the original order of lines. This is done using the `ReadFileInParallel` method, which calls `Task.WhenAll` to await all batch completions and then combines the results in the original order:

```csharp
var tasks = new List<Task<Batch<T>>>();
for (int i = 0; i < numberOfThreads; i++)
{
    var task = ProcessBatch(filePath, numberOfThreads, bytesPerBatch, headerOffset, i);
    tasks.Add(task);
}

var batches = await Task.WhenAll(tasks);
var items = batches.OrderBy(x => x.Index).SelectMany(x => x.Models).ToList();
```

## Advantages of Parallel Reading Processing

Implementing parallel reading in `Carubbi.TextFile` brings several advantages, especially in high-demand and data processing contexts. Some of these benefits include:

- **Significant Time Reduction**: Splitting the file allows simultaneous reading of multiple parts, reducing the total reading time.
- **Scalability**: Increasing the number of threads can further decrease processing time on multi-core machines.
- **Efficiency in High-Volume Data Scenarios**: In environments where large data volumes must be processed quickly, parallel processing offers a scalable and efficient solution.

## Considerations and Possible Improvements

Although the current implementation of parallel processing in `Carubbi.TextFile` already offers performance gains, there are some considerations and potential improvements:

1. **Exception Handling**: Each thread should handle exceptions individually to prevent one thread's failure from affecting the others.
2. **Synchronization Control**: Ensure that aggregated result handling is appropriately synchronized, if additional processing is needed.
3. **Parameter Configuration**: Allow the number of threads to be configurable, making it easier to adapt parallel processing to the machine's characteristics.

## Conclusion

The parallel reading functionality implemented in `Carubbi.TextFile` is an excellent example of how parallel processing can be used to improve the efficiency of large file reading. This method maximizes CPU resources, reduces processing time, and enhances user experience in high-data-demand applications. This approach is particularly useful in scenarios where speed is essential, such as real-time data processing systems and large-scale data analysis.

For developers looking to efficiently handle large text files, `Carubbi.TextFile` provides a robust and scalable solution, leveraging best practices in parallel processing within .NET.
