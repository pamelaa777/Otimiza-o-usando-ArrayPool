# Otimiza-o-usando-ArrayPool

Exercício: Processador de Filtros de Imagem em Lote
Contexto do Problema
Você foi contratado para desenvolver um sistema de processamento de imagens em lote para uma startup de edição de fotos. O sistema precisa aplicar filtros de blur em centenas de imagens de uma só vez, mas está enfrentando problemas de performance devido ao alto uso de memória e pausas frequentes do Garbage Collector.
Sua missão é otimizar o algoritmo usando ArrayPool<T> para reduzir alocações de memória e melhorar significativamente a performance do sistema.
Especificação do Problema
Implemente um sistema que:
Processa uma sequência de imagens (representadas como arrays bidimensionais de pixels RGB)
Aplica um filtro de blur gaussiano em cada imagem
Salva as imagens processadas (simulado por um contador)
Processa pelo menos 500 imagens de tamanho 800x600 pixels cada
Cada pixel é representado por uma estrutura RGB:

public struct PixelRGB
{
    public byte R, G, B;
    
    public PixelRGB(byte r, byte g, byte b)
    {
        R = r; G = g; B = b;
    }
  public static PixelRGB Average(PixelRGB a, PixelRGB b, PixelRGB c, PixelRGB d)
    {
        return new PixelRGB(
            (byte)((a.R + b.R + c.R + d.R) / 4),
            (byte)((a.G + b.G + c.G + d.G) / 4),
            (byte)((a.B + b.B + c.B + d.B) / 4)
        );
    }
}
Solução Trivial (Ineficiente)
Aqui está uma implementação que funciona, mas é ineficiente em termos de alocação de memória:

using System;
using System.Diagnostics;

public class ImageProcessor
{
    private const int IMAGE_WIDTH = 800;
    private const int IMAGE_HEIGHT = 600;
    private const int TOTAL_IMAGES = 500;
    
    public static void ProcessImages()
    {
        Console.WriteLine("Iniciando processamento de imagens (versão trivial)...");
        
        var stopwatch = Stopwatch.StartNew();
        int processedCount = 0;
        
        for (int imageIndex = 0; imageIndex < TOTAL_IMAGES; imageIndex++)
        {
            // Gera uma imagem sintética
            PixelRGB[,] originalImage = GenerateSyntheticImage(imageIndex);
            
            // Aplica filtro blur (cria novo array a cada operação)
            PixelRGB[,] blurredImage = ApplyBlurFilter(originalImage);
            
            // Simula salvamento
            SaveImage(blurredImage, $"processed_{imageIndex}.jpg");
            processedCount++;
            
            if (imageIndex % 50 == 0)
            {
                Console.WriteLine($"Processadas {imageIndex} imagens...");
            }
        }
        
        stopwatch.Stop();
        
        Console.WriteLine($"Processamento concluído!");
        Console.WriteLine($"Imagens processadas: {processedCount}");
        Console.WriteLine($"Tempo total: {stopwatch.ElapsedMilliseconds} ms");
        Console.WriteLine($"Tempo médio por imagem: {stopwatch.ElapsedMilliseconds / (double)processedCount:F2} ms");
    }
    
    private static PixelRGB[,] GenerateSyntheticImage(int seed)
    {
        var image = new PixelRGB[IMAGE_HEIGHT, IMAGE_WIDTH];
        var random = new Random(seed);
        
        for (int y = 0; y < IMAGE_HEIGHT; y++)
        {
            for (int x = 0; x < IMAGE_WIDTH; x++)
            {
                image[y, x] = new PixelRGB(
                    (byte)random.Next(256),
                    (byte)random.Next(256),
                    (byte)random.Next(256)
                );
            }
        }
        
        return image;
    }
    
    private static PixelRGB[,] ApplyBlurFilter(PixelRGB[,] original)
    {
        int height = original.GetLength(0);
        int width = original.GetLength(1);
        
        // PROBLEMA: Nova alocação a cada chamada!
        var blurred = new PixelRGB[height, width];
        
        // Aplicação de blur 2x2 simples
        for (int y = 0; y < height - 1; y++)
        {
            for (int x = 0; x < width - 1; x++)
            {
                blurred[y, x] = PixelRGB.Average(
                    original[y, x],
                    original[y, x + 1],
                    original[y + 1, x],
                    original[y + 1, x + 1]
                );
            }
        }
        
        return blurred;
    }
    
    private static void SaveImage(PixelRGB[,] image, string filename)
    {
        // Simula salvamento - na prática salvaria em disco
        // Para o exercício, apenas dar print
    }
}

Parte Prática
Parte 1: Análise da Solução Trivial
Execute a solução trivial e meça sua performance:


var sw = Stopwatch.StartNew();
var initialMemory = GC.GetTotalMemory(true);

ImageProcessor.ProcessImages();

var finalMemory = GC.GetTotalMemory(true);
sw.Stop();

Console.WriteLine($"Memória inicial: {initialMemory / 1024 / 1024:F2} MB");
Console.WriteLine($"Memória final: {finalMemory / 1024 / 1024:F2} MB");
Console.WriteLine($"Diferença de memória: {(finalMemory - initialMemory) / 1024 / 1024:F2} MB");
Console.WriteLine($"Coleções GC Gen0: {GC.CollectionCount(0)}");
Console.WriteLine($"Coleções GC Gen1: {GC.CollectionCount(1)}");
Console.WriteLine($"Coleções GC Gen2: {GC.CollectionCount(2)}");



Parte 2: Implementação Otimizada
Reescreva a classe ImageProcessor usando ArrayPool<PixelRGB> para:
Reutilizar arrays em vez de alocar novos a cada operação
Gerenciar corretamente o aluguel e devolução dos arrays
Manter a mesma funcionalidade da versão original
Dicas para implementação:
Use ArrayPool<PixelRGB>.Shared.Rent() para alugar arrays
Lembre-se de devolver os arrays com Return()
Arrays 2D podem ser simulados com arrays 1D usando índices calculados: array[y * width + x]
Considere usar try-finally para garantir que arrays sejam devolvidos mesmo em caso de exceção
Parte 3: Comparação e Análise
Após implementar a versão otimizada:
Execute ambas as versões com a mesma carga de trabalho
Colete métricas de performance (tempo, memória, GC)
Compare os resultados e calcule as melhorias percentuais
Documente suas observações
