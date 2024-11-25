# 🎵 Construindo uma API de Conversão de Áudio em .NET na prática 🎧

Quando comecei a trabalhar neste projeto, meu objetivo era criar uma API para lidar com conversão de áudio entre formatos populares como MP3, WAV e OGG. A ideia era simples: processar uploads de arquivos de áudio, validar se eles eram válidos e confiáveis e, finalmente, realizar a conversão para o formato desejado.

Esse tipo de API pode ser útil em diversos contextos, como:
- Serviços de streaming que precisam converter áudio para formatos mais otimizados.
- Pipelines de processamento que requerem formatos intermediários antes de aplicar outras transformações.
- Automatizações que eliminam a necessidade de ferramentas manuais de conversão.

O que eu queria alcançar aqui não era apenas uma API funcional, mas algo que fosse:
1. **Seguro**: Garantir que os arquivos enviados fossem válidos e não pudessem comprometer o sistema.
2. **Modular**: Ter um design que facilitasse a manutenção e a adição de novos formatos no futuro.
3. **Eficiente**: Integrar ferramentas externas confiáveis, como encoders e decoders, para evitar reinventar a roda.

Neste artigo, vou explicar como abordei esses desafios e como você pode implementar algo semelhante em seus próprios projetos. Vou detalhar como:
1. Validar arquivos corretamente, verificando tamanho, extensão e assinatura.
2. Criar um pipeline de conversão que reaproveite lógica e evite duplicação.
3. Integrar ferramentas externas de forma eficiente e prática.

Se você já trabalha com desenvolvimento em .NET e quer aprender mais sobre APIs bem estruturadas, modularidade e segurança, este material deve ser útil.

---

## Validação de Arquivos

Uma das primeiras coisas que implementei foi a validação de arquivos. Trabalhar com uploads de usuários pode ser perigoso. Um arquivo aparentemente inofensivo pode, na verdade, ser algo completamente diferente — por exemplo, um malware renomeado como um MP3. Por isso, validar arquivos antes de processá-los é essencial para garantir a segurança e a confiabilidade de qualquer API.

### Os Três Pilares da Validação

No meu caso, foquei em três aspectos principais:

1. **Tamanho do Arquivo**
   - O servidor precisa estar protegido contra uploads gigantescos que possam sobrecarregar recursos.
   - Aqui, implementei uma verificação simples para rejeitar arquivos acima de um limite configurável.

   ```csharp
   if (formFile.Length > sizeLimit)
   {
       var megabyteSizeLimit = sizeLimit / 1048576;
       throw new ArgumentException($"O arquivo excede o limite de {megabyteSizeLimit:N1} MB.");
   }
   ```

2. **Extensão**
   - Apenas extensões suportadas são permitidas (no meu caso, `.mp3`, `.wav` e `.ogg`). Isso ajuda a evitar arquivos fora do escopo da API.

   ```csharp
   var ext = Path.GetExtension(fileName).ToLowerInvariant();
   if (string.IsNullOrEmpty(ext) || !FileSignature.ContainsKey(ext))
   {
       throw new ArgumentException("Formato não suportado.");
   }
   ```

3. **Assinatura do Arquivo**
   - Para garantir que o conteúdo do arquivo corresponde à sua extensão, usei *magic numbers*. Esses são bytes específicos no início do arquivo que identificam seu tipo.
   - Por exemplo:
     - `.mp3` começa com `0x49, 0x44, 0x33`.
     - `.wav` começa com `0x52, 0x49, 0x46, 0x46`.

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

Mesmo que você não esteja lidando com áudio, essa ideia de validar tamanho, extensão e assinatura pode ser aplicada a qualquer tipo de upload.

---

## Pipeline de Conversão

Quando comecei a implementar a lógica de conversão, percebi que algumas conversões de áudio eram mais complexas do que outras. Por exemplo, transformar MP3 em OGG poderia exigir um passo intermediário: primeiro converter MP3 para WAV e só então WAV para OGG. A solução foi um pipeline de conversão reutilizável que conecta conversores especializados de forma modular.

### Estrutura do Pipeline

O pipeline de conversão é baseado em uma interface simples:

```csharp
public interface IConverter
{
    string From { get; }
    string To { get; }
    Task<byte[]> ConvertAsync(byte[] content);
}
```

Cada conversor implementa essa interface e define:
- `From`: O formato de entrada que ele aceita.
- `To`: O formato de saída que ele gera.
- `ConvertAsync`: A lógica para realizar a conversão.

Isso me permite criar pequenos blocos de lógica que podem ser combinados para formar pipelines maiores. Por exemplo:
- MP3 → WAV → OGG
- OGG → WAV → MP3

## Selecionando o Conversor Certo

O `ConverterSelector` é responsável por encontrar o conversor certo para um par de formatos. Ele recebe uma coleção de conversores registrados e verifica qual deles suporta os formatos de entrada e saída solicitados.

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
O diagrama de classe abaixo demonstra graficamente esta idéia:
[![](https://mermaid.ink/img/pako:eNqlksFOwzAMhl8l8mkT7Wm3aExCICQOiMMqIUE5mDbLqjVxlaZF0-i7kzZlymgFB3KyPznx_9s5QUa5AA5ZiXV9V6A0qFLN3BkIe7gl3QpjhWEnz_tzVVtTaMnuDakpTShgCdaH9fvRite3DRsfu6mPOlt4yDLSVmi79He6sPm591aUIrN0qcHDxdh056REbEwsLXkgfebpx2qV0DO28_Z6X-yapaCqVQqhGxrwB7YX-DdfnIUjmJHiVCT0JOUfUn72HKWQlP-V4oNg0-vPOJ5OaL5sot6XTVcXx5vgMkSghFFY5O7zDX5TsHuhRArchTmaQ2-rc3XYWNo6N8CtaUQEhhq5B77DsnZZU-Voxfhzz7RC_UL0nXdfKVbt1A?type=png)](https://mermaid-js.github.io/mermaid-live-editor/edit#pako:eNqlksFOwzAMhl8l8mkT7Wm3aExCICQOiMMqIUE5mDbLqjVxlaZF0-i7kzZlymgFB3KyPznx_9s5QUa5AA5ZiXV9V6A0qFLN3BkIe7gl3QpjhWEnz_tzVVtTaMnuDakpTShgCdaH9fvRite3DRsfu6mPOlt4yDLSVmi79He6sPm591aUIrN0qcHDxdh056REbEwsLXkgfebpx2qV0DO28_Z6X-yapaCqVQqhGxrwB7YX-DdfnIUjmJHiVCT0JOUfUn72HKWQlP-V4oNg0-vPOJ5OaL5sot6XTVcXx5vgMkSghFFY5O7zDX5TsHuhRArchTmaQ2-rc3XYWNo6N8CtaUQEhhq5B77DsnZZU-Voxfhzz7RC_UL0nXdfKVbt1A)

## Conversões com MP3 e WAV utilizando NAudio

### NAudio.Wave
O namespace `NAudio.Wave` lida com formatos WAV e permite:
- Ler arquivos WAV (`WaveFileReader`).
- Ajustar atributos de áudio, como taxa de amostragem (`WaveFormatConversionStream`).
- Escrever arquivos WAV (`WaveFileWriter`).

### NAudio.Lame
Para MP3, usamos a extensão `NAudio.Lame`, integrada ao encoder LAME MP3.

Exemplo: MP3 para WAV
```csharp
public class Mp3ToWavConverter : IConverter
{
    public string From => "mp3";
    public string To => "wav";

    public async Task<byte[]> ConvertAsync(byte[] content)
    {
        var targetFormat = new WaveFormat(8000, 16, 1); // Formato mono
        await using var outputStream = new MemoryStream();
        await using var mp3Reader = new Mp3FileReader(new MemoryStream(content));
        await using var conversionStream = new WaveFormatConversionStream(targetFormat, mp3Reader);
        await using var writer = new WaveFileWriter(outputStream, conversionStream.WaveFormat);

        await conversionStream.CopyToAsync(writer);

        return outputStream.ToArray();
    }
}
```

Exemplo: WAV para MP3
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

## Conversões com OGG utilizando Opus

### **Opusenc**
Encode OGG a partir de WAV via `opusenc`. Processo:
1. Criar arquivos temporários para entrada e saída.
2. Chamar `opusenc` como processo externo.
3. Ler o OGG gerado.

### **Opusdec**
Decode OGG para WAV via `opusdec`. Processo inverso ao `opusenc`.

Exemplo: WAV para OGG
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

Exemplo: OGG para WAV
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

## Reutilização em Pipelines

Com os conversores e o `ConverterSelector` prontos, criar pipelines é simples. Um exemplo para converter MP3 em OGG:

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
O diagrama de sequência abaixo demonstra o fluxo do pipeline com mais de um conversor:
[![](https://mermaid.ink/img/pako:eNqFkk9rwzAMxb9K0KmD9NSbD4WyXXYYG2thMHwRjpaG1X_myB2j9LtPXewF6kNzip6efnqOcwLjOwIFI30lcoYeBuwjWu0aeQJGHswQ0HGzeXmsxS0dyLCPdecprHb-DY91R8Sdf-577aaegJfrdSGpzFxosGGloW00fAsG7iZ38S1lRiZV80pSObxaOEGLppp7744UeTP-OLMQsNRMjjO0-CqobM7OG1n_Il6yejnYzazzJ5ihRbvKOifI0OKroLI5O6EFS9Hi0MnFni5jGnhPljQoee0wfmrQ7iw-TOy3sgcUx0QtRJ_6PagPPIxSpdAhl1_iX5VbfPe-1OdfHDi_TQ?type=png)](https://mermaid-js.github.io/mermaid-live-editor/edit#pako:eNqFkk9rwzAMxb9K0KmD9NSbD4WyXXYYG2thMHwRjpaG1X_myB2j9LtPXewF6kNzip6efnqOcwLjOwIFI30lcoYeBuwjWu0aeQJGHswQ0HGzeXmsxS0dyLCPdecprHb-DY91R8Sdf-577aaegJfrdSGpzFxosGGloW00fAsG7iZ38S1lRiZV80pSObxaOEGLppp7744UeTP-OLMQsNRMjjO0-CqobM7OG1n_Il6yejnYzazzJ5ihRbvKOifI0OKroLI5O6EFS9Hi0MnFni5jGnhPljQoee0wfmrQ7iw-TOy3sgcUx0QtRJ_6PagPPIxSpdAhl1_iX5VbfPe-1OdfHDi_TQ)

### Por que Essa Abordagem Funciona

- **Reutilização**: Cada conversor faz apenas uma tarefa, e essa lógica pode ser combinada em diferentes pipelines.
- **Modularidade**: Adicionar suporte a um novo formato é apenas uma questão de implementar um novo conversor.
- **Manutenção**: Problemas em um conversor não afetam os outros, já que eles são desacoplados.
  
---

## Conclusão

Construir essa API foi um exercício de modularidade, segurança e eficiência. A validação robusta garante que apenas arquivos confiáveis sejam processados, enquanto o pipeline de conversão permite reaproveitar lógica de forma elegante. 

O design modular não apenas facilita a manutenção, mas também torna simples a adição de novos formatos de áudio no futuro. Essa abordagem pode ser facilmente adaptada para lidar com outros tipos de dados, como imagens ou documentos.

Se você quiser ver o código completo deste projeto, ele está disponível no GitHub: [Carubbi Audio Converter API](https://github.com/rcarubbi/Carubbi-AudioConverter-Api).
