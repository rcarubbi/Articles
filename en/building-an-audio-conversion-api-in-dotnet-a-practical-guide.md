# 🎵 Building an Audio Conversion API in .NET: A Practical Guide 🎧

When I started working on this project, my goal was to create an API capable of handling audio conversion between popular formats like MP3, WAV, and OGG. The idea was simple: process uploaded audio files, validate their integrity, and finally, convert them into the desired format.

This kind of API can be useful in various contexts, such as:
- Streaming services needing to convert audio to optimized formats.
- Processing pipelines requiring intermediary formats before applying further transformations.
- Automation workflows that eliminate the need for manual conversion tools.

My aim was not only to build a functional API but also one that was:
1. **Secure**: Ensuring uploaded files were valid and posed no risk to the system.
2. **Modular**: Designed for easy maintenance and the future addition of new formats.
3. **Efficient**: Integrating reliable external tools, like encoders and decoders, to avoid reinventing the wheel.

In this article, I’ll explain how I approached these challenges and how you can implement something similar in your projects. I’ll cover:
1. Proper file validation by checking size, extension, and signature.
2. Building a conversion pipeline that reuses logic and avoids duplication.
3. Efficiently integrating external tools in a practical manner.

If you’re a .NET developer looking to learn more about well-structured APIs, modularity, and security, this guide will be valuable.

---

## File Validation

One of the first things I implemented was file validation. Handling user uploads can be risky. A seemingly harmless file could actually be something entirely different — for example, malware renamed as an MP3. Thus, validating files before processing them is essential to ensure the security and reliability of any API.

### The Three Pillars of Validation

In my case, I focused on three main aspects:

1. **File Size**
   - The server must be protected against oversized uploads that could overwhelm resources.
   - I implemented a simple check to reject files exceeding a configurable size limit.

   ```csharp
   if (formFile.Length > sizeLimit)
   {
       var megabyteSizeLimit = sizeLimit / 1048576;
       throw new ArgumentException($"The file exceeds the {megabyteSizeLimit:N1} MB size limit.");
   }
   ```

2. **Extension**
   - Only supported extensions are allowed (e.g., `.mp3`, `.wav`, `.ogg`). This helps prevent files outside the API's scope from being processed.

   ```csharp
   var ext = Path.GetExtension(fileName).ToLowerInvariant();
   if (string.IsNullOrEmpty(ext) || !FileSignature.ContainsKey(ext))
   {
       throw new ArgumentException("Unsupported format.");
   }
   ```

3. **File Signature**
   - To ensure that the file's content matches its extension, I used *magic numbers*. These are specific bytes at the beginning of a file that identify its type.
   - Examples:
     - `.mp3` starts with `0x49, 0x44, 0x33`.
     - `.wav` starts with `0x52, 0x49, 0x46, 0x46`.

   ```csharp
   var headerBytes = reader.ReadBytes(FileSignature.Values.SelectMany(x => x).Max(m => m.Length));
   foreach (var (extension, signatures) in FileSignature)
   {
       var validExtension = signatures.Any(signature =>
           headerBytes.Take(signature.Length).SequenceEqual(signature));
       if (validExtension)
           return (true, extension);
   }
   ```

Even if you’re not dealing with audio, the idea of validating file size, extension, and signature can be applied to any type of upload.

---

## Conversion Pipeline

When implementing the conversion logic, I realized that some audio conversions were more complex than others. For example, converting MP3 to OGG might require an intermediate step: first converting MP3 to WAV, then WAV to OGG. The solution was a reusable conversion pipeline that connects specialized converters in a modular fashion.

### Pipeline Structure

The conversion pipeline is based on a simple interface:

```csharp
public interface IConverter
{
    string From { get; }
    string To { get; }
    Task<byte[]> ConvertAsync(byte[] content);
}
```

Each converter implements this interface and defines:
- `From`: The input format it accepts.
- `To`: The output format it generates.
- `ConvertAsync`: The logic to perform the conversion.

This allows me to create small blocks of logic that can be combined into larger pipelines. For example:
- MP3 → WAV → OGG
- OGG → WAV → MP3

### Selecting the Right Converter

The `ConverterSelector` is responsible for finding the right converter for a given pair of formats. It takes a collection of registered converters and determines which one supports the requested input and output formats.

```csharp
public class ConverterSelector : IConverterSelector
{
    private readonly IEnumerable<IConverter> _converters;

    public ConverterSelector(IEnumerable<IConverter> converters)
    {
        _converters = converters;
    }

    public IConverter Select(string from, string to)
    {
        var converter = _converters.FirstOrDefault(x => x.From == from && x.To == to);
        if (converter == null)
            throw new NotSupportedException($"Conversion from {from} to {to} is not supported");

        return converter;
    }
}
```
The class diagram below graphically illustrates this idea:
[![](https://mermaid.ink/img/pako:eNqlksFOwzAMhl8l8mkT7Wm3aExCICQOiMMqIUE5mDbLqjVxlaZF0-i7kzZlymgFB3KyPznx_9s5QUa5AA5ZiXV9V6A0qFLN3BkIe7gl3QpjhWEnz_tzVVtTaMnuDakpTShgCdaH9fvRite3DRsfu6mPOlt4yDLSVmi79He6sPm591aUIrN0qcHDxdh056REbEwsLXkgfebpx2qV0DO28_Z6X-yapaCqVQqhGxrwB7YX-DdfnIUjmJHiVCT0JOUfUn72HKWQlP-V4oNg0-vPOJ5OaL5sot6XTVcXx5vgMkSghFFY5O7zDX5TsHuhRArchTmaQ2-rc3XYWNo6N8CtaUQEhhq5B77DsnZZU-Voxfhzz7RC_UL0nXdfKVbt1A?type=png)](https://mermaid-js.github.io/mermaid-live-editor/edit#pako:eNqlksFOwzAMhl8l8mkT7Wm3aExCICQOiMMqIUE5mDbLqjVxlaZF0-i7kzZlymgFB3KyPznx_9s5QUa5AA5ZiXV9V6A0qFLN3BkIe7gl3QpjhWEnz_tzVVtTaMnuDakpTShgCdaH9fvRite3DRsfu6mPOlt4yDLSVmi79He6sPm591aUIrN0qcHDxdh056REbEwsLXkgfebpx2qV0DO28_Z6X-yapaCqVQqhGxrwB7YX-DdfnIUjmJHiVCT0JOUfUn72HKWQlP-V4oNg0-vPOJ5OaL5sot6XTVcXx5vgMkSghFFY5O7zDX5TsHuhRArchTmaQ2-rc3XYWNo6N8CtaUQEhhq5B77DsnZZU-Voxfhzz7RC_UL0nXdfKVbt1A)

---

## Conversions with MP3 and WAV Using NAudio

### NAudio.Wave
The `NAudio.Wave` namespace handles WAV formats and allows:
- Reading WAV files (`WaveFileReader`).
- Adjusting audio attributes like sample rate (`WaveFormatConversionStream`).
- Writing WAV files (`WaveFileWriter`).

### NAudio.Lame
For MP3, the `NAudio.Lame` extension integrates with the LAME MP3 encoder.

**Example: MP3 to WAV**
```csharp
public class Mp3ToWavConverter : IConverter
{
    public string From => "mp3";
    public string To => "wav";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        var targetFormat = new WaveFormat(8000, 16, 1); // Mono format
        await using var outputStream = new MemoryStream();
        await using var mp3Reader = new Mp3FileReader(new MemoryStream(content));
        await using var conversionStream = new WaveFormatConversionStream(targetFormat, mp3Reader);
        await using var writer = new WaveFileWriter(outputStream, conversionStream.WaveFormat);

        await conversionStream.CopyToAsync(writer);

        return outputStream.ToArray();
    }
}
```

**Example: WAV to MP3**
```csharp
public class WavToMp3Converter : IConverter
{
    public string From => "wav";
    public string To => "mp3";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        await using var outputStream = new MemoryStream();
        await using var waveStream = new WaveFileReader(new MemoryStream(content));
        await using var writer = new LameMP3FileWriter(outputStream, waveStream.WaveFormat, 128);
        await waveStream.CopyToAsync(writer);

        return outputStream.ToArray();
    }
}
```

## Conversions with OGG Using Opus

### **Opusenc**
Encodes OGG from WAV using `opusenc`. Process:
1. Create temporary files for input and output.
2. Invoke `opusenc` as an external process.
3. Read the generated OGG file.

### **Opusdec**
Decodes OGG to WAV using `opusdec`. This process is the reverse of `opusenc`.

**Example: WAV to OGG**
```csharp
public class WavToOggConverter : IConverter
{
    public string From => "wav";
    public string To => "ogg";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        var inputTemp = Path.GetTempFileName();
        var outputTemp = Path.ChangeExtension(inputTemp, "ogg");

        await File.WriteAllBytesAsync(inputTemp, content);

        using var process = new Process
        {
            StartInfo = new ProcessStartInfo
            {
                FileName = "opusenc.exe",
                Arguments = $"{inputTemp} {outputTemp}",
                CreateNoWindow = true,
                UseShellExecute = false
            }
        };

        process.Start();
        process.WaitForExit();

        var result = await File.ReadAllBytesAsync(outputTemp);
        File.Delete(inputTemp);
        File.Delete(outputTemp);

        return result;
    }
}
```

**Example: OGG to WAV**
```csharp
public class OggToWavConverter : IConverter
{
    public string From => "ogg";
    public string To => "wav";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        var inputTemp = Path.GetTempFileName();
        var outputTemp = Path.ChangeExtension(inputTemp, "wav");

        await File.WriteAllBytesAsync(inputTemp, content);

        using var process = new Process
        {
            StartInfo = new ProcessStartInfo
            {
                FileName = "opusdec.exe",
                Arguments = $"{inputTemp} {outputTemp}",
                CreateNoWindow = true,
                UseShellExecute = false
            }
        };

        process.Start();
        process.WaitForExit();

        var result = await File.ReadAllBytesAsync(outputTemp);
        File.Delete(inputTemp);
        File.Delete(outputTemp);

        return result;
    }
}
```

---

## Reusability in Pipelines

With the converters and `ConverterSelector` ready, creating pipelines is simple. For example, converting MP3 to OGG:

```csharp
public class Mp3ToOggConverter : IConverter
{
    private readonly IConverterSelector _selector;

    public Mp3ToOggConverter(IConverterSelector selector)
    {
        _selector = selector;
    }

    public string From => "mp3";
    public string To => "ogg";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        var mp3ToWav = _selector.Select("mp3", "wav");
        var wavToOgg = _selector.Select("wav", "ogg");

        var wavContent = await mp3ToWav.ConvertAsync(content);
        return await wavToOgg.ConvertAsync(wavContent);
    }
}
```
The sequence diagram below illustrates the pipeline flow with more than one converter:
[![](https://mermaid.ink/img/pako:eNqFkjFrwzAQhf-KuCkFZ8qmIRDapUNpaQKFouWQr45pLKnyKaWE_PdeaqmGaIgn37t33z1ZPoH1LYGGkb4SOUsPPXYRB-OUPAEj97YP6FhtXh5rcUsHsuxj3XkKq51_w2PdEXHnn7vOuKkn4OV6XUg6MxcGhrAy0CgD34KBu8ldfEuZkUmtXolTdOPVwglaNK3uvTtS5M344-xCwFIzOc7Q4qugsjk7b2T9i3jJ6uVgN7POn2CGFu0q65wgQ4uvgsrm7IQGBooD9q1c7OkyZoD3NJABLa8txk8Dxp3Fh4n9VvaA5piogehTtwf9gYdRqhRa5PJL_Ktyi-_el_r8C3pwv60?type=png)](https://mermaid-js.github.io/mermaid-live-editor/edit#pako:eNqFkjFrwzAQhf-KuCkFZ8qmIRDapUNpaQKFouWQr45pLKnyKaWE_PdeaqmGaIgn37t33z1ZPoH1LYGGkb4SOUsPPXYRB-OUPAEj97YP6FhtXh5rcUsHsuxj3XkKq51_w2PdEXHnn7vOuKkn4OV6XUg6MxcGhrAy0CgD34KBu8ldfEuZkUmtXolTdOPVwglaNK3uvTtS5M344-xCwFIzOc7Q4qugsjk7b2T9i3jJ6uVgN7POn2CGFu0q65wgQ4uvgsrm7IQGBooD9q1c7OkyZoD3NJABLa8txk8Dxp3Fh4n9VvaA5piogehTtwf9gYdRqhRa5PJL_Ktyi-_el_r8C3pwv60)

### Why This Approach Works

- **Reusability**: Each converter handles only one task, and its logic can be combined into different pipelines.
- **Modularity**: Adding support for a new format is simply a matter of implementing a new converter.
- **Maintainability**: Issues in one converter do not affect others since they are decoupled.

---

## Conclusion

Building this API was an exercise in modularity, security, and efficiency. Robust validation ensures that only trustworthy files are processed, while the conversion pipeline enables elegant reuse of logic.

The modular design not only simplifies maintenance but also makes it easy to add support for new audio formats in the future. This approach can be easily adapted to handle other types of data, such as images or documents.

You can find the full project code on GitHub: [Carubbi Audio Converter API](https://github.com/rcarubbi/Carubbi-AudioConverter-Api).
