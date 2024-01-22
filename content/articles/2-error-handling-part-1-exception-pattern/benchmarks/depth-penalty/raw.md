| Method                        | Categories | Mean       | Error       | StdDev    | Gen0    | Gen1   | Allocated |      
|------------------------------ |----------- |-----------:|------------:|----------:|--------:|-------:|----------:|      
| ThrowIn1Call                  | 001 level  |   4.890 us |   0.8690 us | 0.0476 us |  0.0381 |      - |     560 B |      
| ThrowIn1CallWithStackTrace    | 001 level  |  17.379 us |   1.3236 us | 0.0726 us |  0.4883 |      - |    6376 B |
|                               |            |            |             |           |         |        |           |      
| ThrowIn2Calls                 | 002 levels |   5.681 us |   0.1936 us | 0.0106 us |  0.0381 |      - |     560 B |      
| ThrowIn2CallsWithStackTrace   | 002 levels |  21.069 us |   3.8731 us | 0.2123 us |  0.5798 |      - |    7304 B |      
|                               |            |            |             |           |         |        |           |      
| ThrowIn4Calls                 | 004 levels |   7.095 us |   0.9524 us | 0.0522 us |  0.0763 |      - |     968 B |      
| ThrowIn4CallsWithStackTrace   | 004 levels |  29.203 us |   9.3638 us | 0.5133 us |  0.9155 |      - |   11737 B |      
|                               |            |            |             |           |         |        |           |      
| ThrowIn8Calls                 | 008 levels |   9.648 us |   0.7217 us | 0.0396 us |  0.0763 |      - |     968 B |      
| ThrowIn8CallsWithStackTrace   | 008 levels |  54.139 us |   5.5793 us | 0.3058 us |  1.4648 |      - |   19721 B |      
|                               |            |            |             |           |         |        |           |
| ThrowIn16Calls                | 016 levels |  15.111 us |   2.5158 us | 0.1379 us |  0.1373 |      - |    1760 B |      
| ThrowIn16CallsWithStackTrace  | 016 levels |  75.288 us |  32.8789 us | 1.8022 us |  2.1973 |      - |   28154 B |      
|                               |            |            |             |           |         |        |           |      
| ThrowIn32Calls                | 032 levels |  26.314 us |   1.6150 us | 0.0885 us |  0.2441 |      - |    3320 B |      
| ThrowIn32CallsWithStackTrace  | 032 levels | 136.082 us |  31.8375 us | 1.7451 us |  4.1504 | 0.2441 |   53262 B |      
|                               |            |            |             |           |         |        |           |      
| ThrowIn64Calls                | 064 levels |  45.701 us |   3.8151 us | 0.2091 us |  0.4883 |      - |    6416 B |      
| ThrowIn64CallsWithStackTrace  | 064 levels | 257.381 us |  63.1832 us | 3.4633 us |  7.8125 | 1.4648 |  102939 B |      
|                               |            |            |             |           |         |        |           |      
| ThrowIn128Calls               | 128 levels |  88.841 us |  27.0890 us | 1.4848 us |  0.9766 |      - |   12584 B |      
| ThrowIn128CallsWithStackTrace | 128 levels | 504.649 us | 141.4478 us | 7.7532 us | 15.6250 |      - |  203009 B |

// 4.4 + 0.66x
// 15.3 + 3.8x
