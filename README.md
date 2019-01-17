# BufferExcelWriter

1. New WorkBookDfn()  
2. New WorkSheetDfn()  
2.1 Add header(cells withoutvalue) to sheet  
3.  OpenWriteExcelAsync  
4.  Fill data to sheet.BufferRows  
5.  OpenWriteExcelAsync  
6.  CloseExcelAndGetStreamAsync
7.  Dispose


Samples Project => BufferExcelWriter.Sample

```CSharp
public static async Task WriteTestDataAsync1()
        {
            var header = new RowDfn
            {
                Cells = new List<CellDfn>
                {
                    new CellDfn("Nam®e"),
                    new CellDfn("Index"),
                    new CellDfn("noVal")
                }
            };
            var oddSheet = new WorkSheetDfn("Odd", header); //add headers
            var evenSheet = new WorkSheetDfn("Even", header);
            var wb = new WorkBookDfn("tempFolder");
            wb.Sheets.Add(oddSheet);
            wb.Sheets.Add(evenSheet);


            await wb.OpenWriteExcelAsync(); //open write
            var sw = new Stopwatch();
            foreach (var outerIndex in Enumerable.Range(0, 10))
            {
                sw.Reset();
                var size = 1000;
                foreach (var index in Enumerable.Range(outerIndex * size, size))
                    if (index % 2 == 0)
                        evenSheet.BufferedRows.Add(new RowDfn
                        {
                            Cells = new List<CellDfn>
                            {
                                new CellDfn("Name", $"fo&o{index}"),
                                new CellDfn("Index", index.ToString())
                            }
                        });
                    else
                        oddSheet.BufferedRows.Add(new RowDfn
                        {
                            Cells = new List<CellDfn>
                            {
                                new CellDfn("Nam®e", $"f￥￥￥©$ \"oo{index}"),
                                new CellDfn("Index", index.ToString())
                            }
                        });
                sw.Start();
                await wb.FlushBufferedRowsAsync(true); //flushDataAndClean
                sw.Stop();
                Console.WriteLine(sw.Elapsed);
            }

            using (var fs = File.Create($"{DateTime.Now.Ticks}.xlsx"))
            {
                using (var stream = wb.CloseExcelAndGetStreamAsync().Result)
                {
                    stream.Position = 0;
                    stream.CopyTo(fs);
                }
            }

            sw.Reset();
            sw.Start();
            wb.Dispose();
            sw.Stop();
            Console.WriteLine($"clean cost : {sw.Elapsed}");
            Console.WriteLine("Over");
            Console.ReadLine();
        }
```
