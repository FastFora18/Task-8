## Практическая работа № 8
```Csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

class Program
{
    // Константы для побитового блока
    const int BatchNumberBits = 12;
    const int WeightBits = 14;
    const int MaterialCodeBits = 4;
    const int DefectFlagBits = 1;

    const int BatchNumberMask = (1 << BatchNumberBits) - 1;
    const int WeightMask = (1 << WeightBits) - 1;           
    const int MaterialCodeMask = (1 << MaterialCodeBits) - 1;
    const int DefectFlagMask = 1;                           

    const int BatchNumberShift = WeightBits + MaterialCodeBits + DefectFlagBits;
    const int WeightShift = MaterialCodeBits + DefectFlagBits;                  
    const int MaterialCodeShift = DefectFlagBits;                                

    static void Main(string[] args)
    {
        // 1. Побитовый блок
        Console.WriteLine("--- Побитовый блок ---");
        int packed = PackBatch(123, 2500, 9, 1);
        Console.WriteLine($"Упаковано: {packed}");
        Console.WriteLine($"Номер партии: {UnpackBatchNumber(packed)}");
        Console.WriteLine($"Вес: {UnpackWeight(packed)} г");
        Console.WriteLine($"Материал: {UnpackMaterialCode(packed)}");
        Console.WriteLine($"Флаг брака: {UnpackDefectFlag(packed)}");
        Console.WriteLine($"Проверка брака (без if): {(IsDefect(packed) ? "Брак" : "Норма")}");
        Console.WriteLine();

        // 2. Матрица (отражение относительно побочной диагонали)
        Console.WriteLine("--- Матрица (отражение относительно побочной диагонали) ---");
        int[,] warehouse = new int[7, 7];
        // Заполнение матрицы
        for (int i = 0; i < 7; i++)
        {
            for (int j = 0; j < 7; j++)
            {
                warehouse[i, j] = i * 7 + j + 1;
            }
        }
        PrintMatrix(warehouse);
        ReflectSecondaryDiagonal(warehouse);
        Console.WriteLine("После отражения:");
        PrintMatrix(warehouse);
        Console.WriteLine();

        // 3. Метод с ref, out, params
        Console.WriteLine("--- Метод InspectBoxes ---");
        int[] weights = { 105, 98, 112, 101, 150, 95 };
        bool[] isDefect = { false, false, false, false, false, false };
        int[] tolerances = { 90, 110 }; // допуск от 90 до 110
        InspectBoxes(ref weights, ref isDefect, out int totalGoodWeight, out int defectCount, tolerances);
        Console.WriteLine($"Годных коробок: {weights.Length - defectCount}, их общий вес: {totalGoodWeight} г");
        Console.WriteLine($"Отбраковано: {defectCount} шт.");
        Console.WriteLine("Массив весов после сдвига годных в начало:");
        for (int i = 0; i < weights.Length; i++)
        {
            Console.WriteLine($"[{i}] = {weights[i]} (брак: {isDefect[i]})");
        }
        Console.WriteLine();

        // 4. Рекурсия
        Console.WriteLine("--- Рекурсия (сумма цифр) ---");
        long serial = 987654321;
        Console.WriteLine($"Серийный номер: {serial}");
        Console.WriteLine($"Сумма цифр: {SumDigits(serial)}");
        Console.WriteLine();

        // 5. Перегрузка
        Console.WriteLine("--- Перегрузка CalculateEfficiency ---");
        double eff1 = CalculateEfficiency(1000, 45);
        Console.WriteLine($"Выход годной продукции: {eff1:F2}%");

        int[] outputs = { 200, 220, 190 };
        int[] defects = { 10, 15, 5 };
        CalculateEfficiency(outputs, defects, 650, out double bonus);
        Console.WriteLine($"Коэффициент премии: {bonus:F2}");
    }

    // Упаковка данных в int
    static int PackBatch(int batchNumber, int weight, int materialCode, int defectFlag)
    {
        return ((batchNumber & BatchNumberMask) << BatchNumberShift) |
               ((weight & WeightMask) << WeightShift) |
               ((materialCode & MaterialCodeMask) << MaterialCodeShift) |
               (defectFlag & DefectFlagMask);
    }

    // Распаковка (для вывода в Main)
    static int UnpackBatchNumber(int packed) => (packed >> BatchNumberShift) & BatchNumberMask;
    static int UnpackWeight(int packed) => (packed >> WeightShift) & WeightMask;
    static int UnpackMaterialCode(int packed) => (packed >> MaterialCodeShift) & MaterialCodeMask;
    static int UnpackDefectFlag(int packed) => packed & DefectFlagMask;

    // Проверка флага брака без if
    static bool IsDefect(int packed) => ((packed & DefectFlagMask) != 0);

    // Отражение матрицы относительно побочной диагонали in-place
    static void ReflectSecondaryDiagonal(int[,] matrix)
    {
        int n = matrix.GetLength(0);
        for (int i = 0; i < n - 1; i++)
        {
            for (int j = 0; j < n - 1 - i; j++)
            {
                // Индексы побочной диагонали: i + j == n - 1
                // Меняем местами (i, j) и (n - 1 - j, n - 1 - i)
                int temp = matrix[i, j];
                matrix[i, j] = matrix[n - 1 - j, n - 1 - i];
                matrix[n - 1 - j, n - 1 - i] = temp;
            }
        }
    }

    static void PrintMatrix(int[,] matrix)
    {
        int rows = matrix.GetLength(0);
        int cols = matrix.GetLength(1);
        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                Console.Write(matrix[i, j].ToString().PadLeft(3) + " ");
            }
            Console.WriteLine();
        }
    }

    // Метод с ref, out, params
    static void InspectBoxes(ref int[] weights, ref bool[] isDefect, out int totalGoodWeight, out int defectCount, params int[] weightTolerances)
    {
        int minTol = weightTolerances[0];
        int maxTol = weightTolerances[1];
        totalGoodWeight = 0;
        defectCount = 0;
        int writeIndex = 0;

        for (int i = 0; i < weights.Length; i++)
        {
            bool isBad = (weights[i] < minTol) || (weights[i] > maxTol);
            isDefect[i] = isBad;

            if (isBad)
            {
                defectCount++;
            }
            else
            {
                // Сдвигаем годную коробку в начало массива
                if (writeIndex != i)
                {
                    weights[writeIndex] = weights[i];
                    isDefect[writeIndex] = isDefect[i];
                }
                totalGoodWeight += weights[writeIndex];
                writeIndex++;
            }
        }

        // Оставшиеся элементы в хвосте массива помечаем как брак (логически они отбракованы)
        for (int i = writeIndex; i < weights.Length; i++)
        {
            isDefect[i] = true;
        }
    }

    // Рекурсивный подсчет суммы цифр
    static int SumDigits(long serialNumber)
    {
        if (serialNumber == 0) return 0;
        return (int)(serialNumber % 10) + SumDigits(serialNumber / 10);
    }

    // Перегрузка 1: процент выхода годной продукции
    static double CalculateEfficiency(int totalBoxes, int defectBoxes)
    {
        if (totalBoxes == 0) return 0.0;
        return ((double)(totalBoxes - defectBoxes) / totalBoxes) * 100.0;
    }

    // Перегрузка 2: расчет по сменам с коэффициентом премии
    static double CalculateEfficiency(int[] shiftOutputs, int[] shiftDefects, int targetPlan, out double bonusMultiplier)
    {
        int totalOutput = 0;
        int totalDefects = 0;

        for (int i = 0; i < shiftOutputs.Length; i++)
        {
            totalOutput += shiftOutputs[i];
            totalDefects += shiftDefects[i];
        }

        double baseEfficiency = CalculateEfficiency(totalOutput, totalDefects);

        // Логика премии: 
        // 1. План выполнен (totalOutput >= targetPlan)
        // 2. Эффективность выше 95%
        if (totalOutput >= targetPlan && baseEfficiency > 95.0)
        {
            bonusMultiplier = 1.15;
        }
        else if (totalOutput >= targetPlan)
        {
            bonusMultiplier = 1.05;
        }
        else
        {
            bonusMultiplier = 0.95;
        }

        return baseEfficiency;
    }
}
```
!Результат выполнения
<img width="1436" height="849" alt="t-8 (1)" src="https://github.com/user-attachments/assets/1f952d2f-eac4-40f2-8c0b-a9007b51bf65" />
<img width="1443" height="835" alt="t-8 (2)" src="https://github.com/user-attachments/assets/0cd5a129-b030-4f8f-b989-81f57119fca1" />
<img width="1431" height="838" alt="t-8 (3)" src="https://github.com/user-attachments/assets/1762fc29-0d6f-450a-a241-47314edcfb5e" />
<img width="1429" height="844" alt="t-8 (4)" src="https://github.com/user-attachments/assets/6032e5de-43c5-4220-8411-9d1abfac2284" />
<img width="1513" height="979" alt="t-8 (5)" src="https://github.com/user-attachments/assets/9a5348df-accb-484d-8f8a-0819dc774f29" />

// Делал Аристов Максим
